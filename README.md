# EKA — Evolving Kognitive Assistant

An AI-powered mission support system for astronauts conducting experiments in space-station and ground-based environments.

**Status**: Architecture and documentation complete. Development ready for implementation.

---

## Overview

EKA is a comprehensive AI assistant that:

- 📹 **Detects objects** in video (Vision Engine)
- 🧑‍🚀 **Recognizes activities** from video (Activity Recognition)
- 📋 **Tracks procedures** step-by-step (Experiment Engine)
- 🚨 **Detects safety violations** in real-time (Safety Engine)
- 💡 **Answers questions** from mission documents (Knowledge Assistant)
- 📊 **Aggregates and displays** mission state (Backend + Frontend)
- 🔐 **Maintains complete audit trail** for accountability

**Designed by**: 6-member student team (B.Tech CSE/AI-ML, 2nd/3rd year)  
**Timeline**: 9 weeks  
**Philosophy**: Realistic, modular, production-style student project

---

## Quick Start

### Prerequisites
- Python 3.9+
- PostgreSQL 12+
- NVIDIA GPU (recommended, CPU fallback available)
- Git

### Setup (Development)

```bash
# Clone repository
git clone <repo-url>
cd EKA

# Create virtual environment
python -m venv venv
source venv/bin/activate  # or: venv\Scripts\activate (Windows)

# Install dependencies
pip install -r requirements-dev.txt

# Configure environment
cp .env.example .env
# Edit .env with your settings

# Setup database
python -m alembic upgrade head

# Run tests
pytest tests/

# Start backend server
python backend/main.py

# In another terminal, start frontend
cd frontend
npm install
npm start
```

### Try the Demo Scenario

See `docs/DEMO_SCENARIO.md` for complete crystal growth experiment walkthrough.

```bash
# Load demo data
python scripts/load_demo_scenario.py

# Visit dashboard
open http://localhost:3000/demo
```

---

## Architecture

EKA is organized into 7 core modules:

```
┌──────────────────────────────────────────┐
│          FRONTEND DASHBOARD              │
│         (React, Real-time UI)            │
└────────────────────┬─────────────────────┘
                     │
        ┌────────────▼─────────────┐
        │    BACKEND API           │
        │   (FastAPI, REST)        │
        │   (Authentication)       │
        │   (Audit Logging)        │
        └────┬───────────┬─────────┘
             │           │
      ┌──────▼──────┐    └──────┬──────────┐
      │   AI MODULES │           │ DATABASE │
      │              │           └──────────┘
      ├─ Vision      │
      ├─ Activity    │
      │  Recognition │
      ├─ Knowledge   │
      │  Assistant   │
      └──────┬───────┘
             │
      ┌──────▼────────────────┐
      │ Experiment Engine     │
      │ Safety Engine         │
      └──────────────────────┘
```

**Detailed Architecture**: See `docs/ARCHITECTURE.md`

---

## Module Ownership

| Module | Owner | Language | Focus |
|--------|-------|----------|-------|
| Vision Engine | Member 2 | Python | Object detection (YOLO) |
| Activity Recognition | Member 1 | Python | Action classification (PyTorch) |
| Knowledge Assistant | Member 3 | Python | Q&A from documents (RAG) |
| Experiment Engine | Member 4 | Python | Procedure tracking (State Machine) |
| Safety Engine | Member 4 | Python | Violation detection (Rules) |
| Backend + Database | Member 5 | Python | API, Auth, Logging (FastAPI) |
| Frontend + Integration | Member 6 | TypeScript/React | Dashboard, UX |

---

## Key Features

### 1. Real-Time Vision
- Astronaut and equipment detection
- Bounding boxes and confidence scores
- <100ms inference latency

### 2. Activity Understanding
- 13+ recognized activities
- Context-aware classification
- Works with partial occlusion

### 3. Procedure Tracking
- Multi-step experiment SOPs
- Progress visualization
- Timeline recording

### 4. Safety Vigilance
- Expected-vs-observed checking
- Deterministic rules (no AI in safety path)
- Severity-based alerting

### 5. Mission Knowledge
- Grounded Q&A from approved documents
- Source attribution
- Offline capability

### 6. Comprehensive Audit
- All decisions logged
- Tamper-detected audit trail
- Mission review support

---

## Technology Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| Vision | YOLOv5/v8 | Real-time, production-ready |
| Activity | PyTorch + 3D CNN | Flexible temporal modeling |
| Knowledge | RAG + FAISS + mistral-7b | Offline, grounded, cost-effective |
| Experiment | Python state machine | Deterministic, auditable |
| Safety | Rule-based Python | No AI, explainable |
| Backend | FastAPI + PostgreSQL | Fast, async, reliable |
| Frontend | React + WebSocket | Responsive, real-time |
| Deployment | Docker | Reproducible, portable |

---

## Documentation

Start here:

1. **`README.md`** (this file) — Project overview
2. **`docs/ARCHITECTURE.md`** — System design and data flow
3. **`docs/API_CONTRACTS.md`** — Module interfaces (JSON schemas)
4. **`docs/PROJECT_REQUIREMENTS.md`** — Functional and non-functional requirements
5. **`docs/SECURITY.md`** — Authentication, encryption, audit logging
6. **`docs/DEMO_SCENARIO.md`** — Complete walkthrough of crystal growth experiment

Module-specific documentation:

- **`ai/vision/README.md`** — Vision engine setup and usage
- **`ai/activity_recognition/README.md`** — Activity recognition model
- **`ai/knowledge_assistant/README.md`** — Knowledge base and Q&A
- **`experiment_engine/README.md`** — Procedure tracking
- **`safety_engine/README.md`** — Safety rules and violation detection
- **`backend/README.md`** — API server and database
- **`frontend/README.md`** — Dashboard UI

---

## Development Workflow

### 1. Understand the Architecture

Read `docs/ARCHITECTURE.md` and the module READMEs.

### 2. Work on Your Module

```bash
# Create feature branch
git checkout -b feature/your-feature

# Install module dependencies
cd <your-module>
pip install -r requirements.txt

# Develop and test
pytest tests/

# Commit with clear messages
git commit -m "Add feature X to module Y"

# Push and create PR
git push origin feature/your-feature
```

### 3. Follow API Contracts

**Important**: All module communication must follow the JSON schemas in `docs/API_CONTRACTS.md`.

```python
# Example: Validate module output
from docs.api_contracts import VisionOutput

result = vision_engine.detect(frame)
VisionOutput.parse_obj(result)  # Raises if invalid
```

### 4. Test Independently

Each module must have:
- Unit tests in `<module>/tests/`
- Test data in `data/test/<module>/`
- >80% code coverage

```bash
pytest <module>/tests/ --cov=<module> --cov-report=html
```

### 5. Integrate with Backend

Once your module works standalone, integrate with Backend API.

```python
# backend/routers/<module>.py
@router.post("/api/<module>/action")
async def module_action(request: ModuleRequest, token: str = Depends(oauth2_scheme)):
    result = <module>.process(request)
    audit_log(token.user_id, "<module>.action", request, result)
    return result
```

### 6. Update Documentation

Update module README if you changed:
- API contract
- Configuration options
- Dependencies
- Performance characteristics

---

## Testing Strategy

### Unit Tests
```bash
# Run all tests
pytest tests/

# Run specific module
pytest ai/vision/tests/

# With coverage
pytest tests/ --cov=ai --cov-report=html
```

### Integration Tests
```bash
# Test end-to-end pipeline
pytest tests/integration/

# Test demo scenario
python scripts/run_demo_scenario.py
```

### Performance Tests
```bash
# Measure latency and throughput
pytest tests/performance/ --benchmark
```

---

## Deployment

### Development
```bash
python backend/main.py
npm start  # (in frontend/)
```

### Staging (Docker)
```bash
docker-compose up  # (requires docker-compose.yml)
```

### Production
```bash
# Use Docker + Kubernetes (future)
# See docs/DEPLOYMENT.md
```

---

## Troubleshooting

### "Module not found" errors
```bash
# Install all dependencies
pip install -r requirements-dev.txt
```

### Database connection error
```bash
# Verify PostgreSQL is running
psql -U postgres -d eka_db

# Check DATABASE_URL in .env
```

### GPU out of memory
```bash
# Use smaller models
YOLO_MODEL_PATH=models/weights/yolo5s.pt
ACTIVITY_MODEL_SIZE=small
```

### Slow inference
- Check latency: `result['processing_time_ms']`
- Use smaller models or reduce input resolution
- Enable GPU acceleration

---

## Project Timeline

| Week | Phase | Tasks |
|------|-------|-------|
| 1-2 | Foundation | Backend skeleton, DB schema, frontend setup, module stubs |
| 3-4 | Core Modules | Vision, Activity Recognition implementation |
| 5-6 | Smart Modules | Experiment Engine, Safety Engine |
| 7-8 | Intelligence | Knowledge Assistant, Integration |
| 9+ | Polish | Testing, deployment, documentation, demo |

---

## Success Criteria

- [x] Architecture and documentation complete
- [ ] All 7 modules have working implementations
- [ ] End-to-end demo scenario runs successfully
- [ ] >80% test coverage for critical paths
- [ ] Safety violations detected correctly (100% recall)
- [ ] Knowledge base answers questions with sources
- [ ] Dashboard displays real-time state updates
- [ ] Complete audit trail is maintained
- [ ] Security requirements met (TLS, auth, encryption)
- [ ] Performance benchmarks achieved

---

## Contributing Guidelines

1. **Read documentation first**: Understand the architecture and your module's responsibility
2. **Follow code style**: PEP 8 for Python, Prettier for JS/TS
3. **Test thoroughly**: Unit + integration tests required
4. **Document changes**: Update READMEs and API contracts
5. **Communicate early**: Discuss design decisions with team
6. **Keep modules independent**: Don't modify other members' code
7. **Respect contracts**: Never change API schemas without discussion

---

## Security

**Important**: See `docs/SECURITY.md` for complete security design.

Key principles:
- Authentication required for all API endpoints
- Audit logging of all decisions
- No AI control of safety-critical systems
- Encryption in transit (TLS) and at rest
- Regular vulnerability scanning

```bash
# Check dependencies for vulnerabilities
pip-audit

# Run security tests
pytest tests/security/
```

---

## Performance Targets

| Component | Target | Measurement |
|-----------|--------|-------------|
| Vision inference | <100ms | Per frame |
| Activity recognition | <200ms | Per activity |
| Safety check | <50ms | Per check |
| Knowledge query | <2 seconds | Query to answer |
| Dashboard update | <1 second | Event to display |
| API response | <500ms p95 | Any endpoint |

---

## Common Tasks

### Add a New Activity Class

1. Add class name to `ai/activity_recognition/constants.py`
2. Retrain activity recognition model
3. Update `docs/API_CONTRACTS.md`
4. Add test case in `ai/activity_recognition/tests/`

### Add a New Safety Rule

1. Create rule in `safety_engine/rules/`
2. Load rule: `python safety_engine/load_rules.py`
3. Add test case in `safety_engine/tests/`
4. Document rule in `docs/SECURITY.md`

### Add a New Experiment SOP

1. Create SOP JSON in `data/experiments/`
2. Load: `python experiment_engine/load_experiments.py`
3. Test end-to-end with demo scenario

---

## Resources

- **Codebase**: `https://github.com/sighkalp/EKA-Evolving-Kognitive-Assistant`
- **Documentation**: `docs/` folder
- **Test Data**: `data/test/` folder
- **Demo Scenario**: `docs/DEMO_SCENARIO.md`

---

## Questions & Support

- **Architecture questions**: See `docs/ARCHITECTURE.md`
- **API contract questions**: See `docs/API_CONTRACTS.md`
- **Module-specific issues**: Contact module owner (see team assignments)
- **Security concerns**: See `docs/SECURITY.md` or contact Member 5

---

## License

Project-specific. See LICENSE file for details.

---

## Acknowledgments

Built by a dedicated team of student engineers with guidance from mentors. This project demonstrates real-world software engineering practices in AI systems development.

---

**Project Version**: 1.0 (Architecture)  
**Last Updated**: 2024-01-15  
**Next Milestone**: Core module implementations complete by Week 6

**Let's build something extraordinary! 🚀**