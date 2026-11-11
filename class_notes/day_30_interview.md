# Day 30 Interview Guide — ETL and Data Extraction (Complete Model Answers)

**Audience:** Freshers and junior Data Engineers  
**Use with:** [Day 30 Class Notes](day_30_notes.md)  
**Interview rule:** Give the business context, explain your design, prove correctness and discuss failure/recovery. A definition alone is not enough.

---

# 1. What is ETL? Explain with a real example.

**Model answer:** ETL means Extract, Transform and Load. Extract collects data from a source, Transform applies required cleaning or business rules, and Load writes the result to a destination. For example, TechStore extracts yesterday's orders from PostgreSQL, standardizes status and validates amount, then loads the curated rows into an analytics table. ETL is a repeatable data workflow, not simply copying a file.

**Follow-up: Is every pipeline ETL?** Not necessarily. Some systems use ELT: extract and load raw data first, then transform inside the destination platform. The choice depends on destination capabilities, data volume, governance and processing cost.

**Common trap:** Saying ETL always transforms before loading. That describes ETL, not ELT.

# 2. What is data extraction?

**Model answer:** Extraction is reading a defined set of data from a source system and making it available to a pipeline. I define which fields, rows, time range and consistency boundary are needed. I use explicit columns, source-side filters, deterministic ordering where useful, bounded reads and a least-privilege account. Then I validate row counts, keys and boundaries. Extraction must protect the operational source as well as produce correct data.

# 3. Explain full load and incremental load.

**Model answer:** Full load reads the complete selected dataset each run. It is straightforward and useful for small tables, initial loads or rebuilding a target, but repeated full scans can increase source load and processing time.

Incremental load reads only new or changed data after a saved boundary. It reduces repeated work, but requires a trustworthy change mechanism—such as a change timestamp, compound cursor, CDC or source version. I also design overlap/replay and reconciliation because late changes can be missed.

**Example:** A full load reads all 2 million orders every day. An incremental run may read only today's changes. The actual savings depend on how many records change and the correctness of change capture.

# 4. What is a watermark? Explain its limitation.

**Model answer:** A watermark is a saved progress boundary used to identify what a pipeline has safely processed. For append-only data, a last order ID can work: read IDs greater than the last committed ID. But it cannot detect an update to an older order or a hard delete. For mutable records, I prefer a reliable updated timestamp with a tie-breaking unique key, CDC or another source change mechanism. I also account for late events and checkpoint only after the destination write is durable.

**Interviewer trap:** “order_id > last_id guarantees all changes.” It does not if old rows can change.

# 5. Why do you use explicit columns instead of SELECT *?

**Model answer:** Explicit columns make the data contract visible and stable. If a source table gains a column, SELECT * changes the result shape automatically. It may increase transfer size or break downstream code. Explicit selection also avoids reading unnecessary fields and reduces accidental exposure of sensitive columns. I still inspect source schema and version the contract.

# 6. Why should extraction queries have deterministic ordering?

**Model answer:** Deterministic ordering makes batches and results repeatable. If I paginate or checkpoint, I need a stable order with a unique tie-breaker. ORDER BY a non-unique timestamp alone can produce ambiguous ordering among rows with the same timestamp. I would order by timestamp plus unique ID. Ordering does not itself make a query incremental or consistent; it supports predictable processing.

# 7. What is the difference between fetchone, fetchmany and fetchall?

**Model answer:**
- fetchone returns one row (or None when no row remains).
- fetchmany(size) returns a bounded group of rows.
- fetchall returns all remaining rows.

For a small classroom dataset, fetchall is convenient. For a large source, I use fetchmany or server-side/batched cursor behavior appropriate to the driver, process each batch and avoid accumulating the entire dataset in memory.

# 8. Why can fetchall cause memory problems?

**Model answer:** It materializes every remaining result row in Python memory. Python objects also have overhead beyond raw database bytes. A million records can consume much more memory than the source file size suggests. I use bounded batches, stream results to a destination, monitor memory and test with realistic volumes.

# 9. How do you safely connect Python to PostgreSQL?

**Model answer:** I install the correct driver, keep host/database/user/password in environment variables or a secret manager, create a connection, execute parameterized SQL, consume results and close resources. I do not commit credentials, print secrets or concatenate untrusted values into SQL. I use a read-only source account and suitable timeout/permissions in production.

# 10. What is parameterized SQL and why is it important?

**Model answer:** Parameterized SQL separates the query structure from the values supplied to it. The driver sends values safely according to its protocol, helping prevent SQL injection and handling data types correctly. For Psycopg 3, values use driver placeholders such as %s and are passed separately as a tuple/list. I never put quotes around the placeholder or concatenate user input into SQL.

Example:
```python
cursor.execute(
    "SELECT order_id FROM orders WHERE customer_id = %s",
    (customer_id,)
)
```

The comma makes a one-element tuple. Placeholder syntax is driver-specific.

# 11. How do you extract a day's data without missing fractional timestamps?

**Model answer:** I use an inclusive start and exclusive next-day end. For 10 November, created_at >= midnight on 10 November and created_at < midnight on 11 November. This includes timestamps such as 23:59:59.500 and excludes the next day's midnight. I define timezone explicitly and use consistent timestamp types.

# 12. How do you prove an extraction is correct?

**Model answer:** I use multiple checks:
1. Source query count.
2. Python fetched count.
3. Output file/table count.
4. Unique business key count.
5. Date/watermark boundary checks.
6. Selected totals or key-set reconciliation where appropriate.
7. Schema and null/invalid checks.

Counts alone are insufficient: a file can have the right count but wrong or duplicated keys. The validation depends on the business contract.

# 13. What happens if the source query returns zero rows?

**Model answer:** Zero rows can be a valid business result, such as no orders on a holiday. I distinguish an expected empty window from an unexpected zero caused by wrong date, source, permissions or schema. I check the run's business date, query boundary, source health and baseline. I don't automatically treat zero as success or failure; the pipeline's rule and monitoring decide.

# 14. How would you recover if the process crashes after writing the CSV?

**Model answer:** First determine whether the file is complete and whether the run was validated/published. I avoid exposing partial output by writing to a temporary file, closing and validating it, then atomically renaming it where supported. On restart, I use run identity and source boundary to determine whether to safely replace/replay. I never blindly append a rerun to the same file because it may duplicate records.

# 15. What is reconciliation?

**Model answer:** Reconciliation compares independent evidence to verify the pipeline result. For an extract, I compare source count, fetched count and output count; check unique keys and date boundaries; and compare important aggregates or key sets when required. I explain known exclusions and rejected records. Reconciliation helps detect silent loss, duplication or transformation mistakes.

# 16. What risks does extraction create for an operational database?

**Model answer:** A large scan or expensive query can consume CPU, I/O, memory or locks and slow customer-facing transactions. I use source-side filters, explicit columns, suitable indexes, bounded batches, read replicas where approved, query timeouts and monitoring. I coordinate with database owners for high-volume/backfill work. I don't assume a replica is current; I consider replication lag.

# 17. What is the difference between extraction and transformation?

**Model answer:** Extraction retrieves the selected source data while preserving its meaning. Transformation applies documented rules such as type conversion, status mapping, deduplication or derived calculations. I keep raw data/provenance when possible so a changed business rule can be replayed without losing the original source.

# 18. How would you design a daily orders extraction pipeline?

**Model answer (project defence):**
1. Confirm business date/timezone, fields, source and expected delivery deadline.
2. Inspect schema and identify stable key/change column.
3. Create read-only credentials and secret configuration.
4. Run explicit, bounded SQL.
5. Fetch in batches appropriate to volume.
6. Write to a temporary output with run ID/date.
7. Validate count, keys, schema and time boundaries.
8. Publish final file only after checks pass.
9. Record status, counts, duration, watermark and output path.
10. Alert on failure or unexpected empty output.
11. Re-run safely from the defined boundary and reconcile.

# 19. What is idempotency in an extraction pipeline?

**Model answer:** Idempotency means repeating the same run does not create unintended duplicate effects. For a daily file, deterministic output naming plus safe replace/publish can make reruns idempotent. For database loads, unique keys/upserts or transactionally deduplicated staging are common. I test by executing the same input twice and checking target state.

# 20. What would you monitor?

**Model answer:** Run status, duration, source/output row counts, rejects, duplicate keys, freshness, retries, source query duration, memory, destination write time and last successful watermark. I alert on actionable conditions: missed deadline, unexpected zero rows, abnormal count, excessive rejects or lag. Logs should include run ID and safe error context, never credentials or unnecessary PII.

# 21. Project interview: “Tell me about your extraction project.”

**Strong beginner answer:**  
“I built a TechStore daily order extractor. It reads a defined business-day range from PostgreSQL using explicit columns and a read-only connection. Python executes the query and writes a CSV. I validate source/fetched/file counts, unique order IDs and date boundaries before marking the run successful. I keep credentials in environment variables. I also considered empty days, timestamp precision, reruns and the limitation of ID watermarks for updated records. The project demonstrates extraction, safe access, output generation and data correctness checks.”

**Follow-up prompts to practise:**
- Why did you choose the date range this way?
- What if a record is updated tomorrow?
- What if the job crashes halfway?
- How do you know the CSV has no duplicates?
- What would change for 10 million rows?
- How would you load this into a warehouse?

# 22. Final answer checklist

Before an interview, make sure you can:
- explain each term without memorized jargon;
- draw the pipeline flow;
- explain your SQL and Python line by line;
- derive a sample output from the source rows;
- show validation evidence;
- describe one failure and its recovery;
- state a limitation honestly and propose a safe improvement.
