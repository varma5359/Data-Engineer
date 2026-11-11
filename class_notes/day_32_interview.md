# Day 32 Interview Guide — API-Based Data Ingestion

# 1. What is an API?
**Model answer:** An API is a defined interface through which software systems exchange information or invoke operations. An endpoint represents a resource or operation; request parameters, authentication and response structure are defined by its contract. A Data Engineer uses APIs to retrieve data from SaaS, marketplace, payment or logistics systems when direct database access or files are unavailable.

# 2. Explain request and response.
**Model answer:** A request includes method, URL, headers, optional query parameters/body and authentication. The response includes status code, headers and body. I inspect status and content type before parsing, then validate the body schema and business values. HTTP success alone does not prove data correctness.

# 3. What is pagination and why is it needed?
**Model answer:** Providers limit records per response for performance and quota control. Pagination retrieves subsequent pages until the documented terminal condition. I track page count, stable IDs and cursor progress, and reconcile final records. A one-page request can silently produce an incomplete dataset.

# 4. Offset versus cursor pagination?
**Model answer:** Offset pagination uses page/offset positions and can be simple, but changing source data between requests can shift rows and cause skips/duplicates. Cursor pagination uses a provider-issued continuation token and is often more stable. Cursor is opaque; I send it back exactly as provided and stop on the documented terminal marker.

# 5. How do you avoid an infinite pagination loop?
**Model answer:** I detect repeated cursors, set a maximum page/run budget, monitor progress and alert if the provider returns the same continuation token unexpectedly. I do not rely only on record count because an empty page may be valid or the terminal marker may be separate.

# 6. Explain HTTP status handling.
**Model answer:** 2xx generally indicates request success; 400 means request input needs correction; 401 authentication; 403 permission; 404 resource/endpoint; 429 rate limit; 5xx server issue. I classify errors and use bounded retries only for eligible transient failures. Provider documentation can add specific semantics.

# 7. How do you handle 429?
**Model answer:** I stop immediate requests, read Retry-After if supplied, respect provider quota guidance and retry within a finite attempt/time budget. I record rate-limit events and tune request concurrency. A tight loop would worsen throttling.

# 8. What is exponential backoff?
**Model answer:** Retry waits increase after repeated transient failures, such as 1, 2, 4, 8 seconds, with a cap and random jitter. It reduces pressure on a recovering service and avoids synchronized clients. Backoff is not used to fix permanent 400/401/403 errors.

# 9. Why is timeout not the same as retry?
**Model answer:** Timeout limits how long one request waits. Retry policy decides whether another attempt is appropriate, how many attempts, how long to wait and when to stop. Without a retry budget, a pipeline can run indefinitely.

# 10. How do you secure API credentials?
**Model answer:** Store credentials in environment variables or a secret manager, use least privilege and HTTPS, rotate credentials, redact authorization headers and never commit tokens or print them. If exposed, revoke/rotate immediately. The token should be scoped to the data and operation needed.

# 11. What does response.json() do?
**Model answer:** It parses response body text into Python objects—JSON object becomes dict, array becomes list, strings become str and null becomes None. It can raise a decode error if body is not valid JSON. I check status/content type and then validate expected top-level type and required fields.

# 12. Is valid JSON enough?
**Model answer:** No. JSON can parse but have missing fields, wrong types, duplicate IDs, unexpected nulls or invalid business values. I validate contract and record rules, and report rejected data or fail the page depending on impact.

# 13. How do you checkpoint API pagination?
**Model answer:** Persist the last safe cursor/page boundary only after the records from that page are durably written. If checkpoint fails, replay should be safe using idempotent keys. If checkpoint is written before data, a crash can skip records. Store run/window, cursor, page status and destination evidence.

# 14. What if API data changes during pagination?
**Model answer:** Offset pagination may shift. I prefer snapshot/cursor APIs if available, stable sorting and source versioning. Otherwise use overlap, dedup and reconciliation. I document consistency limitations and may need a follow-up query for late changes.

# 15. How do you design API extraction for large volume?
**Model answer:** Use provider filters/date windows, pagination, bounded processing, controlled concurrency within quota, streaming/page persistence, retry/backoff, checkpoints and idempotent destination writes. Monitor lag, page counts, duplicates, rate limits and failures. Avoid accumulating an unbounded list in memory.

# 16. How would you test without a real API?
**Model answer:** Use mock response fixtures for success, empty page, malformed JSON, missing field, 429, timeout, 503, repeated cursor and terminal page. Assert number of requests, output IDs, retry count and checkpoint behavior. Unit tests should not depend on network availability.

# 17. Explain your Day 32 project.
**Model answer:** “I built an API extractor that sends a documented request, checks HTTP status and response type, validates required fields, follows provider-issued cursors until completion, and writes records to a JSON output. I track page/record counts, detect repeated cursors and use bounded retries for transient errors. I protect credentials and test failure cases with mock responses. I reconcile IDs and output count before success.”

# 18. Follow-up interview prompts
- What if API returns 200 with an error object?
- What if page 5 fails after pages 1–4 were saved?
- What if next_cursor repeats?
- What if Retry-After is missing?
- How do you handle API version changes?
- What if the API returns 10,000 records per page?
- How do you detect missing data when counts are unknown?
- How do you ingest deletes or updated old records?
- What data do you avoid logging?
- How do you prove a rerun is safe?

# 19. Project defence framework
Answer each design question using: business requirement -> API contract -> implementation -> risk -> validation evidence -> recovery. Include actual request/response and test results from your own run.
