# Day 51: Object Storage and Data Lakes

**Phase:** 6 — Cloud and Distributed Processing  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and junior Data Engineer candidates  
**Topic boundary:** Class notes only. Project workbook and interview guide are separate.

> Yesterday you learned why companies use cloud platforms. Today we focus on a storage type used heavily in data engineering: object storage. You will learn what an object is, how it differs from a traditional file system and a relational database, how a data lake is organized, and how to design folders, partitions, file formats and retention rules.

## Learning outcomes

- Explain object storage in simple language.
- Identify an object, bucket, key and metadata.
- Compare object storage, file storage and block storage.
- Explain why object storage is useful for analytics.
- Explain what a data lake is.
- Design raw, processed and curated zones.
- Understand file naming and partitioning.
- Compare CSV, JSON, Parquet and Avro at a high level.
- Explain small-file and schema-evolution problems.
- Design basic retention and access boundaries.

## Today's flow

Source files -> Object storage bucket -> Raw zone -> Processing -> Processed zone -> Curated zone -> Analytics

---

# Part 1 — Begin with files

## 1. What does a data engineer receive?

TechStore receives files from multiple business systems.

| Source | Example file | Data |
|---|---|---|
| Order application | orders_2026_11_10.csv | Order details |
| Payment provider | payments_2026_11_10.json | Payment status |
| Website | click_events_2026_11_10.json | User activity |
| Product system | products.parquet | Product catalogue |

A file is a container for data. The format describes how the data is represented inside it.

A data engineer needs a place to store many files, organize them, protect them and make them available to processing engines.

## 2. What is object storage?

**Beginner definition:** Object storage stores data as objects inside containers called buckets (the exact container name varies by provider).

An object commonly includes:
1. the data itself;
2. a key or object name;
3. metadata describing the object.

For example, an object might contain a CSV file with 10,000 orders.

Object storage is designed for storing and retrieving objects using service APIs. It is not exactly the same as browsing a local hard drive.

## 3. Understand bucket, object and key

Imagine a bucket called techstore-data.

An object key could be:

~~~text
raw/orders/year=2026/month=11/day=10/orders_001.csv
~~~

| Term | Meaning |
|---|---|
| Bucket | Top-level object storage container |
| Object | Stored data plus its object attributes |
| Key | Name used to identify the object within a bucket |
| Metadata | Information describing the object |
| Prefix | A beginning portion of object keys used to group or search objects |

The key looks like a folder path, but many object storage systems use a flat object namespace. Slashes in keys are commonly used as prefixes to create a folder-like view in consoles and tools.

## 4. Is an object key a real folder path?

Not necessarily.

In a traditional file system:
- folders are directory objects;
- files are stored within directories;
- the operating system provides directory operations.

In object storage:
- the object is identified by its key;
- a slash in a key is often just a character;
- tools display prefixes as folders for convenience.

This difference matters when listing, renaming or moving objects. A move may actually be a copy followed by deletion rather than a simple filesystem rename.

## 5. What is object metadata?

Metadata describes an object.

Examples:
- content type;
- content length;
- last modified time;
- encryption information;
- custom business attributes, where supported.

Do not put every business field into object metadata. The row-level business data belongs in the file or a data catalogue/table format. Object metadata is for describing the stored object.

---

# Part 2 — Compare storage types

## 6. Object storage

Object storage is useful for:
- CSV, JSON, Parquet and Avro files;
- images and documents;
- application logs;
- backups;
- data lake datasets;
- long-term archives.

It is commonly accessed through APIs and provider tools.

## 7. File storage

File storage exposes data through a file and directory hierarchy.

Examples include:
- a shared network file system;
- a mounted file share;
- a local laptop drive.

Applications use paths and file operations such as opening, reading, writing and listing directories.

## 8. Block storage

Block storage provides storage volumes made of blocks that can be attached to compute systems.

It is commonly used for:
- operating system disks;
- database storage volumes;
- application disks.

A database engine manages its own data structures on top of block storage.

## 9. Compare object, file and block storage

| Feature | Object storage | File storage | Block storage |
|---|---|---|---|
| Data view | Objects and keys | Files and directories | Blocks/volumes |
| Typical access | API/HTTP or SDK | File protocols | Attached volume interface |
| Common use | Data lakes, backups, media | Shared files | OS and database disks |
| Folder behavior | Prefix-based view in many systems | Real directory hierarchy | Managed by filesystem above blocks |
| Best suited to | Large-scale unstructured/object data | Shared file access | Low-level disk workloads |

No storage type is best for every workload. Select based on access pattern, latency, sharing, durability, scale and cost.

---

# Part 3 — What is a data lake?

## 10. Beginner definition

A **data lake** is a storage environment designed to keep large amounts of data in its original or transformed forms so different processing and analytics tools can use it.

A data lake may contain:
- structured data such as CSV and Parquet;
- semi-structured data such as JSON;
- unstructured data such as images and documents.

Object storage is commonly used as the foundation of a cloud data lake.

A data lake is not automatically well organized. Without ownership, metadata, quality rules and access controls, it can become a data swamp.

## 11. Data lake versus data warehouse

| Area | Data lake | Data warehouse |
|---|---|---|
| Main purpose | Store varied data for multiple uses | Provide governed analytical structures |
| Data forms | Raw and transformed files, many formats | Tables and managed analytical structures |
| Schema approach | Often schema-on-read or flexible ingestion | Often curated schemas and models |
| Users | Data engineers, scientists, analysts | Analysts, BI and business users |
| Typical storage | Object storage | Warehouse-managed storage |
| Governance | Required | Required |

Modern lakehouse platforms combine lake storage with table-management and warehouse-like capabilities. We will study these patterns in later lessons.

## 12. Why keep different data zones?

A practical beginner layout has three zones:

1. Raw zone
2. Processed zone
3. Curated zone

These names are conventions. A company may use bronze/silver/gold or other naming.

## 13. Raw zone

The raw zone stores source data as received, with minimal changes.

Example:

~~~text
s3://techstore-data/raw/orders/ingest_date=2026-11-10/orders_001.csv
~~~

Why preserve raw data?
- reprocess after a transformation bug;
- compare source and target;
- investigate unexpected source changes;
- support audit needs;
- recover from downstream failures.

Raw does not mean "keep everything forever". Apply access control, retention and deletion policies.

## 14. Processed zone

The processed zone contains data that has been standardized or cleaned.

Possible actions:
- standardize date formats;
- normalize column names;
- convert types;
- remove invalid records into a quarantine location;
- deduplicate according to a documented key;
- convert CSV to Parquet.

The transformation rules must be documented. Do not silently discard source records.

## 15. Curated zone

The curated zone contains data shaped for known business use.

Examples:
- daily_sales;
- customer_orders;
- product_performance;
- delivery_sla_summary.

Curated data should have:
- a clear grain;
- documented columns;
- known owners;
- quality checks;
- freshness expectations;
- controlled access.

---

# Part 4 — Design an object storage layout

## 16. Why does layout matter?

A good layout helps engineers:
- locate datasets;
- separate environments;
- identify data state;
- filter by date;
- apply retention;
- grant access by prefix;
- debug failed jobs.

A poor layout makes data hard to find and can cause access or processing mistakes.

## 17. Start with a bucket naming plan

A bucket name should be:
- unique within the provider's naming scope;
- meaningful;
- environment-aware where appropriate;
- free of sensitive information;
- compliant with provider naming rules.

Example naming pattern:

~~~text
company-data-dev
company-data-test
company-data-prod
~~~

These are examples, not guaranteed available names. Real provider naming rules and uniqueness constraints apply.

## 18. Example lake layout

~~~text
company-data-prod/
    raw/
        orders/
            source_system=order_app/
                ingest_date=2026-11-10/
                    orders_001.csv
        payments/
            source_system=payment_api/
                ingest_date=2026-11-10/
                    payments_001.json
    processed/
        orders/
            order_date_year=2026/
                order_date_month=11/
                    part-000.parquet
    curated/
        sales_daily/
            sales_date_year=2026/
                sales_date_month=11/
                    part-000.parquet
    quarantine/
        orders/
            ingest_date=2026-11-10/
                invalid_orders.csv
~~~

Understand the levels:
- raw/processed/curated identifies the data zone;
- orders/payments/sales_daily identifies the dataset;
- source_system identifies origin;
- ingest_date identifies when the pipeline received data;
- date partitions may identify the business date represented by the records.

Do not confuse ingestion date with event date.

## 19. Ingestion date versus event date

Suppose an order was created on November 9 but the file arrived on November 10.

| Field | Value |
|---|---|
| order_date | 2026-11-09 |
| ingest_date | 2026-11-10 |

These dates answer different questions:
- event date: when did the business event happen?
- ingestion date: when did our platform receive the record?

Keeping both can help with late-arriving data and pipeline troubleshooting.

## 20. Naming conventions

Use names that are:
- consistent;
- readable;
- predictable;
- stable over time.

A common convention is lowercase names with underscores.

Avoid:
- spaces;
- random abbreviations;
- names such as final_final_v2;
- personal names;
- secrets or sensitive customer data in keys.

Example:

~~~text
orders_2026_11_10_001.parquet
~~~

The object name should help identify the file, but the data catalogue should hold the complete business definition.

---

# Part 5 — File formats in a data lake

## 21. CSV

CSV is a text format where rows and columns are represented using delimiters.

Example:

~~~csv
order_id,customer_id,amount
101,C001,250.00
102,C002,499.00
~~~

Advantages:
- easy to inspect;
- supported by many tools;
- convenient for simple exchange.

Limitations:
- data types are not strongly stored in the file;
- nested data is awkward;
- large scans can be inefficient;
- delimiter and quoting rules must be handled correctly.

## 22. JSON

JSON stores data as objects, arrays and values.

Example:

~~~json
{
  "order_id": 101,
  "customer": {
    "customer_id": "C001",
    "city": "Hyderabad"
  },
  "amount": 250.00
}
~~~

Advantages:
- supports nested structures;
- common in APIs;
- readable.

Limitations:
- repeated field names add storage overhead;
- inconsistent structures can complicate processing;
- type variation can create schema issues.

## 23. Parquet

Parquet is a columnar file format commonly used for analytical workloads.

In a columnar format, values for a column are stored together in column-oriented structures.

Benefits can include:
- reading only required columns;
- compression and encoding efficiency;
- typed schema information;
- good integration with analytical engines.

Parquet is not a database. It is a file format.

## 24. Avro

Avro is a row-oriented data serialization format with schema support.

It is often used in data exchange and event/data pipeline systems.

A schema helps readers understand how records are represented and how compatible schema changes are handled.

## 25. Compare common formats

| Format | Layout | Common use |
|---|---|---|
| CSV | Text rows and columns | Simple exchange and exports |
| JSON | Nested key-value structures | API responses and events |
| Parquet | Columnar | Analytical storage and scans |
| Avro | Row-oriented with schema | Data serialization and event pipelines |

Choose formats based on workload, compatibility, schema requirements and tools. Do not convert everything to Parquet without checking downstream consumers.

---

# Part 6 — Partitioning object data

## 26. What is partitioning?

Partitioning organizes data into separate groups based on one or more column values, often represented by key prefixes or table metadata.

Example:

~~~text
sales/order_year=2026/order_month=11/order_day=10/
~~~

A query for one date may read only the relevant partition if the query engine and table design support partition pruning.

## 27. Why partition?

Benefits may include:
- reducing the amount of data scanned;
- improving query organization;
- supporting incremental processing;
- simplifying retention by date;
- making data ownership easier to understand.

Partitioning is not automatically faster. The query must filter on useful partition columns and the engine must use the partition metadata.

## 28. Choose a useful partition key

Common candidates:
- event_date;
- order_date;
- ingest_date;
- region, if distribution and access patterns justify it.

Choose based on:
- common query filters;
- data volume;
- number of distinct values;
- late-arriving records;
- retention requirements;
- file size.

## 29. Avoid over-partitioning

Imagine one year of data is split into a separate partition for every customer ID.

If there are millions of customers, this may create too many tiny partitions or files.

Too many tiny files can increase:
- metadata operations;
- file listing;
- task scheduling overhead;
- query planning work.

Choose partition granularity based on data volume and access patterns.

## 30. Partition by ingestion date or business date?

Use ingestion date when the main question is when the platform received the data.

Use business/event date when queries commonly filter by when the event happened.

Some datasets need both, but adding every possible partition column can create excessive partitions. Keep the physical layout practical.

---

# Part 7 — Data lake reliability and governance

## 31. Schema-on-read

Schema-on-read means the structure is interpreted when data is read by a processing or query tool.

This flexibility helps store varied source data, but it does not remove the need for schema documentation and validation.

A reader may interpret the same file differently if type inference or parsing rules differ.

## 32. Schema-on-write

Schema-on-write means data is checked against an expected structure as it is written into a target.

It can prevent malformed data from entering a governed table, but the pipeline must decide what happens to invalid records.

Many real platforms use a combination of both approaches.

## 33. Schema evolution

Schema evolution means the data structure changes over time.

Example:
- yesterday's customer file had customer_id and city;
- today's file adds loyalty_tier.

The pipeline must decide:
- Is the new field optional?
- What default applies to old records?
- Does the target schema support the change?
- Will downstream reports break?
- Who approves the change?

Do not assume every file reader automatically handles every schema change.

## 34. Data quality in a lake

Possible checks:
- required file exists;
- file is not unexpectedly empty;
- schema matches expectation;
- required columns exist;
- primary business key is not null;
- duplicate rules are satisfied;
- numeric values are in valid ranges;
- event dates are plausible;
- source and target row counts reconcile.

A file landing successfully does not mean its data is correct.

## 35. Quarantine

Quarantine is a controlled location for records or files that fail validation.

Example:

~~~text
quarantine/orders/ingest_date=2026-11-10/invalid_orders.csv
~~~

A good quarantine process records:
- source file;
- rejected record or file;
- reason for rejection;
- processing time;
- pipeline run identifier;
- whether the issue can be replayed.

Do not silently delete rejected records.

## 36. Retention and lifecycle

Retention defines how long data is kept.

Lifecycle policies may transition or expire objects according to configured rules and provider support.

Before deleting data, confirm:
- legal and regulatory requirements;
- business retention needs;
- backup dependencies;
- downstream consumers;
- recovery expectations;
- whether a legal hold applies.

Never assume old data is safe to delete just because it is not queried often.

## 37. Access boundaries

A data lake may contain both public business metrics and sensitive personal information.

Possible access design:
- ingestion role writes to raw;
- transformation role reads raw and writes processed;
- curated reporting role reads curated datasets;
- analyst role reads only approved reporting data;
- administrators manage platform configuration through separate controlled access.

Use separate identities and narrow permissions. Do not give every analyst access to every bucket.

---

# Part 8 — Local practice: design and inspect a lake layout

This exercise does not create an AWS bucket. It helps you understand object keys and prefixes using local folders.

## 38. Create a local practice layout

In your practice project, create:

~~~text
cloud_data_lake/
    raw/
        orders/
        payments/
    processed/
        orders/
    curated/
        sales_daily/
    quarantine/
        orders/
~~~

These are local directories. They imitate the naming convention but are not actual cloud object storage.

## 39. Create sample source data

Create a file named orders.csv inside raw/orders.

~~~csv
order_id,customer_id,order_date,amount
101,C001,2026-11-10,250.00
102,C002,2026-11-10,499.00
103,C001,2026-11-11,100.00
~~~

Save the file.

## 40. Read the file with Python

~~~python
import pandas as pd

orders = pd.read_csv("cloud_data_lake/raw/orders/orders.csv")

print(orders)
~~~

Expected output:

~~~text
   order_id customer_id  order_date  amount
0       101        C001  2026-11-10   250.0
1       102        C002  2026-11-10   499.0
2       103        C001  2026-11-11   100.0
~~~

Explanation:
- import pandas loads the pandas library;
- read_csv opens the local CSV file;
- the DataFrame stores the rows and columns;
- print displays the data.

If pandas is not installed in the active environment, install it in the environment you use for this course. Do not install packages into a different Python environment by accident.

## 41. Inspect columns and data types

~~~python
print(orders.columns)
print(orders.dtypes)
print(orders.shape)
~~~

Expected output:

~~~text
Index(['order_id', 'customer_id', 'order_date', 'amount'], dtype='object')
order_id         int64
customer_id     object
order_date      object
amount         float64
dtype: object
(3, 4)
~~~

Your exact dtype display can vary by pandas version and input values.

## 42. Convert date column

~~~python
orders["order_date"] = pd.to_datetime(orders["order_date"])

print(orders.dtypes)
~~~

Expected date type:

~~~text
order_date    datetime64[ns]
~~~

This conversion is useful before date filtering or grouping.

## 43. Write a processed file

~~~python
orders.to_parquet(
    "cloud_data_lake/processed/orders/orders.parquet",
    index=False
)
~~~

Expected behavior: pandas writes a Parquet file if a compatible Parquet engine is installed, commonly pyarrow or fastparquet.

The command may raise an ImportError if no compatible engine is installed. Install and verify the required engine in your active environment.

## 44. Read the processed file

~~~python
processed_orders = pd.read_parquet(
    "cloud_data_lake/processed/orders/orders.parquet"
)

print(processed_orders)
~~~

Expected rows:

| order_id | customer_id | order_date | amount |
|---:|---|---|---:|
| 101 | C001 | 2026-11-10 | 250.00 |
| 102 | C002 | 2026-11-10 | 499.00 |
| 103 | C001 | 2026-11-11 | 100.00 |

The date display may differ depending on the library version.

## 45. Create a daily summary

~~~python
daily_sales = (
    processed_orders
    .groupby("order_date", as_index=False)["amount"]
    .sum()
)

daily_sales = daily_sales.rename(
    columns={"amount": "total_sales"}
)

print(daily_sales)
~~~

Expected result:

| order_date | total_sales |
|---|---:|
| 2026-11-10 | 749.00 |
| 2026-11-11 | 100.00 |

This is a simple example of creating a curated business result.

## 46. Save the curated result

~~~python
daily_sales.to_csv(
    "cloud_data_lake/curated/sales_daily/daily_sales.csv",
    index=False
)
~~~

The file is now stored in the local curated folder.

Remember: local folders are a learning simulation. Actual cloud object storage uses a provider service API, bucket, object keys, permissions and network access.

---

# Part 9 — Common mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Treat object storage as a normal hard drive | Object APIs and key semantics differ | Understand object, bucket and key |
| Assume slash means a real directory | Prefixes can be virtual folder views | Design stable object keys |
| Store every dataset in one folder | Hard to govern and locate | Separate zones and datasets |
| Mix ingestion date with event date | Late data becomes confusing | Keep both concepts explicit |
| Partition by every column | Creates too many tiny partitions | Choose based on query and volume |
| Use CSV for every analytical workload | Repeated text parsing can be expensive | Evaluate Parquet for analytics |
| Assume Parquet is a database | It is a file format | Use a query engine/table layer |
| Delete raw data immediately | Reprocessing and audit may be impossible | Define retention and recovery |
| Ignore schema changes | Downstream jobs can break | Validate and document evolution |
| Silently drop bad records | Data loss is hidden | Quarantine and report rejects |
| Give all analysts raw access | Sensitive data exposure | Use curated access boundaries |

---

# Part 10 — Practice questions

## Exercise 1

What are the three main parts of an object?

**Answer:** Data, key and metadata.

## Exercise 2

What is the difference between a bucket and an object?

**Answer:** A bucket is a container for objects. An object is an individual stored item identified by its key.

## Exercise 3

Why does object storage often display folders even when its namespace is key-based?

**Answer:** Tools interpret prefixes separated by slashes and display them as folder-like groupings.

## Exercise 4

Why preserve raw files?

**Answer:** To support replay, debugging, auditing and reprocessing, subject to access and retention controls.

## Exercise 5

A source file arrives on November 10 but contains orders from November 9. Which dates should be recorded?

**Answer:** Keep the business event/order date and the ingestion date separately.

## Exercise 6

Why is Parquet useful for analytical processing?

**Answer:** It is columnar and stores typed data, allowing analytical engines to read selected columns efficiently and use compression/encoding.

## Exercise 7

What can go wrong if every customer gets a separate partition?

**Answer:** Too many small partitions or files can increase metadata and scheduling overhead.

## Exercise 8

What is quarantine?

**Answer:** A controlled location for records or files that fail validation, with reasons and traceability.

---

# Part 11 — Revision sheet

| Term | Remember |
|---|---|
| Object storage | Stores data as objects |
| Bucket | Container for objects |
| Object key | Name identifying an object |
| Prefix | Beginning portion of object keys |
| Metadata | Information describing an object |
| File storage | Files arranged in directories |
| Block storage | Block-based storage volumes |
| Data lake | Storage environment for varied raw and transformed data |
| Raw zone | Original source data |
| Processed zone | Standardized or cleaned data |
| Curated zone | Business-ready data |
| CSV | Delimited text |
| JSON | Nested key-value data format |
| Parquet | Columnar analytical file format |
| Avro | Row-oriented serialization format with schema |
| Partition | Organized subset of a dataset |
| Schema evolution | Change in data structure over time |
| Quarantine | Controlled area for rejected data |
| Retention | How long data is kept |
| Small-file problem | Overhead from too many tiny files |

## Final self-check

- [ ] Explain object, bucket, key and metadata.
- [ ] Compare object, file and block storage.
- [ ] Explain a data lake and its zones.
- [ ] Design a clear bucket/prefix layout.
- [ ] Distinguish event date from ingestion date.
- [ ] Compare CSV, JSON, Parquet and Avro.
- [ ] Explain partitioning and over-partitioning.
- [ ] Explain schema evolution and quarantine.
- [ ] Describe retention and access controls.
- [ ] Run the local file-layout exercise and inspect its outputs.

**Next lesson — Day 52: AWS S3 and Storage Management.** We will connect the object-storage concepts to AWS S3 buckets, objects, storage classes, lifecycle policies, access control and safe upload/download workflows.
