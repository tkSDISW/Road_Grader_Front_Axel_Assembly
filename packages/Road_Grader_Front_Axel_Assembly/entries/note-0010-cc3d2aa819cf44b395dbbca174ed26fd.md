---
id: note-0010-cc3d2aa819cf44b395dbbca174ed26fd
type: issue
timestamp: 2026-06-11T19:56:53Z
author: 
tags: []
---

## MODEL ISSUE: Steering Knuckle RH — no logical function allocated

**Discovered:** 2026-06-11, LA component review
**Component:** Steering Knuckle RH (uuid: 216a03d6-b3e9-46b7-917c-4bd14cb52f87)
**Layer:** LA
**Status:** Open

### Description
Steering Knuckle RH has 6 component ports and is the highest-connectivity node in the Logical System, sitting at the intersection of the steering, structural, and camber force paths. However, no logical function is currently allocated to it. Its physical behavior — rotating about the kingpin axis under combined steering and camber forces to deflect the right wheel — is unmodeled at the function level.

### Contrast with Steering Knuckle LH
Steering Knuckle LH (uuid: 0d5cd6cd) has `Actuate steering cylinder` allocated to it (a carry-over from the SA era, itself an open item). RH has nothing at all.

### Proposed resolution
Create and allocate a new logical function to Steering Knuckle RH:

- **Function name:** `Deflect right wheel to steering angle`
- **Description:** Rotate about the kingpin axis under cylinder and tie rod input forces to deflect the right wheel hub to the commanded steering angle.

A symmetric equivalent should also be considered for Steering Knuckle LH once the misallocated SA-era `Actuate steering cylinder` function is resolved:

- **Function name:** `Deflect left wheel to steering angle`
- **Description:** Rotate about the kingpin axis as the primary driven knuckle to deflect the left wheel hub to the commanded steering angle.

### Related open items
- Steering Knuckle LH still has SA-era `Actuate steering cylinder` misallocated (open item from 2026-06-11 [LAB] Structure review session)
- Steering Stop Assembly (uuid: a618c726) has `Limit steering travel` allocated correctly — no issue there
