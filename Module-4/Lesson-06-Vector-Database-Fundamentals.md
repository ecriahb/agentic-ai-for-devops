# 🚩 Lesson 06 — Vector Database Fundamentals

> **Vector database ka kaam LLM banna nahi hai; uska kaam embeddings ko efficiently store, index aur search karna hai.**

---

## 🎯 Lesson Goal

Is lesson ke end tak aap samjhoge:

- vector database kya hai
- normal database aur vector database me conceptual difference
- vector index kya hai
- exact vs approximate nearest neighbor
- metadata ka role
- pre-filter vs post-filter retrieval
- persistence, updates aur re-indexing
- Chroma/FAISS ka role
- production design questions

---

# PART 1 — Why We Need It

Small demo:

```text
10 vectors → Python list → compare all
```

Real system:

```text
10,000 / 1,000,000+ chunks
       ↓
Need efficient storage + indexing + search
```

Sirf arrays maintain karna quickly operationally difficult ho jata hai.

---

# PART 2 — English Definition

> A vector database or vector-capable index stores high-dimensional vectors and supports efficient nearest-neighbor search, often together with documents, IDs and metadata.

Hinglish:

```text
Vector DB = embeddings ka searchable system
```

It may manage:

```text
ID
vector
original chunk/document
metadata
index
search
persistence
```

---

# PART 3 — Normal DB vs Vector Search

Traditional lookup:

```sql
SELECT * FROM incidents WHERE environment = 'prod';
```

Exact/filter style query.

Vector search:

```text
"AKS connectivity failed after network rule change"
        ↓ embedding
nearest semantic chunks
```

Both useful hain.

Production retrieval often combines:

```text
metadata filter + semantic vector search
```

Example:

```text
environment = prod
AND
semantic similarity to "AKS network issue"
```

Both operations solve different problems:

```text
Metadata filter  → exact constraint / eligibility
Vector search    → semantic relevance
```

---

# PART 4 — Vector Index

Without optimized index:

```text
Query vector
   ↓
compare with every stored vector
```

Index search ko accelerate karta hai.

Mental model:

```text
Vectors
  ↓
Index Structure
  ↓
Fast Candidate Search
  ↓
Nearest Results
```

Vector DB and vector index same exact thing nahi hote. Database broader lifecycle/features provide kar sakta hai; FAISS primarily vector similarity indexing/search library hai.

---

# PART 5 — Exact vs Approximate Search

## Exact Nearest Neighbor

Every relevant vector accurately compare karne ka goal.

Pros:
- deterministic/exact for chosen metric/index

Cons:
- huge datasets par expensive ho sakta hai

## Approximate Nearest Neighbor (ANN)

Speed ke liye search space intelligently reduce karta hai.

Tradeoff:

```text
Speed / scale ↑
Potential perfect recall ↓
```

Production retrieval is a tradeoff, not magic.

---

# PART 6 — Metadata and Filtering

### Metadata Definition

> **Metadata is structured information associated with a stored document/chunk/vector that can be used for filtering, organization, routing or retrieval control.**

Example:

```json
{
  "source": "aks-networking.md",
  "environment": "production",
  "service": "payment",
  "severity": "critical",
  "version": "v4"
}
```

Hinglish:

```text
Vector metadata se meaning nahi batata.
Metadata structured facts/attributes batata hai.
```

Mental model:

```text
Metadata
   ↓
Exact constraint

Vector similarity
   ↓
Semantic relevance
```

## Pre-filter vs Post-filter

When a vector retrieval system supports metadata filtering, an important design question is **when the metadata constraint is applied relative to vector candidate search**.

### Pre-filter

Conceptually, the metadata constraint is applied **before the vector nearest-neighbor search**:

```text
All stored records
       ↓
Metadata filter
(environment=production
 AND service=payment)
       ↓
Eligible candidates
       ↓
Vector similarity search
       ↓
Top-K
```

Hinglish intuition:

> **Pehle decide karo kaunse documents allowed hain, phir unmein semantic search karo.**

Example:

```text
10,000 documents
        ↓
production + payment
        ↓
800 eligible documents
        ↓
vector search
        ↓
Top-K relevant documents
```

### Post-filter

Conceptually, vector candidates are selected first and the metadata constraint is applied **afterward**:

```text
All stored records
       ↓
Vector search
       ↓
Top candidate set
       ↓
Metadata filter
       ↓
Final results
```

Hinglish intuition:

> **Pehle semantic candidates nikalo, phir un candidates ko metadata condition se filter karo.**

### Why does this matter?

Suppose:

```text
Requested Top-K = 5
```

A post-filter implementation may find five highly similar documents, but after applying:

```text
environment = production
```

some or all of them may be removed. The final result count can therefore be smaller than the requested K unless the system retrieves additional candidates or uses another strategy.

This is one reason filtering semantics matter when designing production retrieval.

### Important: This is not a universal rule for every vector database

**Pre-filter vs post-filter describes retrieval strategies; it should not be assumed that every product implements filtering in exactly one of these ways.** Actual behavior depends on the database, index type, query engine and configuration.

So in production, always verify the library/database's official documentation rather than assuming that a metadata filter automatically means pre-filtering.

### DevOps Example

Suppose the knowledge base contains:

```text
production / payment / critical
staging    / payment / high
production / orders  / high
production / payment / warning
```

Query:

```text
"Which NSG rule is blocking production payment traffic?"
```

A useful retrieval constraint is:

```text
environment = production
service = payment
```

Then semantic similarity ranks the eligible documents.

This gives us the key mental model:

```text
Hard constraint
      +
Semantic relevance
      ↓
More controlled retrieval
```

### Pre-filter vs Post-filter — Quick Comparison

| Aspect | Pre-filter | Post-filter |
|---|---|---|
| Metadata condition | Applied before candidate search | Applied after candidate search |
| Candidate pool | Restricted first | Starts broader |
| Possible final results | More predictable with respect to filter | Can be fewer than requested K |
| Main concern | How efficiently the filtered search is supported | Need enough candidates after filtering |
| Universal behavior? | ❌ No | ❌ No |

> **Interview point:** Don't say "pre-filter is always better." The correct answer is that filtering strategy affects recall, latency and result count, and the actual semantics depend on the vector system and index implementation.

---

# PART 7 — What Gets Stored?

Example chunk record:

```json
{
  "id": "aks-runbook-03",
  "text": "Validate outbound NSG rules for AKS subnet...",
  "metadata": {
    "source": "aks-networking.md",
    "environment": "prod",
    "version": "v4"
  },
  "embedding": [0.12, -0.08, 0.44]
}
```

Real vector has many more dimensions.

---

# PART 8 — Ingestion vs Query Path

## Ingestion

```text
Document
 ↓
Chunk
 ↓
Embedding
 ↓
Vector DB / Index
```

## Query

```text
User query
 ↓
Query embedding
 ↓
Vector search
 ↓
Top-K chunks
```

Do not re-embed entire document collection on every query.

---

# PART 9 — Persistence

Prototype:

```text
process ends → index disappears
```

Persistent system:

```text
Index stored on disk/service
    ↓
application restart
    ↓
index reused
```

But persistence introduces lifecycle questions:

- document changed?
- chunk deleted?
- model changed?
- duplicate ingestion?
- old version stale?

---

# PART 10 — Chroma vs FAISS Mental Model

```text
Chroma
→ developer-friendly collection/store abstraction
→ documents + metadata + embeddings + query workflow

FAISS
→ high-performance vector indexing/search library
→ you usually manage document text/metadata mapping yourself
```

Neither should be treated as the universally best production option. Tool choice depends on scale, deployment, operations, security, filtering and team requirements.

---

# PART 11 — DevOps Knowledge Base Example

```text
AKS runbooks
Terraform docs
pipeline postmortems
Azure networking SOPs
       ↓
chunks + embeddings
       ↓
vector store
       ↓
Query: "pods lost access after NSG change"
       ↓
Top relevant operational knowledge
```

---

# PART 12 — Common Mistakes

1. Vector DB ko LLM memory samajhna.
2. Source text/metadata mapping lose kar dena.
3. Duplicate docs repeatedly ingest karna.
4. Model change ke baad incompatible old vectors use karna.
5. Authorization ko metadata filter se replace karna.
6. Index freshness monitor na karna.
7. Filtering behavior ko database documentation verify kiye bina assume karna.

---

# PART 13 — Production Design Checklist

Before choosing a vector solution, ask:

```text
How many vectors?
Required latency?
Metadata filtering?
Pre-filter or post-filter semantics?
Persistence?
Backups?
Multi-tenancy?
Access control?
Encryption?
Update/delete frequency?
Hybrid search needed?
Managed vs self-hosted?
```

---

# PART 14 — Interview Corner

**Q: What is a vector database?**  
A system designed to store/index high-dimensional vectors and retrieve nearest items efficiently, often with metadata and source content.

**Q: What is ANN?**  
Approximate nearest-neighbor search trades some exactness/recall for faster scalable retrieval.

**Q: Is FAISS a full database?**  
It is primarily a vector similarity search/indexing library, so broader persistence/metadata/application lifecycle may need separate handling.

**Q: What is pre-filtering?**  
Conceptually, applying metadata constraints before vector candidate search so only eligible records participate in retrieval.

**Q: What is post-filtering?**  
Conceptually, applying metadata constraints after vector candidate selection; this can reduce the number of final results available after filtering.

**Q: Is pre-filtering always better than post-filtering?**  
No. The tradeoffs depend on the database, index, filtering implementation, latency requirements and desired recall/result count.

---

# PART 15 — Revision

```text
Chunks
 ↓
Embeddings
 ↓
Vector Store / Index
 ↓
Metadata Constraints
 ↓
Pre-filter / Post-filter semantics
 ↓
Query Embedding
 ↓
Nearest Neighbor Search
 ↓
Top-K
```

Remember:

```text
Metadata filtering = exact constraint
Vector similarity  = semantic relevance
Pre/Post filtering = when the constraint participates in retrieval
```

---

# PART 16 — Homework

1. Normal SQL filter aur vector similarity search ka difference explain karo.
2. Exact vs approximate nearest neighbor ka tradeoff likho.
3. Vector record me text + metadata preserve kyu karna chahiye?
4. Pre-filter aur post-filter ka flow diagram banao.
5. Explain why a post-filtered Top-K search can return fewer than K final documents.

---

# Next Lesson Kyu?

Concept clear hai. Ab actual local tools use karenge.

# 👉 Lesson 07 — ChromaDB & FAISS Basics
