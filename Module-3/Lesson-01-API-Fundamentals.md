# 🚩 Jai Bajrangbali!

# Lesson 01 — API Fundamentals

> **API ko samjhe bina AI API call sirf copy-paste lagti hai. API samajh aane ke baad poora software-to-software flow clear ho jata hai.**

> **Canonical boundary:** Module 1 introduced API use from an application perspective. **Module 3 is the canonical deep-dive for general API mechanics**: client/server, endpoints, contracts, REST, HTTP, JSON, authentication, errors and provider integration. Later modules reference these mechanics instead of re-teaching them.

---

## 🎯 Lesson Goal

Is lesson ke end tak aap samjhoge:

- API kya hai
- Client aur Server kya hote hain
- API endpoint kya hota hai
- Request aur Response kya hote hain
- API contract kya hota hai
- SDK API se kaise related hai
- AI application me API ka exact role

---

## 1. API Kya Hai?

**English Definition:**
> An API (Application Programming Interface) is a defined way for one software system to communicate with another software system.

### Hinglish Explanation

API ko restaurant waiter ki tarah samjho.

```text
Customer
   ↓ order
Waiter
   ↓
Kitchen
   ↓ food
Waiter
   ↓
Customer
```

Software world me:

```text
Your Python App
      ↓ request
     API
      ↓
AI / Azure / GitHub / Kubernetes Service
      ↓ response
     API
      ↓
Your Python App
```

Aapka application directly internal server code ko control nahi karta. Wo documented interface ke through request bhejta hai.

---

## 2. Client vs Server

**Client** = request bhejne wala.

**Server** = request receive karke kaam karne wala.

Example:

```text
Python script = Client
Ollama server = Server
```

Ya:

```text
Terraform automation app = Client
Azure REST API = Server
```

Important point:

> Ek hi system different situations me client bhi ho sakta hai aur server bhi.

---

## 3. Endpoint Kya Hai?

**English Definition:**
> An endpoint is a specific API address used to access a particular resource or operation.

Example conceptual endpoint:

```text
http://localhost:11434/api/chat
```

Mental model:

```text
Server Address
     +
Specific Path
     =
API Endpoint
```

---

## 4. Request Kya Hota Hai?

Request me commonly ye cheezein hoti hain:

```text
Method
URL / Endpoint
Headers
Authentication
Body / Payload
```

Example idea:

```json
{
  "model": "qwen2.5:3b",
  "messages": [
    {"role": "user", "content": "Explain AKS"}
  ]
}
```

At this lesson, request/response are concepts. **Wire-level HTTP mechanics are deliberately left to Lesson 03**, where methods, headers, status codes and idempotency are studied in detail.

---

## 5. Response Kya Hota Hai?

Server response me generally:

```text
Status Code
Headers
Body
```

Body frequently JSON hoti hai.

Example:

```json
{
  "message": {
    "role": "assistant",
    "content": "AKS is Azure Kubernetes Service..."
  }
}
```

This lesson establishes the abstraction. **Detailed HTTP response behavior belongs to Lesson 03.**

---

## 6. API Contract

API contract means agreement:

```text
What URL?
What method?
What authentication?
What input format?
What response format?
What errors can occur?
```

Agar contract ke against request bheji:

```text
Wrong field
Wrong auth
Wrong URL
Wrong HTTP method
       ↓
Request fails
```

Later lessons break this contract into REST, HTTP, JSON and authentication layers.

---

## 7. SDK vs API

Common confusion:

```text
API ≠ SDK
```

API = actual communication interface.

SDK = developer-friendly library jo API calls ko easier banati hai.

Without SDK:

```python
import requests
# manually build HTTP request
```

With SDK:

```python
client.responses.create(...)
```

SDK internally API ko hi call karta hai.

Mental model:

```text
Your Code
   ↓
SDK
   ↓
HTTP API
   ↓
Remote Service
```

---

# 🛠️ DevOps Example

Suppose agent ko pipeline details chahiye:

```text
DevOps AI Assistant
      ↓
Azure DevOps API
      ↓
Pipeline Run Information
      ↓
JSON Response
      ↓
Agent Analysis
```

Ya AKS case:

```text
Python Tool
   ↓
Kubernetes API
   ↓
Pod Status
   ↓
JSON
   ↓
LLM Evidence
```

---

## 🔗 Course Boundary Map

```text
Module 1
→ first programmatic API use

Module 3 — this module
→ general API engineering mechanics

Module 4+
→ APIs used as retrieval/tool/data sources
```

Do not turn this lesson into a second Python SDK setup tutorial; the goal is the reusable API mental model.

---

# ❌ Common Mistakes

1. API aur application ko same cheez samajhna.
2. SDK ko API samajhna.
3. Endpoint aur server ko same samajhna.
4. Request body ko poori request samajhna.
5. Response text dekhkar HTTP status ignore kar dena.
6. Module 1 ke first-call material ko API fundamentals samajhkar skip kar dena — both layers have different purposes.

---

# 🎤 Interview Points

**Q: What is an API?**

An API is a defined interface that allows software systems to communicate using an agreed request and response contract.

**Q: API vs SDK?**

An API defines communication with a service, while an SDK is a developer library that simplifies consuming that API.

---

# 🧠 Revision

```text
Client sends Request
        ↓
Endpoint receives Request
        ↓
Server performs work
        ↓
Server returns Response
```

---

# 🔁 Why Next Lesson?

Ab API ka abstraction clear hai.

But APIs ko design karne ke multiple styles hote hain. Most cloud and web APIs me aap baar-baar **REST** word dekhoge.

So next:

> **Lesson 02 — REST API Basics**
