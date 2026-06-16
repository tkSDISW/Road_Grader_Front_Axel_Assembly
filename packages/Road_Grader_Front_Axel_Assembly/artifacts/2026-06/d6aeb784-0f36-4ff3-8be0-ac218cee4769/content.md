# Continuous Systems Integration and Deployment (CSID)
## Vision Document — SE Knowledge Partner as CSID Foundation
**Author:** SE Knowledge Partner + Engineer
**Date:** 2026-06-16
**Status:** Vision — for paper and roadmap

---

## The Parallel to Software CI/CD

Software CI/CD pipelines transformed how software is built and delivered:
- Every commit triggers automated build, test, and validation
- Defects are caught at the point of introduction, not at release
- Deployment is a routine, repeatable, low-risk operation
- The pipeline is the institutional memory of how to build the system

**CSID applies the same discipline to systems engineering:**
- Every model commit triggers automated analysis, verification, and knowledge capture
- Model defects (missing allocations, untraced requirements, unnamed exchanges, wrong types) are caught at the point of introduction
- Systems artifacts (FMEA, requirements, P-diagrams, review board findings) are generated routinely, not heroically
- The routine library is the institutional memory of how to engineer the system

The difference is that software CI/CD pipelines execute code against code. CSID pipelines execute `routine_def` artifacts against model fabric — semantic reasoning over engineering content rather than unit tests over function calls.

---

## What CSID Looks Like in Practice

### Trigger: every model commit to master

```
Engineer pushes Capella model commit
        ↓
CSID pipeline triggers
        ↓
┌─────────────────────────────────────────────┐
│  Stage 1 — Model Integrity                  │
│  routine: model_integrity_check             │
│  • Duplicate UUID scan (ISSUE-007 pattern)  │
│  • Unnamed exchange scan                    │
│  • Unallocated function scan                │
│  • Missing description scan                 │
│  Output: integrity_report artifact          │
└─────────────────────────────────────────────┘
        ↓ (pass)
┌─────────────────────────────────────────────┐
│  Stage 2 — Traceability Verification        │
│  routine: traceability_check                │
│  • Functions → capabilities traced?         │
│  • Requirements → exchanges traced?         │
│  • PA components → LA components realized?  │
│  • Property values populated?               │
│  Output: traceability_report artifact       │
└─────────────────────────────────────────────┘
        ↓ (pass)
┌─────────────────────────────────────────────┐
│  Stage 3 — Knowledge Artifact Refresh       │
│  routine: fmea_from_functional_chain        │
│    variables: [Steering Control]            │
│  routine: fmea_from_functional_chain        │
│    variables: [Camber Control]              │
│  routine: requirements_from_capability      │
│    variables: [CAP-01 through CAP-06]       │
│  Output: refreshed FMEA, requirements       │
│          artifacts on main branch           │
└─────────────────────────────────────────────┘
        ↓
┌─────────────────────────────────────────────┐
│  Stage 4 — Review Board                     │
│  routine: architecture_review               │
│    roles: [Systems Architect,               │
│            Safety Engineer,                 │
│            Requirements Engineer]           │
│  Output: review_findings artifact           │
│          issues logged to work log          │
└─────────────────────────────────────────────┘
        ↓
┌─────────────────────────────────────────────┐
│  Stage 5 — Deployment Gate                  │
│  Check: zero blocking integrity issues      │
│  Check: all required artifacts present      │
│  Check: no open FMEA items with RPN > 200   │
│  If pass → tag model commit as              │
│            "CSID-verified" baseline         │
│  If fail → notify engineer with             │
│            specific findings                │
└─────────────────────────────────────────────┘
```

---

## The Routine Library — Foundation of CSID

Just as a software CI pipeline depends on a test suite, a CSID pipeline depends on a routine library. The `routine_def` artifacts are the test suite of systems engineering.

### Initial routine library for Road Grader CSID

| Routine ID | Category | Description | Trigger |
|---|---|---|---|
| `model_integrity_check` | Integrity | Scan for duplicate UUIDs, unnamed exchanges, unallocated functions, missing descriptions | Every commit |
| `traceability_check` | Verification | Verify function→capability, requirement→exchange, LA→PA allocation completeness | Every commit |
| `fmea_from_functional_chain` | Analysis | Generate FMEA from named functional chain | On chain change |
| `p_diagram_from_chain` | Analysis | Generate P-diagram parameter structure from chain | On chain change |
| `requirements_from_capability` | Generation | Draft requirements for a named capability with testable values | On capability change |
| `property_value_completeness` | Verification | Identify PA components missing key property values (mass, pressure, torque) | Every commit |
| `exchange_naming_check` | Quality | Find unnamed or auto-named exchanges across all diagrams | Every commit |
| `architecture_review` | Review | Multi-role review board instantiation against current model state | Weekly or on demand |
| `work_log_summary` | Reporting | Generate session summary and open items list from work log | On demand |
| `fmea_comparison` | Analysis | Compare FMEA across two functional chains — identify shared components and cumulative risk | On demand |

---

## Why Routines Beat Notebooks for CSID

A Jupyter notebook CI step requires:
- A Python runtime with maintained dependencies
- API credentials injected at execution time
- Code that knows the exact schema of the model API
- A developer to update it when the API changes

A `routine_def` CSID step requires:
- A YAML artifact in the knowledge repo
- The KP to execute it
- Fresh fabric generated at runtime — no schema dependency

When the capella-fabric MCP adds a new capability (property values, NODE components, deployment links), the `routine_def` author writes new natural language instructions in the prompt template. No code changes. No dependency updates. No broken pipelines.

---

## The CSID Feedback Loop

```
Engineer models
      ↓
Git commit (master)
      ↓
CSID pipeline runs routines
      ↓
Artifacts refreshed (main branch)
      ↓
Issues logged to work log
      ↓
Engineer reviews findings
      ↓
Engineer fixes model
      ↓
Git commit (master)  ← flywheel turns again
```

Each turn of this loop:
- Catches defects earlier (at commit, not at review)
- Refreshes knowledge artifacts automatically (no manual FMEA update)
- Builds the knowledge flywheel (more artifacts = richer context = better next session)
- Creates a verifiable audit trail (every baseline is CSID-tagged)

---

## Connection to Paper

> *The `routine_def` artifact type is not just a productivity tool — it is the foundation of Continuous Systems Integration and Deployment. Just as software CI/CD pipelines transformed software delivery from a periodic high-risk event into a continuous low-risk process, CSID pipelines transform systems engineering delivery from periodic milestone reviews into continuous verified baselines. The Knowledge Partner is both the pipeline executor and the knowledge accumulator — the engine that runs the routines and captures what they produce.*

### The CSID thesis extends the paper's argument in three ways:

1. **From session to pipeline** — the KP is not just useful in individual sessions. It is the execution engine of a continuous engineering process.

2. **From artifact to baseline** — individual knowledge artifacts (FMEA, requirements, review findings) become pipeline outputs that are automatically refreshed and verifiable against specific model commits.

3. **From engineer to team** — CSID makes the KP workflow scale beyond a single engineer. The routine library is shared institutional knowledge. Any engineer on the team can trigger any routine. The pipeline is the team's collective intelligence, not one engineer's expertise.

---

## Immediate Next Steps

1. **Implement `routine_def` artifact type** (cousin) — schema validation, `list_routines`, `validate_routine_def`
2. **Author the first 3 routines** — `model_integrity_check`, `exchange_naming_check`, `fmea_from_functional_chain` (already proven)
3. **Define the CSID pipeline trigger mechanism** — GitHub Actions webhook on master commit calling the KP API? Or engineer-initiated via chat? Start with engineer-initiated, evolve to automated
4. **Define the deployment gate criteria** — what does "CSID-verified baseline" mean for the Road Grader? Propose: zero integrity issues + all required FMEA artifacts present + all capabilities have at least one requirement
