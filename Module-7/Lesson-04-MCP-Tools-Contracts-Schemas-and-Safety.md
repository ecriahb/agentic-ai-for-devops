# 🚩 Jai Bajrangbali!

# Lesson 04 — MCP Tools: Contracts, Schemas & Invocation Boundaries

> **Module 1 ne generic tool-calling safety sikhayi; yahan focus hai MCP tool ko protocol-level capability ke roop me define, discover aur invoke karna.**

---

## 🎯 Lesson Goal

Is lesson ke end tak aap samjhoge:

- MCP tool kya hai
- name, description aur input schema ka role
- typed tool contracts
- discovery metadata vs application policy
- read-only vs mutating capability classification
- structured tool results
- MCP-specific invocation boundaries

> **Boundary:** Generic argument validation, RBAC, retries, approvals and evidence preservation were introduced earlier. Module 7 reuses those rules at the MCP boundary instead of re-teaching them from scratch.

---

# PART 1 — English Definition

An **MCP tool** is a server-exposed executable capability with a discoverable name, description and input contract that an MCP client can invoke.

Example:

```text
get_aks_status(cluster_name)
get_pipeline_status(environment)
get_terraform_changes(environment)
```

Think:

```text
Tool = capability + contract + invocation endpoint
```

---

# PART 2 — What MCP Adds to Existing Tool Calling

Module 1 mental model:

```text
Model proposes tool call
      ↓
Host validates
      ↓
Execute
      ↓
Evidence/result
```

MCP standardizes the capability-facing part:

```text
MCP Server
   ↓
advertise typed tool
   ↓
MCP Client discovers tool
   ↓
Client invokes by protocol
```

So MCP does not replace the trust model.

```text
MCP standardization
        ≠
automatic authorization
```

---

# PART 3 — Tool Contract Anatomy

A useful MCP tool definition has:

```text
name
description
input schema
required fields
optional fields
output/result shape
```

Example:

```text
name:
get_pipeline_status

input:
{
  "environment": "production"
}

output:
{
  "status": "failed",
  "stage": "terraform_apply"
}
```

The contract answers:

```text
What is this capability?
What input does it accept?
What kind of result can I expect?
```

It does not answer:

```text
Who is allowed to use it?
Whether production use is approved?
Whether a business rule permits the operation?
```

---

# PART 4 — Typed Schemas vs Business Rules

Typed declaration:

```python
@mcp.tool()
def get_pipeline_status(environment: str) -> dict:
    ...
```

This communicates structural type information.

Business policy is separate:

```python
ALLOWED_ENVIRONMENTS = {"dev", "stage", "production"}
```

Therefore:

```text
Schema validation
= "Is this input shaped correctly?"

Business validation
= "Is this value allowed here?"
```

The second remains application/server responsibility.

---

# PART 5 — Read-Only vs Mutating Capability

Classify tools explicitly:

```text
READ ONLY
get_pipeline_status
get_aks_status
read_terraform_plan

MUTATING
restart_deployment
apply_terraform
scale_cluster
```

This classification becomes part of the host/server policy layer.

For Module 7 learning, default to:

```text
read-only tools
      ↓
trusted evidence
      ↓
recommendation
```

Write-capable tools are covered in greater depth through the security and integration lessons.

---

# PART 6 — Tool Description Is Discovery Metadata

Weak:

```text
Get status
```

Better:

```text
Return current read-only deployment pipeline status for one approved environment. Does not modify the pipeline.
```

A strong description improves discoverability and model/tool selection.

But remember:

```text
Description = information
Description != enforcement
```

---

# PART 7 — Structured Results

Prefer explicit result shapes:

```json
{
  "status": "failed",
  "stage": "terraform_apply",
  "timestamp": "2026-08-16T10:00:00Z"
}
```

Compared with:

```text
"something failed"
```

Structured results help downstream applications with:

```text
parsing
logging
validation
source mapping
```

But:

```text
structured != verified truth
```

The backend source remains the authority for the actual observation.

---

# PART 8 — Invocation Boundary

Conceptual flow:

```text
Tool discovered
      ↓
Host selects/permits tool
      ↓
MCP client sends tool request
      ↓
MCP server validates request
      ↓
Backend operation
      ↓
Structured result / error
```

Important distinction:

```text
discovery → availability
policy    → permission
invocation → request
execution  → server-side operation
```

Keeping these concepts separate prevents MCP from becoming a magic trust layer.

---

# PART 9 — Error Semantics

A tool should distinguish meaningful failure states, for example:

```text
INVALID_ARGUMENT
UNAUTHORIZED
NOT_FOUND
TIMEOUT
DEPENDENCY_FAILURE
```

Do not transform an error into fake success:

```json
{
  "status": "success",
  "message": "unknown"
}
```

An explicit error should remain an error so the host can decide whether to retry, abstain or surface an evidence gap.

---

# PART 10 — DevOps Example

Tool set:

```text
get_pipeline_status(environment)
get_terraform_changes(environment)
get_aks_status(cluster_name)
```

Discovery tells the host what exists.

The host then decides which read-only tools belong to the investigation.

Example result mapping:

```text
[E1] pipeline → failed during Terraform Apply
[E2] terraform → NSG allow rule removed
[E3] AKS → connectivity degraded
```

The evidence handling pattern is inherited from Modules 1, 5 and 6.

---

# PART 11 — MCP Tool vs Generic API Endpoint

A REST API might expose:

```text
GET /clusters/{id}/status
```

An MCP server can expose:

```text
get_aks_status(cluster_name)
```

The important architectural difference for this course is not that HTTP disappeared; it is that the AI-facing capability has a discoverable, typed MCP contract.

The server may still call REST, SDKs, databases or CLIs internally.

---

# PART 12 — Common Mistakes

- treating discovery as authorization
- using overly broad generic tools
- hiding side effects in tool descriptions
- returning ambiguous success payloads
- losing structured result metadata
- assuming schema validation replaces business policy

For detailed generic tool safety, refer back to Module 1 and Module 6.

---

# PART 13 — Interview Q&A

### Q1. What is an MCP tool?
A discoverable server-exposed capability with a defined input contract that a client can invoke.

### Q2. Does MCP schema validation enforce authorization?
No. Schema validates structure; trusted application/server policy must handle authorization and business rules.

### Q3. Why classify read-only vs mutating tools?
Because side effects change retry, approval and security requirements.

### Q4. Does an MCP server have to call an HTTP API?
No. It can wrap APIs, SDKs, CLIs, databases or local functions.

---

# PART 14 — Revision

```text
Tool
= capability + discoverable contract

Schema
= structure

Policy
= permission/business rule

Invocation
= request

Execution
= server/backend operation
```

Golden rule:

```text
MCP tool call is a request, not authority.
```

---

# PART 15 — Homework

For these tools, define the protocol contract only:

```text
get_terraform_plan(workspace)
get_aks_events(cluster, namespace)
get_pipeline_status(environment)
```

For each write:

```text
name
purpose
required arguments
argument types
result fields
error states
read-only or mutating
```

---

# 🔁 Next Lesson Kyu?

Tools represent executable capabilities. Next we need the read-only counterpart for addressable context/data:

# 👉 Lesson 05 — MCP Resources & Resource Templates
