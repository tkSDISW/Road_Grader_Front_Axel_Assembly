# prompt_def: fmea_from_functional_chain
# version: 1.0
# description: Generates a structured FMEA report artifact from a named functional chain
# variables:
#   - functional_chain_name: Name of the functional chain to analyze (e.g. "Steering Control")
#   - model_repo_url: GitHub URL of the Capella model repo
#   - github_pat: PAT for model repo access
#   - artifact_package: Target artifact package name for the FMEA output
# usage: render_prompt with variables, then execute the rendered prompt as a KP session

---

## KP Task: Generate FMEA from Functional Chain

You are the SE Knowledge Partner operating in engineering analysis mode.

### Step 1 — Clone and resolve the functional chain

Clone the model at `{{ model_repo_url }}` on branch `master` using the provided PAT.

Search for a functional chain named `{{ functional_chain_name }}` across all phases (SA, LA, PA). Resolve its UUID and generate fabric scoped to that chain.

### Step 2 — Analyze the chain for FMEA inputs

From the fabric, extract for each functional exchange in the chain:

- **Exchange name** — the data/signal flow identifier
- **Source function** — the function producing the output
- **Target function** — the function consuming the input
- **Allocated component** — the physical or logical component hosting the source function
- **Exchange type** — hydraulic, mechanical, electrical, control signal, etc. (infer from name and context)

### Step 3 — Generate FMEA table

For each functional exchange, produce one or more FMEA rows using this structure:

| Item | Function | Failure Mode | Failure Effect (Local) | Failure Effect (End) | Severity | Occurrence | Detection | RPN | Recommended Action |
|---|---|---|---|---|---|---|---|---|---|

**Guidance for each column:**

- **Item** — the component hosting the source function
- **Function** — the source function name
- **Failure Mode** — how the exchange could fail (loss of signal, degraded signal, spurious signal, physical fracture, seizure, leakage, etc.)
- **Failure Effect (Local)** — what the target function loses or receives incorrectly
- **Failure Effect (End)** — what the terminal output of the chain loses (trace to the last function in the chain)
- **Severity (S)** — 1–10 scale: 1=no effect, 10=safety critical without warning
- **Occurrence (O)** — 1–10 scale: 1=unlikely, 10=almost certain
- **Detection (D)** — 1–10 scale: 1=certain detection, 10=undetectable
- **RPN** — S × O × D
- **Recommended Action** — design change, redundancy, inspection interval, or requirement to add

**Severity guidance for steering/camber chains:**
- Loss of steering function = S 9–10
- Degraded steering accuracy = S 6–8
- Loss of camber control = S 5–7
- Maintenance access loss = S 2–4

### Step 4 — Save as artifact

Write the completed FMEA table as a structured `table` artifact to the knowledge repo on branch `main` under package `{{ artifact_package }}` with:

- **name:** `FMEA — {{ functional_chain_name }}`
- **tags:** `["fmea", "functional-chain", "{{ functional_chain_name | lower | replace(' ', '-') }}"]`
- **lineage:** UUID of the functional chain artifact

### Step 5 — Log the work

Add a milestone log entry to the work log recording:
- Chain analyzed
- Number of exchanges processed
- Number of FMEA rows generated
- Artifact ID of the saved FMEA
- Any exchanges that could not be analyzed (unnamed, no allocated component, etc.)

### Step 6 — Summary

Report back to the engineer with:
- Total FMEA rows generated
- Top 3 highest RPN items (most critical failure modes)
- Any gaps found (exchanges with no allocated component, functions with no property values that would inform severity)
- Recommended next steps (requirements to add, property values to populate, design changes to consider)
