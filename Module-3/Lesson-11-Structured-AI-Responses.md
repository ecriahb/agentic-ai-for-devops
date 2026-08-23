# 🚩 Jai Bajrangbali!

# Lesson 11 — Structured AI Responses

> **Human ke liye paragraph enough ho sakta hai; application ke liye predictable fields chahiye.**

> **Canonical boundary:** Module 1 introduced structured output as an application safety pattern. **Module 3 focuses on the API/data-contract layer**: JSON response shapes, schema transport, parsing and provider structured-output capabilities. Deep evidence/claim validation remains with Module 1's trusted-RCA pattern and later evaluation/security modules.

---

## 🎯 Lesson Goal

Aap samjhoge:

- free-form text vs structured data
- JSON output
- schema kya hota hai
- provider-side structured output concept
- Pydantic as a local application validator
- structure validation vs truth validation
- parsing failures
- evidence-grounded RCA contract

---

## 1. Problem with Free-Form Output

Model output 1:

```text
The issue appears to be caused by an NSG rule removal...
```

Model output 2:

```text
Root Cause: NSG rule removal
Impact: Connectivity failed
Fix: Restore rule
```

Human dono samajh lega.

Application ko problem:

```text
Where exactly is root_cause?
Is impact present?
What is severity?
Can pipeline parse this safely?
```

---

## 2. Structured Output

Desired contract:

```json
{
  "root_cause": "...",
  "impact": "...",
  "recommended_fix": ["..."],
  "severity": "high",
  "confidence": "medium"
}
```

Now downstream systems can process fields deterministically.

---

## 3. JSON Is Not Enough

Valid JSON:

```json
{
  "root_cause": 900,
  "severity": "banana"
}
```

Syntactically JSON valid hai, but application contract invalid.

Therefore:

```text
JSON format
   ↓
Schema validation
```

Detailed JSON syntax/serialization belongs to Lesson 04; this lesson applies it to AI response contracts.

---

## 4. Schema Kya Hai?

**English Definition:**
> A schema defines the expected structure, field types, required values and constraints of data.

Conceptual RCA schema:

```text
root_cause       → required string
impact           → required string
recommended_fix  → list of strings
severity         → low | medium | high | critical
confidence       → low | medium | high
```

---

## 5. Pydantic Example

```python
from typing import Literal
from pydantic import BaseModel


class IncidentRCA(BaseModel):
    root_cause: str
    impact: str
    recommended_fix: list[str]
    severity: Literal["low", "medium", "high", "critical"]
    confidence: Literal["low", "medium", "high"]
```

Validate:

```python
rca = IncidentRCA.model_validate({
    "root_cause": "NSG rule removed",
    "impact": "AKS connectivity validation failed",
    "recommended_fix": ["Restore required NSG rule"],
    "severity": "high",
    "confidence": "medium"
})

print(rca.model_dump())
```

Here Pydantic is the **application-side shape/type validator**.

---

## 6. Validation Failure

```python
IncidentRCA.model_validate({
    "root_cause": "NSG rule removed",
    "impact": "Connectivity failed",
    "recommended_fix": [],
    "severity": "banana",
    "confidence": "medium"
})
```

Pydantic rejects invalid enum/type according to schema.

This prevents malformed structured data from silently entering downstream systems.

---

## 7. Critical Principle: Structure ≠ Truth

Model can produce perfectly valid schema:

```json
{
  "root_cause": "Database corruption",
  "impact": "All customer data lost",
  "recommended_fix": ["Restore backup"],
  "severity": "critical",
  "confidence": "high"
}
```

But if evidence never mentioned database corruption, it is unsupported.

Therefore:

```text
Schema Validation
        ↓
Structure is valid
        ≠
Claim is factually supported
```

Need both:

```text
Structured Output Validation
         +
Evidence / Business Validation
```

The second layer is not solved by JSON/Pydantic alone.

---

## 8. Evidence-First RCA Contract

Prompt rule:

```text
Use only supplied evidence.
If root cause cannot be supported, state "insufficient evidence".
Do not invent customer impact.
```

Application rule:

```text
No evidence
   ↓
Do not call final RCA reporter
```

Schema rule:

```text
root_cause
impact
recommended_fix
severity
confidence
```

Together:

```text
Prompt Guardrail
 +
Schema Guardrail
 +
Application Guardrail
```

The application guardrail owns factual trust decisions; the schema owns data shape.

---

## 9. Provider-Side Structured Outputs

Modern LLM APIs may support provider-side structured output/schema features.

For supported providers/models, an explicit schema contract can reduce parsing ambiguity compared with simply asking:

```text
"Please return JSON"
```

But provider capabilities and syntax vary by model/API version, so current official provider documentation is the source of truth.

Even perfect schema adherence does not prove factual correctness.

---

## 10. Parsing Strategy

Preferred flow:

```text
Provider structured-output feature (if supported)
        ↓
Parse structured object
        ↓
Pydantic/business validation
        ↓
Use downstream
```

If provider returns plain JSON text:

```python
import json

data = json.loads(raw_text)
rca = IncidentRCA.model_validate(data)
```

Handle:

```text
malformed JSON
missing fields
wrong types
unsupported enum values
provider refusal/error
```

---

# 🛠️ DevOps Example

```text
pipeline.log
 ↓
LLM
 ↓
Structured RCA
 ↓
Pydantic
 ↓
Evidence/business validation
 ↓
Ticket / Slack / Dashboard / Pipeline Gate
```

The schema makes the API result machine-consumable; downstream policy decides whether the claims are trusted.

---

## 🔗 Module Boundary

```text
Module 1
→ structured-output safety + trusted RCA architecture

Module 2
→ prompt/output-contract design

Module 3 — this lesson
→ API/data-contract parsing + schema transport

Later security/evaluation modules
→ broader claim validation, adversarial testing and release gates
```

This prevents Module 3 from becoming a second full structured-RCA security course.

---

# ❌ Common Mistakes

- "return JSON" ko sufficient validation samajhna
- Pydantic ko hallucination detector samajhna
- missing required fields accept karna
- provider schema support ko universal assume karna
- model-generated confidence blindly trust karna
- raw structured output directly deployment automation me execute karna

---

# 🎤 Interview Point

**Q: What is the difference between structured output and validated truth?**

Structured output ensures the data follows an expected shape. Truth validation separately verifies whether claims are supported by trusted evidence and business rules.

**Q: Why use provider-side schema features?**

When supported, they reduce ambiguity in the model-to-application data contract, but they do not replace application-side validation.

---

# 🧠 Revision

```text
JSON
  ↓
Schema
  ↓
Parse
  ↓
Pydantic / Application Validation
  ↓
Business / Evidence Validation
```

Core rule:

```text
Valid structure != valid fact
```

---

# 🔁 Why Next Lesson?

Ab API, HTTP, JSON, auth, configuration, Python, provider differences, errors and structured responses connect ho gaye.

Ab in sab ko ek **first complete AI application** mein integrate karenge.

> **Lesson 12 — Mini Project: First AI Application**
