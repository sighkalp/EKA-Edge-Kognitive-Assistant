# Safety Engine

**Responsibility**: Detect procedure deviations and safety violations in real-time.

---

## Overview

The Safety Engine is a deterministic rule-based system that protects mission integrity. It compares expected vs. observed activities and equipment states, detects safety violations, and issues severity-appropriate alerts. **No AI inference is used for safety-critical decisions.**

**Owner**: Member 4  
**Dependencies**: Experiment Engine, Vision Engine, Activity Recognition  
**Technology**: Rule-based Python engine, deterministic logic

---

## Inputs

### Expected State (from Experiment Engine)
- Current step
- Expected activities
- Required equipment

### Observed State (from Activity Recognition & Vision)
- Recognized activity
- Detected objects

### Safety Rules (Configuration)
- Rule definitions in YAML/JSON format

---

## Outputs

See `docs/API_CONTRACTS.md` for detailed JSON schema.

**Example Output (Normal)**:
```json
{
  "deviation_detected": false,
  "alerts": [],
  "warnings": [],
  "procedure_compliance": {
    "expected_activity": "securing_chamber",
    "observed_activity": "securing_chamber",
    "match_confidence": 0.94
  },
  "status": "success"
}
```

**Example Output (Deviation)**:
```json
{
  "deviation_detected": true,
  "alerts": [
    {
      "severity": "critical",
      "category": "safety",
      "message": "Chamber not secured before reaction initiation",
      "expected": "chamber_sealed",
      "observed": "chamber_unsecured",
      "recommendation": "STOP. Secure chamber immediately.",
      "rule": "SOP-2.3.1"
    }
  ],
  "status": "success"
}
```

---

## Installation

1. **Install dependencies**
   ```bash
   pip install -r safety_engine/requirements.txt
   ```

2. **Load safety rules**
   ```bash
   python safety_engine/load_rules.py \
     --rules safety_engine/rules/ \
     --database postgres://localhost:5432/eka_db
   ```

3. **Configure environment**
   ```bash
   cp .env.example .env
   # Set ENABLE_SAFETY_ENGINE=true
   ```

---

## Usage

### As a Module

```python
from safety_engine import SafetyEngine

# Initialize
engine = SafetyEngine(
    rules_path="safety_engine/rules/",
    database_url="postgresql://..."
)

# Check safety
result = engine.check_safety(
    expected_activity="securing_chamber",
    observed_activity="placing_sample",
    expected_equipment=["chamber", "bolts"],
    observed_equipment=["chamber", "bolts", "beaker"]
)

print(result)
# {
#   "deviation_detected": True,
#   "alerts": [{
#       "severity": "warning",
#       "message": "Expected: securing_chamber, Observed: placing_sample"
#   }]
# }
```

### As a Service (via Backend API)

```bash
curl -X POST http://localhost:8000/api/safety/check \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "expected": {...},
    "observed": {...}
  }'
```

---

## Safety Rules

### Rule Definition Format

```yaml
# safety_engine/rules/chamber_integrity.yaml
rules:
  - id: "RULE_CHAMBER_SEAL_001"
    title: "Chamber must be sealed before reaction"
    severity: "critical"
    condition: |
      current_step == 6 AND
      observed_state.chamber_sealed == False AND
      observed_activity == "initializing_reaction"
    recommendation: "STOP. Seal chamber with cap and tighten bolts."
    reference: "SOP-2.3.1"
    
  - id: "RULE_TEMP_LIMIT_001"
    title: "Reaction temperature must not exceed 60°C"
    severity: "warning"
    condition: |
      observed_state.temperature > 60.0
    recommendation: "Reduce heat source or increase coolant flow."
    reference: "SAFETY-HANDBOOK-4.2"
```

### Built-in Rules

- Chamber integrity (seal, pressure)
- Temperature limits
- Equipment operational checks
- Procedure sequence validation
- Emergency stop triggers

### Adding Custom Rules

```yaml
# Create new rule file
safety_engine/rules/custom_experiment.yaml

# Load rules
python safety_engine/load_rules.py --rules safety_engine/rules/
```

---

## Configuration

### Severity Levels

```python
class SeverityLevel(Enum):
    INFO = "info"           # Informational, no action required
    WARNING = "warning"     # Minor deviation, monitor
    CRITICAL = "critical"   # Safety violation, immediate action
```

### Alert Categories

```python
CATEGORIES = [
    "safety",          # Safety protocol violation
    "procedure",       # Procedural step mismatch
    "equipment",       # Equipment malfunction
    "timeline",        # Timing issue (behind schedule)
    "environmental"    # Environmental violation (temp, pressure)
]
```

---

## Testing

### Unit Tests

```bash
pytest safety_engine/tests/

# With coverage
pytest safety_engine/tests/ --cov=safety_engine
```

### Test Cases

1. **test_normal_compliance**: No deviation on normal execution
2. **test_chamber_seal_violation**: Detect unsealed chamber
3. **test_temperature_violation**: Detect overheating
4. **test_activity_mismatch**: Detect activity deviation
5. **test_missing_equipment**: Detect missing required equipment
6. **test_alert_severity**: Correct severity for each violation
7. **test_api_contract**: JSON output schema validation
8. **test_latency**: <50ms per check

### Test Data

```
data/test/safety/
├── test_scenarios.yaml
├── normal_execution.json
├── deviation_cases/
│   ├── unsealed_chamber.json
│   ├── overheating.json
│   └── wrong_activity.json
└── expected_alerts.json
```

---

## Architecture

### Safety Check Flow

```
Expected State (from Experiment Engine)
    ↓
Safety Rules Loaded
    ↓
Observed State (from Activity + Vision)
    ↓
Rule Evaluation (deterministic)
    ↓
Violations Detected?
    ├─ Yes → Generate Alerts (severity-based)
    └─ No → Return "all clear"
    ↓
Return Result with Recommendations
    ↓
Backend Logs Alert
    ↓
Frontend Displays Alert
```

### Rule Engine

```python
class SafetyEngine:
    def __init__(self, rules_path: str, database_url: str)
    
    def load_rules(self) -> None
    def check_safety(self, expected: dict, observed: dict) -> dict
    def evaluate_rule(self, rule: dict, state: dict) -> bool
    def get_violations(self, state: dict) -> List[dict]
    def get_recommendations(self, violations: List[dict]) -> List[str]
```

---

## Key Principles

### 1. Deterministic (No AI)
- All decisions based on explicit rules
- Reproducible and auditable
- No neural networks in safety path

```python
# ✓ GOOD: Deterministic rule
if observed_chamber_sealed == False and observed_activity == "initializing":
    raise SafetyViolation("Chamber not sealed")

# ✗ BAD: AI-based decision
if ml_model.predict(state) == "unsafe":
    raise SafetyViolation("AI says unsafe")
```

### 2. Fail Safe
- Default to "stop" on uncertainty
- Conservative approach
- Astronaut can override with authorization

```python
if confidence < THRESHOLD:
    return {
        "safe": False,
        "recommendation": "STOP - unclear state"
    }
```

### 3. Complete Audit Trail
- Every safety decision is logged
- Includes rule ID, inputs, outputs, timestamp

```python
audit_log(
    event="safety_check",
    rule_id=rule.id,
    expected=expected,
    observed=observed,
    violation=violation,
    timestamp=now()
)
```

---

## Integration with Other Modules

### With Experiment Engine

```
Experiment Engine provides:
  - current_step
  - expected_activities
  - required_equipment

Safety Engine checks:
  - Are observed activities expected?
  - Is required equipment present?
```

### With Activity Recognition + Vision

```
Activity Recognition → "observed: placing_sample"
Vision → "observed objects: [sample, beaker]"
             ↓
Safety Engine → Compare with expected
             ↓
Rule evaluation
```

### With Backend API

```python
# backend/routers/safety.py
@router.post("/api/safety/check")
async def check_safety(
    request: SafetyCheckRequest,
    token: str = Depends(oauth2_scheme)
):
    result = safety_engine.check_safety(
        expected=request.expected,
        observed=request.observed
    )
    
    # Log for audit
    audit_log(token.user_id, "safety.check", request, result)
    
    # Alert if violation
    if result["deviation_detected"]:
        send_alert_to_dashboard(result)
    
    return result
```

---

## Performance

- Rule evaluation: <50ms per check
- Database query: <10ms
- Total latency: <100ms

Target: Real-time safety decisions without blocking main loop

---

## Troubleshooting

### Rules not loading
```bash
python safety_engine/load_rules.py --rules safety_engine/rules/ --verbose
```

### False positives
- Review rule conditions
- Adjust confidence thresholds
- Add context-aware rules

### Slow rule evaluation
```python
# Cache rule conditions
engine = SafetyEngine(cache_rules=True)
```

---

## Dependencies

See `safety_engine/requirements.txt`:
```
pydantic==2.5.0
pyyaml==6.0.1
sqlalchemy==2.0.23
```

---

## Database

```sql
CREATE TABLE safety_rules (
    id SERIAL PRIMARY KEY,
    rule_id VARCHAR(64) NOT NULL UNIQUE,
    title VARCHAR(256),
    severity VARCHAR(32),
    condition TEXT,
    recommendation TEXT,
    reference VARCHAR(256),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE safety_violations (
    id SERIAL PRIMARY KEY,
    experiment_id INT,
    rule_id VARCHAR(64),
    severity VARCHAR(32),
    message TEXT,
    expected JSONB,
    observed JSONB,
    timestamp TIMESTAMP DEFAULT NOW()
);

CREATE INDEX safety_violations_timestamp_idx 
    ON safety_violations(timestamp DESC);
```

---

## Future Enhancements

- [ ] Dynamic rule loading (add rules without restart)
- [ ] Machine learning for anomaly detection (non-safety path)
- [ ] Predictive alerts (warn before violation)
- [ ] Rule conflict detection
- [ ] Explanation generation (why was alert issued?)

---

## Questions & Support

Contact module owner: Member 4

---

**Last Updated**: 2024-01-15  
**Status**: Ready for implementation
