# 🚩 Lesson 09 — Metadata & Filtering

> **Semantic similarity batata hai “meaning close hai”; metadata batata hai “ye result kis source, environment, version aur scope ka hai.”**

> **Canonical boundary:** Lesson 06 introduces the retrieval concept of **pre-filter vs post-filter** inside its Metadata section. This lesson owns the **practical DevOps metadata design, filtering patterns, traceability and authorization boundary**. It does not re-teach the full pre/post-filter theory.

---

## 🎯 Lesson Goal

Is lesson ke end tak aap samjhoge:

- metadata kya hai
- chunk metadata kyu important hai
- semantic search + metadata filter ka combination
- environment/service/version/source filters
- traceability and citations
- metadata filtering vs authorization
- practical filtering patterns
- stale version problems

---

# PART 1 — Metadata Definition

**English Definition:**
> Metadata is structured information that describes a document or chunk, such as its source, environment, service, version, owner, date or access scope.

Example:

```json
{
  "source": "aks-networking.md",
  "service": "aks",
  "environment": "prod",
  "version": "v4",
  "section": "nsg-checks"
}
```

Text = knowledge. Metadata = knowledge ke baare me structured information.

---

# PART 2 — Why Similarity Alone Is Not Enough

Suppose query:

```text
How do I fix AKS network connectivity?
```

Search returns:

```text
1. dev AKS old runbook v1
2. prod AKS current runbook v4
3. retired cluster migration notes
```

All semantically similar ho sakte hain.

But you may want:

```text
environment = prod
status = current
service = aks
```

Then search scope becomes safer and more relevant.

The actual timing semantics of the filter—pre-filter or post-filter—are covered in **Lesson 06** and depend on the chosen vector system. Here we focus on how to design the metadata itself and apply the intended scope correctly.

---

# PART 3 — Filter + Vector Search

Mental model for the application-level contract:

```text
Allowed Scope
   ↓
Metadata Constraints
   ↓
Eligible Knowledge
   ↓
Semantic Vector Search
   ↓
Top-K
```

A vector database may internally combine these operations differently. The application should define the **intended eligibility constraints** clearly and then verify the actual implementation semantics in the library/database documentation.

Example intent:

```text
Find semantically similar chunks
WHERE environment = prod
AND service = aks
AND status = current
```

---

# PART 4 — Useful DevOps Metadata

Possible fields:

```text
source
service
environment
region
team
document_type
version
status
created_at
updated_at
incident_id
repository
path
section
confidentiality
```

Do not add metadata just because possible hai. Add fields that support retrieval, governance and traceability.

A useful design question for every field is:

```text
Will this field help us
retrieve,
trace,
version,
or govern
this chunk?
```

---

# PART 5 — Source Traceability

Retrieved answer candidate:

```text
Validate outbound NSG rule before redeployment.
```

Without metadata:

```text
Where did this come from? Unknown.
```

With metadata:

```text
source: aks-networking.md
section: Network Validation
version: v4
chunk_id: aks-networking::network-validation::03
```

Now later RAG systems can show source references and support claim-level traceability.

---

# PART 6 — Chroma-Style Filter Example

Conceptual example:

```python
results = collection.query(
    query_embeddings=query_embedding,
    n_results=3,
    where={"environment": "prod"}
)
```

Then only matching metadata scope ke records candidates banenge, subject to library behavior/configuration.

Multiple-field logic depends on the vector store's supported filter syntax, so current official docs verify karna important hai.

For this lesson, the important application contract is:

```text
environment = prod
service = aks
status = current
```

The exact execution semantics belong to the vector-store implementation.

---

# PART 7 — Versioning Example

Suppose same runbook:

```text
v1 → old firewall route
v2 → temporary workaround
v4 → current approved process
```

If all indexed without status/version strategy:

```text
semantic search → old solution may rank first
```

Better metadata:

```json
{
  "version": "v4",
  "status": "current"
}
```

And ingestion lifecycle should remove/deprecate stale content deliberately.

---

# PART 8 — Metadata Is NOT Authorization

Very important production principle:

```text
where={"team": "payments"}
```

is a retrieval filter.

It is **not automatically a security boundary**.

Authorization must be enforced by trusted application/system logic based on authenticated identity and permissions.

Mental model:

```text
Identity
 ↓
Authorization Policy
 ↓
Allowed corpus / tenant
 ↓
Metadata filter + vector search
```

Never trust user-provided metadata alone for access control.

---

# PART 9 — Multi-Tenant Example

Bad:

```text
User says tenant=A
→ app trusts string
→ searches tenant A
```

Better:

```text
Authenticated user
 ↓
Application resolves allowed tenant IDs
 ↓
Server-side enforced filter
 ↓
Vector search
```

This becomes critical for enterprise RAG.

---

# PART 10 — Practical Data Structure

```python
chunks = [
    {
        "text": "Validate AKS NSG rules...",
        "metadata": {
            "source": "aks-networking.md",
            "service": "aks",
            "environment": "prod",
            "status": "current",
            "version": "v4"
        }
    },
    {
        "text": "Clear Terraform state lock...",
        "metadata": {
            "source": "terraform-state.md",
            "service": "terraform",
            "environment": "prod",
            "status": "current",
            "version": "v2"
        }
    }
]

filtered = [
    c for c in chunks
    if (
        c["metadata"]["service"] == "aks"
        and c["metadata"]["environment"] == "prod"
        and c["metadata"]["status"] == "current"
    )
]

print(filtered)
```

First Python-side filtering samjho; vector DB filtering iska scalable/managed version ho sakta hai.

---

# PART 11 — Common Mistakes

1. source filename store na karna
2. unstable/meaningless IDs
3. old and current docs mix karna
4. environment metadata inconsistent (`Prod`, `production`, `PROD`)
5. arbitrary free-text metadata without schema
6. metadata filter ko authorization samajhna
7. deleted source ka stale vector retain karna
8. filter fields define karna but ingestion pipeline me consistently populate na karna

---

# PART 12 — Metadata Schema Thinking

Define controlled vocabulary:

```text
environment: dev | stage | prod
status: draft | current | deprecated
service: aks | terraform | pipeline | networking
```

Useful validation:

```text
required source
required chunk_id
allowed environment values
version format
status allowed values
```

Agar metadata schema inconsistent hai:

```text
Prod
production
PROD
```

then filters silently miss relevant records.

---

# PART 13 — Interview Corner

**Q: Why add metadata to vector records?**  
For filtering, source traceability, versioning, governance and better retrieval scope.

**Q: Is metadata filtering enough for security?**  
No. Authorization must be enforced separately by trusted application/platform controls.

**Q: How can stale documents harm RAG?**  
They can be retrieved as semantically relevant even though their operational guidance is outdated.

**Q: Where should you learn pre-filter vs post-filter semantics?**  
Lesson 06 covers the retrieval theory; this lesson applies the metadata/filter contract to DevOps knowledge design.

---

# PART 14 — Revision

```text
Chunk Text
   +
Metadata
   ↓
Eligible + Searchable + Traceable Knowledge
```

Remember:

```text
Similarity      = relevance signal
Metadata        = scope/context
Authorization   = security decision
Pre/Post filter = retrieval implementation semantics
```

---

# PART 15 — Homework

1. 4 sample docs ke liye metadata schema design karo.
2. `prod + aks + current` filter ka pseudo-code likho.
3. Explain why `environment` filter security control nahi hai.
4. Explain why inconsistent values such as `Prod` vs `prod` can break retrieval.

---

# Next Lesson Kyu?

Ab metadata aur filtering contract clear hai. Next full lifecycle connect hoga:

**index kaise build hota hai aur user query ka retrieval flow end-to-end kaise chalta hai?**

# 👉 Lesson 10 — Indexing & Retrieval Flow
