# Database

Architecture Version: 1.1 — Architecture Standardized
Last Updated: 2026-08-29

**Responsibility**: Persistent storage for mission data, embeddings, events, and audit records.

---

## Overview

The repository standardizes on PostgreSQL + pgvector as the database layer.

Deployment model:
- Local PostgreSQL + pgvector for development
- Optional Supabase PostgreSQL + pgvector for cloud deployment

Persistence is accessed through SQLAlchemy and migrations are managed with Alembic.

---

## Technology Stack

- PostgreSQL
- pgvector
- SQLAlchemy
- Alembic

---

## Schema Goals

The database should support:
- mission metadata and lifecycle state
- experiment definitions and progress
- event logs and timeline data
- vector embeddings for knowledge retrieval
- audit logs and user metadata

## Example Core Tables

```sql
missions (
  id,
  name,
  description,
  status,
  created_at,
  updated_at
)

experiments (
  id,
  mission_id,
  definition_id,
  status,
  current_step,
  started_at,
  updated_at
)

events (
  id,
  mission_id,
  type,
  source_module,
  payload,
  created_at
)

embeddings (
  id,
  document_id,
  chunk_text,
  embedding,
  metadata,
  created_at
)

audit_logs (
  id,
  user_id,
  action,
  resource_id,
  request_payload,
  result_payload,
  created_at
)
```

---

## Local Setup

```bash
createdb eka_db
```

Then configure the application connection string in `.env`.

---

## Migrations

```bash
alembic revision --autogenerate -m "Add new table"
alembic upgrade head
```

---

## Optional Cloud Deployment

Supabase PostgreSQL + pgvector may be used as a deployment option, but local PostgreSQL + pgvector remains the default development database.

---

## Notes

- EKA uses PostgreSQL + pgvector as the canonical vector-enabled database.
- ChromaDB and other external vector stores are not part of the standardized architecture.
- The frontend does not directly access the database; FastAPI owns the database boundary.
