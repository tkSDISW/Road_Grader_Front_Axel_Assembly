---
id: note-0011-bba5830c935640cbb169ebadaefba6d3
type: milestone
timestamp: 2026-06-11T19:58:24Z
author: 
tags: []
---

## ISSUE RESOLVED: Steering Knuckle RH — missing function allocated

### Patches applied

| Commit | Change |
|---|---|
| 1a06202d | Created LogicalFunction: Deflect right wheel to steering angle |
| 86f834ca | Allocated function to Steering Knuckle RH (uuid: 216a03d6) |

### Function created

| Field | Value |
|---|---|
| Name | Deflect right wheel to steering angle |
| UUID | c24b4e23-2022-4609-a27e-d164f0a8819c |
| Type | LogicalFunction |
| Layer | LA |
| Allocated to | Steering Knuckle RH |
| Description | Rotate about the kingpin axis under cylinder and tie rod input forces to deflect the right wheel hub to the commanded steering angle. |

### Still open
- Steering Knuckle LH: create symmetric function Deflect left wheel to steering angle and reallocate away from SA-era Actuate steering cylinder
