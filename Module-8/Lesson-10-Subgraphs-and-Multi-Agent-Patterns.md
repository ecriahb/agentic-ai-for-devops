# 🚩 Jai Bajrangbali!

# Lesson 10 — Subgraphs & Multi-Agent Patterns

> **Multiple agents banana goal nahi hai. Goal hai complex responsibility ko bounded, testable components me divide karna.**

---

# 🎯 Lesson Goal

Is lesson ka purpose **Module 8 ke stateful graph ko decompose karna** hai. Deep multi-agent coordination patterns Module 9 me canonical honge.

Aap samjhoge:

- subgraph kya hota hai
- specialist workflow kya hota hai
- supervisor pattern ka mental model
- handoff contract kaise define hota hai
- multi-agent architecture kab justified ho sakta hai
- Module 9 ka scope yahan exactly kahan se start hoga

---

# PART 1 — English Definitions

A **subgraph** is a graph or workflow component embedded inside a larger graph to encapsulate a bounded responsibility.

A **multi-agent system** coordinates multiple specialized decision-making components. In this course, the detailed coordination patterns are deferred to **Module 9**.

---

# PART 2 — Why Subgraphs First?

Before saying:

```text
network agent
terraform agent
pipeline agent
security agent
```

ask:

```text
Can these simply be bounded subgraphs/functions?
```

Often yes.

Subgraph advantages:

```text
clear input/output contract
isolated tests
bounded tools
separate retry policy
reusable workflow
less coordination ambiguity
```

The first design preference is:

```text
Function/Node
   ↓ if complexity grows
Subgraph
   ↓ if independent decision-making is truly justified
Multi-Agent
```

---

# PART 3 — DevOps Subgraph Design

```text
Main Incident Graph
      ↓
Classify Domain
 ┌────┼──────────────┐
 ↓    ↓              ↓
Pipeline Subgraph  Terraform Subgraph  AKS Subgraph
 ↓    ↓              ↓
Normalized Evidence Results
          ↓
      Main Graph
```

Each subgraph returns a contract, not free-form authority.

---

# PART 4 — Specialist Contract

Example input:

```python
{
  "incident_id": "INC-1042",
  "environment": "production",
  "question": "Inspect Terraform networking changes"
}
```

Output:

```python
{
  "status": "SUCCESS",
  "evidence_ids": ["E2", "E3"],
  "gaps": [],
  "hypotheses": ["network change may be relevant"]
}
```

Main graph should consume structured fields, not trust an internal specialist chat transcript as evidence.

---

# PART 5 — Supervisor Pattern: Introduction Only

At this stage, think of a supervisor simply as a coordinator:

```text
Supervisor
  ↓
Inspect State / Goal
  ↓
Choose Specialist
  ├─ Pipeline
  ├─ Terraform
  └─ AKS
  ↓
Collect Specialist Result
  ↓
Continue or Finish
```

The supervisor can be deterministic, model-assisted or hybrid.

**Do not go deep into delegation strategies, debate, handoff protocols, shared-memory designs or agent communication policies here. Those belong to Module 9.**

---

# PART 6 — Bounded Toolsets

Pipeline specialist:

```text
get_pipeline_status
read_pipeline_log
```

Terraform specialist:

```text
get_terraform_changes
read_plan_summary
```

AKS specialist:

```text
get_aks_status
get_k8s_events
```

Least privilege improves reasoning and security.

---

# PART 7 — MCP Connection

Each specialist can consume different MCP servers:

```text
Pipeline Subgraph → CI/CD MCP
Terraform Subgraph → IaC MCP
AKS Subgraph → Kubernetes/Azure MCP
```

Main graph does not need every server-specific implementation detail.

Still:

```text
MCP discovery != authorization
specialist proposal != execution authority
```

---

# PART 8 — Handoff Contract

Bad handoff:

```text
Agent A: "Network is definitely root cause"
Agent B accepts as fact
```

Better:

```text
Subgraph returns evidence IDs + status + hypothesis separately
```

Example:

```python
{
  "evidence_ids": ["E2", "E3"],
  "hypotheses": ["network change may be causal"],
  "confidence": "medium"
}
```

Downstream validates evidence itself.

The detailed **agent-to-agent communication patterns** will be taught in Module 9.

---

# PART 9 — Shared State vs Private State

Some state may be shared:

```text
incident_id
environment
approved evidence IDs
global loop budget
```

Specialist-private state:

```text
local retry counters
local intermediate plan
specialist-specific messages
```

Avoid giant global state where every component can overwrite everything.

---

# PART 10 — When Multi-Agent Is Actually Justified

Use multi-agent architecture only when responsibilities differ materially, for example:

```text
different security boundaries
different tool domains
different long-running workflows
parallel independent investigations
independent evaluation requirements
```

Not useful merely because an architecture diagram looks advanced.

The decision framework and patterns for **supervisor, router, parallel agents, handoffs, shared state and conflict resolution** are intentionally deferred to **Module 9**.

---

# PART 11 — Parallel Specialist Pattern

A bounded graph may fan out without requiring autonomous agents:

```text
                  ┌→ Pipeline Investigation ─┐
Incident → Fanout ├→ Terraform Investigation ┼→ Evidence Merger
                  └→ AKS Investigation ───────┘
```

Need:

```text
bounded concurrency
partial failure handling
evidence deduplication
consistent source IDs
```

Parallel workflows are not automatically multi-agent systems.

---

# PART 12 — Final Decision Authority

Never let each specialist independently execute remediation.

Safer:

```text
specialists investigate
 ↓
main graph validates evidence
 ↓
main policy proposes action
 ↓
human approval
 ↓
central controlled executor
```

---

# PART 13 — Module Boundary

```text
Module 8
= stateful single-graph orchestration
+ bounded subgraphs
+ introductory supervisor concept

Module 9
= deep multi-agent coordination
+ architecture patterns
+ specialization boundaries
+ handoffs
+ shared state/communication
+ conflict resolution
```

This prevents the same multi-agent theory from being taught twice.

---

# PART 14 — Common Mistakes

- one agent per tool
- free-form handoffs treated as evidence
- no supervisor/graph termination rule
- broad tool permissions to every specialist
- no normalized handoff schema
- duplicated evidence
- no final owner for decision
- teaching Module 9 patterns inside the Module 8 graph lesson

---

# PART 15 — Interview Q&A

### Q1. Subgraph vs multi-agent?
A subgraph is a compositional workflow boundary; multi-agent architecture adds multiple specialized decision-making components. A subgraph can be deterministic and does not need to be an autonomous agent.

### Q2. Why use bounded specialist toolsets?
They reduce security exposure and improve decision relevance.

### Q3. How should a specialist hand off information?
Prefer structured state/evidence contracts with source IDs rather than unverified prose treated as truth.

### Q4. Is parallel fan-out automatically multi-agent?
No. Parallel deterministic subgraphs can execute concurrently without introducing multiple autonomous decision-makers.

### Q5. Where are advanced multi-agent patterns taught?
Module 9 is the canonical deep-dive module for multi-agent coordination.

---

# PART 16 — Revision

```text
Subgraph = bounded reusable workflow
Specialist = domain-limited component
Supervisor = introductory coordinator concept
Handoff = validated structured contract
Shared evidence = source-backed truth layer

Deep multi-agent coordination → Module 9
```

---

# PART 17 — Homework

Design three bounded specialists for:

```text
CI/CD
Terraform
AKS
```

For each list:

```text
allowed tools
private state
shared state
output contract
termination condition
```

Then decide whether each specialist really needs an autonomous agent or can remain a deterministic subgraph.

---

# 🔁 Next Lesson Kyu?

Complex graph bana sakte hain, but production me usko **observe, test, evaluate and secure** bhi karna hoga. Next lesson production readiness par hai.
