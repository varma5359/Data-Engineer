# Day 30 Interview Preparation: ETL and Data Extraction Patterns

**Phase:** 4 — Data Processing and Pipelines  
**Mini-project:** Incremental Extraction Pipeline  
**Level:** Beginner to junior Data Engineer

---

# 1. Core interview questions and complete answers

## 1. What is ETL?

ETL stands for Extract, Transform and Load. A pipeline reads data from source systems, transforms it into the required structure or business meaning, and loads it into a destination such as a database or warehouse.

## 2. What is data extraction?

Data extraction is the process of reading data from a source system. Sources can include relational databases, files, APIs, application logs and object storage.

## 3. What is the difference between ETL and ELT?

In ETL, data is transformed before it is loaded into the target. In ELT, data is first loaded into the target platform and transformed there. ETL may use Python or a processing engine before loading, while ELT often uses warehouse SQL or distributed compute after landing raw data.

## 4. What is full extraction?

Full extraction reads the complete required dataset each time the job runs. It is useful for small reference datasets, complete snapshots or rebuilds. It can become expensive when the source is large and only a small portion changes.

## 5. What is incremental extraction?

Incremental extraction reads only records that are new or changed since a previously saved position. It reduces repeated source reads but requires reliable change tracking and recovery logic.

## 6. What is a watermark?

A watermark is a saved progress value that tells the next extraction run where to continue. It may be a timestamp, increasing ID, sequence number or compound position.

## 7. Is a watermark always a timestamp?

No. It can be a timestamp, sequence number, increasing key, source log position or compound cursor. The correct choice depends on what the source can guarantee.

## 8. What is the difference between created_at and updated_at?

created_at records when a row was first created. updated_at records the most recent modification. If a pipeline must capture updates to existing rows, filtering only on created_at is insufficient.

## 9. Why is updated_at useful for incremental extraction?

When the source reliably updates this column on every insert and update, it can identify new and modified rows. The pipeline can query rows newer than the saved watermark.

## 10. What is the difference between event time and processing time?

Event time is when the business event happened. Processing time is when a system processed it. A record can arrive late, so its event time may be old even though its processing or update time is recent.

## 11. What is a late-arriving record?

It is a record that becomes available later than expected. If extraction filters only on an old event timestamp, it may be missed. A reliable update timestamp, CDC feed or lookback strategy may be needed.

## 12. What is a lookback window?

A lookback window intentionally re-reads a recent time range before the previous watermark. It helps capture delayed records and timestamp edge cases. It can produce duplicates, so the destination must support safe retries or deduplication.

## 13. Why can timestamp-only extraction miss records?

Multiple records may share the same timestamp. If the first run saves that timestamp and the next query uses a strict greater-than comparison, other rows with the same timestamp may be skipped.

## 14. How can a compound watermark solve timestamp ties?

Use timestamp plus a stable unique tie-breaker such as order_id. The next query selects rows with a later timestamp, or the same timestamp and a greater ID. The query must order by the same fields.

## 15. What is an exclusive boundary?

An exclusive condition such as updated_at > watermark returns only values strictly greater than the watermark. A row equal to the watermark is excluded.

## 16. What is an inclusive boundary?

An inclusive condition such as updated_at >= watermark includes rows equal to the watermark. It can re-read records, so the destination must handle duplicates safely.

## 17. Does an increasing ID detect updates?

Usually not. It finds records with IDs greater than the last ID. If an existing row changes but keeps its ID, an ID-only query will not detect that update.

## 18. What is a snapshot?

A snapshot is a representation of the source dataset at a particular time. Comparing snapshots can help identify changes, but a snapshot is not inherently a detailed change log.

## 19. What is Change Data Capture?

CDC captures inserts, updates and deletes from source changes. Depending on the system, it may read database logs, replication streams, triggers or source-specific change feeds.

## 20. What is a checkpoint?

A checkpoint is persisted processing progress. The next run reads it to determine where to continue. A checkpoint may store the last timestamp and unique key successfully processed.

## 21. When should a pipeline update its checkpoint?

Only after the corresponding data has been successfully processed and committed. Saving progress before the destination write can cause records to be skipped if the load fails.

## 22. What happens if the pipeline fails after loading data but before saving the checkpoint?

The next run may extract those records again. That is why the destination should support safe retries through stable keys, deduplication or upsert behavior. Reprocessing is generally safer than skipping unprocessed data.

## 23. How do you make incremental extraction retry-safe?

Use a stable record key, deterministic extraction order, a checkpoint tied to successful processing, idempotent destination writes and reconciliation. Re-running the same batch should not create unintended duplicates or overwrite valid newer data.

## 24. How do deletes get captured?

A normal query against the current source table cannot return a row that has already been deleted. Use CDC delete events, soft-delete flags, tombstones or periodic snapshot comparison.

## 25. Why avoid SELECT * in extraction jobs?

It reads unnecessary columns, transfers more data and can make the pipeline fragile when the source schema changes. Select only the fields required by the downstream process.

## 26. How can extraction affect a source database?

Large or poorly indexed extraction queries consume CPU, I/O, memory and network resources. Schedule appropriately, filter efficiently, read in manageable batches and consider replicas or CDC for heavy workloads.

## 27. Why is deterministic ordering important?

If the pipeline saves the last extracted row as its checkpoint, it must know exactly which row is last. ORDER BY on the checkpoint fields creates a consistent order.

## 28. Why does timezone consistency matter?

A local timestamp and a UTC timestamp may represent different instants. Inconsistent timezone assumptions can cause records to be extracted too early, too late or more than once. Standardize timestamp storage and comparison.

## 29. Is a job's start time a safe watermark?

Not necessarily. The start time does not prove that all source records before that time were extracted and loaded. The watermark should be based on successfully processed source positions.

## 30. What is the difference between extraction and ingestion?

Extraction is the act of reading data from a source. Ingestion is the broader process of bringing data into a platform, often including discovery, extraction, landing, metadata, validation and operational tracking. Organizations sometimes use the terms differently, so explain the scope in context.

---

# 2. SQL and scenario questions

## 31. Write a query for a full extraction

```sql
SELECT
    order_id,
    customer_id,
    amount,
    updated_at
FROM orders
ORDER BY order_id;
```

This returns the selected columns from all rows.

## 32. Write a query for timestamp-based incremental extraction

```sql
SELECT
    order_id,
    customer_id,
    amount,
    updated_at
FROM orders
WHERE updated_at > TIMESTAMP '2026-11-08 10:00:00'
ORDER BY updated_at, order_id;
```

This returns rows strictly later than the stated timestamp. The timestamp is illustrative.

## 33. Write a compound-watermark query

```sql
SELECT
    order_id,
    updated_at
FROM orders
WHERE
    updated_at > TIMESTAMP '2026-11-08 10:00:00'
    OR (
        updated_at = TIMESTAMP '2026-11-08 10:00:00'
        AND order_id > 1002
    )
ORDER BY updated_at, order_id;
```

It returns rows later than the saved timestamp and rows at the same timestamp with a greater ID.

## 34. A pipeline extracts 1,000 rows and fails while loading. What do you do?

I would inspect the transaction and batch design. If the batch was not committed, I would roll it back and retry. If some earlier batches were committed, I would use stable keys and checkpoints to resume safely. I would reconcile source and target counts and capture the failed records and error details.

## 35. A row was updated but your pipeline did not capture it. What could be wrong?

Possible causes include:
- The query filters on created_at rather than updated_at.
- The source does not update updated_at consistently.
- The saved watermark is ahead of the row.
- Timestamp precision or timezone conversion is inconsistent.
- The source update was not visible when the query ran.
- The pipeline has a boundary or tie-handling bug.

I would inspect the source row, update mechanism, checkpoint, query predicate and execution timing.

## 36. Two records have the same updated_at value. How do you handle them?

Use a stable unique tie-breaker such as a primary key. Store both the timestamp and last key in the checkpoint, compare both in the extraction predicate, and order by both fields.

## 37. Your pipeline reprocesses 15 minutes of records every run. Is that wrong?

Not necessarily. It may be an intentional lookback window to capture late-arriving data. It is safe only if repeated records are handled correctly and the overlap size is justified by source delay behavior.

## 38. The source deletes a customer row. How will an incremental query detect it?

A SELECT from the current table will not return the deleted row. I would use CDC delete events, a soft-delete column, tombstones or periodic snapshot reconciliation.

## 39. The source has no updated_at column. What alternatives exist?

Possible options:
- CDC or database log-based capture.
- Source change tables or audit logs.
- Increasing sequence numbers if only inserts matter.
- Periodic full snapshots and comparison.
- Source-provided modification tokens or API cursors.

The choice depends on source capabilities and whether updates and deletes must be captured.

## 40. How would you reduce source database load?

Select only required columns, use efficient filters, ensure suitable indexes, extract in controlled batches, schedule outside peak periods where possible, inspect query plans and consider a read replica or CDC architecture.

---

# 3. Practical project defence

Be prepared to explain the Day 30 Incremental Extraction Pipeline.

### Question 1: What business problem does the project solve?

It avoids repeatedly reading the entire order table when only a subset of records is new or changed. It demonstrates how a pipeline can resume from a saved extraction position.

### Question 2: Why did you use a watermark?

The watermark records the last successfully processed source position. The next run uses it to identify the next batch.

### Question 3: Why did you use updated_at?

The pipeline needs to capture both new orders and changes to existing orders. A trustworthy updated_at field can identify both.

### Question 4: Why did you include order_id in the checkpoint?

Two rows can have the same updated_at timestamp. A unique order_id provides a tie-breaker so the pipeline can resume without skipping a row at the boundary.

### Question 5: Why is ORDER BY required?

The checkpoint is based on the last row in the result. Deterministic ordering ensures that the last row represents the highest processed position under the checkpoint rule.

### Question 6: Why did the project not update the checkpoint immediately?

The extraction exercise does not yet load data into a separate destination. Updating the checkpoint before a successful load would be unsafe. The checkpoint must move only after corresponding data is committed.

### Question 7: What would you add for production?

I would add a destination table, transactional load and checkpoint update where possible, safe upsert logic, retry handling, batch processing, structured logs, metrics, source-to-target reconciliation and monitoring.

---

# 4. Interview traps to avoid

1. Saying incremental extraction always means filtering on created_at.
2. Claiming an increasing ID detects updates to existing rows.
3. Saving the current system time as the checkpoint without proving completeness.
4. Ignoring timestamp ties.
5. Forgetting late-arriving records.
6. Assuming a normal SELECT can detect hard deletes.
7. Using >= without considering duplicate processing.
8. Using > without considering records sharing the watermark timestamp.
9. Treating event time and updated time as the same thing.
10. Claiming exactly-once processing without explaining transaction and retry behavior.
11. Using SELECT * for every extraction.
12. Ignoring source-system load.
13. Failing to use deterministic ORDER BY.
14. Advancing a checkpoint before a successful target commit.

---

# 5. Short revision answers

| Term | Interview-ready explanation |
|---|---|
| ETL | Extract, transform and load data |
| Full extraction | Read the complete required dataset |
| Incremental extraction | Read new or changed data since a saved position |
| Watermark | Saved source progress position |
| Checkpoint | Persisted state used to resume processing |
| updated_at | Last modification timestamp |
| CDC | Capture insert, update and delete changes |
| Lookback | Re-read an overlap period to catch delayed records |
| Compound watermark | Timestamp plus unique tie-breaker |
| Idempotency | Safe repeat execution without unintended repeated effects |
| Reconciliation | Compare source and destination evidence for correctness |

---

# 6. Readiness checklist

- [ ] I can explain ETL and ELT.
- [ ] I can distinguish full and incremental extraction.
- [ ] I can explain watermark and checkpoint.
- [ ] I know why updated_at may be better than created_at.
- [ ] I can explain timestamp boundary conditions.
- [ ] I can write a compound-watermark query.
- [ ] I understand late-arriving records and lookback.
- [ ] I can explain how CDC captures deletes.
- [ ] I know why deterministic ordering matters.
- [ ] I can explain why a checkpoint must follow successful processing.
- [ ] I can discuss source-system performance.
- [ ] I can defend the Incremental Extraction Pipeline project.

**Interview goal:** Explain not only how to extract records, but also how to avoid missed records, handle retries and prove that the pipeline has processed the correct data.
