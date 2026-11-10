# Day 12 — APIs, HTTP Requests, JSON Responses and Pagination
**Phase 2: Python for Data Engineering**

## 1. API fundamentals
An API exposes controlled access to a service. REST APIs commonly use HTTP methods:
- GET: retrieve.
- POST: create/submit.
- PUT/PATCH: replace/update.
- DELETE: remove (permission and semantics vary).

Common status codes: 200 success, 201 created, 400 invalid request, 401 unauthenticated, 403 forbidden, 404 not found, 429 rate limit, 5xx server error.

## 2. Request and response
Install: `python -m pip install requests`.
```python
import requests
response = requests.get("https://api.example.com/orders", timeout=30)
print(response.status_code)
```
Use a real approved endpoint and credentials in actual labs; example domain above is illustrative. Always set a timeout.

## 3. JSON response
```python
# After a successful request:
data = response.json()
print(type(data))
```
Validate response status and shape before using fields. A successful HTTP status can still contain unexpected business data.

## 4. Authentication and secrets
Common approaches include API keys, bearer tokens, OAuth, and service credentials. Store secrets in environment/secret manager—not source code, notebook outputs, or logs. Use HTTPS and least privilege.

## 5. Pagination
An API may return only a limited number of records per response.
- Page/offset: `page=1`, `limit=100`.
- Cursor: next cursor/token supplied by server.
- Link-based: follow a next URL.

Illustrative loop:
```python
all_orders = []
next_url = "https://api.example.com/orders"
while next_url:
    response = requests.get(next_url, timeout=30)
    response.raise_for_status()
    payload = response.json()
    all_orders.extend(payload["data"])
    next_url = payload.get("next")
print("Records collected:", len(all_orders))
```
This example accumulates in memory; large ingestion should write pages/chunks incrementally. Follow only trusted API-provided pagination URLs.

## 6. Reliability
- Timeout every request.
- Retry transient failures with bounded exponential backoff.
- Respect 429 and Retry-After.
- Do not retry invalid credentials or bad requests indefinitely.
- Track page/cursor and counts.
- Handle duplicate records and API updates.
- Checkpoint progress for long extraction.

## 7. Mini-project — API Data Extractor
Extract paginated order data from a sandbox/mock API; preserve raw JSON, record run metadata, validate required fields, and report total pages/records/errors.

## Mistakes
- No timeout.
- Assuming first page is complete.
- Hardcoding token.
- Ignoring rate limits.
- Trusting response shape without validation.
- Restarting from page one without duplicate-safe loading.

## Revision
API ingestion = authentication + request + status handling + JSON parsing + pagination + rate-limit/retry + validation + checkpointing.