# EKA Master Architecture

## Overview

EKA (Evolving Kognitive Assistant) is an AI-powered assistant designed to support astronauts conducting experiments in future space-station and Biomedical Applications (BAS) environments. The system combines computer vision, activity recognition, knowledge retrieval, and safety engineering to provide real-time guidance and ensure experimental integrity.

## Architecture Philosophy

1. **Modularity**: Each subsystem has clear responsibility and defined interfaces
2. **Replaceability**: AI models can be upgraded independently
3. **Safety First**: Deterministic rules govern safety-critical decisions
4. **Offline First**: Core functionality works without internet connectivity
5. **Auditability**: All decisions and actions are logged for mission review
6. **Student-Focused**: Practical technologies, no unnecessary complexity

## System Data Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                        EKA MISSION FLOW                         │
└─────────────────────────────────────────────────────────────────┘

                         ASTRONAUT INPUT
                              │
                    ┌─────────┼─────────┐
                    │                   │
            EXPERIMENT QUERY    VIDEO STREAM + VOICE
                    │                   │
                    ▼                   ▼
        ┌──────────────────┐   ┌────────────────────┐
        │    KNOWLEDGE     │   │   VISION ENGINE    │
        │    ASSISTANT     │   └────────────────────┘
        └──────────────────┘          │
                │                     ▼
                │          ┌──────────────────────┐
                │          │  ACTIVITY RECOGNITION│
                │          └──────────────────────┘
                │                     │
                └────────┬────────────┘
                         │
                    CONTEXT FUSION
                         │
                    ▼    ▼    ▼
         ┌──────────────────────────────┐
         │  EXPERIMENT ENGINE           │
         │  (Track procedure, state)    │
         └──────────────────────────────┘
                         │
                    ▼    ▼
         ┌──────────────────────────────┐
         │  SAFETY ENGINE               │
         │  (Detect deviations, alerts) │
         └──────────────────────────────┘
                         │
                    ▼    ▼
         ┌──────────────────────────────┐
         │  RESPONSE ORCHESTRATOR       │
         └──────────────────────────────┘
                         │
                    ▼    ▼
         ┌──────────────────────────────┐
         │  BACKEND API + DATABASE      │
         └──────────────────────────────┘
                         │
                    ▼    ▼
         ┌──────────────────────────────┐
         │  FRONTEND DASHBOARD          │
         │  (Astronaut + Mission Control)│
         └──────────────────────────────┘
```

## Core Modules

### 1. Vision Engine (`ai/vision/`)
**Responsibility**: Process video streams and detect objects/personnel.

**Inputs**:
- Video frames (OpenCV Mat or numpy array)
- Detection configuration (optional region of interest)

**Outputs**:
- Detected objects with bounding boxes and confidence scores
- Timestamps
- Metadata (person ID, equipment type, etc.)

**Technology**:
- YOLOv5/v8 for real-time object detection
- OpenCV for video processing

**Key Features**:
- Astronaut detection and tracking
- Equipment/container detection
- Real-time performance (runs locally)
- Adaptable to different lighting conditions

---

### 2. Activity Recognition Engine (`ai/activity_recognition/`)
**Responsibility**: Classify human activities from video or skeletal data.

**Inputs**:
- Video frames or skeletal keypoints
- Context (current experiment)

**Outputs**:
- Recognized activity with confidence
- Activity duration
- Timestamp

**Technology**:
- Temporal CNN or transformer-based model
- Pretrained on space/lab activity datasets or adapted general activity models

**Key Features**:
- Recognizes common astronaut actions (placing samples, adjusting equipment, etc.)
- Low latency inference
- Handles partial occlusion

---

### 3. Knowledge Assistant (`ai/knowledge_assistant/`)
**Responsibility**: Answer astronaut questions by retrieving mission/experiment documents.

**Inputs**:
- Astronaut question (text or transcribed voice)
- Mission context

**Outputs**:
- Grounded answer with source documents
- Confidence score
- Suggested follow-up actions

**Technology**:
- RAG (Retrieval-Augmented Generation)
- Embeddings model for semantic search
- PostgreSQL + pgvector for vector storage (within free database stack)
- LLM for response generation:
  - **Primary**: Gemini API (free tier)
  - **Fallback**: Qwen3-4B-Instruct-2507 (local, free)

**Key Features**:
- Answers only from approved mission/experiment documents
- Source attribution
- Offline capability
- Experiment-specific knowledge

---

### 4. Experiment Engine (`experiment_engine/`)
**Responsibility**: Track experiment state and procedure progress.

**Inputs**:
- Experiment definition (SOP document)
- Activity recognition results
- Vision results
- Current experiment state

**Outputs**:
- Current procedure step
- Experiment status (normal, paused, completed)
- Metadata

**Technology**:
- State machine pattern
- Deterministic procedure tracking
- Simple database queries

**Key Features**:
- Parses experiment SOPs
- Tracks multi-step procedures
- Provides procedure context to other modules

---

### 5. Safety Engine (`safety_engine/`)
**Responsibility**: Detect deviations and safety violations, issue alerts.

**Inputs**:
- Expected procedure step (from Experiment Engine)
- Observed activity (from Activity Recognition)
- Observed objects (from Vision)
- Safety rules configuration

**Outputs**:
- Deviation detected (boolean)
- Severity level (info, warning, critical)
- Explanation message
- Recommended action

**Technology**:
- Rule-based logic (deterministic safety rules)
- No direct LLM control of safety decisions
- Simple scoring/matching algorithms

**Key Features**:
- Expected-vs-observed procedure checking
- Safety violation detection
- Severity-based alerting
- Offline operation

---

### 6. Backend API (`backend/`)
**Responsibility**: Serve all modules' outputs via REST API, manage authentication, logging, state persistence.

**Inputs**:
- Requests from frontend
- Module outputs (vision, activity, experiment state, etc.)

**Outputs**:
- JSON API responses
- Mission event logs
- Audit records

**Technology**:
- FastAPI
- PostgreSQL for persistent state and audit logs
- JWT authentication

**Key Features**:
- Module integration orchestration
- Request routing
- Real-time event streaming (WebSocket)
- Audit logging
- User authentication and authorization

---

### 7. Database Layer (`database/`)
**Responsibility**: Persistent storage for missions, experiments, audit logs.

**Technology**:
- PostgreSQL
- Simple schema for missions, experiments, astronauts, events, audit logs

**Key Tables**:
- `missions` - Mission metadata
- `experiments` - Experiment definitions and instances
- `astronauts` - Crew members
- `events` - Mission timeline (activities, detections, alerts)
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
| Vision | YOLOv5/v8 | Fast, accurate, well-documented |
| Activity Recognition | PyTorch + temporal models | Flexible, good for custom datasets |
| Knowledge Assistant | RAG + Gemini API (primary) + Qwen3-4B (fallback) + PostgreSQL + pgvector | Free tier, offline capable, grounded |
| Experiment Tracking | Python state machine | Deterministic, auditable |
| Safety Logic | Rule-based Python | No AI in safety-critical paths |
| Backend | FastAPI + PostgreSQL (Supabase free tier) | Simple, fast, easy to test |
| Frontend | Next.js + WebSocket | Modern React framework with SSR, real-time updates |
| Vector Storage | PostgreSQL + pgvector | Integrated with free database stack |
| Deployment | Docker containers | Easy to replicate, version control |

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
