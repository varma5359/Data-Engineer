# Day 36 Interview Guide — Batch Processing and Partitioning

**Project:** TechStore Partitioned Batch Processor  
**Audience:** Fresher / Junior Data Engineer  
**How to use:** Answer aloud, compare with the model answer, then show the matching project output or code cell.

---

## Part 1 — Core concepts with complete answers

### 1. What is batch processing?

**Model answer:** Batch processing handles a finite group of records together rather than processing each event immediately as it arrives. A batch may run on a schedule, when a file arrives or when a defined amount of data is ready. For example, TechStore may process all orders received during the previous business day at 1 AM.

Batch processing is useful when a business can tolerate some delay and wants predictable, efficient processing windows.

### 2. What is the difference between batch and streaming?

**Model answer:** Batch processes bounded collections of data; streaming continuously processes events as they arrive or in very small time windows. Batch is suitable for daily financial summaries or nightly warehouse loads. Streaming is useful for near-real-time fraud signals or live operational monitoring. The choice depends on latency requirements, volume, complexity, cost and operational maturity. Streaming is not automatically better.

### 3. What is a batch size?

**Model answer:** Batch size is the number of records or amount of data processed in one unit of work. A small batch uses less memory and can reduce the amount of work lost on failure, but it may create more overhead. A large batch can improve throughput by reducing repeated setup, but it can consume more memory and take longer to retry. I choose it through measurement rather than guessing.

### 4. Why not load a huge CSV completely into Pandas?

**Model answer:** Reading the whole file creates a DataFrame in memory. The file's in-memory representation can be much larger than its disk size because Python objects and column structures add overhead. If memory is insufficient, the process can slow down or crash. Chunked reading bounds memory by processing a limited number of rows at a time.

### 5. What does chunksize do in Pandas?

**Model answer:** In `pd.read_csv(path, chunksize=10000)`, Pandas returns an iterator-like TextFileReader that yields DataFrame chunks, each containing up to 10,000 rows. I process each chunk and release references when finished. The final chunk may contain fewer rows. Chunking controls input memory; it does not automatically make every transformation or output operation memory-safe.

### 6. What is partitioning?

**Model answer:** Partitioning organizes a dataset into separate logical or physical groups using a partition key, commonly a date such as year/month/day. A query filtering for one date may read only the relevant partition rather than scan every file. Partitioning is a storage and query-layout decision; it is different from the number of rows processed in a batch.

### 7. Batch versus partition — explain the difference.

**Model answer:** A batch is a processing unit; a partition is a way of organizing stored data. I may read a CSV in 50,000-row batches and write each batch into date-based partitions. Several batches can contribute to one partition, and a batch can contain records belonging to multiple partitions.

### 8. What is partition pruning?

**Model answer:** Partition pruning is the ability of a query engine to skip partitions that cannot satisfy a filter. If a dataset is partitioned by order_date and a query asks for one day, the engine can avoid reading other dates. This saves I/O and can improve performance. The query must filter on the partition column in a way the engine can use.

### 9. How do you choose a partition key?

**Model answer:** I look at common query filters, data distribution, partition size, retention needs and ingestion patterns. A date key is often useful for time-based analytical data. I avoid a key with extremely high cardinality, such as customer_id, if it creates thousands or millions of tiny directories/files. I validate the design using representative queries and actual data volume.

### 10. Why is partitioning by customer_id often a poor choice?

**Model answer:** If there are millions of customers, it can create a very large number of small partitions and files. Listing and managing those files becomes expensive, and queries that need all customers may touch many partitions. Customer-based partitioning can make sense in a specialized workload, but it should be justified by query patterns and data scale.

### 11. What is Parquet?

**Model answer:** Parquet is a columnar storage file format commonly used in analytics. It stores values by column, supports data types and compression, and allows readers to select only needed columns. It is often more efficient than CSV for analytical workloads. CSV remains useful for simple interchange and human inspection.

### 12. CSV versus Parquet?

**Model answer:** CSV is text-based, easy to inspect and widely supported, but types are inferred or supplied separately, and reading selected columns can still require scanning text. Parquet is typed and columnar, often compressed, and supports column pruning. I choose based on the consumer, ecosystem and workload rather than assuming Parquet is always required.

### 13. What is the small-file problem?

**Model answer:** Many tiny files create overhead for storage listing, metadata handling, query planning and task scheduling. A partitioned pipeline that writes one tiny file per record or per small batch can become slow even if the total data volume is modest. Solutions include sensible batch sizes, compaction and partition design that avoids excessive fragmentation.

### 14. What is data skew?

**Model answer:** Data skew occurs when records are distributed unevenly across partitions or processing groups. One partition may contain most of the data while others are small. The large partition becomes a bottleneck, causing long-running tasks and uneven resource use. I inspect key distributions and consider a better partition key, salting or a different processing strategy where appropriate.

### 15. How do you select a chunk size?

**Model answer:** I start with a conservative value and measure memory use, throughput, processing time and failure recovery. The right value depends on row width, transformations, machine memory, I/O and output format. I also test the largest expected records, not only a tiny sample. There is no universal best chunksize.

### 16. Does partitioning always improve query performance?

**Model answer:** No. It helps when queries filter on the partition key and partitions are reasonably sized. It can hurt when the key creates too many tiny partitions, when queries do not filter on it, or when metadata overhead exceeds the scan savings. A partition design must be evaluated against actual query patterns.

### 17. What is partition pruning versus column pruning?

**Model answer:** Partition pruning skips whole partitions based on partition values. Column pruning reads only the columns needed by a query, which is especially useful in columnar formats such as Parquet. They reduce different kinds of unnecessary I/O and can work together.

### 18. Why does date and timezone handling matter?

**Model answer:** Events may be recorded in UTC while business reporting uses a local timezone. If I derive a partition date without defining the timezone, an event near midnight can land in the wrong business day. I preserve the event timestamp, standardize time handling, document the business timezone and derive the partition key consistently.

### 19. What is partition compaction?

**Model answer:** Compaction combines many small files into fewer appropriately sized files, often within the same partition. It reduces metadata and file-open overhead. Compaction must preserve schema, partition values and record correctness, and should not overwrite data unsafely while readers are using it.

### 20. How do you reconcile a partitioned batch output?

**Model answer:** I compare source rows read with rows written across all partitions, accounting for documented rejects or deduplication. I also compare per-date counts and important totals, inspect missing partitions and confirm that the output can be read back. A successful write call alone is not proof of a complete dataset.

---

## Part 2 — Project-based questions

### 21. Explain the TechStore Partitioned Batch Processor.

**Model answer:** The project creates a reproducible order dataset, reads it in bounded Pandas chunks, applies a simple transformation, derives a date partition from the order timestamp and writes Parquet files into a partitioned folder layout. It then reads the full dataset and a single date partition, compares counts and totals, and checks that the result is complete. The purpose is to demonstrate the difference between bounded processing and storage partitioning.

### 22. Why did you read the output back?

**Model answer:** Writing files proves only that the write operation returned. Reading the dataset back checks that the output is discoverable and readable using the intended layout. I compare row counts, date values and selected aggregates with the source. This catches missing files, wrong paths and partition mistakes.

### 23. What happens if one chunk fails after earlier chunks were written?

**Model answer:** The pipeline must not publish a partial dataset as complete. I would write to a run-specific staging location, record completed chunks, validate the full result and publish only after reconciliation. Recovery can resume or replay safely using stable keys and idempotent writes. The exact method depends on the storage system and output format.

### 24. What would you improve for production?

**Model answer:** I would add run IDs, source file manifests, schema validation, bounded retries, staging and atomic publication, partition-level metrics, compaction, retention policy and automated reconciliation. I would also test memory use and query performance using realistic data volumes.

---

## Part 3 — Scenario questions

### 25. A 20 GB CSV causes the notebook kernel to restart. What do you do?

**Strong answer:** I stop loading the entire file into memory. I inspect available memory and row width, then use chunked reading with a measured chunksize. I process each chunk, write to staging outputs and release intermediate objects. I test with representative data and verify total rows and aggregates. If the transformation requires cross-chunk state, I design that state explicitly rather than assuming chunks are independent.

### 26. A query filters by order_date but scans every file.

**Strong answer:** I inspect the folder layout and query plan. I verify that data is actually partitioned by the same date field and that the query filter is compatible with the partition column and type. I check whether the reader recognizes the partition metadata. I then test a date-specific query and compare files or bytes scanned.

### 27. One date partition contains 10 million rows and every other date has 10,000.

**Strong answer:** I investigate whether the source volume is genuinely uneven or whether a late-arriving/backfill process concentrated data. I measure partition sizes and query impact. I may subdivide large dates using a secondary key or use a layout that balances file sizes, but I avoid blindly adding high-cardinality partitions. The solution should reflect query patterns and operational overhead.

### 28. Your output contains 50,000 tiny Parquet files.

**Strong answer:** I inspect the relationship between chunk size, write frequency and partition count. I change the write strategy so each task or compaction step creates appropriately sized files per partition. I compact existing files where safe and measure query planning and read performance before and after.

### 29. A date near midnight is stored in yesterday's partition instead of today's.

**Strong answer:** I inspect the original timestamp and timezone. I define whether the partition date is UTC or the business timezone, convert consistently before deriving the date and add boundary tests around midnight and daylight-saving changes where relevant. I do not simply add or subtract one day without defining the intended business rule.

### 30. A manager asks you to partition by every customer to make customer queries faster.

**Strong answer:** I ask for query patterns and customer cardinality. If there are millions of customers, this may create a small-file and metadata problem. I benchmark alternatives such as date partitioning plus clustering/sorting or a storage engine's indexing/data-skipping features. I choose based on measured workload, not a single query example.

---

## Part 4 — Practical revision checklist

- Explain batch versus streaming.
- Explain batch size versus partition key.
- Demonstrate chunked CSV reading.
- Explain CSV versus Parquet.
- Draw a date-partitioned folder layout.
- Explain partition pruning and column pruning.
- Identify small-file and skew risks.
- Reconcile source and partitioned output.
- Explain one recovery strategy for a partial write.

## Part 5 — Self-assessment

Score 0–2 for each skill: 0 = cannot explain; 1 = partial; 2 = explain with a project example.

| Skill | Score |
|---|---:|
| Batch and streaming | /2 |
| Chunked processing | /2 |
| Batch size trade-offs | /2 |
| Partition key design | /2 |
| Parquet and CSV | /2 |
| Partition pruning | /2 |
| Small files and skew | /2 |
| Project reconciliation | /2 |

**13–16:** ready for junior-level discussion. **9–12:** repeat the project and explain trade-offs. **0–8:** revisit the class lesson and rerun the workbook.
