# EKA Security Design

This document outlines the security architecture and principles for the EKA system.

---

## Security Philosophy

1. **Defense in Depth**: Multiple layers of security controls
2. **Fail Safe**: Safety-critical decisions default to "stop" on uncertainty
3. **Least Privilege**: Users and services get only required permissions
4. **Auditability**: Every action is logged and traceable
5. **No AI in Safety**: Deterministic rules govern safety-critical functions
6. **Encryption by Default**: All sensitive data is encrypted in transit and at rest

---

## Authentication & Authorization

### User Roles

The system supports role-based access control (RBAC):

#### 1. Astronaut
- Can view their own mission state
- Can query knowledge base
- Can acknowledge alerts
- Cannot modify experiment definitions
- Cannot access other astronauts' data

#### 2. Mission Control
- Can view all active missions
- Can view audit logs (limited to last 30 days)
- Can pause/resume experiments
- Cannot override safety decisions
- Read-only access to experiment data

#### 3. Administrator
- Full system access
- Can manage users and roles
- Can modify experiment definitions (offline only)
- Can export audit logs
- Can configure safety rules

#### 4. Service Account (Module-to-Backend)
- Each module has a service account
- Can only call specific backend endpoints
- Scoped to their module's data
- Credentials stored securely in environment

### Authentication Flow

```
Astronaut Login
    ↓
Username + Password
    ↓
Backend (FastAPI)
    ↓
Verify against DB + bcrypt
    ↓
Issue JWT token (valid for 8 hours)
    ↓
JWT = {user_id, role, exp}
    ↓
All subsequent requests include JWT in Authorization header
    ↓
Backend validates JWT signature and expiration
```

### Token Management

- **Token Type**: JWT (JSON Web Token)
- **Algorithm**: HS256 (HMAC-SHA256)
- **Expiration**: 8 hours for astronauts, 1 hour for mission control
- **Refresh**: Refresh tokens (valid 30 days) issued on login
- **Revocation**: Logout deletes token from allowed list

### Password Policy

- Minimum 12 characters
- Must include: uppercase, lowercase, number, special character
- Password reset required every 90 days
- Previous 5 passwords cannot be reused
- Failed login attempts trigger account lockout (5 failures = 15 minute lockout)

---

## API Security

### Endpoints

All API endpoints must:

1. **Require Authentication**
   ```python
   @app.get("/api/mission/{mission_id}")
   async def get_mission(mission_id: str, token: str = Depends(oauth2_scheme)):
       # Verify token
       user = get_user_from_token(token)
       # Check authorization
       if not user.can_view_mission(mission_id):
           raise HTTPException(status_code=403)
   ```

2. **Validate Input**
   ```python
   class MissionQuery(BaseModel):
       mission_id: str = Field(..., min_length=1, max_length=64)
       start_time: datetime
       end_time: datetime
   ```

3. **Log Requests**
   ```
   2024-01-15 10:30:45 | astronaut_001 | GET /api/mission/mission_001 | 200 | 145ms
   ```

4. **Rate Limit**
   - Astronaut: 100 requests/minute
   - Mission Control: 200 requests/minute
   - Service account: 1000 requests/minute (internally managed)

### CORS Policy

```python
CORSMiddleware(
    allow_origins=["http://localhost:3000", "https://eka.mission-control.gov"],
    allow_credentials=True,
    allow_methods=["GET", "POST"],
    allow_headers=["Authorization", "Content-Type"],
)
```

### HTTPS/TLS

- All API communication uses TLS 1.2 or higher
- Self-signed certificates acceptable for development
- Production uses CA-signed certificates
- HSTS header: `Strict-Transport-Security: max-age=31536000`

---

## Data Protection

### Encryption in Transit

- All API endpoints use HTTPS/TLS
- WebSocket connections use WSS (WebSocket Secure)
- Internal service-to-service communication uses mTLS

### Encryption at Rest

Database:

```sql
-- User passwords
password_hash BYTEA (bcrypt with salt rounds=12)

-- Sensitive audit logs
audit_logs.details BYTEA (AES-256-CBC encrypted)
```

Environment variables:

- Stored in `.env` file (never committed to Git)
- Load via `python-dotenv` at startup
- Rotate credentials quarterly

Files:

- Models stored in `models/weights/` directory
- Compressed and checksummed for integrity verification

### Data Retention

| Data Type | Retention | Deletion Method |
|-----------|-----------|-----------------|
| Mission data | 7 years | Secure deletion (overwrite 3×) |
| Audit logs | 3 years | Secure deletion (overwrite 3×) |
| Session tokens | 30 days after exp. | Automatic removal |
| Video streams | 24 hours | Automatic deletion |
| User passwords | Until password reset | Overwritten by new hash |

---

## AI Model Safety

### Knowledge Assistant

**Principle**: Never generate responses outside approved documents.

```python
def query_knowledge_base(question: str) -> dict:
    # Search approved documents only
    results = vector_store.search(question, top_k=3)
    
    if not results or max_score < CONFIDENCE_THRESHOLD:
        return {
            "status": "error",
            "error_code": "NO_ANSWER_FOUND",
            "message": "Cannot answer from approved mission documents"
        }
    
    # Generate response only if we have high-confidence sources
    answer = llm.generate(question, results, template="mission_qa")
    
    # Verify answer references approved documents
    assert answer.sources_match_results(results)
    
    return {
        "answer": answer.text,
        "sources": answer.sources,
        "confidence": answer.confidence
    }
```

### Safety Engine

**Principle**: Only use deterministic rules, never delegate to LLM.

```python
def check_safety_compliance(experiment_state: dict) -> dict:
    # Load safety rules
    rules = load_safety_rules()  # Rules are in config, not model
    
    violations = []
    for rule in rules:
        if rule.condition_violated(experiment_state):
            violations.append({
                "rule": rule.id,
                "severity": rule.severity,
                "message": rule.message,
                "action": rule.recommended_action
            })
    
    # No LLM inference in safety path
    return {
        "violations": violations,
        "safe_to_proceed": len(violations) == 0
    }
```

### Model Updates

- Models are versioned and signed
- Model updates require offline review and testing
- No automatic model updates
- Each model includes a manifest with:
  - Training data lineage
  - Test accuracy metrics
  - Known limitations
  - Approved use cases

---

## Audit & Logging

### Event Logging

Every action is logged:

```json
{
  "timestamp": "2024-01-15T10:30:45.123Z",
  "event_id": "evt_abc123",
  "user_id": "ast_001",
  "action": "query_knowledge",
  "resource": "mission_001",
  "method": "POST",
  "endpoint": "/api/knowledge/query",
  "input": { "query": "How to secure chamber?" },
  "output": { "status": "success", "confidence": 0.92 },
  "result": "success",
  "duration_ms": 1234,
  "ip_address": "192.168.1.100",
  "user_agent": "Mozilla/5.0..."
}
```

### Audit Log Storage

- **Immutable**: Logs cannot be modified or deleted once written
- **Tamper Detection**: Hash chain across logs (each log includes hash of previous)
- **Retention**: 3-year minimum for mission-critical events
- **Query**: Queryable by date range, user, action, resource

```sql
CREATE TABLE audit_logs (
    id BIGSERIAL PRIMARY KEY,
    timestamp TIMESTAMPTZ NOT NULL,
    user_id VARCHAR(64),
    action VARCHAR(64) NOT NULL,
    resource_id VARCHAR(64),
    details JSONB,
    result VARCHAR(32),
    ip_address INET,
    previous_hash BYTEA,  -- SHA256 of previous log
    log_hash BYTEA NOT NULL GENERATED ALWAYS AS (
        sha256(
            CONCAT(
                id, timestamp, user_id, action, 
                resource_id, result, previous_hash
            )::bytea
        )
    ) STORED,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX audit_logs_timestamp_idx ON audit_logs(timestamp DESC);
CREATE INDEX audit_logs_user_idx ON audit_logs(user_id, timestamp DESC);
```

### Audit Log Access

- Only administrator and authorized mission control can access
- Read-only: no audit log modification allowed
- Exports include digital signature for integrity verification

---

## Third-Party Dependencies

### Dependency Scanning

- Weekly scan for known vulnerabilities (using `pip-audit`)
- Automated alerts for CVEs
- Regular dependency updates (monthly security patches)

### Allowed Dependencies

Preferred:
- `fastapi` - Web framework
- `pydantic` - Data validation
- `sqlalchemy` + `alembic` - Database ORM and migrations
- `pytorch` - Deep learning
- `opencv-python` - Computer vision
- `numpy`, `pandas` - Data processing
- `pyjwt` - Token handling
- `python-dotenv` - Environment configuration
- `pytest` - Testing
- `black`, `flake8` - Code formatting

Avoided:
- Untrusted packages without community backing
- Packages with unresolved security issues
- Packages with overly permissive dependencies

### Version Pinning

All dependencies are pinned to specific versions in `requirements.txt`:

```
fastapi==0.104.1
pydantic==2.5.0
sqlalchemy==2.0.23
torch==2.1.1
opencv-python==4.8.1.78
```

---

## Deployment Security

### Environment

- **Development**: Local machine, unencrypted (for testing only)
- **Staging**: Docker container, TLS enabled, auth enabled
- **Production**: Kubernetes cluster (future), TLS + mTLS, airgapped

### Secrets Management

**DO NOT**:
- Commit `.env` files
- Hardcode passwords or API keys
- Log sensitive data
- Print tokens to stdout

**DO**:
- Use environment variables for all secrets
- Rotate secrets quarterly
- Store secrets in secure vault (e.g., HashiCorp Vault, AWS Secrets Manager)
- Use service accounts for inter-service authentication

### Container Security

- Non-root user in Docker images
- Read-only root filesystem where possible
- No privileged mode
- Network policies restrict inter-service communication
- Regular image scanning for vulnerabilities

---

## Incident Response

### Security Incident Process

1. **Detection**: Monitor logs for anomalies
2. **Containment**: Isolate affected components
3. **Eradication**: Remove root cause
4. **Recovery**: Restore from backup if needed
5. **Post-Mortem**: Document and improve

### Common Scenarios

**Token Compromise**:
- Immediately revoke token
- Force user password reset
- Review audit logs for suspicious activity
- Notify user and mission control

**Database Breach**:
- Isolate database from network
- Restore from backup
- Reissue all credentials
- Conduct security audit

**Model Tampering**:
- Detect via model signature verification
- Restore from signed backup
- Audit all model changes
- Review any decisions made with tampered model

---

## Compliance & Standards

- **GDPR**: Personal data handling and deletion
- **OWASP Top 10**: Secure coding practices
- **CIS Benchmarks**: Hardening guidelines
- **ISO 27001**: Information security management

---

## Security Checklist

- [ ] All API endpoints require authentication
- [ ] Passwords hashed with bcrypt (rounds=12)
- [ ] All data in transit encrypted (TLS 1.2+)
- [ ] Audit logs immutable and tamper-detected
- [ ] No credentials in code or logs
- [ ] Dependencies scanned for vulnerabilities
- [ ] Safety engine uses deterministic rules only
- [ ] Knowledge assistant bounds-checked to approved documents
- [ ] Rate limiting configured for all endpoints
- [ ] CORS policy restrictive
- [ ] Tests include security scenarios
- [ ] Team training: OWASP Top 10, secure coding

---

## Security Review Schedule

- **Monthly**: Vulnerability scan and patch
- **Quarterly**: Dependency audit and update
- **Semi-Annual**: Security penetration testing
- **Annual**: Full security audit with external consultant

---

## Questions & Support

For security concerns or incident reports:
- Email: `security@eka-project.local` (internal)
- Emergency: Contact team lead immediately

---

**Document Version**: 1.0  
**Last Updated**: 2024-01-15  
**Next Review**: 2024-04-15
