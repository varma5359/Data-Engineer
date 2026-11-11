# Day 51 Interview Guide — Object Storage and Data Lakes

## 1. What is object storage?

Object storage stores data as objects in containers such as buckets. An object includes its data, a key used to identify it and metadata. It is commonly accessed through APIs and is widely used for data lakes, backups, logs and large collections of files.

## 2. What is a bucket?

A bucket is a top-level container for objects in an object storage service. It provides a namespace and a place to configure access, lifecycle and other storage settings. Exact bucket behavior and naming rules depend on the provider.

## 3. What is an object key?

An object key is the name that identifies an object within a bucket. A key may contain slash characters that tools display as folder-like prefixes. In many object stores, these are not traditional directories.

## 4. Is object storage the same as a file system?

No. A traditional file system exposes directories and file operations. Object storage exposes objects through service APIs and keys. A console may display prefixes as folders, but operations such as rename may be implemented as copy and delete.

## 5. Compare object, file and block storage.

Object storage stores objects and is common for data lakes. File storage exposes a directory hierarchy for shared file access. Block storage provides volumes used by operating systems and databases. The choice depends on access pattern, latency, sharing and workload.

## 6. What is a data lake?

A data lake is a storage environment that keeps large volumes of varied data in original and transformed forms for processing and analytics. It can contain structured, semi-structured and unstructured data. A lake still needs governance, metadata, quality and access controls.

## 7. What is the difference between a data lake and a data warehouse?

A data lake is designed to store varied data and support multiple processing uses. A data warehouse provides governed analytical structures and consistent business models. Modern lakehouse designs combine aspects of both.

## 8. What is a raw zone?

The raw zone stores source data as received, with minimal changes. It supports reprocessing, debugging, audit and source comparison. It must still have access controls and retention rules.

## 9. What is a processed zone?

The processed zone contains standardized or cleaned data. Common operations include type conversion, column naming standardization, validation, deduplication under documented rules and file-format conversion.

## 10. What is a curated zone?

The curated zone contains business-ready datasets for known consumers. It should have a defined grain, documented columns, ownership, quality checks and freshness expectations.

## 11. Why distinguish event date from ingestion date?

Event date describes when the business event happened. Ingestion date describes when the platform received the record. They differ when files arrive late or are replayed. Keeping both supports late-data analysis and pipeline troubleshooting.

## 12. Compare CSV, JSON, Parquet and Avro.

CSV is simple delimited text. JSON supports nested structures and is common in APIs. Parquet is a columnar format useful for analytical scans. Avro is a row-oriented serialization format with schema support. Choose based on consumers, schema and access patterns.

## 13. Why is Parquet useful for analytics?

Parquet stores data in a columnar layout and supports typed values and compression/encoding. Analytical engines can often read only required columns. It is a file format, not a database.

## 14. What is partitioning in a data lake?

Partitioning organizes data into subsets based on values such as date or region. A query engine may skip irrelevant partitions when filters align with the partition design. Poor partition choices can create too many tiny files.

## 15. What is the small-file problem?

A very large number of tiny files increases listing, metadata, file-open and task-scheduling overhead. Compaction can combine small files, but target sizes should follow the engine and workload guidance.

## 16. What is schema-on-read?

Schema-on-read means a reader interprets the data structure when it reads the data. It provides flexibility but does not eliminate schema documentation and validation.

## 17. What is schema evolution?

Schema evolution is a change in dataset structure over time, such as adding a column or changing a type. Pipelines must define compatibility, defaults, validation and downstream impact rather than assuming every change is safe.

## 18. What is quarantine?

Quarantine is a controlled location for invalid files or records. It should preserve traceability such as source file, rejection reason, processing run and replay status. Rejected data should not be silently discarded.

## 19. How would you design a data lake layout?

I would separate data by zone and dataset, use stable naming conventions, record source and ingestion date, choose partitions based on query patterns and volume, and define ownership, access, quality and retention. I would avoid excessive partitions and uncontrolled raw access.

## 20. Scenario: an order file arrives one day late. How do you handle it?

I would preserve its business event date and actual ingestion date. The pipeline would validate and process the late data, update downstream aggregates according to the agreed correction strategy and record the late arrival. I would not pretend the file arrived on its event date.

## 21. Scenario: a dataset has millions of tiny files. What is the concern?

Metadata listing, file opening and task scheduling may dominate processing. I would evaluate compaction and a more suitable file/partition strategy while checking parallelism, recovery and downstream compatibility.

## 22. Explain your Day 51 project.

I designed a raw, processed, curated and quarantine layout for TechStore. I created object-key conventions that distinguish event date from ingestion date, chose file formats and partition keys, built a local folder simulation, read a CSV with pandas, converted it to Parquet, aggregated daily sales and documented access and retention rules.

## Common interview traps

- Saying object keys are always real directories.
- Confusing a data lake with a data warehouse.
- Treating Parquet as a database.
- Partitioning by every column.
- Ignoring late-arriving data.
- Deleting invalid records without traceability.
- Keeping raw data forever without retention controls.
- Giving all users access to every zone.

## Readiness checklist

- [ ] Explain bucket, object, key and metadata.
- [ ] Compare object, file and block storage.
- [ ] Explain data lake zones.
- [ ] Compare CSV, JSON, Parquet and Avro.
- [ ] Explain partitioning and small files.
- [ ] Explain schema evolution and quarantine.
- [ ] Defend an object-key and retention design.
