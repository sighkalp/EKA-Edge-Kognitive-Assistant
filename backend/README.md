# Backend API

Architecture Version: 1.1 — Architecture Standardized
Last Updated: 2026-08-29

**Responsibility**: Expose the application API, orchestrate internal services, and manage data access.

---

## Overview

The FastAPI backend is the system boundary for the repository. It is the only application backend. The frontend is a separate Next.js application, and it will call FastAPI APIs over HTTP and WebSocket.

Responsibilities:
- Expose REST endpoints for frontend clients
- Coordinate internal Python services
- Persist mission, experiment, and audit data
- Authenticate and authorize users
- Stream mission updates through WebSocket
- Standardize responses and error handling

**Stack**:
- Python
- FastAPI
- Uvicorn
- PostgreSQL + pgvector
- SQLAlchemy
- Alembic

---

## Ownership Model

```text
Frontend (Next.js + React + TypeScript)
        |
        | Axios + WebSocket
        v
FastAPI backend
        |
        +--> internal service layer (vision, activity recognition, knowledge, experiment, safety)
        +--> database layer (PostgreSQL + pgvector)
```

This is explicit: frontend -> FastAPI; FastAPI -> internal modules/services.

---

## Development Environment

```bash
# Create environment
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements-dev.txt

# Start backend locally
uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

Local API endpoint:
- http://localhost:8000

---

## Database Standards

- Local PostgreSQL + pgvector for development
- Optional Supabase PostgreSQL + pgvector for cloud deployment
- SQLAlchemy for ORM access
- Alembic for schema migrations

---

## API Surface

Common design rules:
- The frontend is the client.
- FastAPI is the application backend.
- Internal modules are not called directly by the frontend.
- Safety checks remain deterministic and do not depend on an LLM.

### Authentication
- `POST /api/auth/login`
- `POST /api/auth/refresh`
- `POST /api/auth/logout`

### Vision
- `POST /api/vision/detect`

### Activity
- `POST /api/activity/recognize`

### Experiment
- `GET /api/experiment/{id}/state`
- `POST /api/experiment/start`
- `POST /api/experiment/{id}/advance`

### Safety
- `POST /api/safety/check`

### Knowledge
- `POST /api/knowledge/query`

### Mission
- `GET /api/mission/{id}/state`
- `GET /api/mission/{id}/events`

### Audit
- `GET /api/audit/logs`

---

## Persistence

Recommended storage model:
- `missions` — mission metadata and configuration
- `experiments` — experiment instances and progression
- `events` — timer/mission timeline data
- `embeddings` — vector storage for RAG retrieval
- `audit_logs` — operational and security records
- `users` — astronauts/operators/access control metadata

---

## Testing

```bash
pytest
```

---

## Deployment

```bash
docker build -t eka-backend .
docker run -p 8000:8000 eka-backend
```

The backend is part of a Docker-based deployment architecture alongside the Next.js frontend and PostgreSQL + pgvector.

---

## Standardization Notes

- Next.js is frontend-only.
- FastAPI is the only backend.
- Gemini is the primary LLM, not an offline system.
- Ollama + Qwen3-4B-Instruct-2507 is the local fallback.
- Safety logic remains deterministic and LLM-independent.
