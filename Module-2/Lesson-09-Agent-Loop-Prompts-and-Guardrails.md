# 🚩 Jai Bajrangbali!

# Lesson 09 — Agent Loop Prompts & Guardrails

> **Agent ko “keep trying until fixed” bolna unsafe hai. Tool choice, stop conditions, budgets aur execution authority host-controlled hone chahiye.**

> **Canonical scope:** Module 2 owns the **prompt/guardrail design** for an agent loop. Module 1 Lesson 9 owns the basic agent implementation and core loop mechanics; Modules 8–9 own production stateful and multi-agent orchestration. This lesson must not become a second agent-framework course.

## Where This Lesson Fits

```text
Module 1 Lesson 8
→ tool calling
        ↓
Module 1 Lesson 9
→ basic agent loop implementation
        ↓
Module 2 Lesson 9
→ prompt design + guardrails for that loop
        ↓
Module 8
→ stateful graph orchestration
        ↓
Module 9
→ multi-agent coordination
```

---

# 🎯 Lesson Goal

Aap samjhoge:

- agent loop kya hai
- model planner aur host executor separation
- tool request vs tool execution
- loop prompt me evidence rules
- max iterations/no-progress stop
- read-only first
- tool allowlist/argument validation
- human approval for writes
- prompt guardrail vs deterministic guardrail

---

# 1. English Definition

**An agent loop is an iterative application workflow in which a model or planner observes current state, proposes a next action, receives a result, and repeats until a defined stopping condition is reached.**

Mental model:

```text
Observe State
    ↓
Model Proposes Next Step
    ↓
Host Validates
    ↓
Host Executes Allowed Tool
    ↓
Evidence Added
    ↓
Repeat or Stop
```

---

# 2. Prompt's Role in Agent Loop

Agent prompt can say:

```text
- prefer read-only evidence collection
- request only one tool at a time
- do not invent tool output
- stop when enough evidence exists
- if no progress, report the gap
```

This guides planning.

But host must enforce:

```text
allowed tool names
argument schema
target scope
RBAC
iteration budget
approval
```

The implementation details of this loop were already introduced in **Module 1 Lesson 9**. Here we focus on the **instruction and control policy** that should sit around it.

---

# 3. Model = Requester, Not Executor

Model output:

```json
{
  "tool": "get_aks_status",
  "arguments": {"cluster_name":"prod-aks"}
}
```

means:

```text
Please call this tool
```

It does not mean:

```text
Tool was called
Result is known
Permission is granted
```

Host validates and executes.

---

# 4. Unsafe Loop Prompt

```text
You are autonomous. Use any command needed until production is fixed.
```

Problems:

- unlimited scope
- no tool boundary
- no target validation
- no iteration limit
- no approval
- outcome pressure can encourage unsafe writes

---

# 5. Safer Investigation Prompt

```text
You are a read-only incident planner.
Your goal is to identify evidence gaps.
You may request only approved read-only capabilities exposed by the host.
Do not invent tool names, arguments or results.
Request the minimum evidence needed.
Stop if evidence is sufficient, no approved tool can close the gap,
or the host signals a budget limit.
```

Now autonomy is bounded to investigation.

---

# 6. Tool Allowlist

Host:

```python
ALLOWED_TOOLS = {
    "get_pipeline_status",
    "get_terraform_changes",
    "get_aks_status",
}
```

Model requests:

```text
restart_prod_cluster
```

Host response:

```text
POLICY_BLOCKED
```

The model cannot create capabilities by naming them.

---

# 7. Argument Validation

Even valid tool name can have unsafe/wrong arguments.

Model:

```json
{"tool":"get_aks_status", "arguments":{"cluster_name":"customer-secret-cluster"}}
```

Host must validate:

```text
schema
type
allowed cluster
tenant/environment scope
caller authorization
```

Lesson from Module 1:

```text
Tool name validation is not enough.
```

---

# 8. Stop Conditions

Agent must terminate intentionally.

Useful states:

```text
ENOUGH_EVIDENCE
INSUFFICIENT_EVIDENCE
NO_PROGRESS
MAX_ITERATIONS
CAPABILITY_MISSING
POLICY_BLOCKED
TOOL_FAILED
```

Bad design:

```text
while True:
    ask model what next
```

Bound loops.

---

# 9. No-Progress Detection

Agent repeatedly asks:

```text
get_aks_status
get_aks_status
get_aks_status
```

same result each time.

Application can track:

```text
tool + arguments + evidence hash
```

and stop duplicate cycles.

Prompt can discourage repeats, but host should enforce budget/no-progress policy.

---

# 10. Evidence State

After each tool call, store evidence outside model memory:

```python
{
  "id": "E3",
  "kind": "CURRENT_EVIDENCE",
  "operation": "get_aks_status",
  "arguments": {"cluster_name":"prod-aks"},
  "payload": {...}
}
```

Next model step receives relevant state.

Do not rely on:

```text
"Earlier I think the cluster was degraded"
```

Conversation wording is not authoritative state.

---

# 11. Write Actions

If agent later proposes:

```text
restore NSG rule
```

safe path:

```text
Proposal
→ deterministic policy
→ authorization
→ human approval
→ isolated executor
→ post-action evidence
```

Prompt saying “ask approval” is not sufficient if write tool is already directly callable.

---

# 12. Provider Independence

Planner can be:

```text
Ollama
or
OpenAI
```

Host loop remains:

```text
validate
execute
persist evidence
budget
stop
```

If provider produces different tool suggestions, policy must still behave consistently.

This is why provider changes require trajectory evals later.

---

# 13. Common Mistakes

1. Giving shell access directly to model.
2. Unlimited loop.
3. No duplicate/no-progress detection.
4. Tool call treated as result.
5. Tool output not preserved.
6. Conversation memory used as state.
7. Model decides its own authorization.
8. Write tool available during read-only investigation.
9. Retry of non-idempotent write without safeguards.

---

# 14. Production Agent Loop

```text
Input Validation
  ↓
State
  ↓
Planner LLM
  ↓
Tool Proposal
  ↓
Policy + Authorization
  ↓
Executor
  ↓
Evidence Envelope
  ↓
State Update
  ↓
Stop / Loop
```

Later LangGraph makes this state/routing explicit.

For full implementation mechanics see **Module 1 Lesson 9**. For graph-based persistence/routing, continue to **Module 8**.

---

# 15. Interview Q&A

### Q1. What is an agent loop?
An iterative observe-plan-act-observe workflow with explicit state and stopping conditions.

### Q2. Why must host execute tools?
Because model output is untrusted and should not directly own permissions/side effects.

### Q3. What are common stop conditions?
Enough evidence, no progress, budget exhausted, missing capability, policy block or tool failure.

### Q4. Why read-only first?
It reduces blast radius while the system proves investigation reliability.

### Q5. What is excessive agency?
Giving an agent more capabilities, permissions or autonomy than required for its task.

---

# 16. Quick Revision

```text
LLM proposes
Host validates
Tool executes
Evidence returns
State updates
Loop is bounded
```

---

# 🧪 Homework

Design an agent loop for a failed deployment with exactly three allowed tools.

Write:

- allowed tool list
- argument rules
- max iterations
- no-progress rule
- success state
- insufficient-evidence state
- what happens if model requests a write

---

# ➡️ Why Next?

Prompt/agent design banana enough nahi. Hume prove karna hai ki behavior multiple test cases par reliable hai. Next: **Prompt Evaluation**.
