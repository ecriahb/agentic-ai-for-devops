# Module 1 — Lesson 6: Tokens, Cost & Context Budgets

> **Goal:** Understand the mechanics needed for a reliable first API application: tokens, finite context capacity, hosted usage/cost signals, and why oversized DevOps inputs should not be sent blindly.

---

> **Course boundary:** This lesson is a **foundation and operational primer**, not the course's context-engineering curriculum. **Module 2 Lesson 7 — Context Engineering for Logs, Terraform & AKS** is the canonical deep-dive for context selection, normalization, source labeling, trust classes, deduplication, prioritization, redaction and budgeting. Module 4–5 later apply these principles to embeddings and RAG retrieval.

## Where This Lesson Fits

```text
Lesson 05
Request + response mechanics
        ↓
Lesson 06
Tokens + context capacity + usage awareness
        ↓
Lesson 07
Structured output + validation
```

The detailed context-engineering methodology lives in **Module 2 Lesson 7**. Later modules reuse that foundation rather than re-teaching it.

---

# 1. English Definitions

**Token:** A unit of text representation that a language model processes internally.

**Context Window:** The finite amount of tokenized input/state a model can consider for a request, subject to the specific model and API behavior.

**Context Budget:** The practical allocation of available context capacity among instructions, user input, evidence, history, tool results and output requirements.

Simple Hinglish:

```text
Text
  ↓ tokenization
Tokens
  ↓
Model Context
  ↓
Generated Output
```

---

# 2. Why This Topic Comes Here

Lesson 5 me request/response samjha.

Ab next engineering questions:

```text
Request kitna bada ho sakta hai?
Input aur output usage ko kaise observe karein?
Bahut bada DevOps log blindly bhejna kyu risky hai?
```

Ye concepts aage ke context-engineering work ke liye prerequisite hain.

---

# 3. Token != Word

One English word may map to one or multiple tokens. Code, punctuation, JSON, paths and identifiers also consume tokens.

Avoid this simplification:

```text
1 word = 1 token
```

Exact tokenization depends on the model/tokenizer.

Operationally:

```text
More input text
→ usually more input tokens
→ potentially more processing/latency/cost
```

---

# 4. Input vs Output Tokens

Conceptually:

```text
INPUT TOKENS
= instructions + question + supplied context/evidence + relevant history

OUTPUT TOKENS
= model-generated content
```

Hosted APIs commonly expose usage metadata. Treat those values as runtime observations rather than hard-coded course constants.

---

# 5. Context Capacity

A useful mental model is:

```text
┌──────────────────────────────┐
│ Available Context Capacity   │
│                              │
│ Instructions                 │
│ Current question             │
│ Evidence / tool results      │
│ Relevant history             │
│ Output requirements          │
└──────────────────────────────┘
```

If the supplied request exceeds current model/API limits, behavior can include rejection, truncation or other provider/model-specific handling.

> **Treat context as an engineering budget, not an unlimited memory store.**

---

# 6. Why DevOps Engineers Must Care

A bad first approach is:

```text
100 MB pipeline log
+ terraform plan
+ kubectl events
+ all historical incidents
→ send everything to LLM
```

Potential problems:

- unnecessary input volume
- latency and hosted usage/cost
- relevant evidence buried in noise
- secret/sensitive-data exposure
- larger prompt-injection surface

The **full method for solving these problems belongs to Module 2 Lesson 7**. Here, remember only the principle:

```text
Do not maximize context.
Maximize relevant, usable context.
```

---

# 7. Prompt Engineering vs Context Engineering

This lesson only establishes the distinction:

```text
Prompt Engineering
→ What should the model do?

Context Engineering
→ What information should the model receive to do it?
```

Example:

```text
Prompt:
Identify the strongest evidence-supported root-cause hypothesis.

Context:
[E1] Deployment failed during Terraform Apply.
[E2] NSG rule was removed.
[E3] AKS connectivity validation failed.
```

For the full methodology, see:

> **Module 2 → Lesson 7: Context Engineering for Logs, Terraform & AKS**

That lesson owns normalization, source IDs, trust classes, deduplication, prioritization, redaction, context ordering and context testing.

---

# 8. Context != Memory

Keep these concepts separate:

```text
Context
= information supplied to the model for the current request

Application State
= data preserved by the host across workflow steps

Conversation History
= prior messages

Evidence
= observations that support or contradict a claim
```

They can overlap, but they are not interchangeable.

---

# 9. Hosted Cost Thinking

Do not memorize static provider pricing in this lesson.

Understand the drivers:

```text
more input
+ more output
+ more calls
+ more expensive model
= potentially higher hosted usage/cost
```

Useful operational observations include:

```text
usage metadata
request count
latency
model selection
context size
```

---

# 10. Local Ollama Cost Thinking

Local inference may avoid a hosted per-call charge, but it is not resource-free.

You still consume:

```text
CPU / GPU
RAM / VRAM
electricity
latency / throughput
hardware capacity
operations
```

So:

```text
Local ≠ free engineering
Local = different cost model
```

---

# 11. Minimal Context Experiment

This lesson needs only a small experiment; the detailed evidence-engineering lab is owned by Module 2.

Compare:

### Version A — Minimal

```text
Why did the deployment fail?
```

### Version B — Curated evidence

```text
[E1] Deployment failed during Terraform Apply.
[E2] NSG rule aks-subnet-allow was removed.
[E3] AKS connectivity validation failed.
```

Observe:

```text
relevance
unsupported assumptions
output length
latency / usage metadata where available
```

Do not turn this lesson into a full context-cleaning lab; that belongs in Module 2 Lesson 7.

---

# 12. Context Size and Security

One foundational rule:

> **Never send sensitive data simply because context capacity is available.**

Example:

```text
AZURE_CLIENT_SECRET=abc123
```

should be redacted before model input:

```text
AZURE_CLIENT_SECRET=[REDACTED]
```

The complete redaction and sensitive-context handling methodology is covered canonically in Module 2 Lesson 7 and later security modules.

---

# 13. Chunking Preview

Very large knowledge sources may later be handled as:

```text
Large Document
→ Split into chunks
→ Embed/index chunks
→ Retrieve relevant chunks
→ Supply selected context
```

Here we only need the motivation. Detailed chunking belongs to **Module 4** and detailed RAG retrieval belongs to **Module 5**.

---

# 14. Common Beginner Mistakes

1. Treating a token as a word.
2. Assuming context is unlimited.
3. Sending complete logs because more data feels safer.
4. Confusing context with permanent memory.
5. Memorizing stale provider pricing.
6. Treating local inference as zero-cost infrastructure.
7. Re-teaching full context-engineering methods here instead of using the canonical Module 2 lesson.

---

# 15. Interview Q&A

### Q1. What is a token?
A model-processing unit derived from text/code by the model's tokenizer.

### Q2. What is a context window?
The finite amount of tokenized information a model can consider for a request/conversation context, subject to model behavior.

### Q3. Why not send complete DevOps logs?
Because unnecessary volume can increase noise, latency, usage/cost and sensitive-data exposure while burying relevant evidence.

### Q4. Prompt engineering vs context engineering?
Prompt engineering defines the task/instructions; context engineering selects and prepares the information supplied for that task.

### Q5. Is context the same as memory?
No. Context is supplied for the current model interaction; application state and memory are separate application concepts.

---

# 16. Revision Sheet

```text
Token = model-processing unit
Input tokens = supplied instructions/context
Output tokens = generated content
Context window = finite working capacity
Context budget = how we allocate that capacity
Prompt = what to do
Context = what information to use
```

### Canonical ownership map

```text
Module 1
→ Token / context mechanics + usage awareness

Module 2
→ Context-engineering methodology

Module 4
→ Embedding/vector representation

Module 5
→ RAG retrieval + context construction for generation
```

---

# 17. Homework

1. Explain why 1 word is not necessarily 1 token.
2. Explain why context should be treated as a finite budget.
3. Compare a one-line prompt with a small evidence-labeled context bundle.
4. Explain why Module 2 is the canonical home for detailed context engineering.
5. Identify one type of sensitive data that should never be sent to a model without appropriate controls.

---

# ➡️ Why Next Lesson?

We now know how requests, tokens and context capacity behave.

But free-form model output is difficult to validate reliably in an application.

So the next gap is:

```text
Free-form model output
        ↓
Need predictable structure
        ↓
Lesson 7 — Structured Output & Validation
```
