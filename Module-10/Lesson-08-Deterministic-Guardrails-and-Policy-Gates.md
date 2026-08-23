# 🚩 Jai Bajrangbali!

# Lesson 08 — Deterministic Guardrails & Policy Gates

> **Use the LLM for reasoning where uncertainty is acceptable; use deterministic code/policy for boundaries where uncertainty is not acceptable.**

---

# 🎯 Lesson Goal

You will understand:

- deterministic vs probabilistic controls
- the difference between a guardrail and an underlying security control
- input, retrieval, tool, output and action policy gates
- policy decision contracts
- deny/allow/approval-required states
- authorization integration
- evidence gates and budgets
- fail-closed behavior
- policy observability and tests

> **Ownership boundary:** Lessons 02–07 explain the specific attack surfaces. This lesson does not re-teach prompt injection, tool abuse, RAG poisoning, MCP security or multi-agent security in depth. It defines the **common enforcement layer** that applies to all of them.

---

# PART 1 — English Definition

**A deterministic guardrail is application logic that enforces a repeatable rule independently of the LLM's willingness or interpretation.**

A **policy gate** is a decision point that evaluates trusted inputs and returns an explicit enforcement state such as `ALLOW`, `DENY`, `APPROVAL_REQUIRED`, or `INSUFFICIENT_EVIDENCE`.

---

# PART 2 — Model Rule vs Host Rule

Model rule:

```text
Please do not call dangerous tools.
```

Host rule:

```python
if tool_name not in allowed_tools:
    raise PolicyDenied()
```

The second is enforceable and testable.

Golden rule:

```text
Prompt = guidance
Policy code = enforcement
```

---

# PART 3 — Guardrail Stack

```text
Input Gate
 ↓
Identity / Authorization Gate
 ↓
Retrieval Eligibility Gate
 ↓
Tool Selection Gate
 ↓
Argument / Target Gate
 ↓
Execution Risk Gate
 ↓
Output / Citation Gate
 ↓
Approval Gate
 ↓
Post-Action Verification
```

Each layer consumes a different trusted signal. A later layer must not silently assume an earlier layer succeeded.

Domain-specific controls are taught in Lessons 02–07 and are connected here through explicit gates.

---

# PART 4 — Policy Decision Contract

Prefer machine-readable outcomes:

```text
ALLOW
DENY
APPROVAL_REQUIRED
INSUFFICIENT_EVIDENCE
RETRYABLE_FAILURE
POLICY_UNAVAILABLE
```

Avoid only prose:

```text
"This seems mostly safe."
```

Policy decisions should also include a stable reason code, for example:

```json
{
  "decision": "DENY",
  "reason_code": "TOOL_NOT_ALLOWLISTED",
  "policy_version": "p7"
}
```

---

# PART 5 — Input Gate

Validate trusted application inputs before model/tool execution:

```text
request size
supported intent
environment
resource identifiers
content limits
caller identity
```

Example:

```python
if environment not in {"dev", "stage", "production"}:
    return "INVALID_ENVIRONMENT"
```

This does not replace domain-specific input validation elsewhere; it establishes the common policy pattern.

---

# PART 6 — Retrieval Gate

Before context reaches the model:

```text
caller authorized?
source approved?
source current?
classification allowed?
metadata complete?
```

No policy → no document in prompt.

Lesson 05 owns the detailed RAG attack/poisoning mechanics; this lesson owns the **enforcement decision point**.

---

# PART 7 — Evidence Gate

For a workflow that requires current evidence:

```python
required = {"E1", "E2", "E3"}
current = evidence_ids()
if not required.issubset(current):
    return "INSUFFICIENT_EVIDENCE"
```

This prevents the model from filling evidence gaps with guesses.

---

# PART 8 — Tool Gate

```python
ALLOWED_TOOLS_BY_AGENT = {
    "pipeline": {"get_pipeline_status"},
    "terraform": {"get_terraform_changes"},
    "aks": {"get_aks_status"},
}
```

A tool must be allowed for:

```text
caller/agent
task
environment
resource scope
```

Lesson 03 owns the detailed tool-abuse threat model; this lesson turns those requirements into an explicit gate.

---

# PART 9 — Argument Gate

Check:

```text
type
length
format
enum
inventory membership
resource ownership
environment
```

Valid JSON can still be unsafe.

```text
Schema validation != authorization
```

---

# PART 10 — Risk Gate

Example classification:

```text
READ_ONLY          → allow if authorized
LOW_RISK_WRITE     → policy-specific
HIGH_RISK_WRITE    → approval required
DESTRUCTIVE        → deny or special break-glass path
```

The model does not select the authoritative risk class.

---

# PART 11 — Output Gate

Validate:

```text
schema
required sections
citation IDs
source class for current facts
forbidden unsupported claims
secret leakage
unsafe executable content
```

The output gate should reuse the security and evaluation contracts defined earlier rather than inventing a second validation model.

---

# PART 12 — Confidence / Evidence Gate

Host computes decision quality from evidence policy.

Example:

```text
missing evidence → INSUFFICIENT_EVIDENCE
complete but indirect evidence → REVIEW / MEDIUM
multiple authoritative observations align → ALLOW analysis completion
```

The LLM may explain confidence, but it does not override the host rubric.

---

# PART 13 — Approval Gate

```text
proposal
 ↓
policy classifies HIGH_RISK
 ↓
authorization check
 ↓
approval request bound to exact action/target
 ↓
resume
 ↓
revalidate
```

Approval is an input to policy, not a permanent boolean that any future action can reuse.

---

# PART 14 — Loop / Cost Gate

```python
if iterations >= max_iterations:
    return "MAX_ITERATIONS"
if tool_calls >= max_tool_calls:
    return "TOOL_BUDGET_EXCEEDED"
```

Also bound:

```text
tokens
runtime
parallel calls
retrieved context size
```

These limits are enforcement controls, not model suggestions.

---

# PART 15 — Fail Closed

High-risk dependency failure:

```text
policy service unavailable
```

Result:

```text
DENY / POLICY_UNAVAILABLE
```

Never infer permission from an unavailable authorization/policy dependency.

For low-risk read paths, any degraded behavior must be explicitly classified and tested.

---

# PART 16 — Policy as Code

Store versioned rules:

```text
policy_version=p7
```

Test them in CI.

Audit records should include policy version so decisions are reproducible.

---

# PART 17 — Policy Test Examples

```text
P-01 unknown tool → DENY
P-02 read prod status with read role → ALLOW
P-03 Terraform apply from investigator → DENY
P-04 approved exact NSG restore → ALLOW executor
P-05 approval target mismatch → DENY
P-06 missing evidence → INSUFFICIENT_EVIDENCE
P-07 cross-tenant retrieval → DENY
P-08 loop budget exceeded → TERMINATE
P-09 policy dependency unavailable for high-risk action → DENY
```

---

# PART 18 — Observability

Track:

```text
policy decision counts
policy version
reason code
denied operation
agent/caller
environment
approval-required rate
policy service failures
```

Do not log raw secrets.

---

# PART 19 — Vulnerable vs Secure Pattern

Vulnerable:

```python
if llm_says_safe:
    execute()
```

Secure:

```python
proposal = parse()
auth = authorize(identity, proposal)
policy = evaluate(proposal, evidence, env)
if policy == APPROVAL_REQUIRED:
    pause()
```

The executor receives only a policy-approved request.

---

# PART 20 — Common Mistakes

- all controls implemented in prompt text
- prose-only policy decisions
- schema validation treated as authorization
- approval not bound to exact target
- missing policy dependency fails open
- loop/cost has no limits
- model chooses authoritative risk class
- policy changes not versioned/tested
- every security lesson duplicates its own gate instead of using a common enforcement layer

---

# PART 21 — Interview Q&A

### Q1. What should be deterministic in an agent?
Authorization, capability allowlists, argument validation, risk policy, approval requirements, budgets and critical output validation.

### Q2. Why not let the LLM decide policy?
LLM outputs are probabilistic and vulnerable to prompt manipulation; critical boundaries need repeatable enforcement.

### Q3. What is fail-closed behavior?
When a required security decision cannot be safely made, the action is denied rather than allowed by default.

### Q4. How do you make policy auditable?
Version policy, emit structured reason codes and record decisions with caller/action/request metadata.

---

# 🧠 Revision

```text
LLM = Proposal / Reasoning
Policy Engine = Permission / Safety Decision
Guardrail = Enforceable control
```

---

# 📝 Homework

Create 12 policy test cases for the final DevOps AI Assistant and mark which are critical release blockers.

---

# 🔁 Next Lesson Kyu?

We now have explicit enforcement. Next we measure the agent against expected behavior through **evaluation**; security-specific adversarial testing comes immediately after.
