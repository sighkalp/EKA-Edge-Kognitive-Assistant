# Safety Engine

Architecture Version: 1.1 — Architecture Standardized
Last Updated: 2026-08-29

**Responsibility**: Detect deviations from expected safe behavior using deterministic Python rules.

---

## Overview

The safety engine compares the current expected procedure state to observed activity and object state. It raises warnings or critical alerts when operations fall outside expected safe behavior.

**Critical requirement**:
- Safety decisions must be deterministic.
- Safety logic must not depend on an LLM.

**Technology**:
- deterministic Python rules
- explicit rule evaluation in code and configuration

---

## Inputs

- expected step state from the experiment engine
- observed activity from the activity recognition service
- observed objects from the vision service
- configured safety rules

## Outputs

```json
{
  "timestamp": 1629907200.789,
  "deviation_detected": true,
  "check_type": "procedure_compliance",
  "alerts": [
    {
      "severity": "critical",
      "category": "safety",
      "message": "Chamber not secured before reaction initiation",
      "expected": "chamber_sealed",
      "observed": "chamber_unsecured",
      "recommendation": "STOP. Secure chamber immediately.",
      "rule": "SOP-2.3.1"
    }
  ],
  "warnings": [],
  "status": "success"
}
```

---

## Rule Model

The safety logic is implemented as explicit deterministic rules, for example:
- chamber seal checks
- temperature thresholds
- equipment state validation
- procedure sequencing
- emergency-stop triggers

These rules are evaluated in code and configuration, not via a generative model.

### Example rule definition

```yaml
rules:
  - id: "RULE_CHAMBER_SEAL_001"
    title: "Chamber must be sealed before reaction"
    severity: "critical"
    condition: "current_step == 6 and observed_state.chamber_sealed == false"
    recommendation: "STOP. Seal chamber before continuing."
    reference: "SOP-2.3.1"
```

---

## Installation

```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r safety_engine/requirements.txt
```

---

## Usage

### Python usage

```python
from safety_engine import SafetyEngine

engine = SafetyEngine(rules_path="safety_engine/rules/")
result = engine.check_safety(
    expected={"current_step": 6, "expected_activity": "securing_chamber"},
    observed={"current_activity": "initializing_reaction", "chamber_sealed": False}
)

print(result)
```

### Backend API usage

```bash
curl -X POST http://localhost:8000/api/safety/check \
  -H "Content-Type: application/json" \
  -d '{
    "expected": {"current_step": 6, "expected_activity": "securing_chamber"},
    "observed": {"current_activity": "initializing_reaction", "chamber_sealed": false}
  }'
```

---

## Role in EKA

- The safety engine receives procedure context from the experiment engine.
- It evaluates observed mission state from the activity and vision services.
- It returns alerts to the backend, which can forward them to the frontend.
- Safety decisions remain explainable, auditable, and independent from LLM outputs.

---

## Safety Principles

### 1. Deterministic
- the same inputs yield the same output
- no AI or probabilistic interpretation is used for safety-critical decisions

### 2. Fail safe
- default to stop or alert when the observed state is uncertain or contradictory

### 3. Auditable
- each alert should include rule metadata, expected state, observed state, and recommendation

---

## Integration with Other Modules

### With Experiment Engine
- expected state and procedure step come from the experiment engine

### With Activity Recognition + Vision
- observed state comes from activity and object detection outputs

### With Backend API
- FastAPI exposes the safety evaluation endpoint for operational monitoring and frontend alerts

---

## Testing

- Pytest for rule evaluation
- deterministic alert validation
- safety regression tests across critical conditions
- checks that LLMs are not required for safety decisions

---

## Troubleshooting

### False positives
- verify expected activity and observed state alignment
- review rule thresholds and condition ordering
- ensure the current experiment phase is correctly set

### Rules not loading
- verify the configured rule files are present
- check YAML/JSON format and schema validity

---

## Dependencies

Core rule evaluation should remain in pure Python and should not require an LLM runtime. Dependencies are limited to the standard backend runtime and any configuration parser needed for rule definitions.

---

## Acceptance Criteria

- All critical safety checks are deterministic and explainable
- No safety decision depends on an LLM model
- Alerts are returned with expected vs observed values and a recommendation
- The rule engine remains compatible with FastAPI-backed mission monitoring

---

**Status**: Ready for implementation