# CHANGELOG — Project EKA

---

## 2026-09-04 — ARCHITECTURAL RESET

### Completed
- Complete repository reset from old EKA architecture
- Removed: `ai/`, `backend/`, `database/`, `frontend/`, `experiment_engine/`, `models/` directories
- Removed: Old documentation (API_CONTRACTS.md, SECURITY.md, DEMO_SCENARIO.md, PROJECT_REQUIREMENTS.md)
- Created: New modular architecture structure
  - `perception/` — YOLO, MediaPipe, ByteTrack
  - `spatial/` — 3D world model, coordinates, relationships
  - `graph/` — Graph representation, GCN
  - `temporal/` — LSTM activity recognition
  - `protocol/` — Procedure validation engine
  - `safety/` — Safety rule evaluation
  - `ui/` — Local visualization
  - `tts/` — Local voice feedback
  - `data/` — CSV logging
  - `tests/` — Unit and integration tests
  - `scripts/` — Utility scripts

### Documentation Created
- `ARCHITECTURE.md` — Complete master architecture (14,736 characters)
  - Master data flow diagram
  - Technology stack
  - Module responsibilities
  - World model definition
  - Module contracts (JSON interfaces)
  - Design principles
  - Data flow walkthrough

- `PROJECT_STATE.md` — Implementation status
  - Completed items
  - In progress items
  - Not started items
  - Known issues
  - Development timeline

- `TODO.md` — Implementation checklist
  - 7 development phases
  - Module-specific tasks
  - Testing examples
  - Dependency list

- `README.md` — Project overview
  - Quick start guide
  - Repository structure
  - Architecture diagram
  - Technology stack
  - Contributing guidelines

- `CHANGELOG.md` — This file

### Key Changes
- **Architecture**: Shifted from cloud-backend multi-module system to local-first computer vision system
- **Scope**: Focused on real-time video processing and activity recognition
- **Technology**: Replaced FastAPI/React/PostgreSQL with YOLO/MediaPipe/LSTM/PyTorch
- **Data Flow**: Replaced complex microservice architecture with linear pipeline
- **Storage**: Replaced PostgreSQL with CSV-based local logging
- **Safety**: Rule-based deterministic logic (no LLM)

### Infrastructure
- Removed old Git history for modules
- Clean repository state
- All old configuration removed
- `.gitignore` and `.env.example` preserved (no longer needed, but kept for compatibility)

### Status
✅ Architecture foundation complete  
✅ Documentation complete  
✅ Repository structure established  
⏳ Ready for implementation (perception module next)

### Next Task
Implement perception module:
1. YOLO object detection wrapper
2. MediaPipe Pose integration
3. MediaPipe Hands integration
4. ByteTrack object tracking
5. Real-time video input pipeline

---

## Important Notes

### What Changed
- **From**: REST API backend + React dashboard + knowledge assistant
- **To**: Local computer vision pipeline with activity recognition

### What Stayed the Same
- Local-first philosophy (existing principle maintained)
- Modular architecture (strengthened)
- Safety-first approach (rule-based, not AI)
- Student-friendly scope (practical, limited complexity)

### Backward Compatibility
- ❌ No backward compatibility with old architecture
- ❌ Old code is not reusable (different problem domain)
- ✅ This is intentional: clean reset for new direction

---

**Repository State**: Clean, well-documented, ready for development  
**Last Update**: 2026-09-04  
**Updated By**: Architectural Reset Process
