# 🚩 The Agentic AI Blueprint — Module 0: Foundation & Course Architecture

> **From ChatGPT User to Production AI Engineer**

Module 0 is the conceptual foundation of the course **and the canonical navigation layer for Modules 0–12**.

The goal is not only to teach LLM fundamentals. It is to establish **where each concept belongs, what each later module is allowed to assume, and how every lesson connects to the next lesson without unnecessary re-teaching**.

---

# 1. The Course Contract

The course follows one progression:

```text
UNDERSTAND
   ↓
USE
   ↓
CONTROL
   ↓
RETRIEVE
   ↓
ORCHESTRATE
   ↓
STANDARDIZE
   ↓
STATE
   ↓
COORDINATE
   ↓
SECURE + EVALUATE
   ↓
OPERATE AT ENTERPRISE SCALE
```

The final objective is not “learn a framework.” It is to build an **evidence-grounded, policy-controlled DevOps AI system**.

```text
Incident / Question
        ↓
Collect Current Evidence
        ↓
Retrieve Approved Reference Knowledge
        ↓
Reason with Explicit Context
        ↓
Validate Claims / Structure
        ↓
Policy + Authorization
        ↓
Human Approval where required
        ↓
Controlled Execution
        ↓
Verification + Audit
```

## Golden principle

> **The LLM can reason and propose. The host application owns evidence handling, authorization, policy and execution.**

---

# 2. How Lessons Connect

Every lesson in this repository should answer four questions:

```text
1. What did the previous lesson establish?
2. What does this lesson uniquely own?
3. What is deliberately out of scope here?
4. What exact gap does the next lesson solve?
```

## Canonical lesson contract

Each lesson should contain, directly or indirectly:

```text
Previous Lesson
      ↓
Prerequisites
      ↓
Current Lesson Goal
      ↓
Canonical Concepts Owned Here
      ↓
Practical / Experiment
      ↓
Common Failure Modes
      ↓
Revision / Interview
      ↓
Why Next Lesson?
```

### One Concept → One Canonical Home

```text
Teach deeply once.
Reference later.
Apply later in context.
Do not silently re-teach the same concept as if it were new.
```

A later module may revisit a concept **only when the context changes**.

Example:

```text
Embeddings
Module 4 → canonical theory
Module 5 → use embeddings inside RAG
Module 6 → implement embeddings through LangChain
```

---

# 3. Complete Course Architecture — Modules 0–12

The module names below are the repository's current architecture and are the reference point for lesson boundaries.

| Module | Canonical Responsibility | Depends Mainly On |
|---|---|---|
| [0](README.md) | AI/LLM foundations + course architecture | — |
| [1](../Module-1/README.md) | First AI application: API, local model, structured output, basic tools | M0 |
| [2](../Module-2/README.md) | Prompt + Context Engineering | M0–M1 |
| [3](../Module-3/README.md) | API / HTTP / Python application integration | M1 |
| [4](../Module-4/README.md) | Embeddings, similarity, vector stores and retrieval foundations | M0–M3 |
| [5](../Module-5/README.md) | Complete RAG pattern: retrieval → context → grounded generation | M4 |
| [6](../Module-6/README.md) | LangChain implementation/orchestration of AI application patterns | M1–M5 |
| [7](../Module-7/README.md) | MCP protocol, capabilities, transports and MCP-specific security | M1, M6 |
| [8](../Module-8/README.md) | Stateful agent workflows and LangGraph state/graph execution | M1–M7 |
| [9](../Module-9/README.md) | Multi-agent specialization, routing and coordination | M8 |
| [10](../Module-10/README.md) | Security, evaluation, red teaming and production controls | M1–M9 |
| [11](../Module-11/README.md) | Enterprise Azure architecture, operations and deployment concerns | M4–M10 |
| [12](../Module-12/README.md) | Final enterprise DevOps AI assistant capstone | M0–M11 |

## End-to-end progression

```text
M0  AI / LLM Fundamentals
 ↓
M1  First AI Application + Tools
 ↓
M2  Prompt + Context Engineering
 ↓
M3  APIs + Python Integration
 ↓
M4  Embeddings + Vector Retrieval
 ↓
M5  RAG + Grounding
 ↓
M6  LangChain Orchestration
 ↓
M7  MCP Protocol
 ↓
M8  Stateful Agents / LangGraph
 ↓
M9  Multi-Agent Systems
 ↓
M10 Security + Evaluation
 ↓
M11 Enterprise Architecture / Operations
 ↓
M12 Production DevOps AI Capstone
```

---

# 4. Concept Ownership Matrix

This matrix prevents the same concept from being taught from scratch in multiple modules.

| Concept | Canonical Home | Later Modules Should Do |
|---|---|---|
| AI / ML / DL / LLM | M0 | Brief recall only |
| Tokens / context window | M0 | Provider/application implications |
| Hallucination fundamentals | M0 | Apply reduction/evaluation in context |
| Prompt engineering | M2 | Context-specific application |
| Context engineering | M2 | RAG/agent implementation |
| Tool calling fundamentals | M1 | MCP/stateful/multi-agent integration |
| API fundamentals | M3 | Use APIs inside frameworks/tools |
| Embeddings | M4 | RAG/framework implementation |
| Cosine / distance metrics | M4 | Use in retrieval evaluation |
| Vector DB / index | M4 | Use through RAG/frameworks |
| Metadata filtering | M4 | RAG retrieval application |
| Chunking | M4 | RAG-specific tuning in M5 |
| RAG architecture | M5 | Framework implementation in M6 |
| Grounding / citations | M5 | Agent workflows and evaluation |
| LangChain | M6 | Do not re-teach base APIs elsewhere |
| MCP protocol | M7 | Integrate MCP without redefining protocol |
| Stateful graph execution | M8 | Multi-agent composition in M9 |
| Multi-agent coordination | M9 | Secure/evaluate in M10 |
| Security/evaluation | M10 | Enterprise deployment controls in M11 |
| Enterprise architecture | M11 | Capstone integration in M12 |

### Boundary rule

> **A later lesson may mention a canonical concept, but it should link back to the canonical lesson instead of reproducing the entire theory.**

---

# 5. Major Layer Boundaries

## Prompting vs Context Engineering

```text
Prompt Engineering (M2)
→ What should the model do?

Context Engineering (M2)
→ What information should the model receive, in what form, with what constraints?

RAG (M5)
→ How do we retrieve external information and turn it into context?
```

## Retrieval vs RAG

```text
M4
Embedding → Similarity → Vector Search → Retrieved Chunks

M5
Retrieved Chunks → Context → Grounded Prompt → LLM → Answer
```

## Tool Calling vs MCP vs Agents

```text
M1
Basic tool-calling pattern

M7
MCP protocol and standardized capability connectivity

M8
Stateful agent workflows / graph execution

M9
Multi-agent coordination
```

## Security boundaries

```text
M1
Basic secret/tool-safety principles

M7
MCP-specific trust/security concepts

M10
Comprehensive security, evaluation and red teaming

M11
Enterprise identity/network/operations controls
```

---

# 6. Retrieval Ownership — Canonical Example

The course must keep these concepts separate:

```text
Embedding
→ numerical representation

Similarity
→ mathematical relevance signal

Vector Store / Database
→ storage + retrieval system

Metadata
→ structured attributes / constraints

Pre/Post-filter semantics
→ when metadata constraints participate in retrieval

Top-K
→ how many ranked results are selected

RAG
→ retrieval + context construction + generation
```

Therefore the learning chain is:

```text
M4 L02/L03
Embedding representation + creation
        ↓
M4 L04/L05
Similarity / metrics
        ↓
M4 L06
Vector DB, index, metadata filtering and retrieval semantics
        ↓
M4 L07
Chroma/FAISS implementation
        ↓
M5
Complete RAG system
```

This is an example of how course-wide architecture should prevent the same concept from appearing as a duplicate “new topic.”

---

# 7. Practical Learning Architecture

Every practical should evolve the same system instead of creating unrelated demos.

```text
P0 — Observe manually
P1 — One isolated concept
P2 — Combine two concepts
P3 — Add validation / evidence
P4 — Add failure drill
P5 — Add integration
P6 — Add policy / security
P7 — Add evaluation
P8 — Module project
P9 — Cross-module integration
P10 — Enterprise capstone
```

For the retrieval/RAG path, the progression should look like:

```text
Text
 ↓
Embedding
 ↓
Similarity
 ↓
Ranking
 ↓
Metadata filtering
 ↓
Vector store/index
 ↓
RAG context
 ↓
Grounded generation
 ↓
Citations / evaluation
 ↓
Agent/tool integration
```

A practical is not complete merely because the script runs. The learner should be able to explain:

```text
What changed?
Why did it change?
What can fail?
Which part is deterministic?
Which part is model-driven?
What is evidence?
What is inference?
What is the safety boundary?
Why does the next practical exist?
```

---

# 8. Module 0 Lesson Sequence

Module 0 teaches the conceptual language needed by everything that follows. These are the 15 current lesson entries:

| No. | Lesson | Canonical purpose |
|---|---|---|
| 00 | [Orientation](Lesson-00-Orientation.md) | Course mindset, scope and how to learn |
| 01 | [AI Revolution](Lesson-01-AI-Revolution.md) | Why AI systems matter |
| 02 | [AI → ML → DL → LLM](Lesson-02-AI-ML-DL-LLM.md) | Vocabulary and hierarchy |
| 03 | [Next Token Prediction](Lesson-03-Next-Token-Prediction.md) | Core LLM generation mental model |
| 04 | [Transformer & Attention](Lesson-04-Transformer-Attention.md) | Architecture intuition |
| 05 | [Context Window](Lesson-05-Context-Window.md) | Finite model input working space |
| 06 | [Hallucination](Lesson-06-Hallucination.md) | Why fluent output can be wrong |
| 07 | [Prompt Engineering](Lesson-07-Prompt-Engineering.md) | Intro only; canonical deep dive is M2 |
| 08 | [System vs User Prompt](Lesson-08-System-vs-User-Prompt.md) | Instruction hierarchy foundation |
| 09 | [Temperature](Lesson-09-Temperature.md) | Sampling/behavior intuition |
| 10 | [Role Prompting](Lesson-10-Role-Prompting.md) | Role/instruction pattern introduction |
| 11 | [Zero/One/Few-Shot](Lesson-11-Zero-One-Few-Shot.md) | Example-based prompting intuition |
| 12 | [Structured Reasoning](Lesson-12-Structured-Reasoning.md) | Reasoning workflow intuition |
| 13 | [AI Limitations & Safety](Lesson-13-AI-Limitations-Safety.md) | Trust, safety and human-control foundation |
| 14 | [Grand Revision + Mini Project](Lesson-14-Grand-Revision-Mini-Project.md) | Integrate M0 concepts and prove readiness for M1 |

### Module 0 boundary rule

Module 0 **introduces** prompting concepts; it does not attempt to replace Module 2's complete prompt/context engineering curriculum.

Similarly, Module 0 introduces hallucination and context; later modules apply those concepts in RAG, agent and evaluation settings instead of silently re-teaching the foundations.

---

# 9. Definition → Intuition → Example → Failure → Next

The course's preferred teaching pattern is:

```text
Technical Definition
        ↓
Simple Hinglish Intuition
        ↓
DevOps Example
        ↓
Visual / Flow
        ↓
Common Failure / Edge Case
        ↓
Practical
        ↓
Revision
        ↓
Why Next?
```

For important concepts, add:

```text
Canonical source
Library semantics
Version notes
Citation / official reference
```

The goal is to keep explanations approachable without sacrificing technical precision.

---

# 10. Trust Model Used Across Every Module

```text
USER INPUT              = untrusted
RAG REFERENCE DATA      = untrusted external/reference data until validated
MODEL OUTPUT            = untrusted analysis/proposal
LLM TOOL REQUEST        = untrusted proposal
TOOL OUTPUT             = evidence candidate with provenance
CURRENT LIVE STATE      = must come from appropriate tools/systems
AUTHORIZATION           = deterministic host-controlled decision
POLICY                  = host-controlled rule
HUMAN APPROVAL          = explicit risk gate where required
EXECUTOR                = known, isolated implementation
```

This prevents a common design mistake:

> **Do not turn “the model said it” or “the document says it” into automatic authority.**

---

# 11. Course-Wide “Out of Scope” Rule

When a concept is intentionally deferred, say where it moves next.

Examples:

```text
M0 Prompting
→ deep prompt/context engineering is M2

M4 Vector Retrieval
→ complete RAG generation is M5

M6 Tooling
→ protocol standardization is M7

M8 Agent State
→ multi-agent coordination is M9

M10 Security/Evals
→ enterprise deployment concerns continue in M11
```

A lesson should never end with an unexplained “later.”

---

# 12. Module / Lesson Quality Gate

Before a lesson is considered complete:

### Content

```text
[ ] Unique canonical scope
[ ] No unnecessary duplicate teaching
[ ] Clear prerequisites
[ ] Clear in-scope concepts
[ ] Clear out-of-scope concepts
[ ] DevOps example
[ ] Failure modes / edge cases
```

### Progression

```text
[ ] Previous lesson referenced correctly
[ ] Current lesson solves a specific gap
[ ] Next lesson solves a new, explicit gap
[ ] Cross-module reference is accurate
```

### Practical

```text
[ ] Runnable or intentionally no-code
[ ] Dependencies documented
[ ] No hardcoded secrets
[ ] Error handling / failure drill where relevant
[ ] Learner can explain expected vs actual output
```

### Sources

```text
[ ] Official sources preferred
[ ] Current library semantics checked
[ ] Links are live/current
[ ] Claims are not stronger than evidence
```

---

# 13. Before Starting Module 1

The learner should be able to explain, without notes:

```text
1. What is an LLM?
2. How does next-token generation work conceptually?
3. Why does context matter?
4. Why can an LLM hallucinate?
5. What is a prompt?
6. Prompt vs context?
7. Why is a fluent answer not automatically a factual answer?
8. Why must risky actions remain outside model authority?
```

If these are clear, Module 1 can introduce actual API/application work without repeatedly reopening the entire foundation.

---

# 14. Beginner Hands-On Experiments

Module 0 intentionally uses **no-code experiments** so that mental models are established before API/application coding.

Open:

### [Module 0 Hands-On Experiments](examples/README.md)

The experiments cover:

```text
Next-token prediction intuition
Context comparison
Hallucination test
System vs user prompt
Zero-shot vs few-shot
Prompt injection intuition
Fact vs inference
First DevOps AI safety rules
No-code mini project
```

Actual local/API coding starts in Module 1.

---

# 15. Recommended Course Loop

For every lesson:

```text
Read
 ↓
Understand the previous → current → next connection
 ↓
Run matching practical/experiment
 ↓
Change one input
 ↓
Break one assumption
 ↓
Explain expected vs actual behavior
 ↓
Complete homework/interview questions
 ↓
Update notes / code
 ↓
Move to the next lesson
```

When studying with the GitHub branch, the working rule is:

> **Learn → Audit → Improve → Commit → Verify → Continue**

This keeps the learning material and repository synchronized.

---

# 16. Final Mental Model for the Entire Course

```text
M0
Understand the model
        ↓
M1
Call it and give it controlled capabilities
        ↓
M2
Control instructions and context
        ↓
M3
Connect real APIs
        ↓
M4
Build semantic retrieval
        ↓
M5
Ground generation with retrieved knowledge
        ↓
M6
Compose application workflows
        ↓
M7
Standardize external capabilities with MCP
        ↓
M8
Add explicit state and durable workflows
        ↓
M9
Coordinate specialist agents
        ↓
M10
Secure, evaluate and red-team the system
        ↓
M11
Deploy and operate at enterprise scale
        ↓
M12
Integrate everything into the final DevOps AI assistant
```

### Final course principle

> **Every new module should add one new engineering capability without silently redefining an earlier capability.**

🚩 **Jai Bajrangbali — Learn • Build • Break • Validate • Secure • Operate**
