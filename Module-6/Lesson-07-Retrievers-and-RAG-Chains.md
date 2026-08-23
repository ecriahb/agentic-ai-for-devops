# 🚩 Lesson 07 — Retrievers & RAG Chains

> **Module 5 owns RAG behavior and retrieval strategy. This lesson owns the LangChain implementation abstraction: Retriever contracts, `Document` flow, LCEL composition and chain wiring.**

---

## 🎯 Where This Lesson Fits

```text
Module 5
  → RAG pattern
  → thresholds
  → context engineering
  → grounded prompts
  → query rewriting
  → reranking/hybrid search
  → evaluation

Module 6 L06
  → VectorStore integration

Module 6 L07
  → Retriever abstraction
  → LangChain RAG composition
  → LCEL data flow

Module 6 L08+
  → state, tools, reliability and workflow integration
```

So this lesson should answer:

> **How does LangChain wire a retrieval component into a reusable RAG chain?**

---

# PART 1 — What Is a Retriever?

A **retriever** is an application component that accepts a query and returns relevant `Document` objects according to its configured retrieval strategy.

Simple contract:

```text
query
  ↓
Retriever
  ↓
List[Document]
```

The retriever is not the vector database itself.

---

# PART 2 — VectorStore vs Retriever

```text
VectorStore
= backend/search capability

Retriever
= application-facing query → documents interface
```

A retriever may wrap:

```text
vector search
keyword search
hybrid search
metadata filtering
custom retrieval logic
```

The **retrieval strategy itself** is covered in Module 5. Here we focus on the LangChain abstraction.

---

# PART 3 — Basic LangChain Retriever

```python
retriever = vectorstore.as_retriever(
    search_kwargs={"k": 3}
)

docs = retriever.invoke(
    "AKS deployment failed after NSG change"
)

for doc in docs:
    print(doc.page_content)
    print(doc.metadata)
```

Important observation:

```text
Retriever returns Documents
```

So the downstream chain can work with a stable interface instead of knowing whether the backend is FAISS, Chroma or another store.

---

# PART 4 — `Document` Preservation

A useful RAG chain needs:

```text
page_content
metadata
source identity
```

Example:

```python
def format_docs(docs):
    return "\n\n".join(
        f"Source: {doc.metadata.get('source', 'unknown')}\n"
        f"Content: {doc.page_content}"
        for doc in docs
    )
```

Source labeling is an application concern; do not ask the model to invent document identity.

Detailed context-engineering rules remain in **Module 5 Lesson 03**.

---

# PART 5 — The RAG Chain as LCEL Composition

LangChain's Runnable/LCEL model lets us compose components.

Conceptually:

```text
Question
 ├── Retriever → Documents → Formatter → Context
 └── Question stays available
                  ↓
          PromptTemplate
                  ↓
                Model
                  ↓
              Parser
```

Example:

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

prompt = ChatPromptTemplate.from_template("""
Use the supplied context to answer the question.
If the context is insufficient, say so.

Question:
{question}

Context:
{context}
""")

chain = (
    {
        "context": retriever | format_docs,
        "question": RunnablePassthrough(),
    }
    | prompt
    | model
    | StrOutputParser()
)

answer = chain.invoke(
    "Why can AKS workloads lose connectivity after a networking change?"
)

print(answer)
```

This is the main framework concept of the lesson:

```text
small components
   ↓
explicit data flow
   ↓
reusable chain
```

---

# PART 6 — Understanding the Data Flow

Input:

```text
"Why can AKS connectivity fail?"
```

The mapping:

```text
input
 ├──────────────→ question
 │
 └→ retriever → documents → format_docs → context

question + context
        ↓
      prompt
        ↓
      model
        ↓
     parser
```

This is why composition is useful: the flow is visible rather than hidden in a monolithic function.

---

# PART 7 — What This Lesson Does NOT Re-Teach

Do not duplicate Module 5 here:

```text
Top-K theory
threshold tuning
no-context policy design
query rewriting
multi-query
reranking
hybrid search
grounded prompt methodology
citation evaluation
RAG evaluation
```

Instead:

```text
Module 5 = what the RAG system should do
Module 6 = how LangChain components compose to do it
```

A LangChain chain should implement the policies already learned—not redefine them.

---

# PART 8 — Quality Gate Placement

A real application can insert a deterministic gate between retrieval and generation:

```text
Retriever
   ↓
Application quality gate
   ↓
Formatter
   ↓
Prompt
   ↓
Model
```

For example:

```python
if not docs:
    return "INSUFFICIENT_CONTEXT"
```

More advanced threshold/ACL/reranking logic belongs to the retrieval policy layer taught in Module 5, even if implemented inside a LangChain retriever or custom Runnable.

---

# PART 9 — DevOps Example

Question:

```text
AKS pods cannot reach an internal service after a Terraform change.
```

LangChain flow:

```text
Question
   ↓
Retriever
   ↓
Document[]
   ↓
format_docs()
   ↓
PromptTemplate
   ↓
Chat model
   ↓
Parser
```

The framework does not decide that Terraform is the root cause. It only orchestrates the application flow.

---

# PART 10 — Reference vs Current Evidence

If the application has both:

```text
reference runbooks
current incident evidence
```

pass them to the model as separate fields rather than one anonymous string:

```python
{
    "reference": reference_context,
    "evidence": current_evidence,
    "question": question,
}
```

The trust-policy design is Module 5 territory; LangChain's job here is to carry those fields through the chain.

---

# PART 11 — Debugging a RAG Chain

Inspect each boundary:

```text
Retriever output
   ↓
Formatter output
   ↓
Prompt input
   ↓
Model output
   ↓
Parser output
```

Useful debug questions:

```text
Did retriever return expected Documents?
Did metadata survive?
Did formatter include the right text?
Did prompt receive both question and context?
Did model return expected format?
Did parser fail?
```

This is an orchestration debugging skill—not another retrieval theory lesson.

---

# PART 12 — Error Boundaries

Do not collapse all failures into one exception.

Typical categories:

```text
RETRIEVAL_FAILED
CONTEXT_FORMAT_FAILED
MODEL_FAILED
PARSER_FAILED
VALIDATION_FAILED
```

This prepares the application for the dedicated error/retry lesson later in the module.

---

# PART 13 — Interview Q&A

### Q1. What is a retriever in LangChain?

An application-facing component that accepts a query and returns relevant `Document` objects.

### Q2. Is a retriever the same as a vector database?

No. A vector store provides backend search capability; a retriever is a higher-level retrieval interface and strategy.

### Q3. What does LCEL add here?

Composable Runnable components with explicit input/output data flow.

### Q4. Why keep retrieval policy outside generic chain syntax?

Because retrieval thresholds, freshness, authorization, hybrid search and evaluation are application/domain policies; the framework should implement them rather than silently define them.

### Q5. Does using a retriever make RAG grounded automatically?

No. Correct retrieval, context construction, grounding rules and validation are still required.

---

# PART 14 — Practical Exercise

Using Lesson 06's vector store:

1. convert it to a retriever,
2. invoke it with three DevOps queries,
3. inspect returned `Document` metadata,
4. build a `format_docs()` function,
5. compose `retriever → formatter → prompt → model → parser`,
6. log each boundary once for debugging.

Keep the implementation read-only.

---

# PART 15 — Homework

Build two versions of the same workflow:

```text
Version A
manual Python functions

Version B
LangChain Runnable composition
```

Compare:

```text
clarity
reusability
testability
error boundaries
debugging effort
```

Do not compare by line count alone.

---

# 🔁 Next Lesson Kyu?

RAG chain ab reusable hai. Ab multi-turn workflows ke liye ek critical distinction chahiye:

```text
Conversation Memory
vs
Workflow State
vs
Evidence
```

# 👉 Lesson 08 — Memory vs Application State
