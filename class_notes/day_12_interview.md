# Day 12 — APIs and Pagination | Interview

**What is REST?** An architectural style commonly using HTTP resources and methods for service interaction.

**What does 429 mean?** Rate limit exceeded; respect Retry-After and apply controlled backoff.

**Why pagination?** Limits response size and server/client resource use; the extractor must retrieve every page.

**Offset vs cursor pagination?** Offset uses positions/page numbers and may shift as data changes; cursor uses a continuation marker and is often more stable for changing datasets.

**How handle API failure?** Timeout, classify status, bounded retries for transient errors, log context, checkpoint, and avoid duplicate destination writes.

Scenario: extraction stops after page 1. Inspect pagination metadata/next link, iterate until no next page, validate counts, and make loading idempotent. Never expose tokens in logs.