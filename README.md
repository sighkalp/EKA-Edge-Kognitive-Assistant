# Project EKA — Edge Kognitive Assistant

A **local-first computer vision and activity recognition system** for observing and validating experiment procedures in real-time.

## Overview

EKA uses a fixed camera to monitor an astronaut conducting an experiment. It:

1. **Detects objects** in the video feed (YOLOv8n)
2. **Estimates body pose** (MediaPipe Pose)
3. **Tracks hand position** (MediaPipe Hands)
4. **Builds 3D spatial model** of the scene
5. **Recognizes activities** from movement patterns (LSTM)
6. **Validates procedures** against expected protocol (state machine)
7. **Detects safety violations** (rule-based engine)
8. **Displays results** in real-time UI with audio feedback

**Key Feature**: All processing runs locally. No cloud services. No external APIs.

---

## Quick Start

### Prerequisites

- Python 3.8+
- Webcam or video file
- ~2GB RAM minimum

### Installation

```bash
# Clone repository
git clone <repo-url>
cd EKA

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run the system
python -m ui.app
```

### Run with Video File

```bash
python -m ui.app --video path/to/video.mp4
```

---

## Repository Structure

```
EKA/
├── ARCHITECTURE.md          # Master architecture document
├── PROJECT_STATE.md         # Current implementation status
├── TODO.md                  # Implementation checklist
├── CHANGELOG.md             # Development history
├── README.md                # This file
├── requirements.txt         # Python dependencies
│
├── perception/              # Video input, YOLO, pose, tracking
│   ├── __init__.py
│   ├── yolo/
│   ├── pose/
│   ├── hands/
│   └── tracking/
│
├── spatial/                 # 3D world model, coordinates, relationships
│   ├── __init__.py
│   ├── world_model.py
│   ├── coordinates.py
│   ├── relative_mapping.py
│   └── trajectory.py
│
├── graph/                   # Graph representation for GCN
│   ├── __init__.py
│   ├── graph_builder.py
│   ├── graph_features.py
│   └── gcn_model.py
│
├── temporal/                # LSTM for activity recognition
│   ├── __init__.py
│   └── lstm_model.py
│
├── protocol/                # Protocol validation engine
│   ├── __init__.py
│   └── protocol_engine.py
│
├── safety/                  # Safety rule evaluation
│   ├── __init__.py
│   └── safety_engine.py
│
├── ui/                      # Local visualization interface
│   ├── __init__.py
│   └── app.py
│
├── tts/                     # Text-to-speech (local)
│   ├── __init__.py
│   └── speaker.py
│
├── data/                    # CSV data storage
│   ├── raw/
│   ├── processed/
│   └── csv/
│
├── tests/                   # Unit and integration tests
│
└── scripts/                 # Utility scripts
```

---

## Architecture

See `ARCHITECTURE.md` for complete system design.

**Data Flow**:

```
Camera Feed
    ↓
Perception (YOLO + MediaPipe)
    ↓
Spatial Model (3D world representation)
    ↓
Graph Representation
    ↓
GCN (spatial feature extraction)
    ↓
Temporal/LSTM (activity recognition)
    ↓
Protocol Engine (procedure validation)
    ↓
Safety Engine (anomaly detection)
    ↓
UI + TTS (visualization and alerts)
    ↓
CSV Logging
```

---

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Object Detection | YOLOv8n |
| Pose Estimation | MediaPipe Pose |
| Hand Tracking | MediaPipe Hands |
| Object Tracking | ByteTrack |
| Video I/O | OpenCV |
| Spatial Computation | NumPy |
| Graph Neural Network | PyTorch Geometric (GCN) |
| Temporal Model | LSTM (PyTorch) |
| Procedural Logic | Python (rule/state machine) |
| Safety Logic | Python (rule-based) |
| UI | Lightweight local interface |
| Voice | Local TTS |
| Data Logging | CSV |

---

## Current Status

See `PROJECT_STATE.md` for detailed implementation status.

---

## Development Checklist

See `TODO.md` for complete implementation checklist.

---

## Important Design Principles

### 1. Local-First
- All inference runs locally
- No cloud services or APIs
- Data stored in CSV files

### 2. Modularity
- Clear module boundaries
- Independent development possible
- Standard JSON interfaces

### 3. Determinism
- Safety logic uses rules, not AI
- Auditable and reproducible
- Full logging for review

### 4. Simplicity
- Use simplest model that works
- Avoid over-engineering
- Keep dependencies minimal

### 5. Efficiency
- Real-time performance target: 30+ FPS
- Lightweight inference
- Minimal resource usage

---

## Documentation

| File | Purpose |
|------|---------|
| `ARCHITECTURE.md` | Complete system design and module specifications |
| `PROJECT_STATE.md` | Current implementation status |
| `TODO.md` | Development checklist |
| `CHANGELOG.md` | Development history |

---

## Contributing

Follow the architecture strictly. Do not:
- Introduce cloud services
- Add unnecessary dependencies
- Couple modules tightly
- Use LLMs or RAG
- Create circular imports

---

## License

(Add project license here)

---

**Status**: Architecture foundation complete. Ready for module implementation.
