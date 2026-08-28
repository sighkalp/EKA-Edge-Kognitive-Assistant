# Activity Recognition Module

**Responsibility**: Classify human activities from video to understand procedural progress.

---

## Overview

The Activity Recognition Module takes video frames (or skeleton keypoints from pose estimation) and classifies what activity the astronaut is performing. This enables the Experiment Engine to track procedure progress and the Safety Engine to detect deviations.

**Owner**: Member 1  
**Dependencies**: Vision Engine (for object context)  
**Technology**: PyTorch, Temporal CNN or Transformer

---

## Inputs

### Video Frames (Primary)
- **Source**: Raw video stream or extracted frames
- **Format**: RGB images (H×W×3)
- **Resolution**: 224×224 (resized by module)
- **Temporal Window**: 30 frames (1 second at 30 FPS)

### Optional: Skeleton Keypoints
- **Format**: Array of (x, y) positions for body joints
- **Joints**: 17 or 25 keypoints (COCO or OpenPose format)
- **Temporal Window**: 15-30 frames

### Context (Optional)
- Current experiment step
- Current procedure phase
- Expected activities for this step

---

## Outputs

See `docs/API_CONTRACTS.md` for detailed JSON schema.

**Example Output**:
```json
{
  "timestamp": 1629907200.456,
  "subject_id": "astronaut_001",
  "activity": {
    "name": "placing_sample",
    "confidence": 0.94,
    "duration_seconds": 3.2
  },
  "alternative_activities": [
    { "name": "adjusting_equipment", "confidence": 0.04 },
    { "name": "idle", "confidence": 0.02 }
  ],
  "video_segment": { "start_frame": 120, "end_frame": 135 },
  "processing_time_ms": 120,
  "status": "success"
}
```

---

## Activity Classes

```python
ACTIVITY_CLASSES = {
    "idle": 0,
    "walking": 1,
    "reaching": 2,
    "grasping": 3,
    "placing_sample": 4,
    "pouring_liquid": 5,
    "stirring": 6,
    "measuring": 7,
    "securing_chamber": 8,
    "adjusting_equipment": 9,
    "taking_measurement": 10,
    "recording_observation": 11,
    "emergency_stop": 12
}
```

Easily extensible to more activities as needed.

---

## Installation

1. **Install dependencies**
   ```bash
   pip install -r ai/activity_recognition/requirements.txt
   ```

2. **Download pretrained model**
   ```bash
   python ai/activity_recognition/download_model.py
   # Downloads model to models/weights/activity_model.pth (~200MB)
   ```

3. **Configure environment**
   ```bash
   cp .env.example .env
   # Set ACTIVITY_MODEL_PATH
   ```

---

## Usage

### As a Module

```python
from ai.activity_recognition import ActivityRecognizer

# Initialize
recognizer = ActivityRecognizer(
    model_path="models/weights/activity_model.pth",
    temporal_window=30,
    confidence_threshold=0.7
)

# Process video segment (list of frames)
frames = [cv2.imread(f) for f in frame_files]
result = recognizer.recognize(frames, subject_id="ast_001")

print(result)
# Output: activity name, confidence, alternatives
```

### As a Service (via Backend API)

```bash
curl -X POST http://localhost:8000/api/activity/recognize \
  -F "frames=@frames.tar" \
  -H "Authorization: Bearer <token>"
```

### Command Line

```bash
# Process video
python ai/activity_recognition/cli.py --input video.mp4 --output activities.json

# Process frame directory
python ai/activity_recognition/cli.py --input frames/ --output activities.json
```

---

## Configuration

### Model Selection

```python
# Fast model (30 FPS)
recognizer = ActivityRecognizer(model_size="small")

# Balanced (15 FPS)
recognizer = ActivityRecognizer(model_size="medium")

# Most accurate (5 FPS)
recognizer = ActivityRecognizer(model_size="large")
```

### Temporal Window

```python
# Use 15 frames (0.5 seconds at 30 FPS)
recognizer = ActivityRecognizer(temporal_window=15)

# Use 30 frames (1 second at 30 FPS)
recognizer = ActivityRecognizer(temporal_window=30)
```

### Context-Aware Filtering

```python
# Only recognize activities expected in current step
recognizer.set_expected_activities(
    step=5,
    activities=["securing_chamber", "adjusting_equipment"]
)
# Increases confidence for expected activities, penalizes unlikely ones
```

---

## Testing

### Unit Tests

```bash
pytest ai/activity_recognition/tests/

# With coverage
pytest ai/activity_recognition/tests/ --cov=ai.activity_recognition
```

### Test Cases

1. **test_single_activity**: Recognize placing_sample
2. **test_confidence_bounds**: Confidence in [0, 1]
3. **test_temporal_window**: Correctly processes N-frame window
4. **test_api_contract**: JSON output schema validation
5. **test_multi_class_recognition**: Distinguish between 5+ classes
6. **test_low_confidence_rejection**: Reject ambiguous activities
7. **test_latency**: Inference <200ms

### Test Data

```
data/test/activity/
├── placing_sample/
│   ├── frame_001.jpg
│   ├── frame_002.jpg
│   └── ...
├── pouring_liquid/
│   └── ...
└── expected_output.json
```

---

## Performance Benchmarks

On NVIDIA RTX 3080:

| Model | Latency (ms) | FPS | Accuracy | Memory (MB) |
|-------|---|---|---|---|
| Small (3D CNN) | 40 | 25 | 82% | 150 |
| Medium (3D CNN) | 80 | 12 | 88% | 400 |
| Large (Transformer) | 200 | 5 | 93% | 800 |

---

## Model Training

To train on custom dataset:

```bash
# Prepare labeled data
python ai/activity_recognition/training/prepare_dataset.py

# Train model
python ai/activity_recognition/training/train.py --epochs 50

# Evaluate
python ai/activity_recognition/training/evaluate.py
```

See `ai/activity_recognition/training/README.md`.

---

## Integration with Other Modules

### With Vision Engine

```
Vision Output (detected objects)
         ↓
Activity Recognizer (uses object context)
         ↓
"If human + beaker + motion → pouring activity"
```

### With Experiment Engine

```
Activity Recognition → Experiment Engine
"Observed: placing_sample"
Experiment Engine: "Expected: placing_sample"
Status: MATCH ✓ → Proceed
```

### With Safety Engine

```
Activity Recognition + Experiment Engine
         ↓
Safety Engine
"Expected: securing_chamber, Observed: adjusting_equipment"
Status: DEVIATION → Alert
```

---

## Architecture

### Model Architecture Options

**Option 1: 3D CNN** (Recommended for speed)
- Input: T×H×W×C (30×224×224×3)
- 3D convolutions capture temporal patterns
- Output: 13-class softmax
- Inference: 40-80ms

**Option 2: Transformer** (Recommended for accuracy)
- Input: Sequence of frame embeddings
- Multi-head attention captures temporal dependencies
- Output: 13-class softmax
- Inference: 150-250ms

**Option 3: Skeleton-based** (Alternative input)
- Input: 15-30 frames of keypoints
- LSTM or TCN on joint coordinates
- Output: Activity class
- Inference: 20-50ms

---

## Troubleshooting

### "Model not found"
```bash
python ai/activity_recognition/download_model.py
```

### Low accuracy on custom activities
```python
# Lower confidence threshold to catch more
recognizer = ActivityRecognizer(confidence_threshold=0.5)

# Or retrain on your specific data
# See training/README.md
```

### Slow inference
```python
# Use smaller model
recognizer = ActivityRecognizer(model_size="small")

# Or reduce temporal window
recognizer = ActivityRecognizer(temporal_window=15)
```

---

## Dependencies

See `ai/activity_recognition/requirements.txt`:
```
torch==2.1.1
torchvision==0.16.1
timm==0.9.7
numpy==1.24.3
opencv-python==4.8.1.78
```

---

## API Reference

```python
class ActivityRecognizer:
    def __init__(
        self,
        model_path: str,
        model_size: str = "medium",
        temporal_window: int = 30,
        confidence_threshold: float = 0.7,
        device: str = "auto"
    )
    
    def recognize(self, frames: List[np.ndarray], subject_id: str) -> dict
    def recognize_from_file(self, video_path: str) -> Generator[dict]
    def set_expected_activities(self, step: int, activities: List[str])
```

---

## Future Enhancements

- [ ] Multi-person activity recognition (track multiple astronauts)
- [ ] Online learning to adapt to new activities
- [ ] Pose-based activity recognition (skeleton keypoints)
- [ ] Anomaly detection (unusual activity patterns)
- [ ] Activity duration prediction

---

## Questions & Support

Contact module owner: Member 1

---

**Last Updated**: 2024-01-15  
**Status**: Ready for implementation
