# Day 32: API-Based Data Ingestion — Beginner Notebook Class

**Phase:** 4 — Data Processing and Pipelines  
**Time:** 2–3 hours  
**Project:** TechStore Marketplace API Extractor  
**Level:** Complete beginner / fresher  
**Tools:** Python, requests, JSON, Jupyter Notebook/VS Code  
**Prerequisites:** Python dictionaries/lists/loops, JSON, file ingestion (Day 31)

---

# 0. What will you build?

A company may not receive a file or have direct database access. A business system may expose an API: a controlled interface that lets approved clients request data.

Example: TechStore's marketplace provider exposes orders through an API. The provider limits each response to 3 records, returns a page token, and may temporarily respond with rate limits. We need to retrieve all permitted orders without losing records or overwhelming the provider.

Today's flow:

`API documentation -> Configure request -> Send -> Check response -> Parse JSON -> Validate -> Follow pagination -> Save -> Reconcile`

## 0.1 Learning outcomes

You can:
- explain API, endpoint, request, response, status code, headers and JSON;
- inspect a response safely;
- use query parameters rather than manually build URLs;
- validate response shape and records;
- understand pagination and cursor tokens;
- handle timeout, 429 and transient server errors;
- protect credentials;
- save data and checkpoint safely;
- test using mock responses without depending on a live API.

---

# 1. API fundamentals from zero

## 1.1 What is an API?

API means Application Programming Interface. It is a defined way for one software system to request or exchange information with another.

Analogy: A restaurant menu describes what can be ordered; the kitchen has rules for accepting orders and returning meals. An API contract describes available operations, required inputs, permissions and response structure. The analogy is not exact—the API is a technical interface with authentication, validation and machine-readable responses.

## 1.2 Important vocabulary

| Term | Meaning | Example |
|---|---|---|
| API | Interface for software communication | Marketplace provider API |
| Endpoint | Specific resource/operation address | /v1/orders |
| Request | What client sends | GET orders with date/filter |
| Response | What server returns | status, headers, JSON body |
| HTTP method | Intended operation type | GET reads |
| Status code | Result category | 200 success, 401 auth issue |
| Header | Metadata about request/response | Content-Type, Retry-After |
| Query parameter | Filter/control in URL | limit=3 |
| JSON | Structured text payload | orders array |
| Token | Credential or paging marker depending context | access token / next cursor |
| Rate limit | Request quota over time | 100 requests/minute |
| Pagination | Splitting results into pages | page token 1 then 2 |

**Careful:** Authentication token and pagination token are different things. One proves/authorizes identity; the other indicates where to continue reading.

## 1.3 HTTP GET

GET is commonly used to retrieve resources. It should not be used to perform an unintended business update.

Request concept:
`GET /v1/orders?limit=3&status=PAID`

- /v1/orders identifies resource/version.
- limit=3 asks for at most three records if supported.
- status=PAID asks for a filter if supported.
- Provider documentation defines actual meaning.

Never invent endpoint paths or parameters. Use the provider's official documentation and permitted access.

---

# 2. Inspect a safe public test endpoint

For classroom network practice, use a public test API such as JSONPlaceholder. It is a demo service, not real customer data or a production partner.

## 2.1 Install requests

Terminal:
```bash
python -m pip install requests
```

Verify in notebook:
```python
import requests
print(requests.__version__)
```

Exact version varies.

## 2.2 Send a simple GET request

```python
response = requests.get(
    "https://jsonplaceholder.typicode.com/posts/1",
    timeout=10
)

print("Status:", response.status_code)
print("Content-Type:", response.headers.get("Content-Type"))
print("Body:", response.text)
```

Expected response shape (public demo content):
```text
Status: 200
Content-Type: application/json; charset=utf-8
Body: {"userId": 1, "id": 1, "title": "...", "body": "..."}
```

The exact body formatting may vary. The meaningful fields are userId, id, title and body.

### Explain the code

- requests.get sends an HTTP GET request.
- URL identifies resource.
- timeout=10 limits how long this request waits for a response; it is not a retry policy.
- response stores HTTP response details.
- status_code is numeric status.
- headers is metadata.
- text is raw response body as text.

**Do not immediately parse every response as JSON.** First check whether request succeeded and whether body type is expected.

---

# 3. Understand status codes

| Code | Common meaning | Typical action |
|---|---|---|
| 200 | Successful request | Validate payload |
| 201 | Resource created | Usually relevant to POST |
| 204 | Success, no body | Do not parse JSON body |
| 400 | Invalid request | Fix parameters/input |
| 401 | Authentication missing/invalid | Fix credential; stop retries |
| 403 | Permission denied | Check scope/access |
| 404 | Resource/endpoint not found | Verify URL/resource |
| 429 | Rate limit exceeded | Respect Retry-After/backoff |
| 500 | Server error | May be transient; bounded retry policy |
| 503 | Service unavailable | Often transient; bounded retry |
| timeout | No response in allowed time | Retry only within policy |

Status codes are broad categories; provider-specific details may exist in body/headers.

## 3.1 Check status before parsing

```python
if response.status_code == 200:
    print("Request succeeded")
else:
    print("Request failed:", response.status_code)
```

A successful HTTP response does not guarantee valid business data. It may return an empty array or unexpected schema.

## 3.2 Raise for HTTP errors

```python
response.raise_for_status()
```

This raises an HTTPError for unsuccessful status codes (4xx/5xx). It does not validate JSON schema or business values. Use try/except where you need controlled failure handling.

---

# 4. Parse JSON response

## 4.1 Convert response body to Python

```python
payload = response.json()

print(type(payload))
print(payload)
```

For a single post, payload is a dictionary. A list endpoint may return a list of dictionaries.

## 4.2 Inspect one field

```python
print(payload["id"])
print(payload["title"])
```

Expected for /posts/1:
```text
1
sunt aut facere repellat provident occaecati excepturi optio reprehenderit
```

If the API returns different content or unavailable service, inspect actual response. Do not claim a request succeeded if it didn't.

## 4.3 JSON-to-Python mapping

| JSON | Python |
|---|---|
| object | dict |
| array | list |
| string | str |
| number integer | int |
| number decimal | float (unless custom parsing) |
| true/false | bool |
| null | None |

For money, convert carefully to Decimal from the source string or documented representation; do not trust binary float as financial exactness.

---

# 5. Query parameters

## 5.1 Use params dictionary

```python
params = {
    "userId": 1
}

response = requests.get(
    "https://jsonplaceholder.typicode.com/posts",
    params=params,
    timeout=10
)

response.raise_for_status()
posts = response.json()

print("Records returned:", len(posts))
print("First record:", posts[0] if posts else None)
```

Expected for demo endpoint:
```text
Records returned: 10
First record: {'userId': 1, 'id': 1, 'title': '...', 'body': '...'}
```

Why params?
- requests handles URL encoding.
- code separates endpoint from filters.
- easier to change and test.
- provider docs specify exact parameter names.

## 5.2 Avoid string concatenation

Unsafe/fragile:
`url = base + "?status=" + user_input`

Use params for query values. Dynamic URL path/identifiers require validation and safe construction too.

---

# 6. Validate the API response contract

## 6.1 Expected list shape

```python
if not isinstance(posts, list):
    raise ValueError("Expected API response to be a list")

print("List response confirmed")
```

## 6.2 Validate each record

```python
required_post_fields = {"userId", "id", "title", "body"}

for post in posts:
    missing = required_post_fields - set(post.keys())
    if missing:
        print("Invalid post; missing:", missing)
    else:
        print("Valid post ID:", post["id"])
```

For a production extractor, collect invalid records with source/request/run metadata and define whether to fail page or reject row. Do not print every record in a huge response.

## 6.3 Valid JSON vs valid API contract

A response can be:
- valid JSON but wrong top-level type;
- list with missing fields;
- correct fields with wrong value types;
- empty list;
- duplicate IDs;
- incomplete page due provider error.

Parsing and validation are separate stages.

---

# 7. Pagination: why one request may not be enough

A provider may return only 100 records per request. If 1,000 orders exist, one request returns only a portion.

Two common styles:

### Page/offset
`page=1, page=2...` or `offset=0, offset=100...`

### Cursor/token
Response includes `next_cursor`; next request sends that exact token.

Cursor is opaque. Do not calculate it or assume numeric.

## 7.1 Tiny simulated pagination example

Imagine a provider returns:

Page 1:
```json
{"items":[{"id":"A"},{"id":"B"}],"next_cursor":"cursor-2"}
```

Page 2:
```json
{"items":[{"id":"C"},{"id":"D"}],"next_cursor":null}
```

Expected final IDs: A, B, C, D.

Flow:
`request cursor=null -> page1 -> cursor-2 -> page2 -> null -> stop`

The provider's documented terminal signal determines when to stop.

---

# 8. Build pagination logic with mock pages (no internet dependency)

## 8.1 Create simulated page responses

```python
mock_pages = {
    None: {
        "items": [{"id": "A"}, {"id": "B"}],
        "next_cursor": "cursor-2"
    },
    "cursor-2": {
        "items": [{"id": "C"}, {"id": "D"}],
        "next_cursor": None
    }
}
```

## 8.2 Walk through page tokens

```python
all_items = []
cursor_token = None

while True:
    page = mock_pages[cursor_token]
    all_items.extend(page["items"])

    cursor_token = page["next_cursor"]

    if cursor_token is None:
        break

print([item["id"] for item in all_items])
print("Total:", len(all_items))
```

Expected:
```text
['A', 'B', 'C', 'D']
Total: 4
```

Explain:
1. Start with None because first page has no cursor.
2. Read page associated with current token.
3. Extend the result with page items.
4. Replace cursor with next_cursor from response.
5. Stop when documented terminal token is None.

## 8.3 Protect against repeated cursor

A buggy provider/mock might return cursor-2 repeatedly. Use a seen-token guard and max-page budget.

```python
seen_cursors = set()
cursor_token = None
page_count = 0
max_pages = 100

while True:
    if cursor_token in seen_cursors:
        raise RuntimeError("Repeated cursor detected")

    if cursor_token is not None:
        seen_cursors.add(cursor_token)

    page_count += 1
    if page_count > max_pages:
        raise RuntimeError("Page limit exceeded")

    page = mock_pages[cursor_token]
    all_items.extend(page["items"])
    cursor_token = page["next_cursor"]

    if cursor_token is None:
        break

print("Pages read:", page_count)
```

The maximum is a safety budget, not a replacement for provider termination rules.

---

# 9. Rate limits and retry policy

## 9.1 What is 429?

429 means the server is limiting requests. Sending requests repeatedly without waiting makes the situation worse.

If Retry-After is provided, respect it. Otherwise follow documented provider policy with bounded exponential backoff and jitter.

## 9.2 Retry classification

| Failure | Retry? | Reason |
|---|---|---|
| Timeout | Usually bounded retry | Temporary network issue possible |
| 429 | Yes after wait | Quota boundary |
| 503 | Often bounded retry | Temporary unavailability |
| 500 | Policy-dependent bounded retry | Server failure |
| 400 | No | Input/request needs correction |
| 401 | No immediate retry | Credentials need correction |
| 403 | No immediate retry | Permission/scope issue |
| 404 | Usually no | Wrong resource/endpoint |

Always define attempts and total elapsed-time budget.

## 9.3 Exponential backoff concept

Waits may increase: 1 sec -> 2 sec -> 4 sec -> 8 sec, subject to maximum and provider guidance. Add jitter so many clients do not retry at exactly the same time.

Do not implement an unbounded `while retry` loop.

---

# 10. Authentication and secrets

## 10.1 Token safety

A provider may require bearer token:
`Authorization: Bearer <token>`

The token is sensitive:
- store in environment/secret manager;
- never commit to Git;
- do not print it;
- redact request headers in logs;
- use least-privilege scope;
- rotate if exposed.

## 10.2 Environment variable example

```python
import os

api_token = os.environ["MARKETPLACE_API_TOKEN"]
print("Token configured:", bool(api_token))
```

Expected:
```text
Token configured: True
```

This proves only that a nonempty value exists. It does not prove validity or permission.

For a real request:
```python
headers = {
    "Authorization": f"Bearer {api_token}",
    "Accept": "application/json"
}
```

Only use this with an endpoint and token you are authorized to access.

---

# 11. Persist response safely

## 11.1 Save JSON

```python
import json
from pathlib import Path

output_path = Path("../output/posts.json")
output_path.parent.mkdir(parents=True, exist_ok=True)

with open(output_path, "w", encoding="utf-8") as file:
    json.dump(posts, file, indent=2)

print("JSON output saved:", output_path)
```

Explain:
- output folder is created if absent;
- json.dump serializes Python data to JSON;
- indent=2 makes output readable;
- file writing overwrites, so use a run-specific/temp path in production.

## 11.2 Verify saved output

```python
with open(output_path, "r", encoding="utf-8") as file:
    saved_posts = json.load(file)

print("Saved records:", len(saved_posts))
```

Expected count equals the number of posts in memory. Verify keys/IDs too; count alone is not enough.

---

# 12. Incremental API extraction

If API supports a documented updated_since or start/end filter:
- define timezone and boundary semantics;
- save a safe watermark only after destination is durable;
- use overlap if late changes are possible;
- deduplicate with stable event/order ID;
- reconcile changed records and deletes.

An API's created_at filter may not capture updates to old records. Ask whether the provider exposes updated_at, change events or CDC-like feed.

---

# 13. Project: TechStore Marketplace API Extractor

## Business request
Fetch marketplace orders for a defined date window, follow all pages, validate records, save raw JSON, and produce a run summary.

## Project milestones

1. Read provider documentation and confirm permission, endpoint, filters, pagination, quotas and schema.
2. Store token securely.
3. Configure request timeout and safe headers.
4. Send one request; inspect status, headers and body.
5. Validate response shape and required fields.
6. Follow pagination using exact next token.
7. Detect repeated cursor and page-limit breach.
8. Persist each page or use temp output safely.
9. Record request/page/record counts and errors.
10. Reconcile unique IDs and expected boundaries.
11. Test 429, timeout, malformed JSON, empty page and invalid schema using mocks.
12. Publish only after validation.

## Run summary shape

```python
run_summary = {
    "pages_read": page_count,
    "records_received": len(all_items),
    "unique_ids": len(set(item["id"] for item in all_items)),
    "duplicate_ids": len(all_items) - len(set(item["id"] for item in all_items)),
    "status": "SUCCESS"
}

print(run_summary)
```

Use the provider's actual stable ID field. The example id field is illustrative.

---

# 14. Troubleshooting

| Symptom | Check |
|---|---|
| Connection timeout | Network, endpoint, timeout and provider availability |
| 401 | Token validity/expiry |
| 403 | Permission/scope |
| 404 | Endpoint/resource/version |
| 429 | Retry-After/quota |
| 5xx | Provider status and bounded retry |
| JSON decode error | Content type/body; error page may be HTML |
| Missing field | Schema contract/version |
| Repeated cursor | Stop and alert |
| Duplicate records | Pagination consistency/overlap/idempotency |
| Unexpected zero | Filters/date/timezone/source |
| Output count mismatch | Page accounting, validation and persistence |

---

# 15. Exercises with answer guide

1. Why check status before calling json()?
2. What is difference between response.text and response.json()?
3. Why is pagination necessary?
4. Why should cursor tokens be treated as opaque?
5. What should happen on 429?
6. Which errors normally should not retry immediately?
7. Why is timeout different from retry?
8. Why does valid JSON not guarantee valid data?
9. Why should a checkpoint follow durable write?
10. What evidence proves all pages were read?

**Answer guide**
1. Error responses may contain non-JSON or unexpected payload.
2. text is raw body string; json parses into Python values.
3. API limits each response.
4. Provider defines token semantics; arithmetic assumptions can break.
5. Respect Retry-After/provider quota and bounded backoff.
6. 400, 401, 403 and generally 404 need request/permission correction.
7. Timeout limits one wait; retry policy governs another attempt.
8. Schema/types/business constraints may be wrong.
9. Otherwise progress can be recorded before data exists, causing loss.
10. Page count/terminal cursor, unique IDs, counts and reconciliation.

---

# 16. Day 32 final self-check

- [ ] Explain API request/response and endpoint.
- [ ] Send GET and inspect status/headers/body.
- [ ] Parse JSON and validate schema.
- [ ] Use query parameters.
- [ ] Demonstrate pagination with a mock.
- [ ] Detect repeated cursor.
- [ ] Explain rate limit and retry classification.
- [ ] Protect token.
- [ ] Save and reload output.
- [ ] Complete API extractor project and defend recovery strategy.

**Remember:** A successful HTTP response is only the beginning. A reliable API ingestion proves that all permitted pages were retrieved, validated, persisted and reconciled.
