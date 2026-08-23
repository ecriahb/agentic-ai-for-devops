# 🚩 Jai Bajrangbali!

# Lesson 06 — Environment Variables & Secret Management

> **Credential ka meaning Lesson 05 me samjha; ab seekhenge application ko configuration aur secrets safely runtime par kaise milte hain.**

> **Canonical boundary:** Module 1 covers first-project secret hygiene while this lesson is the deeper **API-application configuration boundary**: environment variables, `.env`, CI/CD injection, secret stores and fail-fast configuration. Azure-specific identity/security architecture is referenced rather than re-taught here.

---

## 🎯 Lesson Goal

Aap samjhoge:

- environment variable kya hai
- `.env` file ka role
- `python-dotenv`
- `.gitignore`
- local vs production secret handling
- Azure Key Vault / secret manager concept
- missing-secret validation
- secure logging habits
- CI/CD runtime injection

---

## 1. Hard-Coded Secret Problem

Bad:

```python
api_key = "sk-real-secret-here"
```

Risk:

```text
Code pushed to GitHub
       ↓
Secret exposed
       ↓
Unauthorized API usage / cost / data risk
```

Even private repository ko secret vault samajhna safe design nahi hai.

---

## 2. Environment Variable Kya Hai?

**English Definition:**
> An environment variable is a key-value value supplied to a process by its runtime environment instead of being embedded in source code.

Example:

```text
OPENAI_API_KEY=<secret>
```

Python:

```python
import os

api_key = os.getenv("OPENAI_API_KEY")
```

Mental model:

```text
Runtime Environment
      ↓
Environment Variable
      ↓
Application Process
```

---

## 3. `.env` for Local Development

Local file:

```text
OPENAI_API_KEY=replace_me
OLLAMA_BASE_URL=http://localhost:11434/v1
```

Python:

```python
import os
from dotenv import load_dotenv

load_dotenv()

api_key = os.getenv("OPENAI_API_KEY")
```

Install:

```bash
python -m pip install python-dotenv
```

`.env` is a **development convenience**, not a security boundary or production secret-management service.

---

## 4. `.env` Must Not Go to Git

`.gitignore`:

```gitignore
.env
.venv/
__pycache__/
```

Safe repo pattern:

```text
.env          → local real values, ignored
.env.example  → placeholder names, committed
```

`.env.example`:

```text
OPENAI_API_KEY=your_key_here
OLLAMA_BASE_URL=http://localhost:11434/v1
```

Never put a real secret in `.env.example`.

Important: if a secret was already committed, adding `.gitignore` later does not make the credential safe. Revoke/rotate the secret.

---

## 5. Validate Early

Bad:

```python
api_key = os.getenv("OPENAI_API_KEY")
# failure happens much later
```

Better:

```python
api_key = os.getenv("OPENAI_API_KEY")

if not api_key:
    raise RuntimeError("OPENAI_API_KEY is not configured")
```

Fail fast gives clearer debugging and prevents confusing downstream errors.

---

## 6. Local vs Production

`.env` is convenient for learning/local development.

Production mental model:

```text
Application
    ↓ authenticated identity/access
Secret Manager / Key Vault
    ↓
Secret / Token
```

Azure example:

```text
AKS / App Service / VM
       ↓ Managed Identity / Workload Identity
Azure Key Vault
       ↓ Secret
Application
```

The exact identity mechanism is environment-specific; the important principle is to keep secret retrieval in the application/platform identity layer rather than source code.

---

## 7. Don't Print Secrets

Bad:

```python
print(api_key)
```

Better:

```python
print("API key configured:", bool(api_key))
```

Logs, screenshots and CI/CD output are all potential exposure paths.

---

## 8. CI/CD Secret Injection

Pipeline flow:

```text
GitHub Actions / Azure DevOps
       ↓ secret store / federated identity
Runtime environment or identity token
       ↓
Application
```

Avoid:

```text
Secret in YAML plaintext
Secret in Terraform output
Secret echoed in pipeline logs
```

Prefer the platform's approved secret/identity mechanism.

---

# 🧪 Practical

`.env`:

```text
APP_ENV=dev
DEMO_API_KEY=super-secret-demo
```

Python:

```python
import os
from dotenv import load_dotenv

load_dotenv()

app_env = os.getenv("APP_ENV", "local")
api_key = os.getenv("DEMO_API_KEY")

if not api_key:
    raise RuntimeError("DEMO_API_KEY missing")

print("Environment:", app_env)
print("Secret configured:", True)
```

Expected:

```text
Environment: dev
Secret configured: True
```

Never print the actual key.

---

# 🔗 Course Boundary Map

```text
Module 1
→ first-project secret hygiene / .env basics

Module 3 — this lesson
→ runtime configuration + API secret delivery patterns

Module 10+
→ comprehensive security, threat and control analysis
```

This lesson should not re-teach full identity/RBAC security design from later security modules.

---

# ❌ Common Mistakes

- `.env` ko commit kar dena
- key ko README/code screenshot me expose karna
- production me unmanaged `.env` files spread karna
- secret missing hone par unclear error
- logs me token print karna
- `.gitignore` ko already-committed secret cleanup samajhna
- configuration aur authorization ko same concept samajhna

---

# 🎤 Interview Point

**Q: Is `.env` a production secret manager?**

No. It is mainly a local-development pattern. Production systems should prefer managed secret stores or workload identity mechanisms with controlled access and auditing.

**Q: Why fail fast on missing configuration?**

To stop the application before it reaches a later, less-obvious authentication or integration failure.

---

# 🧠 Revision

```text
Secret meaning
    ↓
Lesson 05

Secret/config delivery
    ↓
Lesson 06

API usage
    ↓
Lesson 08+
```

Core rule:

> **Keep secrets out of source code, logs, prompts and model context.**

---

# 🔁 Why Next Lesson?

Ab API client ko configuration safely mil sakti hai. In concepts ko glue karne ke liye hume sirf wahi Python chahiye jo AI application read, debug aur modify karne ke liye essential hai.

> **Lesson 07 — Minimal Python for AI Applications**
