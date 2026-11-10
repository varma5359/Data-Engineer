# Day 12: API Integration

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** API Data Extractor  
**Learning time:** 2–3 hours

## 1. Learning outcomes
- Explain HTTP and REST API basics.
- Make GET requests using Python requests.
- Understand status codes, headers, parameters and JSON.
- Handle timeouts, errors and rate limits.
- Implement pagination and persist extraction progress.
- Protect API credentials.

## 2. What is an API?
An Application Programming Interface is a defined way for one application to request data or actions from another system.

Example: a retailer retrieves payment status from a payment provider rather than accessing the provider's internal database.

## 3. HTTP request and response
A request commonly contains:
- Method: what operation is requested.
- URL: endpoint.
- Headers: metadata such as authentication and content type.
- Query parameters: filters or pagination.
- Body: data sent for methods such as POST.

A response commonly contains:
- Status code.
- Headers.
- Body, often JSON.

## 4. Common HTTP methods

| Method | Typical use |
|---|---|
| GET | Retrieve resource |
| POST | Create or submit |
| PUT | Replace resource |
| PATCH | Partially update |
| DELETE | Delete resource |

A GET request should not be used to modify server state.

## 5. Common status codes

| Code | Meaning | Typical response |
|---|---|---|
| 200 | OK | Parse response |
| 201 | Created | Record creation result |
| 204 | No content | Handle empty body |
| 400 | Bad request | Fix request parameters |
| 401 | Unauthorized | Check authentication |
| 403 | Forbidden | Check permission |
| 404 | Not found | Check endpoint/resource |
| 429 | Too many requests | Respect rate limit/backoff |
| 500 | Server error | Retry cautiously if transient |
| 503 | Service unavailable | Retry with backoff |

Do not assume every non-200 response has a JSON body.

## 6. Install requests
```bash
python -m pip install requests
```

## 7. Make a GET request
Use a documented public test endpoint or an approved company API.

```python
import requests

url = "https://jsonplaceholder.typicode.com/posts/1"

response = requests.get(url, timeout=15)

print(response.status_code)
print(response.json())
```

Expected: status 200 and a JSON object for a sample post. Network access may be unavailable in a restricted classroom environment; the code example demonstrates the request pattern.

## 8. Set a timeout
```python
response = requests.get(url, timeout=(5, 20))
```

The tuple sets connection and read timeout. A request without a timeout can wait indefinitely.

## 9. Check response status
```python
response = requests.get(url, timeout=15)
response.raise_for_status()

data = response.json()
print(data)
```

`raise_for_status()` raises an HTTPError for unsuccessful status codes. It does not validate that the JSON matches the expected schema.

## 10. Query parameters
```python
params = {"userId": 1}
response = requests.get(
    "https://jsonplaceholder.typicode.com/posts",
    params=params,
    timeout=15
)

response.raise_for_status()
posts = response.json()

print(len(posts))
```

Let the HTTP library encode query parameters. Avoid manually concatenating untrusted values into URLs.

## 11. Headers and authentication
```python
headers = {
    "Accept": "application/json"
}

response = requests.get(url, headers=headers, timeout=15)
```

For bearer-token APIs, pass a token through an approved secret store or environment variable:

```python
import os

api_token = os.environ.get("PAYMENT_API_TOKEN")

if not api_token:
    raise RuntimeError("API token is not configured")

headers = {
    "Authorization": f"Bearer {api_token}",
    "Accept": "application/json"
}
```

Never commit real tokens to Git or print them in logs.

## 12. JSON response validation
```python
payload = response.json()

if not isinstance(payload, list):
    raise ValueError("Expected a list response")

for record in payload:
    if "id" not in record:
        raise ValueError("Record is missing id")
```

A successful HTTP response can still contain unexpected data. Validate structure, types and required fields.

## 13. Pagination
APIs often split results into pages to control response size.

Common pagination styles:
- Page number and page size.
- Offset and limit.
- Cursor or continuation token.
- Next-page URL.

Example page-number pattern:

```python
import requests

all_records = []
page = 1
page_size = 50

while True:
    response = requests.get(
        "https://api.example.com/orders",
        params={"page": page, "limit": page_size},
        timeout=20
    )
    response.raise_for_status()

    payload = response.json()
    records = payload["data"]

    if not records:
        break

    all_records.extend(records)

    if len(records) < page_size:
        break

    page += 1

print(len(all_records))
```

This is a generic example; actual key names and pagination rules must come from the API documentation.

## 14. Cursor pagination
A cursor is an opaque token supplied by the API to identify the next page.

```python
cursor = None
all_records = []

while True:
    params = {"limit": 100}

    if cursor is not None:
        params["cursor"] = cursor

    response = requests.get(
        "https://api.example.com/orders",
        params=params,
        timeout=20
    )
    response.raise_for_status()

    payload = response.json()
    all_records.extend(payload["data"])

    cursor = payload.get("next_cursor")

    if not cursor:
        break
```

Do not invent a cursor or assume that a page number works for every API.

## 15. Retries and rate limits
A 429 response may include a Retry-After header. Follow provider instructions. For transient network/server errors, use bounded retries with backoff. Do not retry authentication or validation errors repeatedly.

Retry design should consider:
- Which status codes are retryable.
- Maximum attempts.
- Delay/backoff.
- Retry-After.
- Idempotency of the request.
- Whether partial results are persisted.

## 16. Mini-project: API Data Extractor
Use an approved API with documented pagination.

### Step 1 — Confirm the contract
Record endpoint, authentication, request limits, response schema, pagination style and data owner.

### Step 2 — Retrieve a sample page
```python
response = requests.get(
    "https://jsonplaceholder.typicode.com/posts",
    params={"_limit": 5},
    timeout=15
)
response.raise_for_status()

sample_records = response.json()
print(len(sample_records))
print(sample_records[0])
```

### Step 3 — Validate required fields
```python
required_fields = {"userId", "id", "title", "body"}

for record in sample_records:
    missing_fields = required_fields - set(record.keys())

    if missing_fields:
        print("Invalid record:", missing_fields)
```

### Step 4 — Save raw response
```python
import json

with open("api_response.json", "w", encoding="utf-8") as file:
    json.dump(sample_records, file, indent=2)
```

### Step 5 — Record extraction metadata
Capture endpoint name, extraction timestamp, page/cursor, number of records, response status and run identifier. Do not store credentials in metadata.

## 17. Common mistakes
| Mistake | Better practice |
|---|---|
| No timeout | Set connection/read timeouts |
| Ignoring status code | Call raise_for_status and handle expected codes |
| Assuming response is valid JSON | Check content type and handle decode errors |
| Reading only first page | Implement documented pagination |
| Infinite retry loop | Bound attempts and use backoff |
| Hardcoded API token | Use secret storage/environment |
| Losing progress | Persist page/cursor checkpoints safely |

## 18. Day 12 revision
- HTTP request and response have defined parts.
- Status codes guide response handling.
- Timeout prevents indefinite waiting.
- Pagination is required for multi-page datasets.
- Successful HTTP status does not prove schema correctness.
- Secrets must not be committed or logged.
