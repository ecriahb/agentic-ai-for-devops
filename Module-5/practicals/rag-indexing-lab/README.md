# RAG Indexing + Query Pipeline Lab

This lab mirrors Lesson 02 — RAG Architecture & Data Flow.

## Pipelines

### Indexing
Source Documents → Loading → Cleaning → Chunking → Metadata → Embedding → Vector Index

### Query
Query Validation → Query Embedding → Candidate Retrieval → Quality Gate → No-Context Check → Context Builder → Grounded Prompt → LLM Generation → Validation

## Current practical model

- Embeddings: `all-MiniLM-L6-v2`
- Vector search: FAISS `IndexFlatIP`
- Local LLM: Ollama `qwen3:0.6b`

## Notes

The query practical demonstrates no-context handling, source labels, grounded prompting, and validation.
