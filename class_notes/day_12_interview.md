# Day 12 Interview Preparation: API Integration

**Mini-project:** API Data Extractor

## 1. Core questions

1. **What is an API?**  
   A defined interface through which one application requests data or operations from another application.

2. **What is REST?**  
   An architectural style commonly used to expose resources over HTTP.

3. **What does GET do?**  
   Retrieves a resource or collection. It is commonly used in data extraction.

4. **What is an endpoint?**  
   A URL representing a resource or operation exposed by an API.

5. **What is an HTTP response?**  
   The server's reply containing a status code, headers and often a response body.

6. **Why inspect the status code?**  
   A request can receive a client or server error. Do not parse a response as valid data without checking the HTTP result.

7. **Why set a timeout?**  
   It bounds how long the client waits for a connection or response and prevents a pipeline from hanging indefinitely.

8. **What does `raise_for_status()` do?**  
   It raises an HTTP error for unsuccessful 4xx/5xx responses.

9. **What is a bearer token?**  
   A credential commonly sent in an Authorization header. It must be stored securely.

10. **What is pagination?**  
    Splitting a large result into multiple requests/pages so the client can retrieve all records in manageable portions.

11. **What is a rate limit?**  
    A restriction on how frequently a client can call an API.

12. **Does valid JSON mean valid business data?**  
    No. JSON parsing checks syntax and structure; schema and business rules must be checked separately.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| GET vs POST | Retrieve vs submit/create |
| Header vs query parameter | Request metadata vs endpoint filtering/input |
| 401 vs 403 | Authentication missing/invalid vs access not permitted |
| 404 vs 500 | Resource not found vs server-side failure |
| Timeout vs connection error | Wait exceeded vs connection could not be established |
| Page vs cursor pagination | Numeric page navigation vs opaque continuation token |
| Parsing vs validation | Decode response vs verify expected fields and rules |

## 3. Practical implementation questions

13. **How do you make a GET request?**  
    Use `requests.get(url, timeout=...)`, inspect the status and parse JSON only after checking the response.

14. **How do you pass query parameters?**  
    Supply a dictionary through the `params` argument. This lets Requests encode the URL correctly.

15. **How do you pass authentication?**  
    Read the token from an environment variable or secret manager and pass it in the documented Authorization header.

16. **How do you handle invalid JSON?**  
    Catch the JSON decoding error, record safe diagnostic metadata and quarantine the response.

17. **How do you implement pagination?**  
    Follow the provider's documented page, offset or cursor mechanism until the documented end condition is reached.

18. **What should you log for an API extraction?**  
    Endpoint identifier, request/page number, status code, duration, records received and safe error details. Do not log tokens or sensitive payloads.

## 4. Scenario questions

### Scenario 1: API returns HTTP 429

**Problem:** Your extractor is sending too many requests.

**Strong answer:**
- Respect the provider's rate-limit documentation.
- Inspect `Retry-After` if supplied.
- Apply bounded exponential backoff.
- Reduce request frequency or batch size.
- Resume from a saved page/cursor checkpoint.

### Scenario 2: API returns HTTP 200 but no records

**Problem:** The request succeeded but the response contains an empty list.

**Strong answer:** Check whether the filter is too restrictive, whether the dataset is genuinely empty and whether the response schema is as documented. A 200 status does not guarantee records exist.

### Scenario 3: The API response adds a new field

**Problem:** The upstream team adds `delivery_zone`.

**Strong answer:** Determine whether the field is optional or required, validate the existing contract, update downstream mappings deliberately and communicate any breaking change.

### Scenario 4: The extractor stops after the first 100 records

**Problem:** The API paginates results.

**Strong answer:** Implement the documented pagination mechanism, collect all pages, stop on the provider's termination condition and persist a checkpoint.

### Scenario 5: Network is temporarily unavailable

**Problem:** The request fails with a connection error.

**Strong answer:** Use a bounded retry policy for transient failures, record the failure and make the job report unsuccessful if it cannot complete. Do not silently publish partial data as complete.

## 5. Troubleshooting questions

19. **Why should you not retry every HTTP error?**  
    Some errors are permanent, such as invalid credentials or malformed requests. Retrying them wastes time and can worsen rate limits.

20. **What if the API returns HTML instead of JSON?**  
    Inspect status and content type, then handle the unexpected response without assuming JSON parsing will succeed.

21. **Why protect API credentials?**  
    A leaked token may allow unauthorized access. Use environment variables or secret-management systems and never commit secrets.

## 6. Mini-project explanation

"I built an API extraction workflow that requests data, checks the response, parses JSON, validates required keys, handles pagination and saves records. I practised the pagination logic with local mock responses so the workflow can be tested even without internet access. For production, I would use the provider's authentication, rate-limit and checkpoint rules."

## 7. Follow-up questions

- How do you prevent duplicate records when a page is retried?
- How do you detect incomplete extraction?
- What if the API changes its response schema?
- How do you handle a cursor that expires?
- How would you make the extractor restartable?
- Which information should not be included in logs?

## 8. Quick revision

- Check HTTP status before parsing.
- Always set a timeout.
- Use documented pagination and bounded retries.
- Protect tokens and sensitive response data.
- Validate schema and business rules after parsing.
