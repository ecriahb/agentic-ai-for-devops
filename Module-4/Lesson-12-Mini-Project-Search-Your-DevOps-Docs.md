# 🚩 Lesson 12 — Mini Project: Search Your Own DevOps Documents

> **Module 4 ka final project: local DevOps documents ko traceable semantic search application me convert karna.**

> **Project boundary:** Lesson 11 is the guided first integrated practical. Lesson 12 is the **acceptance/capstone pass**: repeat the full pipeline, validate retrieval quality, run failure/security checks, and document the production upgrade path. It still stops at retrieval and does not generate an LLM/RAG answer; that belongs to Module 5.

---

## 🎯 Final Project Outcome

User query:

```text
AKS deployment Terraform networking change ke baad fail ho raha hai
```

Application:

```text
User Query
   ↓
Embedding
   ↓
Search Local DevOps Knowledge Index
   ↓
Top Relevant Chunks
   ↓
Rank + Source + Chunk + Score + Text
```

Important:

**Is project ka output LLM-generated RCA nahi hai.**

Module 4 ka goal hai:

```text
Question → Relevant Knowledge
```

Module 5 me:

```text
Question → Retrieval → LLM → Grounded Answer
```

---

# PART 1 — Skills Combined

Is mini-project me hum combine kar rahe hain:

```text
External Knowledge
Embeddings
Vectors
Similarity Search
Cosine / Inner Product intuition
Vector Index
Chunking
Metadata
Indexing
Top-K Retrieval
Traceability
Evaluation
```

---

# PART 2 — Project Folder

```text
Module-4/examples/
│
├── 05_devops_knowledge_base.py
├── requirements.txt
│
└── sample_docs/
    ├── aks-networking.md
    ├── terraform-state.md
    ├── pipeline-failure.md
    └── docker-build.md
```

You can later replace sample docs with your own sanitized `.md` runbooks.

---

# PART 3 — Setup

From `Module-4/examples`:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

If execution policy blocks activation, use your already configured Python environment or follow your organization's approved PowerShell policy process.

Verify:

```powershell
python --version
pip show sentence-transformers
pip show faiss-cpu
```

---

# PART 4 — Stage 1: Source Document Preparation

Each source should be useful, non-empty and safe to index.

Example `aks-networking.md`:

```markdown
# AKS Networking Runbook

## Symptoms
Pods cannot reach internal services after network policy or NSG change.

## Checks
Validate DNS, subnet NSG rules, UDRs and firewall routes.

## Resolution
Restore the required approved rule, validate connectivity and redeploy only after checks pass.
```

### Do NOT put in demo docs

```text
API keys
passwords
connection strings
private certificates
access tokens
customer PII
```

---

# PART 5 — Stage 2: Document Loader

```python
from pathlib import Path

DOC_DIR = Path("sample_docs")
files = sorted(DOC_DIR.glob("*.md"))

if not files:
    raise RuntimeError("No Markdown documents found")
```

Why `sorted()`?

Deterministic ordering makes local debugging/mapping easier.

---

# PART 6 — Stage 3: Chunking

```python
def chunk_by_paragraph(text, max_chars=600):
    paragraphs = [p.strip() for p in text.split("\n\n") if p.strip()]
    chunks = []
    current = ""

    for paragraph in paragraphs:
        candidate = f"{current}\n\n{paragraph}".strip()

        if current and len(candidate) > max_chars:
            chunks.append(current)
            current = paragraph
        else:
            current = candidate

    if current:
        chunks.append(current)

    return chunks
```

### Why this instead of blind 600-character slicing?

Paragraph boundaries preserve a little more natural structure.

Still remember:

```text
Simple demo chunking ≠ universally optimal production chunking
```

---

# PART 7 — Stage 4: Create Traceable Records

```python
records = []

for path in files:
    text = path.read_text(encoding="utf-8").strip()

    if not text:
        print(f"Skipping empty file: {path.name}")
        continue

    chunks = chunk_by_paragraph(text)

    for chunk_no, chunk in enumerate(chunks):
        records.append({
            "id": f"{path.name}::{chunk_no}",
            "source": path.name,
            "chunk_no": chunk_no,
            "text": chunk
        })
```

Output mental model:

```text
records[0]
 ├─ id
 ├─ source
 ├─ chunk_no
 └─ text
```

Later metadata expand kar sakte ho:

```text
service
environment
version
status
team
updated_at
```

---

# PART 8 — Stage 5: Embeddings

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("all-MiniLM-L6-v2")

texts = [record["text"] for record in records]

vectors = model.encode(
    texts,
    normalize_embeddings=True
)
```

Check:

```python
print("Chunks:", len(records))
print("Vector matrix shape:", vectors.shape)
```

Expected concept:

```text
N chunks → (N, embedding_dimension)
```

---

# PART 9 — Stage 6: Build FAISS Index

```python
import faiss
import numpy as np

vectors = np.asarray(vectors, dtype="float32")

dimension = vectors.shape[1]
index = faiss.IndexFlatIP(dimension)
index.add(vectors)

print("Index vectors:", index.ntotal)
```

Validation:

```python
assert index.ntotal == len(records)
```

This protects record/vector mapping assumptions.

---

# PART 10 — Stage 7: User Query

```python
query = input("Ask a DevOps question: ").strip()

if not query:
    raise ValueError("Query cannot be empty")
```

Example queries:

```text
AKS networking failure after NSG change
```

```text
Terraform state is locked
```

```text
Pipeline failed during Terraform Apply
```

```text
Docker build runner has no disk space
```

---

# PART 11 — Stage 8: Query Embedding

```python
query_vector = model.encode(
    [query],
    normalize_embeddings=True
)

query_vector = np.asarray(query_vector, dtype="float32")
```

Dimension validation:

```python
if query_vector.shape[1] != dimension:
    raise RuntimeError("Embedding dimension mismatch")
```

---

# PART 12 — Stage 9: Top-K Search

```python
k = min(3, len(records))

scores, indices = index.search(query_vector, k)
```

FAISS gives index positions.

We map them back:

```python
for rank, (score, idx) in enumerate(zip(scores[0], indices[0]), start=1):
    record = records[idx]

    print(f"\nRank: {rank}")
    print(f"Score: {float(score):.4f}")
    print(f"Source: {record['source']}")
    print(f"Chunk: {record['chunk_no']}")
    print(record["text"])
```

---

# PART 13 — Expected Output Pattern

Query:

```text
pods lost connectivity after NSG rule change
```

Expected pattern (scores are illustrative, not fixed):

```text
Rank: 1
Score: ...
Source: aks-networking.md
Chunk: ...
<AKS network troubleshooting text>

Rank: 2
Score: ...
Source: pipeline-failure.md
Chunk: ...
<related deployment troubleshooting text>
```

Do not hard-code expected numeric score because embedding/library versions and text change results.

---

# PART 14 — Acceptance Criteria

Project tab complete maana jayega jab:

- [ ] local `.md` files discover ho rahe hain
- [ ] empty files safely handle ho rahe hain
- [ ] chunks created and identifiable hain
- [ ] each chunk source traceable hai
- [ ] embeddings successfully create ho rahe hain
- [ ] vector dimension consistent hai
- [ ] FAISS index me all expected vectors add hain
- [ ] arbitrary user query accept hoti hai
- [ ] query embedding banti hai
- [ ] Top-K results milte hain
- [ ] result me rank/source/chunk/score/text show hota hai
- [ ] no secrets sample knowledge base me stored nahi hain
- [ ] at least 4 test queries manually evaluated hain

---

# PART 15 — Retrieval Evaluation

Create test set:

```python
TESTS = [
    ("pods blocked after NSG change", "aks-networking.md"),
    ("terraform state locked", "terraform-state.md"),
    ("apply failed in CI pipeline", "pipeline-failure.md"),
    ("docker build disk full", "docker-build.md")
]
```

Manual metric:

```text
Top-1 accuracy
Top-3 hit rate
```

Example:

```text
4 queries
3 expected docs appeared at rank 1
4 expected docs appeared in top 3

Top-1 = 75%
Top-3 hit rate = 100%
```

This introduces evaluation mindset before RAG.

---

# PART 16 — Failure Scenarios to Test

## Test A — Empty query

Expected: clear validation error.

## Test B — Empty docs folder

Expected: clear startup/indexing error.

## Test C — Add unrelated doc

Check whether correct DevOps result still ranks high.

## Test D — Duplicate content

Observe duplicate Top-K results.

## Test E — Change embedding model

Understand why corpus needs re-indexing.

## Test F — Very ambiguous query

```text
Deployment broken
```

Observe weak/mixed retrieval. This teaches query specificity and retrieval limitations.

---

# PART 17 — Project V1 → V10 Evolution

Module 4 practical progression should be remembered like this:

```text
V1  → First embedding
V2  → Compare sentences
V3  → Multiple DevOps incidents
V4  → Manual semantic ranking
V5  → Cosine similarity understanding
V6  → Chroma collection search
V7  → FAISS index search
V8  → Chunk real runbooks
V9  → Metadata + traceability
V10 → Search your DevOps knowledge base
```

This makes the final system understandable instead of black-box.

---

# PART 18 — Production Upgrade Roadmap

Current project:

```text
Local Markdown
Paragraph chunking
SentenceTransformer
Flat FAISS index
CLI
```

Future production capabilities:

```text
Git/Wiki/SharePoint/document connectors
incremental indexing
content hashing
token/semantic chunking
persistent vector store
hybrid BM25 + vector search
metadata filters
identity-aware authorization
reranking
retrieval evaluation dataset
observability
freshness monitoring
backup/recovery
multi-tenant isolation
```

---

# PART 19 — Security Checklist

Before indexing organizational documents:

```text
Who owns the source?
Who is allowed to retrieve it?
Does it contain secrets?
Does it contain PII?
Is tenant isolation required?
Can old/deleted content remain indexed?
Are logs exposing retrieved sensitive text?
```

Remember:

```text
Vectorization does NOT make sensitive data safe.
```

Embedding and source content require appropriate protection.

---

# PART 20 — Interview Corner

**Q1: Explain an end-to-end semantic document search pipeline.**  
Load and validate documents, split them into traceable chunks, generate embeddings, store/index the vectors, embed each query, run nearest-neighbor search, and map top results back to source text and metadata.

**Q2: How would you evaluate retrieval?**  
Build representative queries with expected relevant documents/chunks and measure metrics such as Top-K hit rate/recall plus qualitative relevance.

**Q3: Why does changing the embedding model require care?**  
Existing document vectors may no longer be compatible with query vectors, so the corpus generally needs controlled re-indexing.

**Q4: Is semantic search itself RAG?**  
No. Retrieval is one part. RAG combines retrieval with generative model context/answer generation.

**Q5: What is a key security risk in enterprise vector search?**  
Unauthorized retrieval of sensitive indexed content; access control must be enforced by trusted application/platform logic.

---

# PART 21 — Final Revision Cheat Sheet

```text
WHY
LLM does not automatically know private/live knowledge

WHAT
Embeddings = numeric semantic representations

COMPARE
Similarity / distance metrics

STORE
Vector index / vector database

PREPARE
Chunking + metadata

INDEX
Docs → chunks → embeddings → index

QUERY
Question → embedding → search → Top-K

OUTPUT
Relevant text + source + score
```

---

# PART 22 — Final Homework

1. Run `05_devops_knowledge_base.py` successfully.
2. Add one sanitized personal DevOps note.
3. Test minimum 5 questions.
4. Record expected vs actual Top-3.
5. Write 3 cases where semantic retrieval returned an imperfect result.
6. Explain how you would improve those cases using chunking, metadata, model choice or query quality.

---

# ✅ Module 4 Completion Mental Model

```text
External Knowledge
       ↓
Documents
       ↓
Chunking
       ↓
Metadata
       ↓
Embeddings
       ↓
Vectors
       ↓
Vector Index / DB
       ↓
Query Embedding
       ↓
Similarity Search
       ↓
Top-K Retrieval
       ↓
Traceable Context
```

---

# 🚀 Why Module 5 Comes Next

Module 4 answers:

```text
Where is the relevant knowledge?
```

Module 5 will answer:

```text
How do we give that knowledge to an LLM
so it generates a grounded answer?
```

Next:

# 👉 Module 5 — Retrieval-Augmented Generation (RAG)
