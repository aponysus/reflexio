# Reflexio – Smarter Retries for Real Systems

Retries seem simple until you need them to behave correctly in production.

Most Python retry libraries treat all failures the same. They give you one
decorator, one backoff model, and one global set of rules.

But real systems don’t fail uniformly:

- A timeout is not the same as a 500.
- A concurrency conflict is not the same as a rate limit.
- Some failures should retry aggressively.
- Some should retry slowly.
- Some should never retry.
- Some should escalate immediately.

**Reflexio** exists because real systems need *intelligent*, structured retry behavior,
not blanket rules.

## Why I Built Reflexio

After years building distributed compute engines, internal APIs, and
latency-sensitive services, I kept running into the same retry problems:

- “Why did we retry this? We shouldn’t have.”
- “Why *didn’t* we retry that? It was safe.”
- “Why did we retry it 7 times?”
- “Why is the service being hammered by identical retries?”
- “Why is a timeout handled the same as a 429?”
- “Why are our logs full of identical retry messages with no structure?”

Retry logic is deceptively subtle. It needs structure, classification,
observability, and predictability. So I wrote a library that reflects how real
services behave.

Reflexio focuses on four core principles:

### **1. Classification drives behavior**
Errors are first mapped into coarse “error classes,” which then determine retry
behavior. This keeps policy logic clean and predictable.

```python
TimeoutError   -> TRANSIENT
ConnectionError -> TRANSIENT
429 / rate limit -> RATE_LIMIT
400 / validation -> PERMANENT
```

### **2. Per-class strategies**
Different error classes should use different backoff strategies:

- fast jitter for TRANSIENT
- slow + capped for RATE_LIMIT
- immediate stop for PERMANENT
- equal jitter for SERVER_ERROR
- fail-fast for UNKNOWN after N attempts

### **3. Observability matters**
Retries without structured events are a debugging nightmare.

Reflexio provides a single callback that surfaces:

- attempt number
- event type (retry, success, permanent_fail, etc.)
- chosen sleep duration
- error class
- strategy used

This makes logging and metrics integration trivial.

### **4. Minimal overhead, explicit API**
The API is intentionally small:

- `RetryPolicy`
- `AsyncRetryPolicy`
- `@retry` decorator
- Strategies (`decorrelated_jitter`, `equal_jitter`, etc.)
- Classifiers

No magic, no stack-trace pollution, no surprise behaviors.

---

## How Reflexio Compares

A simple table to illustrate where Reflexio fits in the retry ecosystem:

| Feature | reflexio | Tenacity | backoff |
|--------|----------|----------|---------|
| Exception classification | ✅ | ⚠️ manual | ❌ |
| Per-class strategies | ✅ | ⚠️ partial | ❌ |
| Structured observability hooks | ✅ | ❌ | ❌ |
| Decorators | ✅ | ✅ | ✅ |
| Async support | ✅ | ⚠️ | ⚠️ |
| Decorrelated jitter | ✅ | ❌ | ⚠️ |
| Small, explicit API | ✅ | ❌ (large) | ⚠️ |
| Deterministic envelopes | ✅ | ❌ | ⚠️ |

Reflexio isn't meant to replace Tenacity or backoff; it’s built for
situations where **controlling retry behavior per class of failure actually
matters**.

---

## Quick Example

```python
from reflexio import retry, default_classifier
from reflexio.strategies import decorrelated_jitter

@retry(
    classifier=default_classifier,
    strategy=decorrelated_jitter(max_s=10.0),
)
def fetch_data():
    ...
```

Async support works similarly:

```python
from reflexio import AsyncRetryPolicy, default_classifier

async_policy = AsyncRetryPolicy(
    classifier=default_classifier,
    strategy=decorrelated_jitter(max_s=5.0),
)
```

---

## Why This Matters

Modern systems rely heavily on:

- upstream APIs
- internal microservices
- message queues
- cloud provider endpoints
- databases behind proxies
- external SaaS integrations

All of them fail in different ways, for different reasons, with different
recommended backoff patterns.

Retries are not an afterthought; they’re part of the **reliability surface** of
your application.

Reflexio makes this reliability explicit, structured, and observable... without
bloated abstractions or hidden behavior.

---

## Try It

**GitHub**  
https://github.com/aponysus/reflexio

**PyPI**  
https://pypi.org/project/reflexio/

```
pip install reflexio
```

---

## Documentation

- [Usage Guide](usage.md)
- [Observability & Metrics](observability.md)
- [Examples & Snippets](snippets/)

# Snippets & Examples

- [pyodbc_classifier.py](pyodbc_classifier.py)
- [async_worker_retry.py](async_worker_retry.py)
- [decorator_retry.py](decorator_retry.py)
- [fastapi_downstream.py](fastapi_downstream.py)
- [httpx_async_retry.py](httpx_async_retry.py)
- [httpx_sync_retry.py](httpx_sync_retry.py)

---

## Contribute

The project is intentionally small and easy to grok. Future planned topics:

- HTTP client helpers (requests/httpx)
- circuit-breaker patterns
- retry budgets
- tracing/metrics adapters
- structured error taxonomy extensions

If any of that interests you, PRs are welcome.

---
