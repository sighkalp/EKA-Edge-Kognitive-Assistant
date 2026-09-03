# Project EKA — Master Architecture

## Project Purpose

Project EKA is a **local-first computer vision and activity recognition system** for observing and validating experiment procedures in real-time. It uses a fixed camera to monitor an astronaut conducting an experiment and detects deviations from the expected procedure.

**Key Constraint**: All processing runs locally. No cloud services, no external APIs, no unnecessary databases.

---

## Master Architecture

```
FIXED CAMERA
    ↓
VIDEO STREAM
    ↓
┌─────────────────────────────┐
│                             │
YOLOv8n              MediaPipe Pose/Hands
│                             │
Object Tracking        Body/Hand Tracking
│                             │
└──────────────┬──────────────┘
               ↓
        3D WORLD MODEL
               ↓
    ┌──────────┼──────────┐
    ↓          ↓          ↓
Absolute    Relative   Trajectories
XYZ           XYZ
    └──────────┼──────────┘
               ↓
        GRAPH REPRESENTATION
               ↓
           GCN / GCNConv
               ↓
         SPATIAL FEATURES
               ↓
          TEMPORAL MODEL
               ↓
             ACTIVITY
               ↓
         PROTOCOL ENGINE
               ↓
          SAFETY ENGINE
               ↓
        UI + TTS + ALERTS
```

---

## Technology Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Object Detection** | YOLOv8n | Real-time detection of objects and astronaut |
| **Pose Estimation** | MediaPipe Pose | Estimate body joint positions |
| **Hand Tracking** | MediaPipe Hands | Detect hand landmarks and positions |
| **Object Tracking** | ByteTrack | Maintain consistent object IDs across frames |
| **Video Processing** | OpenCV | Read video streams, frame manipulation |
| **Spatial Computation** | Python + NumPy | Absolute/relative XYZ, distances, trajectories |
| **Graph Representation** | PyTorch Geometric | Build and manipulate spatial graphs |
| **Graph Neural Network** | GCN/GCNConv | Learn spatial relationships |
| **Temporal Model** | LSTM | Classify activities across time |
| **Protocol Engine** | Python (rule/state machine) | Validate procedure compliance |
| **Safety Engine** | Python (rule-based) | Detect anomalies and safety violations |
| **UI** | Lightweight local interface | Display camera, detections, alerts |
| **Voice Output** | Local TTS | Provide audio feedback |
| **Data Storage** | CSV files | Log detections, activities, states |

---

## Module Responsibilities

### 1. Perception (`perception/`)

**Responsibility**: Capture video and extract perception primitives.

**Input**:
- Video stream from fixed camera

**Output**:
- YOLO detections (class, confidence, bounding box, track ID)
- MediaPipe Pose landmarks
- MediaPipe Hands landmarks

**Components**:
- `yolo/` — YOLOv8n object detection wrapper
- `pose/` — MediaPipe Pose wrapper
- `hands/` — MediaPipe Hands wrapper
- `tracking/` — ByteTrack object tracking

**Key Requirements**:
- Real-time inference
- Consistent object tracking across frames
- No filtering or smoothing (spatial module handles this)

---

### 2. Spatial (`spatial/`)

**Responsibility**: Convert perception outputs into structured 3D spatial representation.

**Input**:
- YOLO detections + tracking IDs
- MediaPipe Pose landmarks
- MediaPipe Hands landmarks
- Camera calibration parameters (optional)

**Output**:
- Absolute XYZ coordinates for each entity
- Relative XYZ coordinates (e.g., wrist relative to torso)
- Distances and directions
- Velocities
- Trajectories (sequence of positions over time)

**World Model Entities**:

*Astronaut (from MediaPipe Pose)*:
- head
- torso
- shoulders (left, right)
- elbows (left, right)
- wrists (left, right)
- hands (left, right)

*Objects (from YOLO)*:
- container
- tool
- sample
- tray
- other detected objects

**Relative Mappings**:
- Wrist relative to torso
- Hand relative to head
- Object relative to torso
- Hand relative to object
- Etc.

**Components**:
- `world_model.py` — Define spatial entities and world state
- `coordinates.py` — Convert image coordinates to XYZ
- `relative_mapping.py` — Compute relative positions and distances
- `trajectory.py` — Maintain position history and compute velocity

**Key Requirements**:
- Support continuous spatial values (not just labels)
- Represent relationships precisely
- Maintain trajectory history for velocity calculation
- Keep computations lightweight

---

### 3. Graph (`graph/`)

**Responsibility**: Convert spatial representation into graph format for GCN.

**Input**:
- Spatial world model (entities and relationships)

**Output**:
- PyTorch Geometric graph with nodes, edges, and features

**Possible Nodes**:
- HEAD, TORSO, LEFT_SHOULDER, RIGHT_SHOULDER
- LEFT_ELBOW, RIGHT_ELBOW, LEFT_WRIST, RIGHT_WRIST
- HANDS
- CONTAINER, TOOL, SAMPLE, TRAY
- Other detected objects

**Possible Edges**:
- HEAD → TORSO
- TORSO → SHOULDERS
- SHOULDERS → ELBOWS
- ELBOWS → WRISTS
- WRISTS → HANDS
- WRISTS/HANDS → OBJECTS

**Node Features**:
- Position (absolute XYZ)
- Velocity (dx, dy, dz)
- Confidence scores

**Edge Features**:
- Relative position
- Distance
- Direction

**Components**:
- `graph_builder.py` — Build graphs from spatial representation
- `graph_features.py` — Compute node/edge feature vectors
- `gcn_model.py` — GCN model using PyTorch Geometric

**Key Requirements**:
- Keep graphs lightweight for local execution
- Consistent node/edge definitions
- Fast graph construction (per-frame)

---

### 4. Temporal (`temporal/`)

**Responsibility**: Model sequences of spatial features for activity recognition.

**Input**:
- Sequence of spatial/GCN feature vectors across multiple frames
- Temporal window (e.g., last 30 frames)

**Output**:
- Activity prediction (activity class + confidence)

**Model**:
- LSTM baseline for temporal sequence modeling

**Components**:
- `lstm_model.py` — LSTM implementation

**Key Requirements**:
- Learn temporal patterns in movement
- Output activity predictions
- Keep model simple (no transformers unless proven necessary)

---

### 5. Protocol Engine (`protocol/`)

**Responsibility**: Validate whether recognized activities follow the expected experiment procedure.

**Input**:
- Recognized activity
- Current protocol state (current step, completed steps, etc.)
- Procedure definition (ordered steps)

**Output**:
- `expected` — Activity matches current procedure step
- `unexpected` — Activity does not match expected step
- Next expected activity

**Example Procedure**:
```
STEP 1 → PICK TOOL
STEP 2 → MOVE TOOL
STEP 3 → INTERACT WITH CONTAINER
STEP 4 → RETURN TOOL
```

**Components**:
- `protocol_engine.py` — State machine for procedure validation

**Key Requirements**:
- Deterministic rule/state-machine based logic
- No LLM
- Maintain current experiment state
- Simple and auditable

---

### 6. Safety Engine (`safety/`)

**Responsibility**: Evaluate system state and determine safety alerts.

**Input**:
- Recognized activity
- Protocol state (expected vs. actual)
- Spatial conditions (object proximity, body configuration)
- Safety rules/constraints

**Output**:
- `SAFE` — All conditions nominal
- `WARNING` — Potential issue, alert astronaut
- `INTERVENTION` — Critical issue, require immediate action

**Example Safety Rules**:
- Object proximity thresholds
- Unexpected procedure deviations
- Risky body positions
- Anomalous trajectories

**Components**:
- `safety_engine.py` — Rule-based safety evaluation

**Key Requirements**:
- Deterministic, rule-based logic
- No AI in safety-critical decisions
- Fast evaluation
- Clear alert messages

---

### 7. UI (`ui/`)

**Responsibility**: Provide lightweight local interface for monitoring.

**Input**:
- Camera feed
- Detections (objects, landmarks)
- Spatial world model
- Current activity
- Protocol state
- Safety alerts

**Output**:
- Visual display (camera + overlays)
- User input (start experiment, etc.)

**Display Elements**:
- Live camera feed with YOLO bounding boxes
- MediaPipe pose/hand landmarks
- Current activity label
- Protocol state (step progress)
- Safety state (visual indicator)
- Active alerts/warnings

**Components**:
- `app.py` — UI application

**Key Requirements**:
- Lightweight (no heavy frameworks)
- Local only (no web server required)
- Real-time rendering
- Clear visual feedback

---

### 8. TTS (`tts/`)

**Responsibility**: Provide local voice feedback.

**Input**:
- Text messages (alerts, confirmations, current state)

**Output**:
- Audio playback (local speaker)

**Use Cases**:
- Confirm activity recognition
- Alert warnings
- Announce protocol state changes
- Voice feedback on safety issues

**Components**:
- `speaker.py` — Local TTS wrapper

**Key Requirements**:
- Use local TTS (no cloud services)
- Low latency
- Reliable on all systems

---

## Data Storage

**Primary Format**: CSV

**Local Storage Location**: `data/` directory

**Log Outputs**:
- `detections.csv` — Frame-level object and landmark detections
- `spatial.csv` — World model state (positions, velocities)
- `activities.csv` — Recognized activities with timestamps
- `protocol.csv` — Protocol state and procedure progress
- `safety.csv` — Safety alerts and conditions
- `system.csv` — Overall system events and timing

**Format Example** (detections.csv):
```
timestamp,frame_id,entity_id,entity_type,class,confidence,x,y,z,track_id
1.0,0,obj_0,container,container,0.95,100.5,150.2,50.0,1
1.0,0,pose_0,torso,astronaut,0.98,200.5,200.3,100.0,null
...
```

**Key Requirements**:
- Append-only during execution
- Human-readable format
- Easy to analyze after experiment
- Minimal I/O overhead

---

## Design Principles

### 1. Local-First
- All inference runs locally
- No cloud services
- No external APIs
- Minimal network usage

### 2. Modularity
- Clear interfaces between modules
- Independent development and testing
- Module outputs feed into next stage

### 3. Simplicity
- Use the simplest model that works
- No unnecessary complexity
- Avoid over-engineering

### 4. Determinism
- Safety-critical decisions use rules, not AI
- Auditable logic
- Reproducible results

### 5. Efficiency
- Real-time performance on consumer hardware
- Lightweight dependencies
- Minimal memory usage

### 6. Auditability
- Log all decisions and state changes
- Timestamp every action
- Enable post-experiment review

---

## Data Flow

### Per-Frame Processing

```
1. Camera → Perception
   - YOLO inference
   - MediaPipe Pose inference
   - MediaPipe Hands inference
   - ByteTrack update

2. Perception → Spatial
   - Convert detections to XYZ
   - Update trajectories
   - Compute relative positions

3. Spatial → Graph
   - Build graph from world model
   - Compute node/edge features

4. Graph → GCN
   - Forward pass through GCN
   - Extract spatial features

5. Features → Temporal
   - Add to feature buffer
   - Compute activity prediction

6. Activity → Protocol
   - Check against expected step
   - Determine expected/unexpected

7. Protocol + Activity + Spatial → Safety
   - Evaluate safety conditions
   - Generate alerts if necessary

8. Everything → UI
   - Render camera with overlays
   - Display current state
   - Show alerts

9. Activity + State → CSV
   - Log to data storage
```

---

## Module Contracts

### Perception Module Output

```json
{
  "frame_id": 42,
  "timestamp": 1.234,
  "detections": [
    {
      "class": "container",
      "confidence": 0.95,
      "bbox": [100, 150, 200, 300],
      "track_id": 1
    }
  ],
  "pose": [
    {"name": "nose", "x": 320, "y": 240, "z": 0, "confidence": 0.98},
    {"name": "left_shoulder", "x": 290, "y": 200, "z": -50, "confidence": 0.97},
    ...
  ],
  "hands": [
    {"name": "left_hand", "landmarks": [...], "confidence": 0.96},
    {"name": "right_hand", "landmarks": [...], "confidence": 0.94}
  ]
}
```

### Spatial Module Output

```json
{
  "frame_id": 42,
  "timestamp": 1.234,
  "entities": {
    "astronaut": {
      "torso": {"xyz": [500, 600, 0], "velocity": [5, 2, 0]},
      "left_wrist": {"xyz": [450, 400, -100], "velocity": [10, -5, 2]}
    },
    "container": {
      "obj_1": {"xyz": [350, 500, 50], "velocity": [0, 0, 0]}
    }
  },
  "relationships": [
    {"from": "left_wrist", "to": "container_obj_1", "distance": 150, "direction": [-0.5, -0.3, 0.8]}
  ]
}
```

### Activity Recognition Output

```json
{
  "activity": "picking_up_tool",
  "confidence": 0.87,
  "timestamp": 1.234,
  "frame_id": 42
}
```

### Protocol Engine Output

```json
{
  "current_step": 2,
  "current_activity": "pick_tool",
  "recognized_activity": "pick_tool",
  "status": "expected",
  "next_step": 3,
  "timestamp": 1.234
}
```

### Safety Engine Output

```json
{
  "state": "WARNING",
  "reason": "hand_approaching_hazard_zone",
  "confidence": 0.92,
  "recommendation": "move hand away from container edge",
  "timestamp": 1.234
}
```

---

## Development Phases

- **Phase 1**: Perception pipeline (YOLO, MediaPipe, ByteTrack)
- **Phase 2**: Spatial module (3D world model, relative mappings)
- **Phase 3**: Graph and GCN
- **Phase 4**: Temporal and activity recognition
- **Phase 5**: Protocol and Safety engines
- **Phase 6**: UI and TTS
- **Phase 7**: Integration and end-to-end testing

---

## Success Criteria

- [ ] Real-time video processing (30+ FPS)
- [ ] Accurate object and pose detection
- [ ] Correct spatial representation (XYZ, relative positions)
- [ ] Graph construction and GCN inference working
- [ ] Activity recognition functional
- [ ] Protocol engine tracks procedure correctly
- [ ] Safety engine detects anomalies
- [ ] UI displays real-time state
- [ ] TTS provides audio feedback
- [ ] Complete pipeline runs end-to-end locally
- [ ] Data logging works correctly
- [ ] Documentation is complete and accurate

---

## Constraints

- **No LLM**: No language models, no RAG, no cloud inference
- **No external APIs**: Everything runs locally
- **No databases**: CSV-based logging only
- **No microservices**: Single application
- **No Kubernetes/Docker unless necessary**: Keep deployment simple
- **No hardware dependencies**: Run on any machine with Python
- **Local execution**: All processing on user's machine

