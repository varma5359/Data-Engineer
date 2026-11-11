# Day 33 Interview Guide — Database Ingestion

# 1. What is database ingestion?
**Model answer:** Database ingestion is extracting or transferring records from a database source into another processing/storage system. It includes source-safe queries, connection management, bounded fetches, type handling, durable writes, progress tracking and reconciliation—not only SELECT.

# 2. Why use a read-only source account?
**Model answer:** It prevents accidental INSERT/UPDATE/DELETE from the extractor and follows least privilege. It does not prevent expensive reads, so I also control query scope, timeout, batch behavior, scheduling and source load.

# 3. Explain fetchone, fetchmany and fetchall.
**Model answer:** fetchone returns one row; fetchmany returns a bounded group; fetchall returns all remaining rows. For large data I use batches or server-side cursor behavior and stream to output. I verify driver semantics because ordinary client-side cursors may buffer results.

# 4. How do you prevent SQL injection?
**Model answer:** I pass values through driver placeholders rather than concatenating them into SQL. For Psycopg 3, %s placeholders receive values separately. Dynamic identifiers cannot be safely passed as ordinary value parameters; use a controlled allow-list or identifier composition API.

# 5. What is a watermark?
**Model answer:** A saved source boundary used to resume extraction. An ID watermark works for append-only records, but misses updates/deletes to existing IDs. Mutable sources need updated_at, compound cursor, CDC or reconciliation strategy.

# 6. Why timestamp plus ID?
**Model answer:** Multiple records can share the same timestamp. A unique ID tie-break makes ordering and continuation deterministic. The predicate must handle timestamp equality and ID comparison; timezone and precision must match source semantics.

# 7. What is batch fetching?
**Model answer:** Fetching a bounded number of rows at a time. It reduces application-level memory and supports progress/recovery. Batch size balances memory, network/driver overhead and destination throughput; measure rather than choose blindly.

# 8. Does fetchmany guarantee low memory?
**Model answer:** Not necessarily. Driver cursor behavior matters; a client-side cursor may buffer results. For very large PostgreSQL extracts, evaluate named/server-side cursor or streaming methods and test memory under realistic volume.

# 9. How do you protect an operational database?
**Model answer:** Read-only access, explicit columns, source-side filters, indexed predicates, query timeout, bounded batches, approved replica/schedule, query-plan review and source monitoring. Coordinate large backfills with database owners.

# 10. Explain replica lag.
**Model answer:** A replica may not yet have applied changes committed on the primary. A watermark captured on primary and read on a lagging replica can skip data. Monitor replication position/lag and use safe overlap or consistent CDC boundary.

# 11. What is snapshot consistency?
**Model answer:** A consistent snapshot represents data at a defined point under database isolation semantics. Multiple batches from a normal query may observe changes during extraction depending on transaction/cursor/isolation. I state the guarantee, choose snapshot/CDC when required and reconcile.

# 12. What is checkpoint ordering?
**Model answer:** Destination write must become durable before progress is checkpointed. If checkpoint is written first and destination fails, records can be skipped. If destination commits and checkpoint fails, replay is safe when writes are idempotent.

# 13. How do you recover from crash during extraction?
**Model answer:** Identify last durable batch/watermark, inspect destination partial state, reconnect, replay from safe boundary, deduplicate/upsert, then reconcile. For files, write temporary output and publish only after validation; avoid appending reruns blindly.

# 14. Explain your project.
**Model answer:** “I extracted TechStore orders from PostgreSQL using explicit columns and parameterized query. I fetched in batches and streamed to CSV, recorded source count/upper boundary, and verified output count and keys. I used read-only access and a temp file before publishing. I understand an ID watermark only captures new append-only rows, so updates/deletes need a change mechanism or reconciliation.”

# 15. Follow-up prompts
- How would you extract 100 million rows?
- How would you capture deletes?
- What if the source updates an old order?
- What if a batch write succeeds but checkpoint fails?
- What if checkpoint succeeds before target write?
- How do you test timestamp ties?
- How do you use an index?
- How would you coordinate a full backfill?
- What if replica is 20 minutes behind?
- How do you prove no row was lost?

# 16. Project defence
Explain requirement, source risk, extraction strategy, batch behavior, progress/checkpoint, validation and recovery. Show actual counts and tests; do not claim exactly-once or snapshot consistency without evidence.
