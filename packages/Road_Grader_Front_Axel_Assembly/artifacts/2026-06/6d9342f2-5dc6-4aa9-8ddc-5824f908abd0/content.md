# KP Routine Specification — `routine_def` Artifact Type
## Specification for cousin (Claude Code) implementation
**Author:** SE Knowledge Partner + Engineer
**Date:** 2026-06-16
**Status:** Draft — for cousin implementation
**Context:** Derived from the FMEA generation session using the Steering Control functional chain

---

## Background — Why Not Jupyter Notebooks

The original KP vision used Jupyter notebooks as the replayable routine format. The Road Grader sessions revealed why this is suboptimal:

| Dimension | Jupyter Notebook | routine_def (proposed) |
|---|---|---|
| Source material | Code cells — execution-centric | Artifact-centric — prompt, knowledge, model |
| Input declaration | Implicit in code | Explicit contract — declared inputs with types |
| Output declaration | Side effects | Explicit — named artifacts to be produced |
| Model awareness | None — requires manual API calls | First-class — model repo declared as resource |
| Replayability | Code-level — brittle to model changes | Semantic-level — adapts to current model state via fabric |
| Knowledge capture | Not included | Built-in — log entry is a required output |
| Authoring | Requires coding skill | YAML + natural language — authoring by engineer or KP |
| Version control | .ipynb JSON — noisy diffs | Clean YAML — readable diffs |

The FMEA generation session demonstrated the routine pattern concretely: it had declared inputs (functional chain name, model repo), produced a declared output (FMEA artifact), used a variable (chain name), and required a resource (the model). A Jupyter notebook would have encoded the API calls. The `routine_def` encodes the engineering intent.

---

## The `routine_def` Artifact Type — Specification

### Overview
A `routine_def` is a structured YAML artifact stored in the knowledge repo that defines a replayable engineering analysis or modeling task. It is not executable code — it is a declarative contract that the KP executes at runtime by combining the declared inputs, variables, prompt template, and resources into a live session.

The KP reads the `routine_def`, resolves all inputs and resources, renders the prompt template with the provided variables, executes the analysis, and writes the declared outputs — all in a single session.

---

### Schema

```yaml
routine_def:
  # ── Identity ──────────────────────────────────────────────────────────────
  id: fmea_from_functional_chain          # unique identifier, snake_case
  name: "FMEA from Functional Chain"      # human-readable name
  version: "1.0"
  description: >
    Generates a structured FMEA report artifact by extracting all functional
    exchanges from a named functional chain and analyzing each for failure
    modes, local effects, end effects, and RPN scores.
  author: SE Knowledge Partner
  tags: [fmea, functional-chain, analysis, safety]

  # ── Variables ─────────────────────────────────────────────────────────────
  # Values the caller must provide at execution time
  variables:
    - name: functional_chain_name
      type: string
      required: true
      description: "Name of the functional chain to analyze (e.g. 'Steering Control')"
      example: "Steering Control"

    - name: artifact_package
      type: string
      required: true
      description: "Target artifact package in the knowledge repo"
      example: "Road_Grader_Front_Axel_Assembly"

    - name: output_branch
      type: string
      required: false
      default: "main"
      description: "Knowledge repo branch for artifact output"

  # ── Resources ─────────────────────────────────────────────────────────────
  # External systems the routine needs access to at execution time
  resources:
    - id: capella_model
      type: capella_model_repo
      description: "Capella model repository containing the functional chain"
      required: true
      branch: master
      mcp_tool: capella-fabric
      validation:
        # Routine will fail pre-flight if these are not found in the model
        require_object:
          - type: FunctionalChain
            name: "{{ functional_chain_name }}"
            phase: [SA, LA, PA]

    - id: knowledge_repo
      type: artifact_repo
      description: "Knowledge repository for artifact input and output"
      required: true
      branch: "{{ output_branch }}"
      mcp_tool: artifact-repo

  # ── Inputs ────────────────────────────────────────────────────────────────
  # Artifacts that must exist before execution begins
  # The routine reads these to inform its analysis
  inputs:
    - artifact_id_pattern: "FMEA — *"
      package: "{{ artifact_package }}"
      required: false
      description: >
        Any existing FMEA artifacts for this project — used to check for
        duplicate analysis and to cross-reference RPN scores across chains
      bind_as: prior_fmea_artifacts

    - artifact_id_pattern: "Work Log — *"
      package: "{{ artifact_package }}"
      required: false
      description: "Project work log — used to add milestone entry on completion"
      bind_as: work_log

  # ── Outputs ───────────────────────────────────────────────────────────────
  # Artifacts the routine is required to produce
  # Execution is not considered complete until all required outputs exist
  outputs:
    - name: "FMEA — {{ functional_chain_name }}"
      type: text
      package: "{{ artifact_package }}"
      branch: "{{ output_branch }}"
      required: true
      description: "Structured FMEA table with S/O/D/RPN scores and recommended actions"
      tags: [fmea, "{{ functional_chain_name | lower | replace(' ', '-') }}", analysis]
      bind_as: fmea_artifact

    - name: work_log_entry
      type: log_entry
      entry_type: milestone
      log_book: "{{ work_log.artifact_id }}"
      required: true
      description: "Milestone entry recording the FMEA execution and artifact ID"

  # ── Pre-flight checks ─────────────────────────────────────────────────────
  # Conditions verified before execution begins
  # Routine aborts with clear error if any check fails
  pre_flight:
    - check: resource_accessible
      resource: capella_model
      error: "Cannot access Capella model repo. Verify PAT and repo URL."

    - check: resource_accessible
      resource: knowledge_repo
      error: "Cannot access knowledge repo on branch {{ output_branch }}."

    - check: object_exists
      resource: capella_model
      object_type: FunctionalChain
      object_name: "{{ functional_chain_name }}"
      error: >
        Functional chain '{{ functional_chain_name }}' not found in model.
        Available chains: [list from browse_model]

    - check: no_duplicate_output
      output: fmea_artifact
      error: >
        FMEA artifact for '{{ functional_chain_name }}' already exists
        (artifact_id: {{ existing.artifact_id }}). Delete it or use
        version: overwrite to replace.

  # ── Prompt template ───────────────────────────────────────────────────────
  # Jinja2 template rendered at execution time with all variables resolved
  # This is the ignition — what the KP reasons over
  prompt_template: |
    You are the SE Knowledge Partner operating in FMEA analysis mode.

    ## Context
    - Functional chain: {{ functional_chain_name }}
    - Model: {{ capella_model.repo_url }} (branch: master)
    - Output package: {{ artifact_package }}
    {% if prior_fmea_artifacts %}
    - Prior FMEAs in this package: {{ prior_fmea_artifacts | length }} found
      Cross-reference RPN scores where chains share components.
    {% endif %}

    ## Execution steps
    The fabric for '{{ functional_chain_name }}' has been generated above.
    Proceed through the following steps in order:

    1. Extract all FunctionalExchange objects from the chain fabric.
       For each exchange identify: source function, target function,
       allocated component (from parent or allocation), exchange type.

    2. Generate FMEA rows per the schema in the routine outputs section.
       Apply severity guidance:
       - Loss of steering function = S 9–10
       - Degraded steering accuracy = S 6–8
       - Loss of camber control = S 5–7
       - Maintenance / lifecycle = S 2–4

    3. Write the FMEA as artifact '{{ outputs.fmea_artifact.name }}'
       to package '{{ artifact_package }}' on branch '{{ output_branch }}'.

    4. Add a milestone log entry to the work log recording:
       - Chain analyzed, exchange count, FMEA row count
       - Artifact ID of the saved FMEA
       - Top 3 RPN items
       - Gaps identified (missing property values, unallocated functions)

    5. Report summary to engineer.

  # ── Post-execution validation ─────────────────────────────────────────────
  post_execution:
    - check: output_exists
      output: fmea_artifact
      error: "FMEA artifact was not written. Check knowledge repo session."

    - check: log_entry_written
      output: work_log_entry
      error: "Work log milestone was not written."

    - report:
        summary_fields:
          - fmea_row_count
          - top_3_rpn_items
          - gaps_identified
          - artifact_id: "{{ fmea_artifact.artifact_id }}"
```

---

## How the KP Executes a `routine_def`

When an engineer says "run routine `fmea_from_functional_chain` with `functional_chain_name = Steering Control`":

```
1. KP reads the routine_def artifact from the knowledge repo
2. KP resolves all variables with provided values and defaults
3. KP runs pre-flight checks — aborts with clear error if any fail
4. KP reads all declared input artifacts (prior FMEAs, work log)
5. KP clones declared model resource and generates fabric for the chain
6. KP renders the prompt_template with all resolved variables and fabric
7. KP executes the reasoning cycle (this conversation)
8. KP writes all declared output artifacts to the knowledge repo
9. KP runs post-execution validation — flags missing outputs
10. KP reports summary to engineer
```

---

## Why This Is Better Than a Jupyter Notebook

A Jupyter notebook would contain Python cells calling the capella-fabric API, parsing YAML, building a FMEA table, and writing to a file. It requires:
- A Python environment with correct dependencies
- API credentials in the environment
- Code maintenance as the model API evolves
- A developer to author and modify it

The `routine_def` requires:
- A YAML file in the knowledge repo
- Variable values from the engineer
- The KP to execute it

The `routine_def` is authored in the same language the engineer already uses to work with the KP — natural language + structured YAML. It's readable, versionable, and maintainable without coding skill. And because the prompt template is what drives execution rather than code, it adapts to model changes automatically — the KP generates fresh fabric every time rather than parsing a fixed schema.

---

## For Cousin Implementation

### New artifact type to add to `artifact-repo` MCP
```
type: routine_def
```
Stored and versioned like any other artifact. Readable by `read_artifact`. Searchable by `search_artifacts`. No special execution support needed in the MCP — execution is handled by the KP at conversation time.

### New KP behavior to implement
When the engineer says "run routine X with variable Y = Z":
1. KP calls `read_artifact` to fetch the `routine_def`
2. KP parses the YAML schema
3. KP resolves variables, runs pre-flight, fetches inputs, clones resources
4. KP renders prompt_template via `render_prompt` (already supported)
5. KP executes analysis and writes outputs

### Optional: `validate_routine_def` tool
A tool that reads a `routine_def` and validates the YAML schema, checks resource accessibility, and confirms input artifacts exist — without executing. Useful for testing new routines before running them.

### Optional: `list_routines` tool
Returns all `routine_def` artifacts in a package. Makes the routine library discoverable.

---

## Naming Convention

```
routines/<category>/<routine_id>_v<N>.yaml
```

Example:
```
routines/analysis/fmea_from_functional_chain_v1.yaml
routines/modeling/name_functional_exchanges_v1.yaml
routines/review/exchange_completeness_check_v1.yaml
routines/requirements/requirements_from_capability_v1.yaml
```

---

## Connection to the KP Knowledge Engine

In the engine analogy:
- `routine_def` = the pre-configured ignition sequence
- Variables = the throttle setting
- Input artifacts = the fuel mixture (knowledge already curated)
- Resources (model) = the compression chamber (current state)
- Prompt template = the spark plug timing
- Output artifacts = the power stroke output
- Post-execution validation = the exhaust check

A routine is not code. It is a structured description of how to fire the engine reliably for a specific task.
