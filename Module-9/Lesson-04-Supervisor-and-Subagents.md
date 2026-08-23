# 🚩 Jai Bajrangbali!

# Lesson 04 — Supervisor & Subagent Pattern

> **Supervisor ka kaam sab kuch khud solve karna nahi; right specialist ko right context ke saath invoke karna aur results ko coordinate karna hai.**

---

## 🔗 Where This Lesson Fits

```text
L01 → Why multi-agent?
L02 → Pattern overview
L03 → Specialist boundaries/contracts
L04 → Supervisor + subagents  ← CANONICAL
L05 → Router + parallel specialists
```

**Scope boundary:** L02 patterns ka comparison deta hai; L04 supervisor delegation ko deeply implement karta hai. L08 later result contracts ko own karta hai. Module 8 ka supervisor/subgraph material yahan multi-agent coordination ke context me specialized hai; state/checkpoint fundamentals wapas nahi padhaye ja rahe.

---

# 🎯 Lesson Goal

Aap samjhoge:

- supervisor pattern ka mental model
- subagents as tools/nodes
- centralized coordination
- delegation contract
- sequential vs parallel delegation
- supervisor state
- supervisor failure modes
- DevOps implementation pattern

---

# PART 1 — Architecture

```text
User / Incident
      ↓
Supervisor
  ├─ Pipeline Specialist
  ├─ Terraform Specialist
  ├─ AKS Specialist
  └─ Knowledge Specialist
      ↓
Supervisor / Synthesis
```

Subagent usually user se direct baat nahi karta; result supervisor ko return karta hai.

---

# PART 2 — Why Supervisor?

Useful when:
- task spans multiple domains
- repeated delegation needed
- one coordinator should own overall workflow
- subagents are focused
- centralized incident state desired

Not useful when:

```text
one deterministic route is enough
```

then router is simpler.

---

# PART 3 — Delegation Contract

Supervisor should not send vague:

```text
"Investigate everything"
```

Better:

```python
{
  "task": "Identify Terraform networking changes around failure window",
  "environment": "production",
  "incident_id": "INC-1042",
  "known_facts": ["Pipeline failed during Terraform Apply [E1]"]
}
```

Focused task produces focused result.

---

# PART 4 — Subagent as Tool Mental Model

```text
Supervisor decision
     ↓
call_terraform_specialist(task)
     ↓
subagent graph/agent
     ↓
normalized result
     ↓
Supervisor state
```

This is similar to Module 1 tool calling, but the capability itself now contains reasoning/workflow.

Still:

```text
subagent invocation request = untrusted proposal
host policy decides what runs
```

---

# PART 5 — Stateless Subagents by Default

For many cases:

```text
Invocation 1 starts fresh
Invocation 2 starts fresh
```

Benefits:
- context isolation
- lower accidental memory contamination
- easier tests
- predictable scope

Supervisor keeps durable incident state.

If a specialist truly needs multi-turn state, use explicit bounded state—not accidental chat history.

---

# PART 6 — Sequential Delegation

```text
Supervisor
 ↓
Pipeline Agent → failure during terraform_apply
 ↓
Supervisor
 ↓
Terraform Agent → finds NSG deletion
 ↓
Supervisor
 ↓
AKS Agent → validates degraded connectivity
```

This is evidence-driven multi-hop coordination.

---

# PART 7 — Parallel Delegation

If domains are independent:

```text
Pipeline Agent ─┐
Terraform Agent ├─→ Supervisor
AKS Agent ──────┘
```

Parallel benefits:
- lower wall-clock latency

But requires:
- deterministic result merge
- stable evidence IDs
- independent tool scopes
- branch-level failure handling

The detailed router/fan-out mechanics belong to **L05**; here we focus on the supervisor's coordination decision.

---

# PART 8 — Supervisor State

Example:

```python
{
  "incident_id": "INC-1042",
  "completed_agents": ["pipeline", "terraform"],
  "findings": [...],
  "evidence_ids": ["E1", "E2"],
  "next_agent": "aks",
  "iteration": 3
}
```

Do not store hidden chain-of-thought.

Store workflow decisions/results required to coordinate the team.

---

# PART 9 — Supervisor Policy Boundary

Supervisor may decide:

```text
which approved specialist to call
what focused task to delegate
whether known evidence gaps remain
whether synthesis can start
```

Supervisor must NOT own:

```text
user authorization
backend RBAC
production write approval
source authenticity as a reasoning claim
```

Those stay in trusted application/policy layers.

---

# PART 10 — Failure Modes

## Infinite delegation

```text
Supervisor → A → Supervisor → A → ...
```

Guard:

```text
max iterations
max delegations
duplicate-task detection
```

## Delegation ambiguity

Two specialists both investigate the same domain.

Guard:

```text
responsibility map from L03
```

## Result trust

Subagent says:

```text
"Root cause definitely NSG"
```

Supervisor must inspect structured findings/evidence IDs, not confidence language alone.

The standardized result contract is covered in **L08**.

---

# PART 11 — Practical Pseudocode

```python
def supervisor(state):
    if "E1" not in state["evidence_ids"]:
        return {"next_agent": "pipeline"}
    if "E2" not in state["evidence_ids"]:
        return {"next_agent": "terraform"}
    if "E3" not in state["evidence_ids"]:
        return {"next_agent": "aks"}
    return {"next_agent": "synthesize"}
```

Deterministic supervisor is a great learning baseline before LLM-driven delegation.

---

# PART 12 — DevOps Example

```text
Incident
 ↓
Supervisor checks current evidence
 ↓
E1 missing → Pipeline specialist
 ↓
E1 found, E2 missing → Terraform specialist
 ↓
E2 found, E3 missing → AKS specialist
 ↓
E1/E2/E3 present → Synthesis
```

The supervisor controls **who runs next**; specialist logic remains inside specialist boundaries.

---

# PART 13 — Interview Q&A

### Q1. What does a supervisor do?
Coordinates specialist agents, controls delegation/context, tracks progress and decides which approved specialist should run next.

### Q2. Why keep subagents stateless by default?
To isolate context and reduce memory contamination when each invocation is independent.

### Q3. Supervisor vs router?
Supervisor can coordinate repeatedly across steps; router usually performs bounded dispatch. Detailed router/parallel behavior is L05.

### Q4. How do you prevent supervisor loops?
Application-controlled iteration/delegation limits, duplicate-task detection and explicit terminal states.

---

# PART 14 — Revision

```text
Supervisor = coordinator
Subagent = focused specialist
Delegation = explicit task contract
Result = structured finding
Policy = outside model reasoning
```

---

# PART 15 — Homework

Design a supervisor decision table for E1/E2/E3 evidence collection. Add conditions for:
- tool timeout
- specialist failure
- duplicate finding
- enough evidence
- max delegations reached

---

# 🔁 Next Lesson Kyu?

Supervisor handles repeated delegation. Next we’ll study **Router + Parallel Specialists**, where independent work can be dispatched and merged deterministically.
