# Day 12 Interview Preparation: API Integration

**Mini-project:** API Data Extractor

## 1. Core questions

1. **What is REST?** An architectural style commonly used to expose resources over HTTP.
2. **What does GET do?** Retrieves a resource or collection.
3. **Why set a timeout?** To bound how long a connection or response can block a pipeline.
4. **What is pagination?** Splitting a large result into manageable pages.
5. **What is a bearer token?** A credential sent to authorize a request; it must be protected.
6. **What does raise_for_status do?** Raises an HTTP error for unsuccessful response status codes.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| 401 vs 403 | Authentication missing/invalid vs authenticated but not permitted |
| Page vs cursor pagination | Numeric page position vs opaque continuation token |
| Connect vs read timeout | Time to establish connection vs wait for response data |
| 4xx vs 5xx | Request/client issue vs server-side issue |
| HTTP success vs schema validity | Server accepted request vs payload matches expected contract |

## 3. Practical questions

7. **How do you extract all API records?** Follow the documented pagination method until the API signals no next page, while persisting progress.
8. **How do you handle 429?** Respect Retry-After or provider policy and use bounded backoff.
9. **How do you protect credentials?** Use secret management or secure runtime variables, least privilege and log redaction.
10. **How do you resume after failure?** Persist cursor/page checkpoint only after the corresponding page is safely stored.

## 4. Scenario questions

### Scenario A: API returns 200 but data is empty
Check filters, date range, pagination, response schema and whether empty is valid for that interval. Do not treat empty as success without business validation.

### Scenario B: API starts returning 429
Reduce request rate, respect Retry-After, apply backoff and monitor quota usage.

### Scenario C: Page 8 fails after pages 1–7 were saved
Resume from the last committed cursor/page. Ensure rerunning a page does not duplicate destination records.

## 5. Troubleshooting
- ConnectionError: network, DNS or endpoint reachability.
- Timeout: slow service or insufficient timeout.
- HTTPError: inspect status and safe response details.
- JSON decode error: response may be HTML, empty or malformed.
- Missing records: inspect pagination and filters.

## 6. Follow-up questions
- What is an idempotency key?
- What is rate limiting?
- Why might offset pagination miss records while data changes?
- What is exponential backoff?
- How do you handle API schema evolution?
- What metadata do you persist for audit?

## 7. Mini-project defence
Describe endpoint contract, authentication, pagination, validation, checkpointing, retries, raw response retention and duplicate-safe loading. Explain which details must come from the API documentation rather than assumptions.
