# EKA Demo Scenario: Crystal Growth Experiment

## Scenario Overview

**Mission**: ISS Research Mission 2026  
**Astronaut**: Dr. Sarah Chen  
**Experiment**: Controlled Crystal Growth in Microgravity  
**Duration**: 45 minutes  
**Objective**: Grow single crystals for material science research  

---

## Experiment Background

Crystal growth in microgravity produces higher-quality crystals than on Earth because convection is eliminated. This experiment uses a solution-based growth chamber to grow a large, pure crystal under controlled conditions.

**Safety-Critical Constraints**:
- Chamber must be sealed before chemical reaction begins (prevents contamination)
- Reaction temperature must not exceed 60°C (risk of pressure buildup)
- Manual intervention is required if growth rate exceeds 2mm/hour

---

## Detailed Procedure (SOP-2024-CRYSTAL-01)

### Step 1: Prepare Workspace (5 min)
**Activities**: Walking, arranging materials
**Expected Objects**: Sample container, growth chamber, calibration kit
**Safety Rules**: None critical

**What Happens**:
- Astronaut enters lab module
- Vision engine detects Dr. Chen and growth equipment
- Experiment engine transitions to "Step 1: Prepare Workspace"
- Timeline records: "Step 1 started at 10:30:00 UTC"

**EKA Response**:
```json
{
  "experiment": {
    "current_step": 1,
    "status": "in_progress"
  },
  "vision": {
    "objects": [
      { "name": "astronaut", "id": "ast_001", "confidence": 0.99 },
      { "name": "growth_chamber", "id": "obj_001", "confidence": 0.92 },
      { "name": "sample_container", "id": "obj_002", "confidence": 0.88 }
    ]
  },
  "safety": { "alerts": [] }
}
```

---

### Step 2: Inspect Materials (5 min)
**Activities**: Taking measurement, visual inspection, recording observations
**Expected Objects**: Sample vial, calibration scale, documentation
**Safety Rules**: None critical

**What Happens**:
- Astronaut measures sample weight using precision balance
- Activity recognition detects "taking_measurement"
- Knowledge assistant may be queried: "What is the target crystal weight?"
- Answer sourced from SOP-2024-CRYSTAL-01 page 2

**Knowledge Query Example**:
```json
{
  "query": "What is the target crystal weight for this experiment?",
  "answer": "The target crystal weight is 2.5 ± 0.1 grams. Crystals heavier than 3.0 grams must be sectioned per safety protocol.",
  "sources": [
    {
      "document": "SOP-2024-CRYSTAL-01",
      "page": 2,
      "section": "2.1 - Material Specifications"
    }
  ],
  "confidence": 0.98
}
```

---

### Step 3: Prepare Growth Solution (8 min)
**Activities**: Pouring liquid, dissolving salt, stirring
**Expected Objects**: Beaker, growth solution, stirring rod, sample vial
**Safety Rules**:
- Solution temperature must be 35-45°C before use
- All containers must be labeled with experiment ID

**What Happens**:
- Activity recognition detects: "pouring_liquid" → "dissolving_salt" → "stirring"
- Vision detects beaker, solution, stirring rod
- Experiment engine remains in Step 3
- If astronaut exceeds 10 minutes, safety engine issues warning (info level)

**Normal Scenario**:
```json
{
  "activity": {
    "name": "pouring_liquid",
    "confidence": 0.93,
    "duration": 2.3
  },
  "safety": {
    "warnings": [
      {
        "severity": "info",
        "message": "Step 3 is slightly behind schedule (8m vs 6m avg). No action needed, continue at current pace."
      }
    ]
  }
}
```

---

### Step 4: Calibrate Temperature Probe (3 min)
**Activities**: Inserting probe, reading calibration value
**Expected Objects**: Temperature probe, calibration standard, readout device
**Safety Rules**: Probe must be inserted to marking line (prevents air bubbles)

**What Happens**:
- Activity recognition detects "inserting_probe"
- Vision confirms probe placement
- Temperature probe reading is logged
- If deviation occurs, safety engine alerts

**Deviation Scenario**: Astronaut inserts probe too shallow

```json
{
  "safety": {
    "deviation_detected": true,
    "alerts": [
      {
        "severity": "warning",
        "message": "Temperature probe inserted to 5cm (expected 8cm per SOP-2024-CRYSTAL-01:3.2). Inaccurate readings may result. Please reinject probe to full depth.",
        "recommendation": "Reinject probe to 8cm marking.",
        "rule": "SOP-2024-CRYSTAL-01:3.2"
      }
    ]
  }
}
```

---

### Step 5: Transfer Solution to Chamber (5 min)
**Activities**: Pouring, sealing cap, securing chamber
**Expected Objects**: Growth chamber, solution, sealing cap, bolts
**Safety Rules**:
- **CRITICAL**: Chamber must be sealed before moving to next step
- Cap must be tight (pressure gauge should show 0 PSI initially)
- At least 3 bolt turns minimum, torque <10 N⋅m

**What Happens**:
- Activity recognition: "pouring_solution" → "sealing_chamber"
- Vision confirms: solution in chamber, cap in place
- Experiment engine verifies: "chamber_sealed" status
- If any seal is loose, safety engine detects before next step

**Critical Safety Event**: Astronaut forgets to seal chamber

Timeline:
1. Dr. Chen pours solution into chamber (✓)
2. Dr. Chen begins reaching for tool (Step 6 is temperature control setup)
3. Dr. Chen activates chemical reaction without sealing chamber
4. Safety engine detects mismatch:

```json
{
  "safety": {
    "deviation_detected": true,
    "alerts": [
      {
        "severity": "critical",
        "category": "safety",
        "message": "CRITICAL SAFETY VIOLATION: Attempting to initialize reaction with unsealed growth chamber. This violates SOP-2024-CRYSTAL-01:5.1 and SAFETY-RULE-CHAMBER-SEAL-001. Proceeding would risk pressure explosion and contamination.",
        "expected_state": "chamber_sealed_and_pressurized",
        "observed_state": "chamber_unsealed_with_open_solution",
        "recommendation": "STOP IMMEDIATELY. Seal chamber with cap and tighten bolts to 5 N⋅m. Verify pressure gauge reads 0 PSI. Confirm green LED is lit.",
        "rules": ["SOP-2024-CRYSTAL-01:5.1", "SAFETY-RULE-CHAMBER-SEAL-001"],
        "confidence": 1.0
      }
    ],
    "violations": [
      {
        "code": "CRITICAL_SAFETY_VIOLATION",
        "category": "chamber_integrity"
      }
    ]
  }
}
```

**Frontend Display**:
```
🚨 CRITICAL ALERT 🚨
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Chamber not sealed before reaction start!

Expected: Chamber fully sealed + pressurized
Observed: Open chamber with liquid

ACTION REQUIRED:
1. STOP - Do not activate reaction
2. Seal chamber with cap
3. Tighten bolts to 5 N⋅m torque
4. Wait for green LED (sealed confirmation)

Reference: SOP-2024-CRYSTAL-01 Section 5.1
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

System prevents step advancement until deviation is resolved.

---

### Step 6: Initialize Reaction (2 min)
**Activities**: Pressing start button, verifying LED indicator
**Expected Objects**: Control panel, LED indicator
**Safety Rules**:
- Green LED must be lit before pressing start (confirms chamber sealed)
- Initial temperature must be 35-45°C

**Normal Execution**:

```json
{
  "activity": {
    "name": "pressing_control_button",
    "confidence": 0.96
  },
  "vision": {
    "objects": [
      {
        "name": "led_indicator",
        "status": "green_lit",
        "confidence": 0.99
      }
    ]
  },
  "safety": {
    "alerts": []
  }
}
```

---

### Step 7: Monitor Reaction (12 min)
**Activities**: Observing chamber, taking temperature readings, recording data
**Expected Objects**: Chamber, temperature readout, observation window
**Safety Rules**:
- Temperature must remain 35-60°C (alert if exceeds 50°C)
- Growth rate must not exceed 2mm/hour (manual intervention if rate spikes)

**Monitoring Timeline**:

```
10:45:00 - Reaction started, initial temp 38.2°C ✓
10:47:30 - Crystal visible, growth rate 0.8mm/h ✓
10:50:15 - Temp rising, now 48.3°C (info alert: approaching limit)
10:52:45 - Temp 51.2°C (warning alert: exceeds 50°C target)
```

**Warning Alert at T+10m**:

```json
{
  "safety": {
    "alerts": [
      {
        "severity": "warning",
        "category": "temperature",
        "message": "Reaction temperature has exceeded 50°C target (current: 51.2°C). This may result in uncontrolled crystal growth or pressure buildup. Monitor closely.",
        "recommendation": "Reduce heat source or increase coolant flow (refer to SOP Step 7.3). If temp continues rising beyond 60°C, consider emergency stop.",
        "rule": "SAFETY-RULE-TEMP-LIMIT-001"
      }
    ]
  }
}
```

Astronaut responds by reducing heater intensity per SOP Step 7.3.

---

### Step 8: Complete Reaction (3 min)
**Activities**: Monitoring final crystal formation, pressing stop button
**Expected Objects**: Chamber with crystal, control panel
**Safety Rules**: None critical

**Observation**:

```json
{
  "vision": {
    "objects": [
      {
        "name": "crystal",
        "description": "Large clear crystal visible in chamber",
        "size_estimate": "2.4 grams",
        "confidence": 0.87
      }
    ]
  },
  "activity": {
    "name": "pressing_control_button",
    "confidence": 0.96
  }
}
```

---

### Step 9: Cool and Extract (7 min)
**Activities**: Waiting for cooling, carefully removing crystal
**Expected Objects**: Chamber, cooling chamber, forceps, storage vial
**Safety Rules**:
- Crystal must cool to <30°C before removal (touch test: cool to hand)
- Must use approved storage container
- Do not touch crystal with bare hands

**Knowledge Query**:

```json
{
  "query": "How do I know when the crystal is cool enough to handle?",
  "answer": "The crystal should be cool enough to touch with a bare hand for 3 seconds without discomfort. Alternatively, wait 5 minutes after stopping the reaction. If in doubt, err on the side of caution—a few extra minutes of cooling prevents thermal cracking.",
  "sources": [
    {
      "document": "SOP-2024-CRYSTAL-01",
      "page": 9,
      "section": "9.2 - Cooling Procedures"
    }
  ]
}
```

---

### Step 10: Document Results (3 min)
**Activities**: Writing observations, weighing final product, photographing
**Expected Objects**: Crystal, documentation form, scale, camera
**Safety Rules**: None critical

**Final Measurement**:

```json
{
  "activity": {
    "name": "taking_measurement",
    "confidence": 0.92
  },
  "vision": {
    "objects": [
      {
        "name": "scale",
        "reading": "2.47 grams",
        "confidence": 0.94
      }
    ]
  }
}
```

---

## Complete Mission Timeline

```
START: 2026-01-15 10:30:00 UTC

10:30 - 10:35  │ Step 1: Prepare Workspace        │ ✓ Completed
10:35 - 10:40  │ Step 2: Inspect Materials        │ ✓ Completed
10:40 - 10:48  │ Step 3: Prepare Solution         │ ✓ Completed (+2m)
10:48 - 10:51  │ Step 4: Calibrate Probe          │ ✓ Completed
10:51 - 10:56  │ Step 5: Transfer & Seal Chamber  │ ✓ Completed
10:56 - 10:58  │ Step 6: Initialize Reaction      │ ✓ Completed
10:58 - 11:10  │ Step 7: Monitor (ALERT: Temp)    │ ⚠ Warning, resolved
11:10 - 11:13  │ Step 8: Complete Reaction        │ ✓ Completed
11:13 - 11:20  │ Step 9: Cool & Extract           │ ✓ Completed
11:20 - 11:23  │ Step 10: Document Results        │ ✓ Completed

END: 2026-01-15 11:23:00 UTC
Duration: 53 minutes (8 min over 45 min target due to monitoring alert)

Final Outcome: SUCCESS
- Crystal successfully grown: 2.47 grams
- All safety protocols followed
- One warning alert (temperature), responded appropriately
- Complete audit trail recorded
```

---

## Audit Log Snapshot

```json
{
  "mission_id": "mission_2026_001",
  "astronaut": "ast_chen_sarah",
  "experiment_id": "exp_crystal_01",
  "events": [
    {
      "timestamp": "2026-01-15T10:30:00Z",
      "event": "experiment_start",
      "step": 1,
      "status": "success"
    },
    {
      "timestamp": "2026-01-15T10:52:45Z",
      "event": "safety_alert",
      "severity": "warning",
      "alert_id": "alert_temp_001",
      "message": "Temperature exceeded 50°C",
      "resolved_at": "2026-01-15T10:55:30Z"
    },
    {
      "timestamp": "2026-01-15T11:23:00Z",
      "event": "experiment_complete",
      "outcome": "success",
      "crystal_mass_grams": 2.47,
      "audit_trail_events": 47
    }
  ]
}
```

---

## Key EKA Behaviors Demonstrated

### ✓ Vision Engine
- Detects astronaut, equipment, crystal formation
- Provides confidence scores

### ✓ Activity Recognition
- Recognizes actions: pouring, sealing, measuring, pressing buttons
- Tracks activity duration

### ✓ Experiment Engine
- Tracks 10-step procedure
- Reports current step and progress
- Associates expected activities with each step

### ✓ Safety Engine
- **Detects critical deviation**: Unsealed chamber before reaction
- **Detects warning**: Temperature exceeds safe threshold
- Issues severity-appropriate alerts
- Provides actionable recommendations

### ✓ Knowledge Assistant
- Answers astronaut questions with document sources
- Provides specific pages and sections

### ✓ Backend + Database
- Aggregates all module outputs
- Records complete audit trail
- Timestamps all events
- Calculates procedure progress

### ✓ Frontend Dashboard
- Displays real-time object detections
- Shows current step and progress bar
- Shows alerts prominently (color-coded by severity)
- Shows Q&A interface
- Updates timeline with completed steps

---

## Test Success Criteria

- [ ] All 10 steps execute without system crash
- [ ] Vision detects objects with >85% accuracy
- [ ] Activity recognition classifies actions with >80% accuracy
- [ ] Safety engine detects the critical chamber-seal deviation
- [ ] Safety engine detects temperature warning
- [ ] Knowledge assistant answers questions with correct sources
- [ ] Audit log records all 47+ events
- [ ] Frontend dashboard updates in real-time (<1s latency)
- [ ] Complete timeline is queryable and exportable
- [ ] No data loss or corruption

---

**Document Version**: 1.1  
**Last Updated**: 2026-08-29  
**Reference**: Used for end-to-end system testing and demo presentation
