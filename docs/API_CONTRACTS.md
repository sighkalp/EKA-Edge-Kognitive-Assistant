# API Contracts - EKA Module Interfaces

All modules communicate through these standardized JSON interfaces. **Do not change these contracts without updating this document and notifying all team members.**

---

## 1. Vision Engine Output

**Source Module**: `ai/vision/`

**Endpoint (if exposed)**: `POST /api/vision/detect`

**Output JSON**:

```json
{
  "timestamp": 1629907200.123,
  "frame_id": 42,
  "objects": [
    {
      "id": "obj_001",
      "class_name": "astronaut",
      "confidence": 0.95,
      "bbox": {
        "x": 100,
        "y": 150,
        "width": 200,
        "height": 300
      },
      "attributes": {
        "suit_color": "white",
        "helmet_on": true
      }
    },
    {
      "id": "obj_002",
      "class_name": "sample_container",
      "confidence": 0.88,
      "bbox": {
        "x": 400,
        "y": 200,
        "width": 80,
        "height": 100
      }
    }
  ],
  "processing_time_ms": 45,
  "status": "success"
}
```

**Field Descriptions**:
- `timestamp`: Unix timestamp of frame capture
- `frame_id`: Unique frame identifier for tracking
- `objects`: Array of detected objects
  - `id`: Unique object ID (for tracking across frames)
  - `class_name`: Object category (astronaut, equipment, sample, chamber, etc.)
  - `confidence`: Detection confidence [0.0, 1.0]
  - `bbox`: Bounding box in pixels
  - `attributes`: Optional object-specific metadata
- `processing_time_ms`: Inference latency
- `status`: "success" or "error"

**Error Response**:
```json
{
  "status": "error",
  "error_code": "INVALID_FRAME",
  "message": "Frame data is null or corrupted"
}
```

---

## 2. Activity Recognition Output

**Source Module**: `ai/activity_recognition/`

**Endpoint (if exposed)**: `POST /api/activity/recognize`

**Output JSON**:

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
    {
      "name": "adjusting_equipment",
      "confidence": 0.04
    },
    {
      "name": "idle",
      "confidence": 0.02
    }
  ],
  "video_segment": {
    "start_frame": 120,
    "end_frame": 135
  },
  "processing_time_ms": 120,
  "status": "success"
}
```

**Field Descriptions**:
- `timestamp`: When activity was recognized
- `subject_id`: ID of the person performing the activity
- `activity`: Primary recognized activity
  - `name`: Activity class name (e.g., placing_sample, securing_chamber, taking_measurement)
  - `confidence`: Recognition confidence [0.0, 1.0]
  - `duration_seconds`: How long the activity took
- `alternative_activities`: Top N alternatives (for debugging/analysis)
- `video_segment`: Frame range of the activity
- `processing_time_ms`: Model inference time
- `status`: "success" or "error"

**Activity Classes** (non-exhaustive):
- `idle`
- `walking`
- `adjusting_equipment`
- `placing_sample`
- `securing_chamber`
- `taking_measurement`
- `recording_observation`
- `emergency_stop`

---

## 3. Experiment Engine Output

**Source Module**: `experiment_engine/`

**Endpoint (if exposed)**: `GET /api/experiment/{experiment_id}/state`

**Output JSON**:

```json
{
  "experiment_id": "exp_20240101_001",
  "mission_id": "mission_001",
  "status": "in_progress",
  "current_step": {
    "step_number": 5,
    "title": "Secure chamber and initiate reaction",
    "duration_seconds": 120,
    "elapsed_seconds": 45,
    "expected_activities": [
      "securing_chamber",
      "initializing_equipment"
    ]
  },
  "procedure": {
    "total_steps": 12,
    "completed_steps": 4,
    "skipped_steps": 0
  },
  "timeline": [
    {
      "step": 1,
      "name": "Prepare workspace",
      "status": "completed",
      "timestamp": 1629907100
    },
    {
      "step": 2,
      "name": "Gather materials",
      "status": "completed",
      "timestamp": 1629907150
    }
  ],
  "context": {
    "equipment_present": ["balance", "beaker", "sample"],
    "environment": "micro_gravity"
  },
  "timestamp": 1629907200,
  "status": "success"
}
```

**Field Descriptions**:
- `experiment_id`: Unique experiment instance ID
- `mission_id`: Parent mission ID
- `status`: "pending", "in_progress", "paused", "completed", "error"
- `current_step`: Details of the current procedure step
  - `step_number`: Procedural step number
  - `title`: Human-readable step description
  - `duration_seconds`: Expected step duration
  - `elapsed_seconds`: How long we've been on this step
  - `expected_activities`: Activities expected during this step
- `procedure`: Overall procedure progress
- `timeline`: Historical record of completed steps
- `context`: Contextual metadata
- `timestamp`: When state was captured
- `status`: "success" or "error"

---

## 4. Safety Engine Output

**Source Module**: `safety_engine/`

**Endpoint (if exposed)**: `POST /api/safety/check`

**Output JSON**:

```json
{
  "timestamp": 1629907200.789,
  "deviation_detected": false,
  "check_type": "procedure_compliance",
  "alerts": [],
  "warnings": [
    {
      "id": "warn_001",
      "severity": "info",
      "category": "procedure",
      "message": "Current step is running longer than expected (45s vs 30s avg)",
      "recommendation": "Monitor progress, ready to assist if needed"
    }
  ],
  "procedure_compliance": {
    "expected_activity": "securing_chamber",
    "observed_activity": "securing_chamber",
    "match_confidence": 0.94
  },
  "violations": [],
  "status": "success"
}
```

**With Deviation**:

```json
{
  "timestamp": 1629907200.789,
  "deviation_detected": true,
  "alerts": [
    {
      "id": "alert_001",
      "severity": "critical",
      "category": "safety",
      "message": "Chamber not secured before reaction initiation. This violates safety protocol SOP-2.3.1.",
      "expected": "chamber_secured",
      "observed": "chamber_unsecured",
      "source_rules": ["SOP-2.3.1", "SAFETY-RULE-005"],
      "recommendation": "STOP. Secure chamber immediately before proceeding.",
      "confidence": 1.0
    }
  ],
  "violations": [
    {
      "violation_code": "CRITICAL_SAFETY",
      "rule_reference": "SOP-2.3.1",
      "description": "Critical safety protocol violation"
    }
  ],
  "status": "success"
}
```

**Field Descriptions**:
- `timestamp`: When check was performed
- `deviation_detected`: Boolean indicating any deviation
- `check_type`: "procedure_compliance", "equipment_check", "safety_rule", etc.
- `alerts`: Array of alerts (critical safety violations)
  - `severity`: "info", "warning", "critical"
  - `category`: "safety", "procedure", "equipment", "timeline"
  - `message`: Human-readable alert
  - `recommendation`: Suggested action for astronaut
  - `confidence`: [0.0, 1.0]
- `warnings`: Non-critical issues or suggestions
- `violations`: Structured safety violations with rule references
- `status`: "success" or "error"

**Severity Levels**:
- `info`: Informational, no action required
- `warning`: Minor deviation, monitor situation
- `critical`: Safety violation, immediate action required

---

## 5. Knowledge Assistant Output

**Source Module**: `ai/knowledge_assistant/`

**Endpoint (if exposed)**: `POST /api/knowledge/query`

**Request JSON**:

```json
{
  "query": "How do I secure the reaction chamber?",
  "context": {
    "experiment_id": "exp_20240101_001",
    "current_step": 5,
    "mission_id": "mission_001"
  }
}
```

**Output JSON**:

```json
{
  "query": "How do I secure the reaction chamber?",
  "answer": "To secure the reaction chamber: 1) Ensure all seals are in place, 2) Tighten bolts in clockwise order, 3) Verify pressure gauge reads zero, 4) Check LED indicator turns green. Refer to SOP section 2.3.1 for detailed diagrams.",
  "confidence": 0.92,
  "sources": [
    {
      "document_id": "SOP_2024_v1",
      "document_name": "Experiment_SOP.pdf",
      "section": "2.3.1",
      "page": 7,
      "snippet": "To secure the reaction chamber: Ensure all seals are in place..."
    },
    {
      "document_id": "SAFETY_HANDBOOK",
      "document_name": "Safety_Handbook.pdf",
      "section": "4.2",
      "page": 12,
      "snippet": "Chamber security is critical..."
    }
  ],
  "suggested_next_steps": [
    "Monitor pressure gauge during initialization",
    "Document chamber security verification"
  ],
  "timestamp": 1629907200,
  "status": "success"
}
```

**Error Response**:

```json
{
  "query": "Can I hack into NASA's servers?",
  "answer": null,
  "status": "error",
  "error_code": "QUERY_OUT_OF_SCOPE",
  "message": "This query is not related to approved mission documents. Please ask mission-relevant questions."
}
```

**Field Descriptions**:
- `query`: Original astronaut question
- `answer`: Grounded answer from mission documents
- `confidence`: Answer confidence [0.0, 1.0]
- `sources`: Array of source documents with page/section references
  - `document_id`, `document_name`: Source identification
  - `section`, `page`: Location in document
  - `snippet`: Relevant extracted text
- `suggested_next_steps`: Follow-up actions or queries
- `timestamp`: When query was answered
- `status`: "success" or "error"

**Special Cases**:
- If query is not answerable from approved documents: `status: "error"`, `error_code: "QUERY_OUT_OF_SCOPE"`
- If answer confidence is low: Include warning in response
- Never fabricate answers or go beyond approved documents

---

## 6. Backend Integration Response

**Endpoint**: `GET /api/mission/{mission_id}/state`

**Aggregated Output JSON** (combines all modules):

```json
{
  "mission_id": "mission_001",
  "timestamp": 1629907200,
  "state": {
    "vision": { /* Vision Engine output */ },
    "activity": { /* Activity Recognition output */ },
    "experiment": { /* Experiment Engine output */ },
    "safety": { /* Safety Engine output */ },
    "status": "nominal"
  },
  "alerts": [
    {
      "source": "safety_engine",
      "severity": "warning",
      "message": "Step 5 is running longer than expected"
    }
  ],
  "astronaut": {
    "id": "ast_001",
    "name": "Dr. Smith",
    "current_location": "Lab Module A"
  }
}
```

---

## Implementation Notes

1. **Timestamps**: Always use Unix timestamps (seconds since epoch) with optional milliseconds
2. **Confidence Scores**: Always in range [0.0, 1.0]
3. **Status Field**: All responses include a top-level `status` field ("success" or "error")
4. **Error Handling**: Errors include `error_code` and `message`
5. **IDs**: Use descriptive prefixes (e.g., `obj_`, `ast_`, `exp_`, `mission_`)
6. **Backwards Compatibility**: New fields are added as optional; existing fields are never removed

---

## Module Testing

Each module must be testable in isolation with mock data conforming to these contracts.

**Test Example**:
```python
# Test Vision Engine output format
vision_output = vision_engine.detect(frame)
assert vision_output["status"] == "success"
assert "objects" in vision_output
for obj in vision_output["objects"]:
    assert "id" in obj
    assert "confidence" in obj
    assert 0.0 <= obj["confidence"] <= 1.0
```

---

## Version Control

- **Current Version**: 1.0
- **Last Updated**: 2024-01-15
- **Next Review**: When new module is added or existing contract changes

**Change Log**:
- v1.0: Initial contracts for all 5 core modules
