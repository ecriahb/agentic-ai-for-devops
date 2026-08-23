# Module 1 — Lesson 5: First API Call & Response Object

> **Goal:** Beginner ko first AI request ka complete request→model→response flow samjhana, `client.responses.create()` ko word-by-word decode karna, response object inspect karna, aur hosted/local calls ko same mental model se understand karna.

---

## Where This Lesson Fits

```text
Lesson 03
Hosted provider setup
      ↓
Lesson 04
Local provider setup
      ↓
Lesson 05
First request + response mechanics
      ↓
Lesson 06
Tokens + context capacity
      ↓
Lesson 07
Structured output + validation
```

**Canonical ownership:** this lesson owns the first end-to-end API request/response mechanics and response-object inspection. It does not own deep HTTP/REST semantics (Module 3), prompt/context methodology (Module 2), or tool-calling mechanics (Lesson 08).

---

# 1. English Definition

**An API call is a programmatic request sent by a client application to a service, followed by a structured response returned by that service.**

AI case:

```text
Python App
   ↓ request
Model API
   ↓ processing
Existing Model
   ↓
Response Object
```

---

# 2. Why This Topic Comes Here

Lesson 3:

```text
OpenAI hosted setup ready
```

Lesson 4:

```text
Ollama local runtime ready
```

Ab actual request samajhna hai.

Important learning rule:

> Code run karna enough nahi. Har line explain kar paana chahiye.

---

# 3. Minimal Hosted Example

```python
import os
from dotenv import load_dotenv
from openai import OpenAI

load_dotenv()

client = OpenAI()
model = os.getenv("OPENAI_MODEL")

response = client.responses.create(
    model=model,
    input="Explain AKS in two simple lines.",
)

print(response.output_text)
```

---

# 4. Line-by-Line Breakdown

## `import os`
Environment variables read karne ke liye.

## `from dotenv import load_dotenv`
Local `.env` values ko process environment me load karne ke liye.

## `from openai import OpenAI`
Official Python SDK se client class import.

## `load_dotenv()`

```text
.env
 ↓
process environment
```

## `client = OpenAI()`
SDK client create hota hai.

```text
Client != Model
```

## `model = os.getenv("OPENAI_MODEL")`
Model name configuration se read hota hai.

## `client.responses.create(...)`
Existing model se new response generate karne ki request.

## `response = ...`
Returned structured response object variable me store hota hai.

## `response.output_text`
Generated text ka convenient aggregated view.

---

# 5. Sabse Important Beginner Correction

Wrong:

```text
responses.create()
= new AI model create kar raha hai
```

Correct:

```text
responses.create()
= existing model se response create/generate karne ki API request
```

Formula:

```text
Existing Model + Input → New Response
```

---

# 6. Request Kya Hai?

**A request is the input and configuration sent by the application to a service.**

Here:

```python
model=model,
input="Explain AKS in two simple lines."
```

Request can conceptually include:

```text
model selection
instructions/input
structured-output requirements
tools
other supported configuration
```

---

# 7. Response Kya Hai?

**A response is the structured result returned after the service processes a request.**

Response is not just a Python string.

Mental model:

```text
response
├── id
├── model
├── status
├── output
├── usage
└── output_text convenience property
```

Exact fields can evolve by SDK/API version, so inspect current official SDK docs for version-sensitive behavior.

---

# 8. Response Object vs Text

Wrong mental model:

```text
response = final text
```

Better:

```text
response = structured API result
response.output_text = one convenient view of generated text
```

Why full object matters:

- tracing
- request/response IDs
- model metadata
- usage
- structured items
- tool calls
- status/error workflows

Later tool calling will require inspecting more than text.

---

# 9. Inspect the Response

Learning code:

```python
print("Type:", type(response))
print("ID:", response.id)
print("Model:", response.model)
print("Status:", response.status)
print("Usage:", response.usage)
print("Output text:", response.output_text)
```

Expected idea:

```text
Type: SDK Response object
ID: provider-generated identifier
Model: selected/resolved model
Status: completion status
Usage: token usage information when available
Output text: generated answer
```

Do not hard-code exact IDs/token counts as expected output.

---

# 10. `output_text` Convenience Property

The response can contain multiple output items/content blocks. `output_text` provides a convenient aggregated text representation when output-text blocks exist.

This teaches:

```text
Full response object
      ↓
Application chooses the fields it needs
```

Later tool calling will require inspecting more than text.

---

# 11. Local Ollama Equivalent

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:11434/v1/",
    api_key="ollama",
)

response = client.responses.create(
    model="qwen3:4b",
    input="Explain AKS in two simple lines.",
)

print(response.output_text)
```

Mental model remains:

```text
Client
→ Request
→ Model Runtime
→ Model
→ Response
```

Only provider destination changes.

---

# 12. Hosted vs Local Call

```text
Hosted OpenAI
client → internet → OpenAI API → hosted model

Local Ollama
client → localhost → Ollama API → local model
```

Common application concepts:

```text
request
response
model selection
errors
validation
```

---

# 13. Expected Output

Prompt:

```text
Explain AKS in two simple lines.
```

A valid answer should roughly explain Azure Kubernetes Service in two simple lines.

Do not expect exact wording because generation is model-dependent.

Evaluation questions:

```text
Was it factually correct?
Did it follow two-line constraint?
Was it concise?
Did it invent anything?
```

---

# 14. Response Metadata in DevOps

Imagine AI incident assistant processes 100 incidents.

You may need:

```text
incident_id
request_id
response_id
model
latency
usage
status
prompt/version
```

Why?

```text
debugging
cost analysis
model comparison
audit
regression investigation
```

Pretty text alone is not enough for production operations.

---

# 15. Error Flow

API call can fail before model returns output.

```text
Python
 ↓
Client
 ↓
Authentication? fail → error
Network? fail → error
Model access? fail → error
Rate/quota? fail → error
Model processing? fail → error
 ↓ success
Response
```

So:

```text
No response text
```

does not necessarily mean model gave an empty answer.

---

# 16. Basic Safe Error Handling

```python
try:
    response = client.responses.create(
        model=model,
        input="Explain AKS in two simple lines.",
    )
    print(response.output_text)
except Exception as exc:
    print("Request failed:", type(exc).__name__)
    print(str(exc))
```

Learning goal:

```text
Failure category identify karo
not random code edit
```

---

# 17. Failure Drills

## Drill A — Missing model config
Host should detect missing configuration early.

## Drill B — Invalid model
Provider/model access error observe karo.

## Drill C — Missing cloud credential
Authentication/setup failure identify karo.

## Drill D — Stop Ollama
Local connection failure identify karo.

## Drill E — Restore correct setup
Confirm success path.

Write for each:

```text
Layer:
Error:
Root cause:
Fix:
```

---

# 18. Request vs Prompt vs Context

Do not mix these concepts:

```text
Request
= full API operation/configuration

Prompt/Input
= instructions/question content

Context
= information supplied to help model answer
```

Later Module 2 deepens prompt/context engineering. This lesson only needs the distinction.

---

# 19. Model Output Is Not Evidence

Suppose model says:

```text
Your AKS failed because NSG rule was removed.
```

If no real evidence was supplied:

```text
This is a hypothesis, not confirmed RCA.
```

Critical rule introduced here:

```text
Response object = API result
not automatically factual truth
```

---

# 20. First DevOps Mapping

Today:

```text
Question
→ Model
→ Response
```

Future:

```text
Incident
→ Model asks for tool
→ Host executes read-only tool
→ Evidence returned
→ Model reasons from evidence
→ Structured RCA
```

This first API call is the smallest building block of the later agent.

---

# 21. Common Beginner Confusions

1. `OpenAI()` = model. ❌
2. `create()` = create model. ❌
3. `response` = plain string. ❌
4. `output_text` = entire response object. ❌
5. API success = answer is true. ❌
6. ChatGPT works = API must work. ❌
7. Local model = no API. ❌
8. Same prompt = same exact answer every time/provider. ❌
9. Error before response = hallucination. ❌
10. Response metadata is useless. ❌

---

# 22. Practical

Run in order:

```powershell
python examples/01_first_ai_call.py
python examples/02_ollama_ai_call.py
```

For both write down:

```text
Provider:
Model:
Endpoint type:
Response type:
Response ID/status available?:
Text field:
Usage available?:
Latency observation:
```

---

# 23. Small Coding Exercise

Create `inspect_response.py`:

```python
print("MODEL:", response.model)
print("STATUS:", response.status)
print("USAGE:", response.usage)
print("TEXT:", response.output_text)
```

Then answer:

```text
Which field is for human-readable answer?
Which fields are useful for observability?
```

---

# 24. Production Notes

A production wrapper should normalize provider output into your own application contract.

Example:

```python
{
    "provider": "openai",
    "model": "...",
    "request_id": "...",
    "status": "success",
    "text": "...",
}
```

Why?

```text
Provider-specific object
      ↓ normalize
Internal app contract
```

This reduces coupling.

---

# 25. Interview Q&A

### Q1. What does `client.responses.create()` do?
It sends a request to generate a response using an existing model and supplied input/configuration.

### Q2. Does `create()` create a model?
No.

### Q3. What is a response object?
A structured SDK result containing generated output and related metadata.

### Q4. What is `response.output_text`?
A convenience property for aggregated output text.

### Q5. Why inspect response metadata?
Tracing, usage, debugging, audit and model comparisons.

### Q6. Request vs response?
Request is what the application sends; response is what the service returns.

### Q7. Does successful API response guarantee factual correctness?
No.

### Q8. Hosted and local model calls me common pattern?
Client → request → model/runtime → response → validation.

---

# 26. Revision Sheet

```text
Client = application communication object
Request = model + input + configuration
Model = existing AI model
Response = structured API result
output_text = convenient generated text
Response ID = useful trace identifier
Usage = token/usage metadata
Successful call != truthful answer
```

---

# 27. Homework

1. Explain every line of the first API call without notes.
2. Draw request→response architecture for OpenAI and Ollama.
3. Print safe response metadata.
4. Intentionally break model configuration and identify the error layer.
5. Explain response object vs output text in your own words.
6. Give one DevOps reason why request IDs/usage metadata matter.

---

# 28. Why Next Lesson?

Ab API call ka mechanics clear hai.

Next question:

```text
Model text ko process kaise measure karta hai?
Kitna context bhejna chahiye?
Hosted usage/cost aur long logs par kya effect hota hai?
```

➡️ **Lesson 6 — Tokens, Cost & Context Budgets**