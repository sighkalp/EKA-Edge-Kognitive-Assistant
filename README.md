# E.K.A — Evolving Kognitive Assistant

Architecture Version: 1.1 — Architecture Standardized
Last Updated: 2026-08-29

EKA is an AI-powered mission support system for astronauts conducting experiments in space-station and ground-based environments.

---

## Overview

EKA combines computer vision, activity understanding, procedural tracking, safety compliance, and knowledge-assisted decision support.

Core capabilities:
- Vision detection for astronauts and equipment
- Human activity recognition from video
- Experiment procedure tracking with a Python state machine
- Deterministic safety checks based on rules
- Grounded Q&A from approved mission documents
- Real-time mission state aggregation for the frontend
- Persistent mission and audit data in PostgreSQL + pgvector

**System ownership model**
- Frontend: Next.js + React + TypeScript
- Backend: Python + FastAPI + Uvicorn
- Internal services/modules: Python services invoked by FastAPI
- Database: PostgreSQL + pgvector with SQLAlchemy + Alembic
- Deployment: Docker

---

## Architecture Summary

EKA follows a clear ownership flow:

Frontend -> FastAPI -> Internal module services and persistence layer

- The frontend is a Next.js React TypeScript client that calls the backend via Axios and receives live updates via WebSocket.
- FastAPI is the application backend and API entry point.
- FastAPI orchestrates internal Python services for vision, activity recognition, knowledge retrieval, experiment state, and safety checks.
- PostgreSQL + pgvector stores mission metadata, experiment state, embeddings, and audit records.
- The frontend does not own or implement backend logic; it is not the application backend.

---

## Standardized Stack

| Layer | Technology |
|------|------------|
| Frontend | Next.js, React, TypeScript |
| API client | Axios |
| Real-time updates | WebSocket |
| Backend | Python, FastAPI, Uvicorn |
| Database | PostgreSQL + pgvector |
| ORM/Migrations | SQLAlchemy, Alembic |
| Local development database | Local PostgreSQL + pgvector |
| Optional cloud deployment database | Supabase PostgreSQL + pgvector |
| Vision | Ultralytics, YOLO11n, PyTorch, OpenCV |
| Activity recognition | PyTorch, Torchvision, R3D-18 |
| Embeddings | Sentence Transformers, sentence-transformers/all-MiniLM-L6-v2 |
| Knowledge assistant | Custom Python RAG pipeline |
| LLM primary | Gemini API |
| LLM fallback | Ollama + Qwen3-4B-Instruct-2507 |
| PDF extraction | PyMuPDF |
| Experiment engine | Python state machine |
| Safety engine | Deterministic Python rules |
| Testing | Pytest, Vitest, React Testing Library |
| Deployment | Docker |

---

## System Flow

```text
Frontend (Next.js + React + TypeScript)
        |
        | Axios requests + WebSocket streams
        v
FastAPI (Python API backend)
        |
        +--> Vision Service (Ultralytics + YOLO11n)
        +--> Activity Service (PyTorch + R3D-18)
        +--> Knowledge Service (custom Python RAG + pgvector)
        +--> Experiment Engine (Python state machine)
        +--> Safety Engine (deterministic Python rules)
        |
        v
PostgreSQL + pgvector (SQLAlchemy/Alembic)
```

The safety layer must remain LLM-independent and deterministic.

---

## Project Structure

```text
EKA/
├── README.md
├── .env.example
├── docs/
│   ├── ARCHITECTURE.md
│   ├── API_CONTRACTS.md
│   ├── PROJECT_REQUIREMENTS.md
│   ├── SECURITY.md
│   └── DEMO_SCENARIO.md
├── backend/
│   └── README.md
├── frontend/
│   └── README.md
├── database/
│   └── README.md
├── ai/
│   ├── vision/
│   │   └── README.md
│   ├── activity_recognition/
│   │   └── README.md
│   └── knowledge_assistant/
│       └── README.md
├── experiment_engine/
│   └── README.md
├── safety_engine/
│   └── README.md
└── docker/
    └── (deployment assets, if added later)
```

---

## Development Workflow

### 1. Local setup

```bash
# clone repo
git clone <repo-url>
cd EKA

# create virtual environment
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# install Python dependencies
pip install -r requirements-dev.txt

# configure environment
cp .env.example .env
```

### 2. Start database

Use local PostgreSQL + pgvector for development.

```bash
createdb eka_db
```

### 3. Run backend

```bash
uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

### 4. Run frontend

```bash
cd frontend
npm install
npm run dev
```

Development frontend URL:
- http://localhost:3000

### 5. Production build

```bash
cd frontend
npm run build
npm start
```

---

## Documentation Index

- `README.md` — repository overview and architecture standard
- `docs/ARCHITECTURE.md` — system design and module interactions
- `docs/API_CONTRACTS.md` — JSON contracts and ownership boundaries
- `docs/PROJECT_REQUIREMENTS.md` — product/design requirements
- `docs/SECURITY.md` — security and audit expectations
- `docs/DEMO_SCENARIO.md` — example mission flow

Module documentation:
- `backend/README.md` — API backend and orchestration
- `frontend/README.md` — user-facing dashboard and client behavior
- `database/README.md` — PostgreSQL + pgvector schema and migrations
- `ai/vision/README.md` — object detection with YOLO11n
- `ai/activity_recognition/README.md` — activity classification with R3D-18
- `ai/knowledge_assistant/README.md` — knowledge retrieval and grounding
- `experiment_engine/README.md` — procedure state tracking
- `safety_engine/README.md` — deterministic safety rules

---

## Testing

- Python tests: Pytest
- Frontend tests: Vitest + React Testing Library

Example commands:

```bash
pytest
cd frontend && npm test
```

---

## Deployment

EKA is deployed with Docker containers.

- Frontend is served as a Next.js app container.
- Backend runs as a FastAPI/Uvicorn container.
- PostgreSQL + pgvector is the primary data layer for local and cloud deployments.
- Supabase PostgreSQL + pgvector is optional for cloud hosting.

---

## Notes

- Next.js is frontend-only; it does not replace the Python FastAPI backend.
- Gemini API is the primary LLM for knowledge generation; it is not an offline component.
- Ollama + Qwen3-4B-Instruct-2507 is the local/offline fallback.
- The safety engine is rule-based and must not depend on an LLM.
- No implementation code is being added in this documentation-only standardization pass.

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
# backend
uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000

# frontend
cd frontend
npm run dev
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
# Use smaller operating settings or reduce input size
YOLO_MODEL=YOLO11n
ACTIVITY_MODEL=R3D-18
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

**Project Version**: 1.1 (Architecture Standardized)  
**Last Updated**: 2026-08-29  
**Next Milestone**: Core module implementation planning and documentation alignment

**Let's build something extraordinary! 🚀**
