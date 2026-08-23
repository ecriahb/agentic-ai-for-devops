# 🚩 Lesson 11 — DevOps Knowledge Base Practical

> **Ab hum actual local DevOps documents ko searchable semantic knowledge base me convert karenge.**

> **Practical boundary:** Lessons 01–10 teach the individual concepts and lifecycle. This lesson is the **first integrated practical**: load real local DevOps documents, apply the existing chunk/metadata/embedding/index rules, and observe retrieval. It does not introduce a new retrieval theory layer or RAG generation.

---

## 🎯 Lesson Goal

Is lesson me theory ko ek real practical pipeline me convert karenge:

- multiple Markdown files load karna
- document metadata attach karna
- paragraph-aware chunking
- chunks embed karna
- FAISS index build karna
- user query embed karna
- Top-K results retrieve karna
- source/score print karna
- empty docs and duplicate ingestion ke issues samajhna

Final runnable example:

```text
examples/05_devops_knowledge_base.py
```

---

# PART 1 — Sample Knowledge Base

Folder:

```text
sample_docs/
├── aks-networking.md
├── terraform-state.md
├── pipeline-failure.md
└── docker-build.md
```

Ye intentionally different DevOps domains cover karte hain so semantic ranking visible ho.

Example knowledge:

```text
AKS networking
Terraform state
Pipeline failure
Docker build
```

---

# PART 2 — Target User Experience

User asks:

```text
AKS deployment failed after subnet security rule change
```

Application should return something like:

```text
#1 aks-networking.md
Relevant AKS/NSG troubleshooting chunk

#2 pipeline-failure.md
Terraform Apply failure investigation chunk
```

Not:

```text
#1 docker-build.md
```

unless that document is genuinely semantically relevant.

---

# PART 3 — Architecture

```text
Markdown Files
      ↓
Load
      ↓
Validate
      ↓
Chunk  ← Lesson 08 rules
      ↓
Metadata  ← Lesson 09 schema/filter rules
      ↓
Sentence Embeddings
      ↓
FAISS Index  ← Lesson 07 tooling
      ↓
User Query
      ↓
Query Embedding
      ↓
Top-K Search
      ↓
Source + Chunk + Score
```

The point is integration. The detailed theory remains in the canonical lessons rather than being re-taught here.

---

# PART 4 — Step 1: Load Documents

```python
from pathlib import Path

DOC_DIR = Path("sample_docs")

files = sorted(DOC_DIR.glob("*.md"))

if not files:
    raise RuntimeError("No Markdown documents found")

for path in files:
    text = path.read_text(encoding="utf-8").strip()
    print(path.name, len(text))
```

### Why `.strip()`?

Whitespace-only files ko meaningful document treat nahi karna.

### Important validation

```text
file exists
supported extension
non-empty text
reasonable size
expected encoding
```

---

# PART 5 — Step 2: Chunk Documents

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

This is intentionally simple and inspectable.

Real production chunking may use token-aware/recursive/semantic approaches.

---

# PART 6 — Step 3: Build Chunk Records

```python
records = []

for path in files:
    text = path.read_text(encoding="utf-8").strip()

    if not text:
        continue

    for chunk_no, chunk in enumerate(chunk_by_paragraph(text)):
        records.append({
            "id": f"{path.name}::{chunk_no}",
            "source": path.name,
            "chunk_no": chunk_no,
            "text": chunk
        })
```

Why records instead of only strings?

Because retrieval ke baad hume source mapping chahiye.

---

# PART 7 — Step 4: Embed Chunks

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("all-MiniLM-L6-v2")

texts = [r["text"] for r in records]

vectors = model.encode(
    texts,
    normalize_embeddings=True
)
```

At this stage:

```text
records[0] ↔ vectors[0]
records[1] ↔ vectors[1]
```

Ordering ko preserve karna critical hai.

---

# PART 8 — Step 5: Build FAISS Index

```python
import faiss
import numpy as np

vectors = np.asarray(vectors, dtype="float32")

dimension = vectors.shape[1]
index = faiss.IndexFlatIP(dimension)
index.add(vectors)
```

Checks:

```python
print("Records:", len(records))
print("Vectors:", vectors.shape)
print("Index size:", index.ntotal)
```

Expected relationship:

```text
len(records) == vectors.shape[0] == index.ntotal
```

If not, mapping bug hai.

---

# PART 9 — Step 6: Query

```python
query = input("Ask a DevOps question: ").strip()

if not query:
    raise ValueError("Query cannot be empty")

query_vector = model.encode(
    [query],
    normalize_embeddings=True
)

query_vector = np.asarray(query_vector, dtype="float32")
```

---

# PART 10 — Step 7: Search

```python
k = min(3, len(records))

scores, indices = index.search(query_vector, k)

for rank, (score, idx) in enumerate(
    zip(scores[0], indices[0]),
    start=1
):
    record = records[idx]

    print(f"\n#{rank}")
    print("Score:", round(float(score), 4))
    print("Source:", record["source"])
    print("Chunk:", record["chunk_no"])
    print(record["text"])
```

### Why `min(3, len(records))`?

If only 2 chunks hain aur `k=3` blindly diya, unnecessary edge behavior aa sakta hai. Defensive code better hai.

---

# PART 11 — Example Test Queries

Run these one by one:

```text
AKS pods cannot connect after NSG rule change
```

```text
Terraform apply is blocked because state is locked
```

```text
Docker build failed because runner disk is full
```

```text
Pipeline failed during Terraform Apply
```

Observe:

- source ranking
- score differences
- duplicate/overlapping chunks
- irrelevant result positions

---

# PART 12 — Retrieval Is Not RCA

Very important:

Query:

```text
Production AKS deployment failed
```

Retrieved runbook says:

```text
Check NSG rules
```

This does **not** prove NSG is root cause.

Correct mental model:

```text
Retrieval result = relevant knowledge candidate
Live evidence     = incident truth
```

Module 1 ka evidence-grounding principle yahan bhi apply hota hai.

---

# PART 13 — Failure Handling

### No documents

```python
if not records:
    raise RuntimeError("No usable knowledge chunks found")
```

### Empty query

Reject before model call.

### Embedding/index mismatch

Validate dimensions.

### Unsupported/binary files

Skip or use dedicated parser.

### Sensitive docs

Do not ingest secrets, credentials, tokens or data user is not authorized to retrieve.

---

# PART 14 — Evaluation Sheet

Create a small manual test set:

| Query | Expected Top Source |
|---|---|
| pods blocked after NSG change | aks-networking.md |
| terraform state locked | terraform-state.md |
| apply failed in pipeline | pipeline-failure.md |
| docker build disk full | docker-build.md |

Run and record whether expected source appears in Top-1/Top-3.

This is first retrieval evaluation.

---

# PART 15 — Production Improvements

Current lab:

```text
local Markdown
simple chunking
SentenceTransformer
FAISS flat index
CLI query
```

Future production:

```text
multiple document loaders
content hash/change detection
persistent index
metadata filters
authorization
hybrid search
reranking
retrieval evaluation
observability
managed vector store
```

---

# PART 16 — Interview Corner

**Q: How would you build a searchable DevOps knowledge base?**  
Load trusted sources, validate and chunk them, attach metadata, embed chunks, index vectors, embed user queries, retrieve Top-K relevant chunks and return traceable source information.

**Q: Why preserve record-to-vector ordering?**  
FAISS returns vector indices; application needs a deterministic mapping back to source text and metadata.

**Q: Why isn't retrieved knowledge automatically root cause evidence?**  
Because semantic relevance does not prove that the retrieved scenario occurred in the current incident.

---

# PART 17 — Revision

```text
Files
 ↓
Chunks
 ↓
Records
 ↓
Embeddings
 ↓
FAISS
 ↓
Query
 ↓
Top-K
 ↓
Traceable DevOps Knowledge
```

---

# PART 18 — Homework

1. Sample knowledge base run karo.
2. 4 test queries ka Top-3 result note karo.
3. Ek fifth document khud add karo.
4. Re-run and compare ranking.
5. Ek intentionally ambiguous query test karo.

---

# Next Lesson Kyu?

Ab components individually aur integrated practical me samajh aa gaye. Next lesson me isko **mini-project acceptance criteria** ke saath build/test karenge, so Module 4 ka complete artifact ready ho.

# 👉 Lesson 12 — Mini Project: Search Your Own DevOps Documents
