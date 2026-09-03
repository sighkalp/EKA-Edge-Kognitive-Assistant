# TODO — Project EKA Implementation Checklist

---

## Phase 1: Perception Pipeline

- [ ] Implement YOLO wrapper (`perception/yolo/`)
  - [ ] Load YOLOv8n model
  - [ ] Implement inference function
  - [ ] Return detections with confidence and bounding boxes

- [ ] Implement MediaPipe Pose (`perception/pose/`)
  - [ ] Load MediaPipe Pose model
  - [ ] Extract pose landmarks
  - [ ] Return 17 body keypoints with confidence

- [ ] Implement MediaPipe Hands (`perception/hands/`)
  - [ ] Load MediaPipe Hands model
  - [ ] Extract hand landmarks
  - [ ] Detect left and right hands

- [ ] Implement ByteTrack (`perception/tracking/`)
  - [ ] Integrate ByteTrack for object tracking
  - [ ] Maintain consistent object IDs across frames
  - [ ] Handle track initialization and termination

- [ ] Implement video input (`perception/`)
  - [ ] OpenCV video capture
  - [ ] Support camera input
  - [ ] Support video file input
  - [ ] Real-time frame processing

---

## Phase 2: Spatial Module

- [ ] World Model (`spatial/world_model.py`)
  - [ ] Define entity classes (Astronaut, Object, etc.)
  - [ ] Implement world state container
  - [ ] Entity ID management

- [ ] Coordinates (`spatial/coordinates.py`)
  - [ ] Image-space to 3D coordinate conversion
  - [ ] Camera calibration handling
  - [ ] Depth estimation (if available)

- [ ] Relative Mapping (`spatial/relative_mapping.py`)
  - [ ] Compute wrist relative to torso
  - [ ] Compute hand relative to head
  - [ ] Compute object relative to torso
  - [ ] Compute hand relative to object
  - [ ] Generic distance/direction calculations

- [ ] Trajectory (`spatial/trajectory.py`)
  - [ ] Maintain position history per entity
  - [ ] Compute velocity from trajectory
  - [ ] Compute acceleration
  - [ ] Interpolate missing frames

- [ ] Integration
  - [ ] Connect perception to spatial module
  - [ ] Verify XYZ outputs
  - [ ] Test relative position calculations

---

## Phase 3: Graph Representation

- [ ] Graph Builder (`graph/graph_builder.py`)
  - [ ] Define node types (body parts, objects)
  - [ ] Build edges from body skeleton
  - [ ] Add object-to-body edges
  - [ ] Convert to PyTorch Geometric format

- [ ] Graph Features (`graph/graph_features.py`)
  - [ ] Compute node features (position, velocity)
  - [ ] Compute edge features (distance, direction)
  - [ ] Feature normalization

- [ ] GCN Model (`graph/gcn_model.py`)
  - [ ] Implement GCN using PyTorch Geometric
  - [ ] Define model architecture
  - [ ] Implement forward pass
  - [ ] Test with sample graphs

---

## Phase 4: Temporal & Activity Recognition

- [ ] LSTM Model (`temporal/lstm_model.py`)
  - [ ] Implement LSTM cell
  - [ ] Define sequence length (e.g., 30 frames)
  - [ ] Activity classification head
  - [ ] Forward pass

- [ ] Integration
  - [ ] Connect spatial/GCN features to LSTM
  - [ ] Implement inference pipeline
  - [ ] Activity prediction output

---

## Phase 5: Protocol & Safety Engines

- [ ] Protocol Engine (`protocol/protocol_engine.py`)
  - [ ] State machine implementation
  - [ ] Procedure definition format
  - [ ] Expected/unexpected determination
  - [ ] State transitions

- [ ] Safety Engine (`safety/safety_engine.py`)
  - [ ] Rule-based safety evaluation
  - [ ] Proximity checking
  - [ ] Anomaly detection
  - [ ] Alert generation (SAFE/WARNING/INTERVENTION)

---

## Phase 6: UI & TTS

- [ ] UI Application (`ui/app.py`)
  - [ ] Display camera feed
  - [ ] Overlay YOLO bounding boxes
  - [ ] Overlay pose landmarks
  - [ ] Display current activity
  - [ ] Display protocol state
  - [ ] Display safety alerts
  - [ ] Real-time rendering

- [ ] TTS Speaker (`tts/speaker.py`)
  - [ ] Local TTS initialization
  - [ ] Audio feedback for activities
  - [ ] Alert announcements
  - [ ] Procedure state announcements

---

## Phase 7: Integration & Testing

- [ ] End-to-End Pipeline
  - [ ] Connect all modules
  - [ ] Test complete data flow
  - [ ] Performance profiling
  - [ ] Optimization if needed

- [ ] CSV Logging
  - [ ] Implement logging for detections
  - [ ] Implement logging for activities
  - [ ] Implement logging for protocol state
  - [ ] Implement logging for safety alerts

- [ ] Unit Tests
  - [ ] Test perception module
  - [ ] Test spatial module
  - [ ] Test graph module
  - [ ] Test temporal module
  - [ ] Test protocol engine
  - [ ] Test safety engine

- [ ] Integration Tests
  - [ ] Test perception → spatial
  - [ ] Test spatial → graph
  - [ ] Test graph → GCN
  - [ ] Test GCN → temporal
  - [ ] Test temporal → protocol
  - [ ] Test protocol → safety
  - [ ] Test safety → UI

- [ ] Performance Testing
  - [ ] Measure FPS on target hardware
  - [ ] Profile memory usage
  - [ ] Profile CPU usage
  - [ ] Optimize hotspots

- [ ] Documentation
  - [ ] Update module READMEs
  - [ ] Document module interfaces
  - [ ] Document configuration options
  - [ ] Create usage examples

---

## Specific Implementation Tasks

### Perception Module

```python
# perception/yolo/__init__.py
def load_yolo(model_size='n'):
    """Load YOLOv8n model"""
    pass

def detect(frame):
    """
    Input: OpenCV frame
    Output: {
        'detections': [
            {'class': str, 'confidence': float, 'bbox': [x1,y1,x2,y2], 'track_id': int}
        ]
    }
    """
    pass
```

### Spatial Module

```python
# spatial/world_model.py
class Entity:
    """Base entity class"""
    def __init__(self, entity_id, entity_type, xyz, confidence):
        self.id = entity_id
        self.type = entity_type
        self.xyz = xyz  # [x, y, z]
        self.confidence = confidence
        self.velocity = [0, 0, 0]
        self.trajectory = []

# spatial/coordinates.py
def image_to_xyz(u, v, depth):
    """Convert image coordinates to 3D"""
    pass

# spatial/relative_mapping.py
def compute_relative(entity_a, entity_b):
    """
    Output: {
        'dx': float,
        'dy': float,
        'dz': float,
        'distance': float,
        'direction': [float, float, float]
    }
    """
    pass
```

### Activity Recognition

```python
# temporal/lstm_model.py
class ActivityLSTM(torch.nn.Module):
    def __init__(self, input_size, hidden_size, num_classes):
        pass

    def forward(self, features_sequence):
        """
        Input: [seq_len, feature_dim]
        Output: activity_logits [num_classes]
        """
        pass
```

### Protocol Engine

```python
# protocol/protocol_engine.py
class ProtocolEngine:
    def __init__(self, procedure_def):
        pass

    def validate(self, recognized_activity):
        """
        Output: {
            'status': 'expected' or 'unexpected',
            'current_step': int,
            'next_step': int
        }
        """
        pass
```

### Safety Engine

```python
# safety/safety_engine.py
class SafetyEngine:
    def __init__(self, safety_rules):
        pass

    def evaluate(self, activity, protocol_state, spatial_data):
        """
        Output: {
            'state': 'SAFE' or 'WARNING' or 'INTERVENTION',
            'reason': str,
            'recommendation': str
        }
        """
        pass
```

---

## Testing Examples

```python
# tests/test_perception.py
def test_yolo_loads():
    model = perception.yolo.load_yolo()
    assert model is not None

def test_detect_frame():
    frame = cv2.imread('test_frame.jpg')
    detections = perception.yolo.detect(frame)
    assert 'detections' in detections
    assert len(detections['detections']) > 0

# tests/test_spatial.py
def test_world_model():
    entity = spatial.Entity('obj_1', 'container', [100, 200, 50], 0.95)
    assert entity.id == 'obj_1'
    assert entity.xyz == [100, 200, 50]

def test_relative_mapping():
    entity_a = spatial.Entity('wrist', 'body_part', [100, 100, 0], 1.0)
    entity_b = spatial.Entity('torso', 'body_part', [150, 150, 0], 1.0)
    relative = spatial.relative_mapping.compute_relative(entity_a, entity_b)
    assert 'distance' in relative
```

---

## Dependency Installation

```bash
# When implementing, add to requirements.txt:
# - ultralytics (YOLO)
# - mediapipe
# - opencv-python
# - torch
# - torch-geometric
# - numpy
# - pyttsx3 (for local TTS)
```

---

## Progress Tracking

Update this file as modules are completed. Mark items `[x]` when done.

**Last Updated**: 2026-09-04  
**Next Checkpoint**: Perception module complete
