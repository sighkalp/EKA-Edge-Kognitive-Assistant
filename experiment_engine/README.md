# Experiment Engine

Architecture Version: 1.1 — Architecture Standardized
Last Updated: 2026-08-29

**Responsibility**: Track procedure state and progress across an experiment lifecycle.

---

## Overview

The experiment engine uses a Python state machine to keep track of which step is active, what is expected next, and whether the procedure is progressing normally.

It is responsible for turning an experiment definition or SOP into a deterministic runtime state that can be checked by the safety engine and presented to the frontend.

**Technology**:
- Python state machine
- deterministic experiment state handling

---

## Inputs

- experiment definition or SOP data
- current mission context
- recognized activities from the activity recognition service
- detected objects from the vision service

## Outputs

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
    "expected_activities": ["securing_chamber", "initializing_equipment"]
  },
  "procedure": {
    "total_steps": 12,
    "completed_steps": 4,
    "skipped_steps": 0
  },
  "timeline": [
    { "step": 1, "name": "Prepare workspace", "status": "completed", "timestamp": 1629907100 }
  ],
  "status": "success"
}
```

---

## Example Experiment Definition

```json
{
  "experiment_id": "crystal_growth_2024",
  "title": "Controlled Crystal Growth in Microgravity",
  "steps": [
    {
      "number": 1,
      "title": "Prepare Workspace",
      "duration_seconds": 300,
      "expected_activities": ["walking", "arranging_materials"],
      "equipment": ["sample_container", "growth_chamber", "calibration_kit"]
    }
  ]
}
```

---

## Installation

```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r experiment_engine/requirements.txt
```

---

## Usage

### Python usage

```python
from experiment_engine import ExperimentEngine

engine = ExperimentEngine()
engine.load_experiment("crystal_growth_2024")
engine.start_experiment(
    experiment_definition_id="crystal_growth_2024",
    astronaut_id="ast_001",
    mission_id="mission_001"
)

state = engine.get_state()
print(state)
```

### Backend API usage

```bash
curl http://localhost:8000/api/experiment/exp_001/state \
  -H "Authorization: <token>"
```

---

## Role in EKA

- Provides step-level procedural context to the safety engine.
- Helps the frontend visualize mission progress.
- Supports the knowledge assistant with task context.
- Remains deterministic and independent from LLM-based reasoning.

---

## State Machine Model

A simplified lifecycle is:

```text
INIT -> READY -> IN_PROGRESS -> PAUSED -> IN_PROGRESS -> COMPLETED
```

The engine should manage:
- current_step
- elapsed time
- completed and skipped steps
- transition validation
- timeline updates
- experiment completion state

---

## Configuration

The canonical state configuration is kept in code and experiment definitions, not in LLM-generated logic. The engine uses explicit procedural data for expected activities and transitions so that safety and UI layers can rely on deterministic state.

---

## Integration with Other Modules

### With Activity Recognition
- observed activity is compared against expected activities for the current step

### With Vision
- detected objects provide mission context and equipment presence information

### With Safety Engine
- expected state is passed to the safety engine for deviation detection

### With Backend API
- FastAPI exposes current experiment state for frontend and operational services

---

## Testing

- Pytest for state transition validation
- procedure completion and timeline checks
- contract validation against the standardized API output
- check that safety-critical transitions are deterministic and reproducible

---

## Troubleshooting

### State mismatch
- verify the experiment definition is loaded correctly
- review the current step and timeline log
- check whether a transition was skipped or failed validation

### Missing expected activities
- confirm the mission context includes the current recognized activities
- verify the experiment definition metadata matches the expected procedure step

---

## Dependencies

The module should depend on the standard Python runtime and explicit experiment data definitions, without introducing AI-based procedural decisions.

---

## Acceptance Criteria

- Current experiment state is deterministic and reproducible
- The engine tracks the active step, completed work, and expected next activities
- Safety engine receives consistent procedure context
- The output contract remains JSON-compatible and usable by the frontend and backend APIs

---

**Status**: Ready for implementation