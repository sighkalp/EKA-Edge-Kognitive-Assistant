# Vision Engine

Architecture Version: 1.1 — Architecture Standardized
Last Updated: 2026-08-29

**Responsibility**: Detect astronauts, tools, and relevant scene objects from video frames.

---

## Overview

The vision engine is the repository’s local inference service for object detection. It processes raw frames, identifies relevant objects, and emits structured detections that can be consumed by FastAPI and then surfaced to the frontend.

This module is a backend-internal service. The frontend does not call the vision engine directly; it receives the processed results through FastAPI.

**Technology stack**:
- Ultralytics
- YOLO11n
- PyTorch
- OpenCV
- Python inference service

---

## Inputs

- raw video frames or sampled frames
- optional ROI and confidence settings
- optional mission context metadata
- output target for downstream safety and activity evaluation

## Outputs

The engine emits detection payloads that are consumed by FastAPI and then exposed to the frontend.

```json
{
  "timestamp": 1629907200.123,
  "frame_id": 42,
  "objects": [
    {
      "id": "obj_001",
      "class_name": "astronaut",
      "confidence": 0.95,
      "bbox": { "x": 100, "y": 150, "width": 200, "height": 300 },
      "attributes": { "suit_color": "white", "helmet_on": true }
    }
  ],
  "processing_time_ms": 45,
  "status": "success"
}
```

---

## Key Features

### Object classes
- astronaut
- tools
- equipment
- sample containers
- chamber or work area objects relevant to mission operations

### Performance goals
- fast local inference for mission monitoring
- deterministic, structured output suitable for downstream safety checks
- support for current mission workflows and frontend dashboard consumption

---

## Installation

```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r ai/vision/requirements.txt
```

Model weights are managed through the configured Ultralytics workflow for YOLO11n.

---

## Configuration

The active architecture standard uses YOLO11n.

```env
YOLO_MODEL=YOLO11n
YOLO_CONFIDENCE_THRESHOLD=0.6
```

This replaces older YOLOv5-yolo5s-era references and standardizes the repository on the current Ultralytics model naming.

---

## Usage

### Python usage

```python
from ai.vision import VisionEngine

engine = VisionEngine(
    model_name="YOLO11n",
    confidence_threshold=0.6,
    device="auto"
)

result = engine.detect(frame)
print(result)
```

### Backend API usage

```bash
curl -X POST http://localhost:8000/api/vision/detect \
  -F "frame=@frame.jpg"
```

---

## Operational Notes

- Inference is performed in Python, using Ultralytics with YOLO11n.
- OpenCV is used for frame processing and normalization.
- Output is shaped according to the EKA API contracts.
- The vision engine is a backend-internal service, not a frontend component.
- The module feeds downstream activity recognition and safety checks with structured object detections.

---

## Integration with EKA

- The frontend calls FastAPI.
- FastAPI invokes the vision module when mission frames need analysis.
- The vision engine returns a structured detection payload.
- The activity recognition and safety engines consume those outputs as upstream mission state.

This module exists to normalize scene understanding before safety and procedural logic evaluate the mission.

---

## Testing

- Pytest for backend service validation
- model output validation against contract samples
- frame-level regression checks for detection quality
- detection fidelity checks for mission-critical objects

---

## Troubleshooting

### Slow inference
- confirm the correct YOLO11n model is loaded
- reduce image size or ROI complexity
- check whether GPU or CPU execution is configured correctly

### Low detection accuracy
- verify confidence threshold and frame quality
- review lighting and occlusion conditions
- use the configured mission-specific object labels

### Model not loading
- ensure the Ultralytics environment is installed correctly
- verify the selected model name matches `YOLO11n`

---

## Dependencies

Core packages should be pinned in `ai/vision/requirements.txt` and should cover:
- ultralytics
- torch
- opencv-python
- numpy
- pillow

---

## Acceptance Criteria

- The repository standard names the model as Ultralytics + YOLO11n
- Detection output remains structured and JSON-compatible
- Inference remains local and Python-based
- Backend FastAPI owns the public API boundary
- The module supports downstream safety and activity recognition workflows

---

**Status**: Ready for implementation