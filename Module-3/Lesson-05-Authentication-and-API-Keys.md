# 🚩 Jai Bajrangbali!

# Lesson 05 — Authentication & API Keys

> **API reachable hona aur API authorized hona do alag problems hain.**

> **Canonical boundary:** Module 1 introduced safe local secret handling. This lesson owns the **API authentication/authorization model**: credentials, bearer tokens, 401/403, least privilege, rotation and workload identity. Lesson 06 then focuses on how applications receive configuration/secrets safely at runtime.

---

## 🎯 Lesson Goal

Aap samjhoge:

- Authentication vs Authorization
- API key kya hai
- Bearer token kya hai
- headers me credentials ka role
- 401 vs 403
- key rotation and least privilege
- managed identity / workload identity ka higher-level idea
- AI APIs ke authentication patterns

---

## 1. Authentication vs Authorization

**Authentication:** Who are you?

**Authorization:** What are you allowed to do?

```text
Identity prove
    ↓
Authentication
    ↓
Permission check
    ↓
Authorization
```

Example:

```text
Valid Azure identity
      ↓
Authentication successful
      ↓
But no permission on resource
      ↓
Authorization fails
```

---

## 2. API Key Kya Hai?

**English Definition:**
> An API key is a credential value used by an API service to authenticate, identify or authorize an application according to that service's design.

Conceptual flow:

```text
Python App
   ↓ credential
Provider API
   ↓ verify
Request accepted/rejected
```

Important:

> API-key security is provider-specific; never assume one header/name works for every API.

Credential **storage** is covered in Lesson 06. Here we focus on what the credential means to the API.

---

## 3. Bearer Token

Many APIs use an HTTP header like:

```http
Authorization: Bearer <secret-token>
```

The server validates the presented credential according to its authentication and authorization rules.

Do not log the full token.

---

## 4. API Key in Header

A common pattern is:

```python
headers = {
    "Authorization": f"Bearer {api_key}",
    "Content-Type": "application/json"
}
```

But never assume every provider uses the same header. Follow the provider's official documentation.

For OpenAI API usage, the current API pattern uses API keys with Bearer authentication; production code should obtain the credential through a secure application configuration/identity mechanism rather than embedding it in client-side code.

---

## 5. 401 vs 403

```text
401 Unauthorized
→ authentication credential missing/invalid/expired

403 Forbidden
→ authenticated identity lacks required permission
```

Debugging:

```text
401
 ↓
Credential present?
Correct auth scheme?
Expired/revoked?
Correct endpoint/provider?

403
 ↓
Role/RBAC correct?
Resource scope correct?
Policy restrictions?
```

---

## 6. Least Privilege

**English Definition:**
> Least privilege means granting only the permissions required to perform the intended task.

DevOps AI agent example:

Bad:

```text
Agent gets Owner/Admin access to everything
```

Better:

```text
Read-only access for investigation
Human approval before remediation
Narrow write permissions only where required
```

The exact secret-storage mechanism does not change the authorization principle.

---

## 7. Credential Rotation

Secrets permanent nahi samajhne chahiye.

Good lifecycle:

```text
Create
 ↓
Store securely
 ↓
Use
 ↓
Monitor
 ↓
Rotate
 ↓
Revoke when unused/compromised
```

If a credential is committed publicly:

1. Treat it as compromised.
2. Revoke/rotate it immediately.
3. Then clean up repository/history as required.

---

## 8. Better Than Long-Lived Secrets

Cloud production scenarios me possible ho to identity-based authentication preferable ho sakta hai:

```text
Managed Identity
Workload Identity
Service Principal with controlled credentials
OIDC federation
```

Goal:

```text
Fewer static secrets
+ short-lived credentials
+ auditable identity
```

---

# 🛠️ DevOps Example

AI assistant ko Azure logs read karne hain:

```text
Agent Application
       ↓
Workload Identity / Managed Identity
       ↓
Read permission
       ↓
Log Analytics / Azure resource
       ↓
Evidence
```

The identity establishes who the application is; RBAC determines what it may read.

---

## 🔗 Boundary With Lesson 06

```text
Lesson 05
→ What authentication/authorization means
→ What API credentials do

Lesson 06
→ Where runtime configuration comes from
→ .env / environment variables / secret stores
```

Do not turn Lesson 05 into a `.env` tutorial; that belongs to Lesson 06.

---

# ❌ Common Mistakes

- key source code me hard-code karna
- API key ko authorization ke equivalent samajhna
- 401 aur 403 confuse karna
- one admin credential sab apps me reuse karna
- old keys rotate na karna
- client-side browser code me server secret expose karna
- authentication successful hone ko authorization successful samajhna

---

# 🎤 Interview Point

**Q: Authentication vs authorization?**

Authentication verifies identity; authorization determines what that identity is permitted to do.

**Q: How would you secure AI API credentials in production?**

Prefer managed/short-lived identity where supported; otherwise use centralized secret management, least privilege, rotation, restricted access and safe logging.

---

# 🧠 Revision

```text
Credential
   ↓
Authentication
   ↓
Who are you?

Identity + Permissions
   ↓
Authorization
   ↓
What may you do?
```

---

# 🔁 Why Next Lesson?

Ab authentication ka meaning clear hai. Next question:

```text
Application ko credential/configuration milegi kahan se?
Code me hard-code kiye bina runtime par kaise load hogi?
```

> **Lesson 06 — Environment Variables & Secret Management**
