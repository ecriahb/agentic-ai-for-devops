# Module 1 — Lesson 7: Structured Output & Validation

> **Goal:** Free-text LLM answer ko predictable application contract me convert karna, schema validation samajhna, Pydantic ka role practically dekhna, aur sabse important distinction learn karna: **valid structure does not guarantee factual truth.**

---

## Where This Lesson Fits

```text
Lesson 06
Tokens + context capacity
      ↓
Lesson 07
Structured output + validation
      ↓
Lesson 08
Tool calling
```

**This lesson owns the application-level structured-output contract and layered validation foundation. It does not own the complete evaluation/red-team discipline (Module 10) or final RAG evaluation (Module 5).**

---

# 1. English Definitions

**Structured Output:** A model response constrained to a defined machine-readable structure so software can parse and consume it reliably.

**Schema:** A contract that defines expected fields, data types and allowed values.

**Validation:** The process of checking whether data satisfies required structural, business and evidence rules.

Simple Hinglish:

```text
LLM free text
   ↓
Defined schema
   ↓
Validated object
   ↓
Application logic
```

---

# 2. Why This Topic Comes Here

Lesson 6 me humne model ko better context dena seekha.

Ab problem:

```text
Model answer useful ho sakta hai
but har baar format alag ho sakta hai
```

Example:

```text
Run 1: "NSG seems to be the issue."
Run 2: "Root cause appears network-related..."
Run 3: bullets
```

Automation ko predictable fields chahiye.

---

# 3. Free Text vs Structured Output

Free text:

```text
The NSG change likely disrupted connectivity and you should restore the rule.
```

Structured:

```json
{
  "root_cause": "Required NSG rule was removed",
  "impact": "Deployment failed",
  "recommended_fix": "Restore required rule and validate connectivity",
  "confidence": "high"
}
```

Benefits:

- machine readability
- downstream APIs/UI
- validation
- testing
- audit

---

# 4. Pydantic Model

```python
from pydantic import BaseModel
from typing import Literal

class RCA(BaseModel):
    root_cause: str
    impact: str
    recommended_fix: str
    confidence: Literal["low", "medium", "high"]
```

This creates a Python-side data contract.

---

# 5. What Schema Validation Checks

Pydantic can check:

```text
required fields
field types
allowed literal values
basic constraints
```

Example valid shape:

```python
RCA(
    root_cause="NSG rule removed",
    impact="Deployment failed",
    recommended_fix="Restore rule",
    confidence="high",
)
```

Invalid:

```python
confidence="super-certain"
```

because allowed values are limited.

---

# 6. What Schema Validation Does NOT Check

This is the key lesson.

The following can be schema-valid:

```json
{
  "root_cause": "Database outage",
  "impact": "All customers were down for 2 hours",
  "recommended_fix": "Restart production database",
  "confidence": "high"
}
```

But if evidence never mentioned a DB outage, this is factually unsupported.

Therefore:

```text
Schema-valid != Factually true
JSON-valid != Evidence-supported
```

---

# 7. Layered Validation Mental Model

```text
LLM Candidate
     ↓
1. Parse / JSON Validation
     ↓
2. Schema / Type Validation
     ↓
3. Source-ID Validation
     ↓
4. Evidence-Claim Validation
     ↓
5. Business / Policy Validation
     ↓
6. Human Approval for Risky Action
```

Each layer catches a different class of problem.

---

# 8. DevOps Evidence Example

Current evidence:

```text
[E1] Deployment failed during Terraform Apply.
[E2] Terraform removed aks-subnet-allow.
[E3] AKS subnet connectivity validation failed.
```

Reasonable structured RCA:

```json
{
  "root_cause": "Removal of aks-subnet-allow is the strongest evidence-supported cause of the connectivity failure.",
  "impact": "Deployment failed and AKS connectivity validation failed.",
  "recommended_fix": "Review and restore the required NSG rule through the controlled Terraform workflow, then validate connectivity.",
  "confidence": "high"
}
```

Unsupported:

```text
"20,000 customers were affected"
```

unless evidence says so.

---

# 9. Facts vs Inference vs Recommendation

A strong schema can separate them:

```python
class RCA(BaseModel):
    confirmed_facts: list[str]
    root_cause_hypothesis: str
    evidence_gaps: list[str]
    recommended_next_checks: list[str]
    confidence: Literal["low", "medium", "high"]
```

Why useful?

```text
Fact != inference != recommendation
```

This reduces accidental overclaiming.

---

# 10. Structured Output Practical

Run:

```powershell
python examples/03_structured_output.py
```

Inspect:

```text
Model output
→ parser/schema
→ Python object
```

Then deliberately test failures.

---

# 11. Failure Drill A — Missing Field

Input candidate:

```json
{
  "root_cause": "NSG removed",
  "impact": "Deployment failed"
}
```

Expected:

```text
Schema validation fails because required fields missing.
```

---

# 12. Failure Drill B — Wrong Enum

```json
{
  "root_cause": "NSG removed",
  "impact": "Deployment failed",
  "recommended_fix": "Review Terraform",
  "confidence": "100_percent"
}
```

Expected: validation error.

---

# 13. Failure Drill C — Valid Schema, False Claim

```json
{
  "root_cause": "Database outage",
  "impact": "All customers affected",
  "recommended_fix": "Restart database",
  "confidence": "high"
}
```

Schema may pass.

Evidence validator should fail.

This is the most important practical exercise.

---

# 14. Source/Citation Validation

If output includes:

```json
{
  "sources": ["E1", "E2", "E9"]
}
```

but evidence store only contains E1-E3:

```text
E9 = invalid citation
```

Host should reject or downgrade the result.

Never trust model-generated source IDs blindly.

---

# 15. Deterministic Fields

Some fields should be calculated by host rather than model.

Example:

```text
Confirmed impact
= parse known pipeline status
```

If evidence says:

```text
Deployment failed
```

host can set:

```python
impact = "Deployment failed"
```

instead of asking model to invent broader impact.

---

# 16. Confidence Policy

Model self-confidence is not objective confidence.

Better learning policy:

```text
No current evidence → low / insufficient evidence
One source only → medium max
Multiple independent agreeing sources → may be high
Conflicting sources → lower
```

Later modules formalize eval/calibration.

---

# 17. Abstention / Unknown State

Your schema should allow uncertainty.

Example:

```python
class RCA(BaseModel):
    root_cause: str | None
    status: Literal["SUCCESS", "INSUFFICIENT_EVIDENCE"]
```

Why?

If model is forced to always populate `root_cause`, it may guess.

Correct behavior:

```text
No evidence
→ INSUFFICIENT_EVIDENCE
```

---

# 18. Structured Evidence vs Structured Output

Do not mix:

```text
Structured Evidence
= normalized tool data

Structured Output
= model-generated result in schema
```

Flow:

```text
External System
→ Tool
→ Structured Evidence
→ Model Reasoning
→ Structured Output
→ Validation
```

---

# 19. OpenAI vs Ollama

Both can participate in structured workflows, but exact provider/model structured-output capabilities may differ by current API/model support.

Stable course pattern:

```text
Provider/model generates candidate
Host validates with Pydantic/business rules
```

Never put your trust logic only in provider behavior.

---

# 20. Error Handling

Possible failure classes:

```text
invalid JSON
missing field
wrong type
provider refusal/error
unsupported structured capability
schema mismatch
evidence mismatch
invalid citation
```

Normalize them separately.

Do not do:

```python
except Exception:
    return default_rca_that_looks_successful
```

---

# 21. Production Architecture

```text
LLM Provider
    ↓
Structured Candidate
    ↓
Pydantic / JSON Schema
    ↓
Citation Validator
    ↓
Evidence Validator
    ↓
Policy Engine
    ↓
Approved Application State
```

For write recommendations:

```text
Approved Application State
→ Human Approval
→ Controlled Executor
```

For deeper evaluation/red-team methodology, see **Module 10**. This lesson establishes the validation layers that later evaluations test.

---

# 22. Common Beginner Mistakes

1. JSON = truth.
2. Pydantic = hallucination prevention.
3. Required root cause field even when evidence missing.
4. Model-generated confidence trusted directly.
5. Model-generated citations never checked.
6. Facts and recommendations mixed.
7. Destructive command included as automatic output action.
8. Parser exception converted into success.
9. Structured evidence and structured output confused.
10. Provider-specific schema support treated as application security.

---

# 23. Practical Acceptance Test

Learner should create four cases:

```text
Case 1: valid schema + supported facts → PASS
Case 2: invalid schema → FAIL
Case 3: valid schema + unsupported claim → FAIL
Case 4: insufficient evidence → ABSTAIN
```

If your app cannot distinguish these, structured-output design is incomplete.

---

# 24. Interview Q&A

### Q1. What is structured output?
Model output constrained to a machine-readable schema.

### Q2. What is Pydantic used for?
Python data/schema validation and parsing.

### Q3. Does Pydantic prove factual correctness?
No.

### Q4. What is layered validation?
Separate checks for syntax/schema, sources, evidence, policy and authorization.

### Q5. Why allow abstention?
To avoid forcing unsupported conclusions.

### Q6. Why validate citation IDs?
The model can invent or reference nonexistent sources.

### Q7. Structured evidence vs structured output?
Evidence is normalized tool/source data; structured output is the model's final schema-constrained response.

### Q8. Why calculate some fields deterministically?
To reduce hallucination for facts already derivable from authoritative evidence.

---

# 25. Revision Sheet

```text
Structured Output = predictable model data shape
Schema = expected fields/types
Pydantic = Python schema validator
Schema-valid != factual truth
Evidence validation = separate layer
Citation IDs = must be checked
Confidence = policy/calibration concern
No evidence = abstain
```

---

# 26. Homework

1. Define an RCA Pydantic model.
2. Test missing field and invalid confidence.
3. Create one schema-valid but false RCA and explain why Pydantic cannot catch it.
4. Add `sources: list[str]` and validate against known evidence IDs.
5. Add `INSUFFICIENT_EVIDENCE` status.
6. Decide which RCA fields should be model-generated vs host-calculated.

---

# 27. Why Next Lesson?

Ab model predictable structured result de sakta hai.

But real DevOps RCA ke liye model ko current system evidence chahiye.

Next:

```text
LLM ko external capability request kaise karwayein
without giving it direct execution authority?
```

➡️ **Lesson 8 — Tool Calling / Function Calling**