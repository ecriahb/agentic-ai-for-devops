# 🚩 Lesson 05 — Document Loaders & Text Splitters

> **Module 4 already owns chunking theory and retrieval implications. This lesson owns the LangChain implementation layer: loaders, `Document` objects, metadata propagation and splitter APIs.**

---

## 🎯 Where This Lesson Fits

```text
Module 4
  → Why chunking matters
  → chunking strategies
  → overlap trade-offs

Module 6 L05
  → How LangChain loads documents
  → How LangChain represents documents
  → How splitters transform them
  → How metadata survives the pipeline

Module 6 L06
  → Embedding + vector-store integrations
```

**Canonical boundaries:**

- Chunking theory → **Module 4 Lesson 08**
- Metadata design/filtering → **Module 4 Lesson 09**
- Indexing lifecycle → **Module 4 Lesson 10**
- RAG context construction → **Module 5 Lesson 03**

---

# PART 1 — What a LangChain Loader Does

A **document loader** converts an external source into LangChain `Document` objects.

Mental model:

```text
Source
  ↓
Loader
  ↓
Document(page_content + metadata)
```

The loader is not the embedding model, vector store or retriever.

---

# PART 2 — Loader vs Parser

A loader usually coordinates source access and document creation; parsing/extraction may happen inside or underneath that loader depending on the source type.

For example:

```text
PDF
 ↓
PDF extraction/parser
 ↓
Document objects
```

A successful file read does not guarantee perfect extraction. Always inspect representative documents before indexing them.

---

# PART 3 — `Document` Contract

Typical shape:

```python
from langchain_core.documents import Document

Document(
    page_content="AKS subnet requires approved NSG rules.",
    metadata={
        "source": "aks-networking.md"
    }
)
```

Think of it as:

```text
Document
├── page_content → retrieval text
└── metadata     → source/provenance attributes
```

The exact metadata fields depend on the loader and application.

---

# PART 4 — First Practical: `TextLoader`

```python
from langchain_community.document_loaders import TextLoader

loader = TextLoader(
    "sample_docs/aks-networking.md",
    encoding="utf-8",
)

docs = loader.load()

print(type(docs))
print(len(docs))

for doc in docs:
    print(doc.page_content[:200])
    print(doc.metadata)
```

Observe:

```text
list[Document]
text
metadata
```

The learning goal here is the API contract, not re-learning document ingestion theory.

---

# PART 5 — Metadata Propagation

Application metadata can be added before splitting:

```python
for doc in docs:
    doc.metadata.update({
        "team": "platform",
        "environment": "production",
        "status": "approved",
    })
```

After splitting, downstream chunks should retain the relevant metadata.

Verify it rather than assuming it:

```python
for chunk in chunks[:3]:
    print(chunk.metadata)
```

**Do not let the LLM invent security or authorization metadata.**

---

# PART 6 — What a Text Splitter Does

The splitter converts `Document` objects into smaller `Document` objects.

```text
Document
   ↓
Splitter
   ↓
Chunk Documents
```

The detailed question of *why* chunking affects retrieval quality is already covered in Module 4 Lesson 08.

Here we focus on **LangChain splitter behavior and configuration**.

---

# PART 7 — Recursive Character Splitter

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=80,
)

chunks = splitter.split_documents(docs)

print("Documents:", len(docs))
print("Chunks:", len(chunks))

for i, chunk in enumerate(chunks[:3], start=1):
    print(f"\nChunk {i}")
    print(chunk.page_content)
    print(chunk.metadata)
```

The important API behavior:

```text
split_documents(...)
→ returns Document objects
→ metadata remains attached
```

---

# PART 8 — Choosing Splitter APIs

LangChain exposes multiple splitter approaches.

For this course, understand the implementation categories:

```text
Character-based
Recursive character
Token-aware
Structure-aware / format-aware
```

Do not memorize a single “best” splitter. Module 4 owns the retrieval trade-offs and evaluation mindset.

---

# PART 9 — DevOps Mapping

Different source types may use different ingestion strategies:

```text
Markdown runbook
→ structure-aware / recursive splitter

Terraform documentation
→ section/resource-aware strategy

Pipeline logs
→ event/time-window grouping

PDF architecture document
→ parser + structure-preserving splitter
```

The application decides the loader/splitter combination based on source characteristics.

---

# PART 10 — Stable IDs and Metadata

LangChain's `Document` object is not automatically your complete production identity system.

Application should establish deterministic identity such as:

```text
source + version + section + chunk_number
```

Example:

```text
aks-networking:v4:checks:004
```

Stable IDs remain important for:

```text
update
delete
deduplication
citations
auditing
```

The lifecycle theory belongs to Module 4; this lesson shows where those values fit into LangChain objects.

---

# PART 11 — Ingestion Validation

Before passing chunks to embeddings/vector stores, validate:

```text
page_content not empty
metadata contains required source identity
expected source type
approved status/classification
no accidental secrets
```

Example:

```python
valid_chunks = [
    chunk
    for chunk in chunks
    if chunk.page_content.strip()
    and chunk.metadata.get("source")
]
```

---

# PART 12 — Common Implementation Failures

### Loader import/version mismatch

Use the package documented for your installed LangChain stack.

### Empty extraction

Inspect `page_content` before indexing.

### Metadata lost

Check the resulting `Document` objects after splitting.

### Unexpected chunk count

Inspect source structure and splitter settings rather than assuming the library failed.

### Duplicate ingestion

Use deterministic IDs/application-level ingestion control.

---

# PART 13 — Security Boundary

Before indexing organization data:

```text
source allowlist
 ↓
classification / secret scanning
 ↓
metadata + access scope
 ↓
loader
 ↓
splitter
 ↓
index
```

Security ownership remains with the host application/platform. LangChain does not automatically turn arbitrary documents into trusted content.

---

# PART 14 — Interview Q&A

### Q1. What does a LangChain document loader return?

Standardized `Document` objects containing text plus metadata.

### Q2. Why inspect loaded documents before splitting?

A successful load can still contain bad extraction, missing text or incorrect metadata.

### Q3. What is `split_documents()` useful for?

It applies the configured splitter to `Document` objects while preserving document-level metadata on the produced chunks.

### Q4. Where should chunk-size strategy be learned in this course?

Module 4 Lesson 08. Module 6 Lesson 05 focuses on implementing that strategy with LangChain APIs.

### Q5. Does LangChain decide authorization?

No. Authorization must be enforced by trusted application/data-layer controls.

---

# PART 15 — Practical Exercise

Take `aks-networking.md` and build:

```text
loader
→ metadata enrichment
→ RecursiveCharacterTextSplitter
→ validation
→ stable chunk ID assignment
```

Print:

```text
chunk ID
source
text length
first 100 characters
metadata
```

Then inspect one chunk manually for boundary quality.

---

# 🔁 Next Lesson Kyu?

Ab LangChain ke paas clean, traceable `Document` chunks hain.

Next step:

```text
Document chunks
   ↓
Embedding integration
   ↓
Vector store integration
```

# 👉 Lesson 06 — Embeddings & Vector Stores in LangChain
