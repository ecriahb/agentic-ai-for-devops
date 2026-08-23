# Module 1 — Lesson 0: Roadmap & Mental Model

> **Goal:** Beginner ko pehle hi clear karna ki Module 1 me hum kya build karenge, kis order me build karenge, aur LLM/application/tool ke roles alag kyun hain.

## Where This Lesson Fits

```text
Module 0
AI / LLM fundamentals
        ↓
Module 1
Application foundation: APIs → models → tools → first controlled agent
        ↓
Module 2
Canonical prompt/context-engineering methodology
```

**This lesson owns the Module 1 roadmap and the host-controlled application mental model. It does not re-teach Module 0's LLM theory or Module 2's detailed prompt/context methodology.**

## English definition
**An AI application uses an existing model as a reasoning component while the host application controls data, tools, validation, policy and execution.**

## Module 0 → Module 1 bridge
Module 0 me humne samjha ki LLM token prediction karta hai, context limited hota hai aur hallucination possible hai. Ab Module 1 ka sawal hai:

```text
Agar LLM perfect truth machine nahi hai,
to use real application me safely kaise use karein?
```

## Core mental model

```text
User / Incident
     ↓
Python Host Application
     ↓
Prompt / Context
     ↓
LLM (OpenAI or Ollama)
     ↓
Model Response / Tool Request
     ↓
Host Validation
     ↓
Tool Execution
     ↓
Evidence
     ↓
Grounded Final Answer
```

Remember:

```text
LLM = reasoner
Host = controller/executor
Tool = capability
Tool result = evidence
Schema = output contract
Policy = deterministic boundary
```

## Canonical Module 1 sequence

0. Module 1 Roadmap & Mental Model
1. ChatGPT UI vs API
2. Development Environment & Secret Management
3. OpenAI Cloud API Setup
4. Zero-Cost Local AI with Ollama
5. First API Call & Response Object
6. Tokens, Cost & Context Budgets
7. Structured Output & Validation
8. Tool Calling / Function Calling
9. From Tool Calling to a Basic DevOps Agent
A. Complete Lab Code
B. Troubleshooting Playbook
C. Interview & Revision Sheet
D. Official References

## Why this order?

```text
UI vs API
→ application setup
→ cloud provider
→ local provider
→ first call
→ cost/context mechanics
→ structured contract
→ tools
→ agent loop
→ complete lab
```

Hum direct agent par jump nahi karte, kyunki agent actually in sab concepts ka combination hai.

## Lesson ownership map

| Lesson | Canonical responsibility | Out of scope / later owner |
|---|---|---|
| 00 | Module roadmap + host/model/tool mental model | Deep LLM theory → Module 0; prompt methodology → Module 2 |
| 01 | UI vs API and application integration | API protocol deep dive → Module 3 |
| 02 | Local Python environment + basic secret hygiene | Enterprise security controls → Module 10/11 |
| 03 | Hosted OpenAI setup + provider access | General API mechanics → Module 3; first-call anatomy → Lesson 05 |
| 04 | Ollama/local-provider path | Agent implementation → Lessons 08–09 |
| 05 | First request + response object | Deep API architecture → Module 3 |
| 06 | Tokens/context capacity + usage awareness | Context-engineering methodology → Module 2; RAG context → Module 5 |
| 07 | Structured output + validation | Evaluation depth → Module 10 |
| 08 | Basic tool/function calling | MCP protocol → Module 7; multi-agent tools → Module 9 |
| 09 | First bounded DevOps agent loop | Stateful graphs → Module 8; multi-agent orchestration → Module 9 |

**Canonical rule:** Learn a concept deeply once. Later modules may apply it to their own context, but should reference the canonical lesson instead of re-teaching the foundation.

## Recurring DevOps incident
Course me same incident repeatedly evolve hoga:

```text
Terraform Apply started
      ↓
NSG rule aks-subnet-allow removed
      ↓
AKS connectivity validation failed
      ↓
Deployment failed
```

Early stage me LLM sirf text analyze karega. Later stage me tools evidence collect karenge. Final stage me host evidence validate karke RCA allow karega.

## Two provider tracks

```text
Track A: OpenAI Cloud API
Track B: Ollama Local LLM
```

Provider badal sakta hai, engineering rules nahi:

```text
same evidence
same tool contracts
same validation
same authorization
same safety policy
```

## Module completion outcome
Module 1 ke end tak learner ko explain kar pana chahiye:

- API kya hai
- SDK kya hai
- API key kya hai
- local vs hosted model difference
- response object kya hota hai
- token/context/cost relationship
- structured output kyun chahiye
- tool calling me model aur host ka role
- agent loop kya hota hai
- evidence grounding kyun zaroori hai
- no evidence → no forced RCA

## Practical rule
Har lesson ke saath `PRACTICAL-ROADMAP.md` follow karo. V10 par direct jump mat karo.

## Why next lesson?
Ab roadmap clear hai. Next lesson me sabse basic confusion solve karenge: **ChatGPT UI aur API same model family use kar sakte hain, lekin software architecture completely different hota hai.**
