# Day 32 Interview Preparation: API-Based Data Ingestion

**Mini-project:** API Ingestion Pipeline  
**Target:** Fresher / Junior Data Engineer  
**Method:** Answer aloud first; compare with the model answer; then connect it to your project.

---

# 1. Core concepts — complete model answers

## 1. What is an API?

**Model answer:** An API is a documented interface through which systems request data or actions. For ingestion it defines endpoints, parameters, authentication, response structure and limits.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 2. What is REST?

**Model answer:** REST is an architectural style commonly used for web APIs: resources are addressed by URLs and HTTP methods describe operations. REST does not require JSON.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 3. What is an endpoint?

**Model answer:** An endpoint is a specific API address for a resource/action, such as /orders.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 4. Explain GET and POST.

**Model answer:** GET retrieves data and should not create side effects. POST commonly submits/creates. Do not retry POST blindly unless idempotency is supported.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 5. What does HTTP 200 mean?

**Model answer:** HTTP 200 means request succeeded at HTTP level. Validate response schema and business values; 200 can still contain zero records or an application error.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 6. Explain 401, 403 and 404.

**Model answer:** 401 means authentication missing/invalid; 403 access refused; 404 resource/endpoint absent. Fix cause instead of infinite retries.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 7. What is HTTP 429?

**Model answer:** 429 means rate limit exceeded. Respect Retry-After, reduce frequency and retry with controlled delay.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 8. What is pagination?

**Model answer:** Pagination splits a large collection. Follow next-page/cursor until documented end and avoid skip/repeat.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 9. Offset versus cursor pagination?

**Model answer:** Offset uses skip/limit and can shift as data changes. Cursor pagination follows a provider token and is often more stable.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 10. What is authentication?

**Model answer:** Authentication proves caller identity using API key/OAuth/signed request. Store securely and never log it.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 11. What is a rate limit?

**Model answer:** A request quota. Throttle and back off to avoid violating provider policy or overloading service.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 12. Why set a timeout?

**Model answer:** Timeout bounds waiting; otherwise a stalled network call can block the pipeline indefinitely.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 13. What should be retried?

**Model answer:** Retry transient network errors, selected 5xx and 429. Invalid request/credentials usually require correction.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 14. What is exponential backoff?

**Model answer:** Wait grows after failures, reducing pressure. Cap delay and add jitter.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 15. What is an API checkpoint?

**Model answer:** Saved progress such as cursor, page token, event ID or timestamp; save after returned data is durable.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 16. How ingest API updates?

**Model answer:** Use updated-since/change-feed/event endpoint; overlap recent window and reconcile when timestamp semantics are imperfect.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 17. What is an idempotency key?

**Model answer:** Unique request identifier recognized by provider to prevent duplicate effects on retry, if supported.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 18. Why validate JSON?

**Model answer:** JSON syntax parsing is not schema/business validation; check required keys, types, nulls, IDs and constraints.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 19. Streaming versus pagination?

**Model answer:** Pagination retrieves finite pages; streaming continuously delivers events and requires reconnect/replay/offset handling.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.

## 20. Design API ingestion.

**Model answer:** Secure credentials, request with timeout, check status, validate payload, paginate/throttle/retry, write idempotently, checkpoint after durable write, log lag/counts and reconcile.

**Follow-up:** Give a concrete example from API Ingestion Pipeline; state what evidence proves your answer.


---

# 2. Scenario questions

## 21. How handle changing page size?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 22. What if next_cursor is null?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 23. How protect token in Git?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 24. How backfill 90 days?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.

## 25. How prevent duplicate pages?

**Strong answer structure:** clarify requirement/source behavior; identify data-loss/duplicate/quality/security/performance risk; state exact control; explain failure/recovery; name verification metric. Do not say only “I will handle it”—describe the boundary, key, rule, retry/checkpoint and test.


---

# 3. Project defence — API Ingestion Pipeline

## 90-second project answer

“My project is API Ingestion Pipeline. It solves this business problem: A marketplace API returns limited pages, may fail temporarily and requires credentials. Build a polite, restartable extractor. I define the source contract and record identity, read data in controlled units, validate structure/business rules, process valid records, isolate invalid data, persist safely and verify counts/keys/values. I record run metadata and progress only after durable success. I test duplicates, boundary cases and failures so reruns are safe. For production I would add managed secrets, least-privilege access, automated tests, scheduling, monitoring/alerts, scale tests and a runbook.”

## 1. Why choose this project?

**Answer:** It models a recurring Data Engineering responsibility and demonstrates more than a happy-path script.

## 2. What is the source contract?

**Answer:** Documented fields, types, required values, identity, delivery behavior and change expectations; I validate before processing.

## 3. How prove output correctness?

**Answer:** Reconcile counts, accepted/rejected totals, unique keys, business totals and source boundary.

## 4. What if job fails halfway?

**Answer:** Inspect last durable safe point, replay uncertain input idempotently, reconcile, then advance progress.

## 5. How prevent duplicates?

**Answer:** Stable business/event key, unique constraint or idempotency token and deterministic deduplication.

## 6. Largest limitation?

**Answer:** Classroom version uses small data and simplified infrastructure; production needs scale tests, secret management, concurrency and monitoring.

## 7. First improvement?

**Answer:** Automated boundary/failure tests, structured run metadata, alerting and repeatable deployment/runbook.

---

# 4. Practical interview exercise

- Draw source, validation, processing, destination, rejected path, checkpoint and monitoring. Label when progress becomes safe.
- Job processed 800 records, persisted 700, then crashed. Explain state inspection, retry, duplicate prevention and success checks.
- Compare classroom and production: scale, security, concurrency, retries, contract, observability and cost.
- Identify in your code: validation, error handling, persistence, progress and correctness verification. If absent, state limitation and next step.

# 5. Interview traps

- “HTTP 200 means correct data.”
- “I retry every error.”
- “I use timestamp” without ties/late records/deletes.
- “I remove duplicates” without identity key.
- “Job succeeded” without counts/reconciliation.
- “Exactly once” without mechanism/failure window.
- “I log everything” including secrets/PII.
- “Production-ready” based only on a local happy-path run.

# 6. Short revision answers

- **What is an API?** An API is a documented interface through which systems request data or actions. For ingestion it defines endpoints, parameters, authentication, response structure and limits.
- **What is REST?** REST is an architectural style commonly used for web APIs: resources are addressed by URLs and HTTP methods describe operations. REST does not require JSON.
- **What is an endpoint?** An endpoint is a specific API address for a resource/action, such as /orders.
- **Explain GET and POST.** GET retrieves data and should not create side effects. POST commonly submits/creates. Do not retry POST blindly unless idempotency is supported.
- **What does HTTP 200 mean?** HTTP 200 means request succeeded at HTTP level. Validate response schema and business values; 200 can still contain zero records or an application error.
- **Explain 401, 403 and 404.** 401 means authentication missing/invalid; 403 access refused; 404 resource/endpoint absent. Fix cause instead of infinite retries.
- **What is HTTP 429?** 429 means rate limit exceeded. Respect Retry-After, reduce frequency and retry with controlled delay.
- **What is pagination?** Pagination splits a large collection. Follow next-page/cursor until documented end and avoid skip/repeat.

# 7. Readiness checklist

- [ ] Core answers explained without reading.
- [ ] One business example connected.
- [ ] End-to-end flow drawn.
- [ ] Failure and recovery explained.
- [ ] Key/checkpoint choice defended.
- [ ] Correctness checks named.
- [ ] Limitations stated honestly.
- [ ] Production improvements discussed.

**Interview goal:** explain decision + reason + failure mode + verification, not just tool name.
