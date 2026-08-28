# Backend API & Database

**Responsibility**: Serve API endpoints, manage authentication, coordinate modules, persist data, and maintain audit logs.

---

## Overview

The Backend is the central hub of EKA. It:

- Exposes REST API endpoints for all modules
- Handles user authentication and authorization
- Coordinates between AI modules
- Manages PostgreSQL database
- Maintains comprehensive audit logs
- Streams real-time updates via WebSocket

**Owner**: Member 5  
**Framework**: FastAPI  
**Database**: PostgreSQL  
**Authentication**: JWT  

---

## Installation

```bash
pip install -r backend/requirements.txt
python backend/main.py
```

Server runs at `http://localhost:8000`

---

## API Endpoints

All endpoints require JWT token in `Authorization: Bearer <token>` header.

### Authentication
- `POST /api/auth/login` — Login user
- `POST /api/auth/logout` — Logout user
- `POST /api/auth/refresh` — Refresh token

### Vision
- `POST /api/vision/detect` — Detect objects in frame

### Activity
- `POST /api/activity/recognize` — Recognize activity

### Experiment
- `GET /api/experiment/{id}/state` — Get experiment state
- `POST /api/experiment/start` — Start experiment
- `POST /api/experiment/{id}/advance` — Advance step

### Safety
- `POST /api/safety/check` — Check safety compliance

### Knowledge
- `POST /api/knowledge/query` — Query knowledge base

### Mission
- `GET /api/mission/{id}/state` — Get mission state
- `GET /api/mission/{id}/events` — Get event timeline

### Audit
- `GET /api/audit/logs` — Query audit logs

---

## Database Schema

See `docs/ARCHITECTURE.md` for ER diagram.

**Main Tables**:
- `users` — Astronauts and mission control
- `missions` — Mission metadata
- `experiments` — Experiment instances
- `events` — Mission timeline events
- `audit_logs` — Immutable decision logs

---

## Development

```bash
# Create migrations
alembic revision --autogenerate -m "Add new table"

# Run migrations
alembic upgrade head

# Seed test data
python backend/scripts/seed_demo_data.py
```

---

## Testing

```bash
pytest backend/tests/

# With coverage
pytest backend/tests/ --cov=backend
```

---

## Deployment

```bash
docker build -t eka-backend .
docker run -p 8000:8000 -e DATABASE_URL=postgres://... eka-backend
```

---

**For detailed setup**: See `backend/README.md` after implementation

**Status**: Ready for implementation
