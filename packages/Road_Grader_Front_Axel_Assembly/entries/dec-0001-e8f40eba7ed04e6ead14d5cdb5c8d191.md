---
id: dec-0001-e8f40eba7ed04e6ead14d5cdb5c8d191
type: decision
timestamp: 2026-06-11T18:13:40Z
author: 
tags: []
---

## Design Decision: Active Camber Sub-System Architecture

**Source:** CAD review and engineer clarification, 2026-06-11

### Decision
The active camber sub-system uses a single hydraulic actuator driving the right knuckle directly, with a mechanical camber assembly transferring the resulting force symmetrically to the left knuckle. The Camber Valve Assembly is a two-way valve.

### Architecture
```
Camber Valve Assembly (2-way valve, in Hydraulic System)
  ↓ Camber Hydraulic Supply
Camber Actuator
  ↓ Camber Actuator Force RH
Steering Knuckle RH (directly driven)
  ← Camber Assembly (mechanical linkage) →
Steering Knuckle LH (force transferred via assembly)
```

### Rationale
- Single actuator reduces hydraulic complexity
- Camber Assembly provides symmetric force transfer — no second actuator required
- Two-way valve allows active camber adjustment in both directions
- Consistent with CAD evidence — actuator visibly connects to RH knuckle only

### Model impact
- C 11 (Camber Actuator Force RH) correctly connects Camber Actuator → Steering Knuckle RH only
- C 9/C 10 (Camber Adjustment RH/LH) from Camber Assembly to both knuckles correctly model the symmetric transfer
- C 7/C 8 (Camber Hydraulic Supply/Return) model the two-way valve circuit
- No LH actuator exchange required — asymmetry is by design

### Patch applied
| Commit | Change |
|---|---|
| d0ba44ea | Added descriptions to Camber Assembly, Camber Actuator, Camber Valve Assembly capturing this design decision |
