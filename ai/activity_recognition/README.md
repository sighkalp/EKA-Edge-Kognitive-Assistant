# Activity Recognition Module

Architecture Version: 1.1 — Architecture Standardized
Last Updated: 2026-08-29

**Responsibility**: Recognize astronaut activities from video clips and provide structured context to the experiment and safety engines.

---

## Overview

The activity recognition engine classifies operational activities from video frames using a 3D video model. It is designed to support procedure tracking and safety validation.

**Technology stack**:
- PyTorch
- Torchvision
- R3D-18

---

## Inputs

- sequence of RGB video frames
- optional mission context
- optional expected activities from the current experiment step

## Outputs

Example output:

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

## Standardized Model

The active architecture uses:
- `R3D-18` as the activity recognition model

Environment example:

```env
ACTIVITY_MODEL=R3D-18
```

This replaces older generic “3D CNN” wording and removes model-size configuration patterns such as `ACTIVITY_MODEL_SIZE=small`.

---

## Use in the System

- The frontend does not interact with this module directly.
- FastAPI is the API boundary.
- The module provides structured predictions to the experiment engine and safety engine.

---

## Testing

- Pytest for service validation
- contract checks against EKA API schemas
- activity classification regression tests
