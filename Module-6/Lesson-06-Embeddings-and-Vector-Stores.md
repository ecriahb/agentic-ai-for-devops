# 🚩 Lesson 06 — Embeddings & Vector Stores in LangChain

> **Module 4 owns embedding/vector theory. This lesson owns the LangChain integration layer: embedding wrappers, vector-store adapters, persistence hooks and retriever hand-off.**

---

## 🎯 Where This Lesson Fits

```text
Module 4
  → embeddings theory
  → vector dimensions
  → similarity metrics
  → vector DB/index fundamentals

Module 6 L06
  → LangChain embedding interfaces
  → vector-store integrations
  → document/metadata association
  → retriever hand-off

Module 6 L07
  → Retriever abstraction + RAG chain composition
```

Do not re-teach Module 4 mathematics here; use it as prerequisite knowledge.

---

# PART 1 — Embedding Interface in LangChain

An embedding integration gives the application a standard way to encode:

```text
Document text → vector
Query text    → vector
```

Example local model wrapper:

```python
from langchain_huggingface import HuggingFaceEmbeddings

embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/all-MiniLM-L6-v2"
)

query_vector = embeddings.embed_query(
    "AKS subnet connectivity issue"
)

print(len(query_vector))
```

The vector dimension depends on the selected embedding model.

---

# PART 2 — `embed_documents()` vs `embed_query()`

For ingestion:

```python
vectors = embeddings.embed_documents([
    "AKS subnet NSG requirements",
    "Terraform state locking"
])
```

For query time:

```python
query_vector = embeddings.embed_query(
    "Why did AKS networking fail?"
)
```

Mental model:

```text
Index time → embed_documents()
Query time → embed_query()
```

The document and query representations must remain compatible under the selected embedding model/configuration.

---

# PART 3 — Vector Store Integration

A vector-store integration connects LangChain `Document` objects and embeddings to a searchable vector backend.

Conceptually:

```text
Document objects
      ↓
Embedding wrapper
      ↓
Vector store
      ↓
Similarity search
```

The framework adapter is not the vector mathematics itself.

---

# PART 4 — FAISS Integration

```python
from langchain_community.vectorstores import FAISS

vectorstore = FAISS.from_documents(
    chunks,
    embeddings,
)

results = vectorstore.similarity_search(
    "AKS deployment failed after NSG change",
    k=3,
)
```

The useful abstraction is that returned items remain application-level documents rather than bare integer vector positions.

Inspect:

```python
for doc in results:
    print(doc.page_content)
    print(doc.metadata)
```

---

# PART 5 — Chroma Integration Mental Model

A higher-level store can manage:

```text
IDs
Documents
Metadata
Embeddings
Query
Persistence/configuration
```

Example conceptual flow:

```text
Document chunks
      ↓
Chroma integration
      ↓
Collection
      ↓
Search
      ↓
Document results
```

Store-specific behavior—especially filtering, persistence and deployment—is still implementation-specific.

---

# PART 6 — Store Abstraction vs Backend Behavior

LangChain can provide a common interface, but this does not mean every vector store behaves identically.

Verify backend-specific semantics for:

```text
filter syntax
score semantics
persistence
updates/deletes
index type
scaling
consistency
```

Abstraction reduces integration friction; it does not erase backend differences.

---

# PART 7 — Same Embedding Contract

If documents were indexed with:

```text
sentence-transformers/all-MiniLM-L6-v2
```

then query embedding must use a compatible representation strategy.

Store operational metadata such as:

```text
embedding_model
embedding_version
dimension
index_version
```

When the embedding strategy changes, treat the existing index as a lifecycle decision rather than blindly mixing vectors from incompatible representations.

Detailed embedding theory remains in Module 4 Lesson 02/03.

---

# PART 8 — Metadata Association

A vector result is useful only when application context survives:

```text
vector
 ↕
Document
 ↕
metadata
```

Example:

```python
for doc in results:
    print(doc.metadata.get("source"))
```

Metadata design/filtering theory remains in Module 4 Lesson 09. Here the goal is verifying that the integration preserves it.

---

# PART 9 — Persistence and Lifecycle

A learning demo may rebuild an index every run.

Production asks:

```text
Where is it persisted?
How are updates applied?
How are deletions handled?
How is index version tracked?
How is rollback performed?
```

LangChain does not automatically answer those architecture questions; the backend/deployment design does.

---

# PART 10 — Direct Search vs Retriever

Direct backend call:

```python
results = vectorstore.similarity_search(
    "AKS connectivity issue",
    k=3,
)
```

Retriever hand-off:

```python
retriever = vectorstore.as_retriever(
    search_kwargs={"k": 3}
)
```

The first is backend-facing search. The second exposes a reusable application retrieval interface.

**Lesson 07 owns the retriever abstraction.**

---

# PART 11 — DevOps Example

Knowledge chunks:

```text
aks-networking.md
terraform-networking.md
pipeline-failure.md
rollback.md
```

Query:

```text
Pods cannot reach an internal service after Terraform networking change.
```

At this lesson level we verify:

```text
chunks → embeddings → vectorstore → Document results
```

We do not redesign the retrieval strategy here; Module 5 owns advanced RAG retrieval behavior.

---

# PART 12 — Common Integration Mistakes

### Wrong model contract

Query and indexed vectors are incompatible.

### Metadata lost

Source traceability disappears.

### Store backend assumed identical

Filter/persistence behavior may differ.

### Re-indexing ignored

Model or source changes leave stale data.

### Secrets stored casually

Vectorization does not make sensitive data safe.

### `k` treated as truth

`k=3` is a retrieval request, not a quality guarantee.

---

# PART 13 — Interview Q&A

### Q1. What does LangChain add over raw FAISS?

A document-oriented integration and standard interfaces that connect embeddings, vector stores and retrievers to the rest of the workflow.

### Q2. Does a vector-store abstraction erase backend differences?

No. Filtering, persistence, score semantics and operational behavior remain backend-specific.

### Q3. Why track embedding model/version?

Because indexed and query vectors must be compatible and model changes may require controlled re-indexing.

### Q4. What is the next abstraction after a vector store?

The retriever: a reusable application-facing `query → documents` interface.

---

# PART 14 — Practical Exercise

Take the chunks produced in Lesson 05 and:

1. instantiate a LangChain embedding wrapper,
2. build a local FAISS vector store,
3. query it with one DevOps question,
4. print returned `page_content` and metadata,
5. convert the store into a retriever.

Verify:

```text
chunk count
embedding dimension
returned sources
metadata preservation
```

---

# 🔁 Next Lesson Kyu?

Ab framework ke through vectors searchable hain.

Next:

```text
VectorStore
    ↓
Retriever
    ↓
Context formatter
    ↓
Prompt | Model | Parser
```

# 👉 Lesson 07 — Retrievers & RAG Chains
