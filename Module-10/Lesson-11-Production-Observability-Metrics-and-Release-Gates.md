# 🚩 Jai Bajrangbali!

# Lesson 11 — Production Observability, Metrics & Release Gates

> **Security controls become operational only when their decisions are observable, measurable and able to block unsafe promotion.**

---

# 🎯 Lesson Goal

You will understand:

- production telemetry for agent/security controls
- reliability vs quality vs security/trust metrics
- zero-tolerance invariants
- release scorecards
- CI/CD and canary gates
- model/prompt/tool/RAG/policy drift
- incident triggers
- kill switches and rollback
- cost and unbounded-consumption controls

> **Ownership boundary:** Lesson 09 defines agent-evaluation methodology and datasets. Lesson 10 defines adversarial/red-team case design. This lesson owns the **operationalization** of those results into dashboards, alerts, release decisions and rollback controls.

---

# PART 1 — Three Operational Metric Families

```text
Reliability
Quality
Security / Trust
```

Reliability:

```text
latency
availability
queue age
errors
```

Quality:

```text
groundedness
retrieval performance
routing quality
abstention correctness
```

Security/Trust:

```text
policy violations
secret leaks
unauthorized retrieval
approval bypass
unknown tools
```

Do not collapse these into one score.

---

# PART 2 — Security Invariants

Some controls should be absolute:

```text
prod write without authorization = 0
prod write without approval = 0
cross-tenant retrieval = 0
secret exposure = 0
unknown tool execution = 0
```

A single critical violation can block/rollback a release even when average quality is high.

---

# PART 3 — Runtime Behavioral Metrics

Track behavior that can change between versions:

```text
average tools/run
unexpected tool rate
loop-limit rate
no-progress rate
handoffs/run
conflict rate
abstention rate
validation failure rate
policy-denial rate
```

Compare candidate vs known-good baseline rather than using isolated raw counts.

---

# PART 4 — Security Signal Metrics

Examples:

```text
prompt-injection detection signals
policy-denied actions after untrusted content
unknown tool proposals
external-destination proposals
secret-redaction events
RAG ACL denials
MCP trust failures
approval mismatches
```

A detector score is a signal, not proof of an attack.

---

# PART 5 — RAG/MCP/Multi-Agent Operational Metrics

### RAG

```text
stale-source retrieval
ACL denial rate
source-version mismatch
index freshness lag
no-context rate
```

### MCP

```text
unknown server attempts
auth failures
capability drift
malformed responses
rate-limit events
```

### Multi-Agent

```text
agent routing drift
handoff loops
private→shared promotions
conflict rate
specialist failure rate
```

These metrics should point operators toward the appropriate evaluation or red-team suite for deeper diagnosis.

---

# PART 6 — Cost / Unbounded Consumption

Monitor:

```text
tokens/request
model calls/run
tool calls/run
retrieved chunks
workflow duration
parallelism
cost/team
```

Runtime controls:

```text
budgets
rate limits
max iterations
max context
queue/backpressure
```

Security and FinOps reinforce each other because unbounded behavior can become a reliability or denial-of-service problem.

---

# PART 7 — Release Scorecard

Example:

```text
Functional suite        PASS
RAG Hit@3               96%
Citation validity       100%
Trajectory suite        PASS
Critical security suite PASS
Unknown tool execution  0
Secret leak             0
P95 latency             18s
Cost/request            within budget
```

The exact thresholds are product/risk specific.

---

# PART 8 — Critical vs Non-Critical Thresholds

Critical:

```text
security invariant violation
approval bypass
cross-scope data access
secret leakage
unknown capability execution
```

→ release blocked.

Non-critical quality changes may trigger review based on predefined tolerances.

```text
routing accuracy -0.5%
```

might be a warning rather than an automatic block depending on risk class.

---

# PART 9 — CI/CD Gate

```text
PR
 ↓
unit / contract tests
 ↓
offline agent evals
 ↓
security / red-team regression suite
 ↓
release scorecard
 ↓
critical failure?
 ├─ yes → BLOCK
 └─ no  → stage / canary
```

A manual approval must not silently override a failed critical invariant; exceptions need an explicit, auditable process.

---

# PART 10 — Canary Monitoring

A new model/prompt/tool/RAG/policy bundle gets limited traffic.

Watch candidate vs baseline for:

```text
validation failures
unexpected tools
policy denials
latency
cost
RAG source mix
abstention
security signals
```

Canary monitoring is **runtime detection**, not a replacement for offline evaluation.

---

# PART 11 — Configuration / Behavior Drift

Behavior can change because:

```text
model version changed
prompt changed
MCP server changed
RAG corpus/index changed
policy changed
tool schema changed
agent graph changed
```

Every trace/eval result should record relevant configuration versions:

```text
model_version
prompt_version
tool_version
index_version
policy_version
graph_version
```

Without version metadata, regressions are difficult to reproduce.

---

# PART 12 — Kill Switches

Plan how to quickly disable:

```text
write capabilities
specific MCP server
specific model deployment
specific agent
RAG source collection
high-risk workflow
```

A kill switch should operate at the application/infrastructure/policy layer—not by editing a prompt and hoping the model obeys.

---

# PART 13 — Security Incident Triggers

Examples:

```text
secret leak detected
unauthorized data access
unknown tool execution
approval bypass
suspicious MCP traffic
repeated prompt-injection successes
critical evaluator regression
```

Response may include:

```text
disable capability
revoke identity
preserve traces
rollback release
rotate credentials
open security incident
```

---

# PART 14 — Audit Record

For high-risk decisions retain enough metadata to reconstruct what happened:

```text
request/incident ID
agent/model/prompt version
policy version
source IDs
tool calls
authorization result
approval result
final action/status
```

Apply data minimization and retention controls; do not turn audit into another secret store.

---

# PART 15 — Privacy-Aware Telemetry

Use:

```text
redaction
sampling
access-controlled traces
shorter retention for sensitive payloads
IDs/hashes instead of full content
```

Observability must not create a second data-leak channel.

---

# PART 16 — Release Rollback

If canary shows unsafe behavior:

```text
stop candidate traffic
 ↓
rollback code/prompt/model/policy bundle
 ↓
block affected capability
 ↓
restore stable version
 ↓
preserve failing trace/case
 ↓
add regression test
```

The rollback target should be known-good and independently validated.

---

# PART 17 — Common Mistakes

- uptime green = agent considered safe
- only final answer quality monitored
- no security metrics
- one average score hides critical failure
- no configuration version in traces
- no kill switch
- canary observes only HTTP errors
- traces log secrets
- release exception process undefined
- offline eval and runtime monitoring treated as the same thing

---

# PART 18 — Interview Q&A

### Q1. What metrics should be zero-tolerance?
Unauthorized production writes, secret exposure, cross-tenant retrieval and unknown tool execution are common examples.

### Q2. Why track configuration versions?
Agent behavior can change with model, prompt, tool, RAG, graph or policy versions even when surrounding application code is unchanged.

### Q3. What is a release gate?
A rule that blocks promotion when required tests/metrics do not meet defined thresholds.

### Q4. Why need kill switches?
To disable risky capabilities immediately without depending on model behavior or a full redeployment.

---

# 🧠 Revision

```text
Operate Trust =
Observe Controls
+ Measure Runtime Behavior
+ Block Unsafe Releases
+ Detect Drift
+ Roll Back Quickly
```

---

# 📝 Homework

Create a production dashboard with 15 metrics and identify:

```text
5 alerts
5 warning thresholds
5 release-blocking invariants
```

---

# 🔁 Next Lesson Kyu?

All security, evaluation and operational controls are ready. Final lesson combines them into the **Secure DevOps Agent Evaluation Harness**.
