# Day 32 Project Workbook — Marketplace API Extractor

**Project goal:** Build a controlled API extraction using a public demo endpoint for practice and a documented/authorized provider endpoint for real work.

## Step 0 — Environment
Terminal:
```bash
python -m pip install requests
```
Notebook:
```python
import requests
import json
from pathlib import Path
```
Expected: no output; imports complete without error.

## Step 1 — One request
```python
response = requests.get(
    "https://jsonplaceholder.typicode.com/posts/1",
    timeout=10
)
print(response.status_code)
```
Expected: 200 if public demo endpoint is reachable. If network is unavailable, note that and continue using mock responses below.

## Step 2 — Inspect before parse
```python
print("Status:", response.status_code)
print("Content type:", response.headers.get("Content-Type"))
print("Body preview:", response.text[:120])
```
Check status and content type. A proxy/error page may return HTML even when you expected JSON.

## Step 3 — Parse and validate
```python
response.raise_for_status()
payload = response.json()

required = {"userId", "id", "title", "body"}
missing = required - set(payload.keys())

print("Payload type:", type(payload).__name__)
print("Missing fields:", missing)
print("Post ID:", payload.get("id"))
```
Expected for /posts/1: dict, missing fields empty, ID 1.

## Step 4 — Query list
```python
response = requests.get(
    "https://jsonplaceholder.typicode.com/posts",
    params={"userId": 1},
    timeout=10
)
response.raise_for_status()
posts = response.json()

print("Type:", type(posts).__name__)
print("Count:", len(posts))
print("First ID:", posts[0]["id"] if posts else None)
```
Expected demo response: list, 10 posts, first ID 1. Network/provider behavior can vary; inspect actual response.

## Step 5 — Simulate cursor pagination offline
```python
mock_pages = {
    None: {"items": [{"id":"A"},{"id":"B"}], "next_cursor":"next-1"},
    "next-1": {"items": [{"id":"C"},{"id":"D"}], "next_cursor":None}
}

all_items = []
cursor_token = None
page_count = 0
seen_tokens = set()

while True:
    if cursor_token in seen_tokens:
        raise RuntimeError("Cursor repeated")
    if cursor_token is not None:
        seen_tokens.add(cursor_token)

    page_count += 1
    if page_count > 10:
        raise RuntimeError("Page limit exceeded")

    page = mock_pages[cursor_token]
    all_items.extend(page["items"])
    cursor_token = page["next_cursor"]

    if cursor_token is None:
        break

print("IDs:", [item["id"] for item in all_items])
print("Pages:", page_count)
```
Expected:
```text
IDs: ['A', 'B', 'C', 'D']
Pages: 2
```

## Step 6 — Simulate a repeated cursor
Change the second mock page's next_cursor to "next-1". The seen-token guard should raise RuntimeError. This prevents an infinite loop. Restore None after the drill.

## Step 7 — Simulate HTTP failure classification
```python
status_examples = [200, 401, 403, 404, 429, 503]

for status in status_examples:
    if status == 200:
        action = "validate payload"
    elif status == 429:
        action = "wait according to quota"
    elif status in [500, 503]:
        action = "bounded retry policy"
    else:
        action = "correct request or permission"
    print(status, "->", action)
```
Expected:
```text
200 -> validate payload
401 -> correct request or permission
403 -> correct request or permission
404 -> correct request or permission
429 -> wait according to quota
503 -> bounded retry policy
```
This is a teaching classification; production decisions should follow provider docs.

## Step 8 — Save and reload data
```python
output_path = Path("../output/posts.json")
output_path.parent.mkdir(parents=True, exist_ok=True)

with open(output_path, "w", encoding="utf-8") as file:
    json.dump(posts, file, indent=2)

with open(output_path, "r", encoding="utf-8") as file:
    saved_posts = json.load(file)

print("Saved records:", len(saved_posts))
print("IDs unique:", len({post["id"] for post in saved_posts}) == len(saved_posts))
```
Expected count equals the original posts count; unique IDs should be True for this endpoint.

## Step 9 — Add a run summary
Record:
- request count;
- page count;
- records received;
- unique/duplicate IDs;
- invalid records;
- retry count;
- last safe cursor;
- output path;
- status and error code.

Do not include token or sensitive response content in logs.

## Step 10 — Failure drills
- Network unavailable: use mock pages and explain that a live test is not a code failure.
- 401: do not retry repeatedly; fix credentials.
- 429: wait; respect provider guidance.
- JSON parse error: inspect body/content type; quarantine or fail.
- Missing required key: fail page or reject record under contract.
- repeated cursor: stop.
- crash before write: replay page.
- crash after write before checkpoint: idempotently replay.
- checkpoint before write: explain potential skipped data and correct ordering.

## Acceptance checklist
- [ ] Request status inspected before parse.
- [ ] JSON structure and record fields validated.
- [ ] Query parameters used correctly.
- [ ] Pagination completed to documented terminal marker.
- [ ] Repeated cursor and max-page safety tested.
- [ ] Retry classification bounded and explained.
- [ ] Credentials protected.
- [ ] Output reload/count/key checks pass.
- [ ] Run summary and recovery strategy documented.
