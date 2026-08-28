# Experiment Engine

**Responsibility**: Track multi-step experiment procedures and current progress.

---

## Overview

The Experiment Engine is the procedural brain of EKA. It loads experiment SOPs, maintains current state, tracks progress, and provides context to other modules about what should be happening at each moment.

**Owner**: Member 4  
**Dependencies**: None (self-contained)  
**Technology**: Python state machine, simple database

---

## Inputs

### Experiment Definition (SOP)
- **Format**: JSON or YAML procedure definition
- **Contents**: Steps, expected activities, equipment, safety rules

Example:
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
    },
    {
      "number": 2,
      "title": "Inspect Materials",
      "duration_seconds": 300,
      "expected_activities": ["taking_measurement", "visual_inspection"]
    }
  ]
}
```

### Activity Recognition Output
- Current observed activities

### Vision Engine Output
- Currently detected objects

---

## Outputs

See `docs/API_CONTRACTS.md` for detailed JSON schema.

**Example Output**:
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

## Installation

1. **Install dependencies**
   ```bash
   pip install -r experiment_engine/requirements.txt
   ```

2. **Load experiment definitions**
   ```bash
   python experiment_engine/load_experiments.py \
     --source data/experiments/ \
     --database postgres://localhost:5432/eka_db
   ```

3. **Configure environment**
   ```bash
   cp .env.example .env
   # Set DATABASE_URL
   ```

---

## Usage

### As a Module

```python
from experiment_engine import ExperimentEngine

# Initialize
engine = ExperimentEngine(
    database_url="postgresql://user:pass@localhost:5432/eka"
)

# Load experiment
engine.load_experiment("crystal_growth_2024")

# Start experiment
engine.start_experiment(
    experiment_definition_id="crystal_growth_2024",
    astronaut_id="ast_001",
    mission_id="mission_001"
)

# Check state
state = engine.get_state()
print(f"Current step: {state['current_step']['number']}")
print(f"Expected activities: {state['current_step']['expected_activities']}")

# Advance step (called when step is complete)
engine.advance_step()

# Check progress
progress = engine.get_progress()
print(f"{progress['completed_steps']}/{progress['total_steps']} complete")
```

### As a Service (via Backend API)

```bash
curl http://localhost:8000/api/experiment/exp_001/state \
  -H "Authorization: Bearer <token>"
```

---

## Configuration

### Experiment Definitions

```
data/experiments/
├── crystal_growth_2024.json
├── protein_crystallization_2024.json
├── materials_testing_2024.json
└── README.md
```

Example experiment definition structure:
```json
{
  "id": "crystal_growth_2024",
  "title": "Controlled Crystal Growth",
  "description": "Growing single crystals in microgravity",
  "duration_minutes": 45,
  "steps": [
    {
      "number": 1,
      "title": "Prepare Workspace",
      "description": "Arrange materials on workbench",
      "duration_seconds": 300,
      "expected_activities": ["walking", "arranging_materials"],
      "equipment": ["sample_container", "growth_chamber"],
      "safety_rules": [],
      "transition_condition": "astronaut_ready"
    }
  ]
}
```

---

## Testing

### Unit Tests

```bash
pytest experiment_engine/tests/

# With coverage
pytest experiment_engine/tests/ --cov=experiment_engine
```

### Test Cases

1. **test_load_experiment**: Load SOP definition
2. **test_start_experiment**: Initialize experiment instance
3. **test_state_tracking**: Current step updates correctly
4. **test_step_advancement**: Progress to next step
5. **test_timeline_recording**: All steps recorded with timestamps
6. **test_expected_activities**: Correct activities for each step
7. **test_procedure_completion**: Mark experiment as complete
8. **test_api_contract**: JSON output schema validation

### Test Data

```
data/test/experiments/
├── simple_3step_experiment.json
├── expected_state.json
└── test_timeline.json
```

---

## Architecture

### State Machine

```
┌────────┐
│ INIT   │ (Experiment loaded, not started)
└───┬────┘
    │ start_experiment()
    ▼
┌────────┐
│ READY  │ (Waiting for astronaut to begin)
└───┬────┘
    │ on_step_start()
    ▼
┌──────────────┐
│ IN_PROGRESS  │ (Currently executing a step)
└───┬──────────┘
    │ on_step_complete() or advance_step()
    ▼
┌────────┐
│ PAUSED │ (Experiment halted temporarily)
└───┬────┘
    │ resume_experiment()
    ▼
┌──────────────┐
│ IN_PROGRESS  │
└────────────┬─┘
    (loop until last step)
             │
    after_step(N) of total_steps
             ▼
         ┌──────────┐
         │COMPLETED │
         └──────────┘
```

### Database Schema

```sql
CREATE TABLE experiments (
    id SERIAL PRIMARY KEY,
    definition_id VARCHAR(64) NOT NULL,
    mission_id VARCHAR(64) NOT NULL,
    astronaut_id VARCHAR(64) NOT NULL,
    status VARCHAR(32) NOT NULL,
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    current_step INT,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE experiment_steps (
    id SERIAL PRIMARY KEY,
    experiment_id INT REFERENCES experiments(id),
    step_number INT NOT NULL,
    title VARCHAR(256),
    status VARCHAR(32) NOT NULL,  -- pending, in_progress, completed
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    duration_seconds INT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

---

## Integration with Other Modules

### With Activity Recognition

```
Activity Recognition → "observed: securing_chamber"
             ↓
Experiment Engine → "expected: securing_chamber"
             ↓
Match ✓ → Proceed to next step
```

### With Safety Engine

```
Experiment Engine → "expected activities for step 5: securing_chamber"
             ↓
Safety Engine → Compares with observed activity
             ↓
"If observed ≠ expected → DEVIATION"
```

### With Backend API

```python
# backend/routers/experiment.py
@router.get("/api/experiment/{experiment_id}/state")
async def get_experiment_state(
    experiment_id: str,
    token: str = Depends(oauth2_scheme)
):
    state = experiment_engine.get_state(experiment_id)
    return state
```

---

## Key Methods

```python
class ExperimentEngine:
    def __init__(self, database_url: str)
    
    # Lifecycle
    def load_experiment(self, definition_id: str) -> None
    def start_experiment(self, definition_id: str, astronaut_id: str, mission_id: str) -> str
    def advance_step(self, experiment_id: str) -> None
    def pause_experiment(self, experiment_id: str) -> None
    def resume_experiment(self, experiment_id: str) -> None
    def complete_experiment(self, experiment_id: str) -> None
    
    # Query
    def get_state(self, experiment_id: str) -> dict
    def get_current_step(self, experiment_id: str) -> dict
    def get_progress(self, experiment_id: str) -> dict
    def get_timeline(self, experiment_id: str) -> List[dict]
    
    # Context
    def get_expected_activities(self, experiment_id: str) -> List[str]
    def get_required_equipment(self, experiment_id: str) -> List[str]
```

---

## Performance

- State lookup: <10ms
- Step advancement: <50ms
- Timeline generation: <100ms

---

## Troubleshooting

### "Experiment definition not found"
Check `data/experiments/` directory and load definitions:
```bash
python experiment_engine/load_experiments.py
```

### Database connection error
```bash
# Verify PostgreSQL is running
psql -U user -d eka_db

# Check DATABASE_URL in .env
```

### State inconsistency
```python
# Verify state with database
state = engine.get_state(experiment_id)
timeline = engine.get_timeline(experiment_id)
assert state['current_step'] == len(timeline) + 1
```

---

## Dependencies

See `experiment_engine/requirements.txt`:
```
sqlalchemy==2.0.23
psycopg2-binary==2.9.9
pydantic==2.5.0
```

---

## API Reference

See `docs/API_CONTRACTS.md` for full JSON schema.

Key endpoint:
```
GET /api/experiment/{experiment_id}/state
```

---

## Future Enhancements

- [ ] Dynamic step duration prediction
- [ ] Step skip capability (if authorized)
- [ ] Experiment branching (alternate procedures based on results)
- [ ] Real-time timeline visualization

---

## Questions & Support

Contact module owner: Member 4

---

**Last Updated**: 2024-01-15  
**Status**: Ready for implementation
