# 🚩 Jai Bajrangbali!

# Lesson 11 — Production Safety, Observability & Evaluation

> **A graph that runs is not necessarily a graph you can trust. Production agents need graph-level policy, traceability, tests, metrics and failure containment.**

---

# 🎯 Lesson Goal

This lesson is the **Module 8 production gate**: how to operate a stateful graph safely and measure graph behavior. Comprehensive agent-security threats and red-team methodology remain canonical in **Module 10**.

You will learn:

- graph/node/edge observability
- state-transition tracing
- routing, tool-use and termination evaluation
- graph-level safety tests and failure injection
- cost/latency/SLO thinking
- controlled rollout and kill switches
- clear boundary to Module 10 security/evaluation depth

---

# PART 1 — Production Trust Stack

```text
Identity / Auth
      ↓
Authorization / Policy
      ↓
Graph State Contract
      ↓
Safe Routing
      ↓
Tool / MCP Guardrails
      ↓
Evidence Grounding
      ↓
Output Validation
      ↓
Human Approval for Writes
      ↓
Observability + Audit
```

These layers come from earlier modules. This lesson focuses on how the **graph coordinates and exposes them**.

---

# PART 2 — What Should Be Observable?

At minimum capture:

```text
request_id
thread_id
incident_id
node entered
node exited
route selected
route reason/tool category
duration
retries
iteration number
tool name + normalized args
retrieved source IDs
evidence IDs
model name
validation result
interrupt/approval state
final status
```

Never log secrets blindly.

---

# PART 3 — State-Transition Tracing

A useful trace is not just an API request duration:

```text
Trace: INC-1042

START
 ↓
validate_input
 ↓
collect_pipeline
 ↓
collect_terraform
 ↓
route=collect_aks
 ↓
evidence_gate
 ↓
analyze
 ↓
validate
 ↓
END
```

For each node record:

```text
input state version
output state version
transition
latency
status
error
```

This answers:

> **Why did the graph take this path?**

---

# PART 4 — Stage-Level Latency

Track:

```text
classification_ms
tool_collection_ms
retrieval_ms
model_ms
validation_ms
approval_wait_ms
checkpoint_ms
```

Then distinguish:

```text
LLM bottleneck
vector retrieval bottleneck
MCP/tool bottleneck
checkpoint storage bottleneck
human wait
```

This is graph-specific operational visibility; Module 6 covers general orchestration observability.

---

# PART 5 — Routing Evaluation

For each fixture define:

```text
input state
expected route
allowed alternatives
expected terminal state
```

Examples:

```text
Terraform Apply failure → Terraform branch
ImagePullBackOff → registry/image path
No relevant evidence → abstain/insufficient evidence
Max iterations → MAX_ITERATIONS_REACHED
```

Measure:

```text
route accuracy
wrong-branch rate
fallback correctness
unnecessary branch changes
```

---

# PART 6 — Tool-Selection Evaluation

For each incident fixture:

```text
expected useful tools
forbidden/unnecessary tools
expected args
```

Measure:

```text
correct tool selected?
arguments valid?
duplicate call?
unsafe tool proposed?
```

This evaluates the **graph's tool-use behavior**, not generic LLM quality.

---

# PART 7 — Evidence Evaluation

Check:

```text
Did system collect required evidence?
Did it preserve source IDs?
Did it separate reference vs current evidence?
Did retries duplicate evidence?
Did stale evidence get refreshed?
```

Possible metrics:

```text
evidence completeness
evidence precision
freshness compliance
duplicate evidence rate
```

---

# PART 8 — Termination Evaluation

Test:

```text
success case
no useful tools
all tools fail
max iterations
no progress
human reject
policy block
validation fail
```

Critical metric:

```text
runaway loop rate = 0 on accepted test suite
```

Termination is a first-class graph behavior.

---

# PART 9 — Graph-Level Failure Injection

Deliberately break:

```text
MCP timeout
vector store empty
checkpoint store unavailable
malformed tool result
rate limit
partial parallel failure
node exception
routing function returns unknown category
```

Document expected state/status for each.

Do not confuse this with the full adversarial security/red-team program in Module 10.

---

# PART 10 — Security Boundary for This Module

Module 8 verifies that graph control flow respects existing policy:

```text
write node cannot bypass approval
router cannot invent arbitrary executable node
model cannot change max_iterations
checkpoint cannot become authorization source
resource/tool results remain untrusted data
```

The deeper threat catalog—prompt injection variants, tool abuse, data exfiltration, MCP attacks, red teaming and security evaluation—is owned by **Module 10**.

---

# PART 11 — Cost Controls

Agent loops multiply cost.

Track:

```text
LLM calls per incident
tokens per node
tool calls per incident
retrieval calls
retry count
average iterations
checkpoint writes
```

Use deterministic code for simple decisions instead of unnecessary model calls.

---

# PART 12 — SLO Thinking

Possible graph SLOs:

```text
95% investigations finish < 60 sec (excluding human wait)
99% no unauthorized write transition reaches executor
100% current-fact claims retain evidence IDs
< 1% max-iteration termination on known test set
checkpoint/recovery success rate target
```

SLOs make graph reliability measurable.

---

# PART 13 — Kill Switch and Feature Flags

Production graph should support:

```text
disable all writes
disable one MCP server
force read-only mode
cap max iterations
switch model
turn off model-assisted routing
force deterministic workflow
```

Emergency controls remain outside model reasoning.

---

# PART 14 — Deployment Strategy

Safer rollout:

```text
Offline graph evaluation
 ↓
Shadow mode
 ↓
Read-only internal users
 ↓
Limited production investigations
 ↓
Human-approved actions
 ↓
Highly controlled automation only if justified
```

---

# PART 15 — Audit Trail

For each graph execution record:

```text
which nodes ran
which routes were selected
which evidence was collected
which model/version ran
which graph version ran
which policy version applied
which human decision occurred
which final terminal state was reached
```

Reproducibility requires versioning.

---

# PART 16 — Common Mistakes

- only final answer logged
- no graph/node trace
- evaluation only on happy paths
- model judge used as sole evaluator
- no termination tests
- no kill switch
- cost invisible
- write automation before read-only reliability proven
- reproducing Module 10's full security catalog here

---

# PART 17 — Interview Q&A

### Q1. What should you evaluate in a stateful agent besides final answer quality?
Routing, tool selection, arguments, evidence handling, termination, recovery, latency, cost and safety-gate behavior.

### Q2. Why is stage-level observability important?
It shows which node, route or dependency caused failure/latency instead of hiding everything inside one request.

### Q3. Why use deterministic evaluators where possible?
Properties such as route allowlists, citation IDs, loop counts, state schemas and terminal statuses can be checked exactly.

### Q4. Where is the deeper security evaluation taught?
Module 10 is the canonical security/evaluation deep dive; Module 8 only verifies that graph control flow respects those boundaries.

---

# PART 18 — Revision

```text
Trace graph behavior
Evaluate routing/tool use/termination
Test failure and recovery
Measure cost/latency
Keep policy external
Roll out gradually

Deep agent security → Module 10
```

---

# PART 19 — Homework

Create a 15-case graph evaluation sheet with columns:

```text
incident
expected route
expected tools
forbidden tools
expected evidence
should abstain?
max iterations
termination state
recovery behavior
final grounded?
safety result
```

Then mark which security cases should later be expanded in Module 10.

---

# 🔁 Next Lesson Kyu?

Ab graph ke individual concepts, production controls aur evaluation boundaries clear hain. Final lesson me **Module 1–8 ko ek Stateful DevOps Incident Response Agent** me combine karenge.
