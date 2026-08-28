# EKA Master Architecture

Architecture Version: 1.1 — Architecture Standardized
Last Updated: 2026-08-29

## Overview

EKA is an AI-powered mission support assistant designed to support astronauts and ground teams during space-based and laboratory experiments. The system coordinates computer vision, activity analysis, retrieval-augmented knowledge, procedure tracking, and deterministic safety validation.

## Architecture Philosophy

1. Clear ownership boundaries
2. Replaceable model components
3. Safety-first rule logic
4. Local-first development with optional cloud deployment
5. Auditability and traceability
6. Standardized stack and documentation

## Ownership and Request Flow

Frontend -> FastAPI -> Internal Python services/modules -> PostgreSQL + pgvector

The frontend is a Next.js React TypeScript application. It communicates with the backend through Axios and WebSocket channels. FastAPI is the only application backend. It orchestrates internal Python services for vision, activity recognition, knowledge retrieval, experiment state management, and safety evaluation.

## System Data Flow

```text
┌──────────────────────────────────────────────────────────────────┐
│                      EKA MISSION FLOW                           │
└──────────────────────────────────────────────────────────────────┘

                     ASTRONAUT / OPERATOR INPUT
                                │
                                v
                  ┌────────────────────────────┐
                  │  Frontend (Next.js)       │
                  │  React + TypeScript        │
                  │  Axios + WebSocket         │
                  └────────────┬───────────────┘
                               │
                               v
                  ┌────────────────────────────┐
                  │  FastAPI Backend           │
                  │  Python application API    │
                  │  Auth, orchestration       │
                  └───────┬─────────┬──────────┘
                          │         │
                          │         ├──────────────> Vision
                          │         │                 Ultralytics + YOLO11n
                          │         │
                          │         ├──────────────> Activity
                          │         │                 PyTorch + R3D-18
                          │         │
                          │         ├──────────────> Knowledge
                          │         │                 custom RAG + pgvector
                          │         │
                          │         ├──────────────> Experiment
                          │         │                 Python state machine
                          │         └──────────────> Safety
                          │                           deterministic Python rules
                          v
                  ┌────────────────────────────┐
                  │ PostgreSQL + pgvector      │
                  │ SQLAlchemy + Alembic       │
                  └────────────────────────────┘
```

## Core Modules

### 1. Frontend (`frontend/`)
**Responsibility**: Display mission data and allow operator interaction.

**Technology**:
- Next.js
- React
- TypeScript
- Axios for REST calls
- WebSocket for live data updates

**Key points**:
- Frontend-only concern.
- No backend responsibilities are assigned to Next.js API routes.
- The frontend calls FastAPI endpoints and subscribes to live updates.

---

### 2. Backend API (`backend/`)
**Responsibility**: Expose the application API and orchestrate internal services.

**Technology**:
- Python
- FastAPI
- Uvicorn

**Responsibilities**:
- Authentication and authorization
- Request routing
- Service orchestration
- Response shaping
- Event and audit handling
- Database persistence access

**Ownership**:
- Frontend calls FastAPI.
- FastAPI calls internal Python modules/services.

---

### 3. Vision Engine (`ai/vision/`)
**Responsibility**: Detect astronauts, equipment, and relevant objects in video frames.

**Inputs**:
- Video frames
- Optional ROI and detection settings

**Outputs**:
- Detected objects with bounding boxes and confidence scores
- Frame timestamps and metadata

**Technology**:
- Ultralytics
- YOLO11n
- PyTorch
- OpenCV

**Key features**:
- Local inference support
- Real-time object detection
- Object tracking metadata

---

### 4. Activity Recognition Engine (`ai/activity_recognition/`)
**Responsibility**: Classify astronaut activities from video segments.

**Inputs**:
- Video frames or sampled clips
- Current mission context

**Outputs**:
- Predicted activity label
- Confidence score
- Time window metadata

**Technology**:
- PyTorch
- Torchvision
- R3D-18

**Key features**:
- Temporal video classification
- Context-aware recognition
- Consistent contract with backend and experiment engine

---

### 5. Knowledge Assistant (`ai/knowledge_assistant/`)
**Responsibility**: Answer operational questions grounded in approved mission and experiment documents.

**Inputs**:
- User question
- Mission or experiment context

**Outputs**:
- Grounded answer
- Source attribution
- Confidence score

**Technology**:
- Custom Python RAG pipeline
- PostgreSQL + pgvector
- Embeddings: Sentence Transformers, sentence-transformers/all-MiniLM-L6-v2
- Gemini API as primary LLM
- Ollama + Qwen3-4B-Instruct-2507 as local/offline fallback
- PyMuPDF for PDF extraction

**Important constraints**:
- Gemini is not described as offline.
- ChromaDB or other vector databases are not part of the standardized architecture.

---

### 6. Experiment Engine (`experiment_engine/`)
**Responsibility**: Track procedure state and expected progress across an experiment.

**Technology**:
- Python state machine

**Outputs**:
- Current step state
- Procedure progress
- Timeline and completion metadata

**Key features**:
- Multi-step SOP tracking
- Context for downstream safety checks
- Deterministic progression logic

---

### 7. Safety Engine (`safety_engine/`)
**Responsibility**: Evaluate whether observed actions and conditions match expected safe behavior.

**Technology**:
- Deterministic Python rules
- No LLM dependency

**Key features**:
- Procedure-compliance checks
- Severity-based alerts
- Human-readable recommendations
- Explicitly independent from LLM reasoning

---

### 8. Database Layer (`database/`)
**Responsibility**: Store mission state, experiment data, embeddings, events, and audit records.

**Technology**:
- PostgreSQL + pgvector
- SQLAlchemy
- Alembic

**Deployment model**:
- Local PostgreSQL + pgvector for development
- Supabase PostgreSQL + pgvector as optional cloud deployment

**Core data groups**:
- missions
- experiments
- events
- embeddings
- audit_logs
- users and roles

---

## Testing Strategy

- Backend: Pytest
- Frontend: Vitest + React Testing Library
- Contract validation for module interfaces
- Deterministic safety tests for critical rule behavior

## Deployment

- Docker-based deployment
- Containerized frontend and backend services
- PostgreSQL + pgvector as the canonical persistent database

## Standardization Notes

- Replace YOLOv5/v8 wording with Ultralytics + YOLO11n.
- Replace vague “3D CNN” wording with R3D-18.
- Remove model-size patterns such as `ACTIVITY_MODEL_SIZE=small`.
- Use a clear YOLO configuration such as `YOLO_MODEL=YOLO11n`.
- Keep the frontend and backend architecture distinct: Next.js is frontend only.
- Keep safety deterministic and independent from any LLM.
- Document the API ownership relationship explicitly: frontend calls FastAPI; FastAPI orchestrates internal services.
- `audit_logs` - All system decisions and state changes

---

### 8. Frontend Dashboard (`frontend/`)
**Responsibility**: Display mission state, experiment progress, alerts, and facilitate astronaut interaction.

**Inputs**:
- API responses from backend

**Outputs**:
- Visual displays
- User input (questions, commands)

**Technology**:
- Next.js for SSR/SSG and full-stack React framework
- WebSocket for real-time updates

**Key Features**:
- Real-time activity and object visualization
- Experiment progress tracking
- Alert and notification system
- Astronaut Q&A interface
- Mission control view

---

## Module Communication Interfaces

All modules communicate via standardized JSON contracts (see `docs/API_CONTRACTS.md`).

```
Vision → Backend → Dashboard
Activity Recognition → Backend → Dashboard
Experiment Engine → Backend → Dashboard
Safety Engine → Backend → Dashboard
Knowledge Assistant → Backend → Dashboard
```

**Key Principle**: Each module publishes its outputs; the backend aggregates and serves them to the frontend and other modules.

---

## Deployment Architecture

```
┌──────────────────────────┐
│   Astronaut Spacesuit    │
│  (Video + Microphone)    │
└──────────────┬───────────┘
               │
        ┌──────▼──────────┐
        │   Local Edge    │
        │  (Spacesuit)    │
        │  - Vision       │
        │  - Activity Rec │
        │  - Knowledge    │
        └──────┬──────────┘
               │
        ┌──────▼──────────────┐
        │  Space Station      │
        │  - Backend API      │
        │  - Database         │
        │  - Experiment Eng   │
        │  - Safety Engine    │
        └──────┬──────────────┘
               │
        ┌──────▼──────────────┐
        │ Mission Control     │
        │  (Earth)            │
        │  - Dashboard        │
        │  - Archive          │
        └─────────────────────┘
```

**Rationale**:
- Vision and Activity Recognition run on-device for latency
- Experiment and Safety Engines run on-station for consistency
- Backend aggregates and persists
- Mission Control has read-only access with audit trail

---

## Technology Stack Summary

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| Vision | Ultralytics + YOLO11n + PyTorch + OpenCV | Standardized, fast, modern object detection |
| Activity Recognition | PyTorch + Torchvision + R3D-18 | Temporal video classification for operational actions |
| Knowledge Assistant | Custom Python RAG + PostgreSQL + pgvector + Gemini API + Ollama fallback | Grounded, traceable answers with optional local fallback |
| Experiment Tracking | Python state machine | Deterministic, auditable |
| Safety Logic | Deterministic Python rules | No LLM dependency in safety-critical paths |
| Backend | Python + FastAPI + Uvicorn | Modern async API backend |
| Frontend | Next.js + React + TypeScript + Axios + WebSocket | Modern client UI with real-time updates |
| Vector Storage | PostgreSQL + pgvector | Standard database layer for embeddings and retrieval |
| Deployment | Docker containers | Reproducible and portable |

---

## Development Phases

### Phase 1: Foundation (Weeks 1-2)
- [x] Architecture documentation
- [ ] Backend skeleton + database schema
- [ ] Frontend skeleton
- [ ] Module stubs and API contracts

### Phase 2: Core Modules (Weeks 3-6)
- [ ] Vision Engine (detect astronauts, equipment)
- [ ] Activity Recognition (classify actions)
- [ ] Experiment Engine (track procedure)
- [ ] Safety Engine (detect deviations)

### Phase 3: Intelligence (Weeks 7-8)
- [ ] Knowledge Assistant (Q&A)
- [ ] Backend integration

### Phase 4: Polish (Week 9+)
- [ ] Frontend UI/UX refinement
- [ ] Testing and deployment
- [ ] Documentation and demo

---

## Security Considerations

See `docs/SECURITY.md` for detailed security design.

**Key Principles**:
1. Authentication and authorization for all API endpoints
2. Audit logging of all decisions
3. No direct LLM control of safety systems
4. Data encryption in transit and at rest
5. Role-based access (astronaut, mission control, admin)

---

## Success Criteria

- [ ] All modules have working prototypes
- [ ] Modules communicate via clean JSON interfaces
- [ ] System successfully runs a complete simulated experiment scenario
- [ ] Safety engine correctly detects deviations
- [ ] Knowledge assistant answers questions with sources
- [ ] Frontend displays real-time updates
- [ ] Audit trail is complete and searchable
- [ ] Documentation is current and complete
