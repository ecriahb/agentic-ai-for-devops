# 🚩 Jai Bajrangbali!

# Lesson 07 — Shared State, Private State & Context Engineering

> **Multi-agent system me sab agents ko sab kuch dikhana transparency nahi, context leakage hai. Shared state minimal, typed aur source-aware hona chahiye.**

---

## 🔗 Where This Lesson Fits

```text
L06 → Handoff transfers active control
L07 → Shared/private state + per-agent context  ← CANONICAL
L08 → Result/evidence communication contracts
L09 → Conflict resolution + synthesis
```

**Scope boundary:** Module 2 owns generic context engineering. Module 8 owns single-graph state schemas/reducers. This lesson specializes those ideas for **cross-agent visibility, context projection, ownership and leakage control**.

---

# 🎯 Lesson Goal

Aap samjhoge:
- shared vs private state
- working context vs durable evidence
- context isolation
- state ownership
- reducers and merge behavior
- sensitive-data boundaries
- context bloat prevention

---

# PART 1 — Core Mental Model

```text
SHARED STATE
= facts needed across agents

PRIVATE STATE
= specialist-local execution data

EVIDENCE STORE
= source-backed observations

CONVERSATION CONTEXT
= user interaction continuity

AUTHORIZATION STATE
= trusted policy/identity data
```

These are different trust classes.

---

# PART 2 — What Belongs in Shared State?

Good shared fields:

```python
{
  "incident_id": "INC-1042",
  "environment": "production",
  "evidence": [...],
  "agent_status": {...},
  "open_questions": [...],
  "final_status": "INVESTIGATING"
}
```

Bad shared field:

```text
"all_agent_internal_reasoning": [...]
```

Shared state should contain only what another component legitimately needs.

---

# PART 3 — Private Specialist State

Terraform agent may need:

```text
candidate modules
plan sections inspected
query attempts
local retry counters
intermediate hypotheses
```

These do not need to pollute global state.

Only normalized outputs should cross the boundary.

---

# PART 4 — Evidence is Special

Evidence should be append-only or explicitly versioned:

```python
{
  "id": "E2",
  "source": "terraform_plan",
  "operation": "inspect_network_changes",
  "payload": {...},
  "timestamp": "...",
  "agent": "terraform_specialist"
}
```

Agent commentary is not evidence.

---

# PART 5 — Reducer / Merge Semantics

Parallel agents may write to shared channels simultaneously.

Example:

```text
Pipeline Agent → [E1]
Terraform Agent → [E2]
AKS Agent → [E3]
```

Shared reducer must merge without overwriting.

But naive append can duplicate evidence on retry.

Production reducer may dedupe by:

```text
evidence_id
source + operation + arguments hash
source event ID
```

The detailed reducer mechanics are taught in Module 8; here the focus is **what should be shared and why**.

---

# PART 6 — Context Projection per Agent

Do not treat shared state as the prompt.

Terraform prompt receives:

```text
incident summary
E1 pipeline fact
Terraform task
Terraform tools
```

It does NOT automatically receive:

```text
all AKS logs
all user history
all runbooks
all other agents' model outputs
```

This is the multi-agent version of context engineering: **project only the minimum useful view into each specialist**.

---

# PART 7 — Shared State is Not Prompt

Important:

```text
Graph state != LLM context window
```

Application may store rich structured state, then create a minimal prompt view for each agent.

Example:

```python
def build_terraform_context(state):
    return select_relevant_fields(state)
```

Different agents can therefore see different projections of the same workflow state.

---

# PART 8 — Sensitive Data

Suppose security agent can access secret-scanner findings.

Do not automatically copy those raw payloads to:
- knowledge agent
- pipeline agent
- final user response

Use classification:

```text
PUBLIC
INTERNAL
RESTRICTED
SECRET
```

and enforce context projection.

This is application-level data minimization, not a prompt trick.

---

# PART 9 — State Ownership

Define owner:

```text
Supervisor owns workflow_status.
Evidence collector owns raw evidence append.
Validator owns validation_status.
Human approval node owns approval_decision.
```

Avoid multiple agents casually mutating the same control field.

For every shared field define:

```text
owner
allowed writers
merge rule
trust class
```

---

# PART 10 — Stale State

Operational facts change.

Shared state should include:

```text
observed_at
TTL
freshness_class
```

On resume from checkpoint:

```text
volatile evidence → refresh before final action
```

Checkpointed state is not automatically current truth.

---

# PART 11 — Common Mistakes

- full chat copied to all agents
- model reasoning stored as facts
- no state owner
- parallel writes overwrite each other
- retry duplicates evidence
- secrets spread through shared state
- stale evidence reused after long pause

---

# PART 12 — Interview Q&A

### Q1. Shared vs private state?
Shared state contains cross-agent workflow data; private state contains specialist-local execution/context.

### Q2. Why not share all messages?
It increases token cost, confusion, privacy risk and accidental contamination.

### Q3. Is graph state the same as prompt context?
No. State is application data; each agent should receive a carefully projected context view.

### Q4. How handle parallel state updates?
Use explicit reducer/merge semantics with stable IDs and deduplication; the detailed graph reducer design belongs to Module 8.

---

# PART 13 — Revision

```text
Share facts, not everything.
Private state stays local.
Evidence is source-backed.
State projection builds prompts.
Reducers need dedupe.
Freshness matters.
```

---

# PART 14 — Homework

Create a `MultiAgentIncidentState` with:
- shared fields
- private Terraform fields
- evidence list
- authorization object
- freshness timestamps

Explain which agent may update each field and what context projection each specialist should receive.

---

# 🔁 Next Lesson Kyu?

State boundaries clear hain. Ab agents ke outputs ko **communication/evidence contracts** me standardize karenge so synthesis deterministic ho sake.
