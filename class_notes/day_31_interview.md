# Day 31 Interview Guide — File-Based Data Ingestion

**Use with:** Day 31 class notes and project workbook.  
**Goal:** Answer as a junior engineer who understands how data is controlled, not as someone who only knows read_csv.

# 1. What is file-based ingestion?

**Model answer:** File-based ingestion is the process of receiving files from a producer, discovering the expected delivery, validating its contract and records, and moving accepted data into a pipeline or destination. It also records rejected data and run evidence. For example, TechStore receives order CSV files from a payment/logistics partner each morning. A safe pipeline checks filename, schema, data types, business rules, duplicate keys and counts before publishing.

# 2. Explain CSV and its limitations.

**Model answer:** CSV is a text format representing tabular rows and columns, usually with a header and delimiter. It is easy to exchange and inspect, but it does not enforce types, constraints or nested structure. Python's DictReader returns values as strings. We must convert and validate integers, decimals and dates explicitly. Quoting, delimiters, encoding and line breaks must follow the producer contract.

# 3. Why do you validate the header before reading records?

**Model answer:** The header is the file's schema contract. If a required column is missing or renamed, record logic may fail or—worse—read the wrong field. I check required fields, duplicate names and unexpected fields before processing. A breaking schema mismatch is usually a file-level error; I quarantine and alert rather than partially ingesting.

# 4. Difference between file-level and record-level errors?

**Model answer:** A file-level error means the file cannot be safely interpreted: malformed CSV/JSON, missing required header, wrong encoding or truncation. A record-level error means the file structure is valid but one row violates a rule, such as invalid customer ID or negative amount. The contract decides whether row errors can be rejected individually or must fail the batch.

# 5. Why does CSV data arrive as strings?

**Model answer:** CSV is text; it does not store a type system. The value "501" is text until our program parses it. I convert it to integer and validate positivity. I parse money as Decimal, not float, and dates using an agreed format. Conversion errors become explicit reject reasons rather than hidden defaults.

# 6. How do you handle invalid rows?

**Model answer:** I validate each rule, collect reasons and separate accepted from rejected records. Reject output should include a stable source key, source file, run ID and reason, with sensitive data minimized. I reconcile input count against accepted plus rejected. I don't silently drop rows or turn invalid values into zero unless the business rule explicitly says so.

# 7. How do you handle malformed JSON?

**Model answer:** JSON parser errors are file-level failures because the payload cannot reliably be parsed. I quarantine the file, record safe error context and notify the producer/owner. After syntax parsing, I still validate structure: expected keys, types, arrays and record fields. Valid JSON does not necessarily mean valid business data.

# 8. How do you process multiple files?

**Model answer:** I discover files using a documented pattern and sort them deterministically if order matters. For each file I record identity, size, arrival and status; validate its schema; process rows; write outputs; reconcile; and mark success only after durable publication. I isolate file-level failures so one malformed delivery does not silently corrupt others. Batch-level policy decides whether the overall run is success, partial success or failure.

# 9. How do you prevent duplicate ingestion?

**Model answer:** I use a file ledger and record-level identity. File identity may include partner, delivery ID, checksum, date and version—not filename alone. The target also needs a unique business/event key and replay-safe write behavior. If the same file arrives again, I skip or safely replay it. If the name is reused with different content, I treat it as a correction/conflict according to contract.

# 10. What is idempotency for file ingestion?

**Model answer:** Reprocessing the same delivery should not create duplicate effects. I test this by running the same input twice and checking output/target key counts and values. Idempotency can use file ledger, deterministic output replacement, unique keys, upsert or transactional staging. The exact control depends on destination and whether a partner can send corrections.

# 11. What is a checksum and when is it useful?

**Model answer:** A checksum/hash is a fingerprint computed from file bytes. It can identify byte-identical files and detect content changes under the same filename. It does not prove the business content is valid, and different bytes may represent semantically identical data (for example different row order). Use it as one delivery-identity signal, not the only validation.

# 12. How do you handle a file still being uploaded?

**Model answer:** I avoid processing an incomplete file. A robust producer contract can upload under a temporary name and rename to a final ready name, or publish a manifest/ready marker after upload. The consumer only processes completed deliveries. File size stability is a weaker heuristic and should not replace an explicit delivery protocol when possible.

# 13. What is schema evolution?

**Model answer:** Schema evolution is a change to fields, names, types or constraints over time. An additive optional field may be compatible; removing/renaming a required field or changing a type may break consumers. I compare incoming schema with the contract, classify the change, version approved mappings and test old/new producer-consumer combinations. Unknown breaking changes are quarantined and escalated.

# 14. Explain input = accepted + rejected.

**Model answer:** For record-level validation, every parsed input record must have one accounted outcome: accepted or rejected. If input is 100, accepted 97 and rejected 3, reconciliation is 100 = 97 + 3. If it doesn't balance, a record was lost, duplicated in accounting or not processed. File-level failures are tracked separately because rows may not have been parsed at all.

# 15. Why count reconciliation alone is not enough?

**Model answer:** Counts can match while values or keys are wrong. I also check unique keys, required fields, allowed values, totals, schema and source-file lineage. For critical feeds, I compare key sets or selected hashes/aggregates. The checks depend on business risk and volume.

# 16. How would you handle duplicate partner order IDs?

**Model answer:** First classify: identical record repeated (replay), same key with changed values (correction/conflict), or key is not actually unique under the partner contract. I define a deterministic rule such as partner version/update timestamp, stage conflicts and report them. I don't blindly keep the first/last row based on file order.

# 17. What happens if accepted output is empty?

**Model answer:** Empty accepted output may be valid if all records are rejected or no records were delivered, but it can indicate schema drift or wrong mapping. I inspect reject reasons, input count, expected delivery and historical baseline. A system-wide error should fail/alert rather than report successful ingestion with zero accepted rows.

# 18. How do you safely publish output?

**Model answer:** Write to a temporary location, close it, validate counts/schema/keys and then publish using atomic rename or a destination-specific commit protocol. Keep run ID and source lineage. If output is object storage, use its appropriate upload/manifest/commit pattern; local filesystem rename assumptions do not automatically apply.

# 19. Explain the Day 31 project.

**Model answer:** “I built a partner file ingestion workflow. It discovers expected CSVs, checks headers, reads records, converts ID/date/amount types, applies status and amount rules, and separates accepted and rejected records with reasons. It writes outputs and reconciles input = accepted + rejected, then checks duplicate partner keys. I also tested malformed files, missing headers and reruns. The design keeps rejected evidence and prevents silent data loss.”

# 20. Follow-up questions to practise

- How would you process a 20 GB CSV?
- What if the partner changes amount from decimal to string?
- What if two files contain the same order?
- What if one file is malformed but the other nine are valid?
- What if the file name is identical but contents changed?
- How do you archive files without losing retry ability?
- What data should not be written into reject logs?
- How would you monitor expected file arrival?
- How would you move accepted records into PostgreSQL?
- How would you test ingestion without waiting for a partner?

# 21. Project defence answer structure

When asked about any design decision, answer in four parts:
1. **Requirement:** what business behavior is needed?
2. **Choice:** what did you implement?
3. **Reason/risk:** why and what can go wrong?
4. **Evidence:** which test, count or output proves it?

Example: “I validate headers before rows because a missing required column can invalidate the whole delivery. I compare required and received columns and reject the file on mismatch. I tested a missing-status header and confirmed no output was published.”

# 22. Self-check

You are interview-ready when you can explain CSV/JSON parsing, schema contract, record rejection, duplicate delivery, idempotency, reconciliation, file publication and recovery using your own project evidence.
