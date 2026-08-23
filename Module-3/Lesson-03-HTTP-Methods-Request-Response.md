# 🚩 Jai Bajrangbali!

# Lesson 03 — HTTP Methods, Request & Response

> **API abstraction samajhne ke baad ab wire-level behavior dekhenge: HTTP request actually kaise banti hai, response ko kaise diagnose karte hain, aur retry/idempotency ka kya role hai.**

---

## 🎯 Lesson Goal

Aap samjhoge:

- HTTP kya hai
- GET, POST, PUT, PATCH, DELETE
- request ke parts
- response ke parts
- headers and body
- status code families
- idempotency ka basic idea
- AI/DevOps API debugging approach

---

## 1. Where This Lesson Fits

```text
Lesson 01 — API Fundamentals
        ↓
What is an API / client / server / endpoint?
        ↓
Lesson 02 — REST API Basics
        ↓
Resources + REST design style
        ↓
Lesson 03 — HTTP Mechanics  ← YOU ARE HERE
        ↓
Method + headers + body + status + retry semantics
```

**Boundary:** Lesson 01 owns the API abstraction. This lesson owns the HTTP transport semantics. JSON details are covered in Lesson 04, and authentication details in Lesson 05.

---

## 2. HTTP Kya Hai?

**English Definition:**
> HTTP is an application-layer protocol used to exchange requests and responses between clients and servers.

Mental model:

```text
Client
  ↓ HTTP Request
Server
  ↓ HTTP Response
Client
```

HTTPS same communication ko TLS encryption ke saath protect karta hai.

---

## 3. HTTP Methods

### GET — Read

```text
GET /pipelines/123
```

Meaning: information fetch karo.

### POST — Create / Submit Action

```text
POST /chat/completions
```

LLM inference commonly POST hota hai because aap prompt payload submit karte ho.

### PUT — Replace

Usually complete resource representation replace karne ke semantics.

### PATCH — Partial Update

Specific fields change karna.

### DELETE — Remove

Resource delete karna.

The exact semantics of a method come from the API contract; HTTP method name alone does not tell you the complete business behavior.

---

## 4. HTTP Request Anatomy

```text
METHOD + URL
Headers
Body
```

Conceptual example:

```http
POST /v1/chat HTTP/1.1
Authorization: Bearer <token>
Content-Type: application/json

{"message":"Analyze deployment failure"}
```

### Headers

Headers metadata carry karte hain:

```text
Authorization
Content-Type
Accept
User-Agent
Correlation / Request IDs
```

### Body

Body actual payload ho sakta hai:

```json
{
  "model": "example-model",
  "prompt": "Analyze this incident"
}
```

**JSON syntax itself is intentionally covered in Lesson 04.**

---

## 5. HTTP Response Anatomy

```text
Status Code
Headers
Body
```

Example:

```text
200 OK
Content-Type: application/json

{"result":"Deployment failed during Terraform apply"}
```

A successful HTTP status does not automatically mean the business operation succeeded; the response body/application status may still need validation.

---

## 6. Status Code Families

| Family | Meaning | Example |
|---|---|---|
| 2xx | Success | 200, 201 |
| 3xx | Redirect | 301, 302 |
| 4xx | Client/request problem | 400, 401, 403, 404, 429 |
| 5xx | Server-side problem | 500, 502, 503 |

### Important Codes for AI Apps

**400 Bad Request** — payload/schema wrong.

**401 Unauthorized** — credentials missing/invalid.

**403 Forbidden** — identity known, but permission insufficient.

**404 Not Found** — endpoint/resource/deployment name wrong or unavailable.

**429 Too Many Requests** — rate/quota limit hit.

**500/502/503** — provider/server/upstream failure; often retry strategy relevant.

Detailed credential semantics belong to Lesson 05; here they are used as part of HTTP diagnosis.

---

## 7. 401 vs 403

Common interview/debugging question:

```text
401 → Who are you? Authentication failed.
403 → I know who you are, but you cannot do this. Authorization failed.
```

We will study credentials and least privilege in Lesson 05.

---

## 8. Idempotency — Basic Idea

**English Definition:**
> An idempotent operation can be repeated without creating additional unintended state changes after the first successful application.

GET is normally safe to repeat.

A POST that creates a deployment may not be safe to retry blindly because duplicate execution could happen.

For AI inference, repeating a failed request may be operationally safe in many cases, but it still costs tokens and may produce a different output. So retry policy should understand the operation.

This concept becomes important again in Lesson 10 when we design bounded retry behavior.

---

# 🧪 Python Practical

```python
import requests

url = "https://httpbin.org/get"

response = requests.get(url, timeout=10)

print("Status:", response.status_code)
print("Headers:", response.headers.get("Content-Type"))
print("Body:", response.json())
```

Run:

```bash
pip install requests
python 01_api_get_request.py
```

---

# 🛠️ DevOps Debugging Flow

```text
API call failed
   ↓
Check URL/endpoint
   ↓
Check HTTP method
   ↓
Check status code
   ↓
Check response body/error
   ↓
Check authentication/permissions
   ↓
Check payload
   ↓
Check rate limit/server availability
```

Don't immediately conclude "LLM problem".

---

# ❌ Common Mistakes

- 401 ko network issue bol dena
- 429 par infinite immediate retry
- 500 response ko malformed prompt samajhna
- timeout specify na karna
- destructive POST/DELETE blindly retry karna
- response body read kiye bina sirf status code print karna
- Lesson 01 API abstraction ko HTTP wire-level behavior ke saath confuse karna

---

# 🎤 Interview Point

**Q: How do you troubleshoot an API failure?**

Start with endpoint and method, inspect the HTTP status code and response body, verify authentication/authorization and payload, then handle transient failures such as rate limiting or server errors with controlled retries.

---

# 🔁 Why Next Lesson?

HTTP body me structured data kaise encode hota hai? AI APIs me mostly **JSON** dikhega.

> **Lesson 04 — JSON for AI Applications**
