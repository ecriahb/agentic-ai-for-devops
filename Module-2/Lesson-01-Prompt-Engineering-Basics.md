# 🚩 Jai Bajrangbali!

# Lesson 01 — Prompt Engineering Basics

> **Prompt engineering ka matlab clever sentence likhna nahi; model ko clear task, useful context, boundaries aur expected output dena hai.**

> **Canonical ownership:** Module-2 is the definitive course module for general Prompt Engineering. Module-0 introduces the idea; later modules apply it to RAG, orchestration, agents and security instead of repeating the full theory.

## Where This Lesson Fits

```text
Module 0
→ introduces what prompts are and why model behavior depends on instructions
        ↓
Module 1
→ uses prompts inside API/tool exercises
        ↓
Module 2 Lesson 1
→ owns the general prompt-engineering mental model
        ↓
Lesson 2
→ turns that model into a repeatable five-part contract
```

**Scope:** This lesson explains *why* prompt engineering matters and the core anatomy. Detailed framework construction belongs to Lesson 2; advanced hallucination/context/evaluation topics belong to Lessons 6–10.

---

# 🎯 Lesson Goal

Is lesson ke end tak aap samjhoge:

- prompt kya hota hai
- prompt engineering kya solve karta hai
- vague prompt aur engineered prompt me difference
- instruction, context, constraints aur output contract ka role
- DevOps incidents me evidence-first prompting kyu important hai
- prompt ko security boundary kyu nahi samajhna chahiye

---

# 1. English Definition

**Prompt engineering is the practice of designing instructions, context, constraints and output requirements so that a language model performs a task more reliably and predictably.**

Simple Hinglish:

```text
LLM ko sirf "kya chahiye" nahi,
"kis evidence se",
"kin rules ke andar",
aur "kis format me"
answer dena hai — ye define karna prompt engineering hai.
```

---

# 2. Why This Topic Comes After Module 1

Module 1 me humne seekha:

```text
LLM call
→ tool request
→ host executes tool
→ evidence
→ structured RCA
```

Lekin same evidence par bhi weak instruction ho to model:

- extra assumptions kar sakta hai
- wrong impact invent kar sakta hai
- format change kar sakta hai
- facts aur recommendations mix kar sakta hai

So next engineering question:

```text
Model ko reliable instructions kaise dein?
```

---

# 3. Prompt Anatomy

A useful prompt often contains:

```text
ROLE
  ↓
CONTEXT / EVIDENCE
  ↓
TASK
  ↓
CONSTRAINTS
  ↓
OUTPUT CONTRACT
```

This is only the **mental model** here. Lesson 2 owns the detailed construction and trade-offs of each block.

Example:

```text
ROLE:
You are a read-only DevOps incident analyst.

CONTEXT:
[E1] Terraform Apply removed aks-subnet-allow.
[E2] AKS subnet connectivity validation failed.

TASK:
Identify the strongest evidence-supported root-cause hypothesis.

CONSTRAINTS:
Do not invent downtime or customer impact.
If evidence is insufficient, say so.

OUTPUT:
Root Cause
Evidence
Evidence Gaps
Recommended Next Checks
```

---

# 4. Weak Prompt vs Strong Prompt

## Weak

```text
Why did my AKS deployment fail?
```

Problems:

```text
No evidence
No scope
No environment
No output format
No hallucination boundary
```

Model may answer from general knowledge.

## Better

```text
Analyze the supplied production deployment evidence.
Use only evidence for incident-specific factual claims.
Separate facts from inference.
Return Root Cause, Evidence Gaps and Next Checks.
```

Now behavior is more constrained.

---

# 5. Prompt != Context != Evidence

Keep these concepts separate:

| Concept | Meaning | DevOps example |
|---|---|---|
| Prompt | Instructions to the model | "Identify the failure stage" |
| Context | Information supplied to perform the task | pipeline metadata, deployment state |
| Evidence | Observations that support a claim | failed Terraform Apply, removed NSG rule |

```text
Prompt  → what should the model do?
Context → what information is available?
Evidence → what observations support the claim?
```

A prompt cannot turn missing evidence into truth.

Later Module-5 will use retrieved context in a RAG pipeline; Module-2 owns the general prompting principles.

---

# 6. Real DevOps Scenario

Incident evidence:

```text
[E1] Deployment failed during Terraform Apply.
[E2] NSG rule aks-subnet-allow was removed.
[E3] AKS subnet connectivity validation failed after the change.
```

Good task:

```text
Using E1-E3 only:
1. list confirmed facts
2. identify strongest supported hypothesis
3. state missing evidence
4. recommend read-only validation
```

Expected safe behavior:

```text
Confirmed:
- rule removed
- connectivity validation failed
- deployment failed

Supported hypothesis:
- NSG removal is strongly associated with the later connectivity failure

Unknown:
- exact customer impact
- node health unless separately checked
- whether restoration has fixed the problem
```

---

# 7. Specificity Without Over-Constraining

Too vague:

```text
Analyze this.
```

Too rigid:

```text
The answer must say NSG is definitely root cause.
```

Better:

```text
Choose the strongest evidence-supported explanation.
If evidence does not support a root cause, return INSUFFICIENT_EVIDENCE.
```

Good prompt engineering guides without forcing false conclusions.

---

# 8. Output Contract

Humans prose tolerate kar sakte hain, applications nahi.

Bad:

```text
Tell me what happened.
```

Better:

```text
Return:
- Confirmed Evidence
- Likely Root Cause
- Confirmed Impact
- Evidence Gaps
- Recommended Next Checks
```

Detailed machine-level structured-output validation belongs to **Lesson 7**; this lesson only introduces the concept of an output contract.

Remember:

```text
Good format != factual truth
```

---

# 9. Prompt Engineering vs Context Engineering

For this course, keep the boundary explicit:

```text
Prompt Engineering
→ designing instructions, constraints and output contracts

Context Engineering
→ selecting, transforming, prioritizing and budgeting information
```

**Lesson 1 introduces this distinction. Lesson 7 owns the detailed context-engineering methodology.**

Module-2 teaches the general prompting principles. Later modules apply them to RAG, orchestration and agents.

---

# 10. Prompt is Not a Security Boundary

System prompt me likhna:

```text
Never delete production.
```

useful hai, but sufficient nahi.

Real security:

```text
Tool allowlist
+ argument validation
+ RBAC
+ policy
+ human approval
```

Prompt = behavior guidance.
Host application = enforcement.

Module-10 owns the comprehensive security treatment.

---

# 11. Provider Independence

Same prompt ko different providers/models par run kar sakte ho. Example:

```text
Ollama / local model
or
OpenAI API
```

Practical:

```powershell
$env:LLM_PROVIDER="ollama"
python Module-2/examples/dual_provider_prompt_playground.py
```

Then compare the same prompt using your configured provider.

Compare:

- structure adherence
- unsupported assumptions
- evidence handling
- verbosity
- latency

Do not judge only wording.

---

# 12. Common Beginner Mistakes

1. **Prompt ko sirf question samajhna** — production prompt is an instruction contract.
2. **Too much irrelevant context** — noise can reduce useful signal.
3. **Desired conclusion prompt me inject kar dena** — confirmation bias.
4. **No abstention rule** — model may be forced to answer.
5. **No output contract** — automation becomes less predictable.
6. **Prompt ko authorization samajhna** — unsafe.
7. **One successful run ko proof samajhna** — prompts need evaluation across cases.
8. **Same theory ko har module me copy karna** — reference the canonical Module-2 lesson instead.

---

# 13. Production Checklist

Before using a prompt in an application, ask:

```text
Is task explicit?
Is evidence scope clear?
Are prohibited assumptions explicit?
Can model abstain?
Is output machine-consumable?
Are dangerous actions host-controlled?
Is prompt versioned?
Is there an eval dataset?
```

---

# 🎤 Interview Q&A

### Q1. What is prompt engineering?

Designing instructions, context, constraints and output requirements to improve model reliability for a task.

### Q2. Does a better prompt eliminate hallucination?

No. It can reduce risk, but factual grounding and application validation are still required.

### Q3. Prompt vs context?

Prompt tells the model what to do; context supplies information needed to do it.

### Q4. Why define an output contract?

To make responses more predictable and machine-consumable.

### Q5. Is a system prompt a security control?

It is a behavioral control, not an authorization boundary.

---

# 🧠 Quick Revision

```text
Prompt Engineering
=
Instruction Design
+ Relevant Context
+ Constraints
+ Output Contract
+ Evaluation
```

Core rule:

```text
Prompt guides.
Evidence grounds.
Host validates.
```

---

# 🧪 Homework

Take this weak prompt:

```text
Fix my AKS problem.
```

Rewrite it with:

- Role
- Context
- Task
- Constraints
- Output

Then run the same prompt against two model/provider configurations and record three behavior differences.

---

# 🔗 Cross-Module Ownership

This lesson is the **canonical foundation** for general Prompt Engineering.

- **Module-0:** introduces LLM/prompt concepts only; do not duplicate this full lesson.
- **Module-1:** uses prompts during the first API/agent exercises.
- **Module-5:** applies prompting to RAG grounding and retrieved context.
- **Module-6:** implements prompts through LangChain abstractions.
- **Module-8:** uses prompts inside stateful agent graphs.
- **Module-9:** applies prompting to specialized multi-agent roles.
- **Module-10:** evaluates and secures prompt-driven agent behavior.

## ➡️ Why Next?

Ab prompt ke core pieces samajh aa gaye. Next lesson me hum ek repeatable framework banayenge:

```text
Role + Context + Task + Constraints + Output
```
