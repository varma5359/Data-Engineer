# Day 36 — Batch Processing and Partitioning

**Phase:** 4 — Data Processing and Pipelines  
**Syllabus:** Batch processing, partitioning, date-based partitions, file organization, Parquet partitioning and partition pruning  
**Mini-project:** TechStore Partitioned Batch Processor  
**Level:** Beginner → junior Data Engineer  
**Study time:** 2–3 hours

> Work cell by cell. Run code, inspect output, then answer the checkpoint. The purpose is to understand why a pipeline processes data in bounded units and how partitioning changes storage and query work.

## 0. Learning outcomes

You will be able to:
- explain batch processing in plain English and compare it with streaming;
- identify a batch boundary and batch size;
- process records in small chunks rather than loading everything at once;
- explain partition, partition key, partition directory and partition pruning;
- create date-based partitions and write Parquet files;
- avoid common partition mistakes such as too many tiny files and skew;
- reconcile records across batches/partitions;
- build a small partitioned output and verify it.

## 1. Business scenario

TechStore receives millions of orders. Analysts often ask:
- “Show orders from 2026-01-10.”
- “Calculate daily revenue for January.”
- “Compare last month's orders.”

If all orders are stored in one huge file, every query may need to inspect much more data than necessary. If data is organized by date, a query for one day can read only that date's partition (when the engine and filter allow pruning).

We will practice with a tiny dataset first. The same idea applies to larger data, but large-scale performance also depends on file size, engine, statistics, compression, data layout and query design.

## 2. Batch processing from zero

A **batch** is a bounded group of records processed together. A batch boundary may be:
- a fixed record count (e.g. 1,000 rows);
- a time window (hour/day);
- a source file;
- a source partition;
- a scheduled arrival window.

Batch processing means the system collects or reads a bounded unit, processes it, writes a result and records progress. It does not require the data to arrive in real time.

```text
Source records -> Choose batch boundary -> Process batch -> Write result -> Verify -> Next batch
```

### Batch vs streaming

| Batch | Streaming |
|---|---|
| Process bounded groups | Process events continuously or in small time/event windows |
| Often scheduled | Often continuously running |
| Easier to rerun a defined batch | Requires event/state/replay design |
| Good for daily reports/backfills | Good for live alerts/event reactions |
| Latency depends on schedule/window | Lower latency, but more operational complexity |

Neither is universally better. Choose based on business latency, cost, volume and operational requirements.

## 3. First example: process records in batches

```python
orders = list(range(1, 11))
batch_size = 3

for start in range(0, len(orders), batch_size):
    batch = orders[start:start + batch_size]
    print(batch)
```

Expected output:
```text
[1, 2, 3]
[4, 5, 6]
[7, 8, 9]
[10]
```

### Explain every important part
- `orders`: ten sample order IDs.
- `batch_size = 3`: each full batch contains at most three values.
- `range(0, len(orders), batch_size)`: starts at zero, moves by three, and stops before ten.
- `orders[start:start + batch_size]`: selects the current slice.
- Last batch contains one record because ten is not divisible by three.

**Checkpoint:** Why is the last batch smaller? What would change if batch_size were 4?

## 4. Batch size is a trade-off

Small batches:
- use less memory per batch;
- reduce the amount to retry after a failure;
- may increase overhead (more queries, writes, commits or files).

Large batches:
- reduce per-batch overhead;
- can improve throughput;
- require more memory and make retries larger;
- may exceed source/target limits.

There is no universal best batch size. Measure memory, throughput, latency, database limits and retry cost.

## 5. Read CSV in chunks

### Create a sample CSV

```python
import pandas as pd

sample = pd.DataFrame({
    "order_id": ["O1", "O2", "O3", "O4", "O5", "O6"],
    "order_date": ["2026-01-10", "2026-01-10", "2026-01-11", "2026-01-11", "2026-01-12", "2026-01-12"],
    "amount": [100, 200, 50, 75, 120, 80]
})
sample.to_csv("orders.csv", index=False)
```

### Read in chunks

```python
for chunk in pd.read_csv("orders.csv", chunksize=2):
    print("Chunk rows:", len(chunk))
    print(chunk)
```

Expected chunk sizes: 2, 2, 2. The loop receives a DataFrame chunk at each iteration.

**Why chunksize?** It bounds how much CSV data pandas reads into a DataFrame at one time. It does not automatically make every later operation memory-safe; avoid collecting all chunks into one giant list if the goal is bounded memory.

## 6. What is partitioning?

A **partition** is a logical subset of data grouped by a partition key. A partition key is a field used to divide records into groups.

Examples:
- date: `order_date`;
- country: `country`;
- department: `department`.

Partitioning is primarily a data organization technique. Batch processing is a processing technique. They are related but not identical:
- You can process a batch without partitioning storage.
- You can store data in partitions and process many partitions in one batch.

## 7. Partition by date (concept first)

Our orders:

| order_id | order_date | amount |
|---|---|---:|
| O1 | 2026-01-10 | 100 |
| O2 | 2026-01-10 | 200 |
| O3 | 2026-01-11 | 50 |
| O4 | 2026-01-11 | 75 |
| O5 | 2026-01-12 | 120 |
| O6 | 2026-01-12 | 80 |

Date is a good partition candidate for daily reporting because common queries filter by date. It may not be good if queries rarely filter date or the table is extremely small.

## 8. Create date partitions in pandas

```python
sample["order_date"] = pd.to_datetime(sample["order_date"])
sample["partition_date"] = sample["order_date"].dt.strftime("%Y-%m-%d")
sample
```

The date is parsed into a datetime type. `.dt.strftime` creates a date string suitable for a directory name.

### Group records by partition

```python
for partition_date, group in sample.groupby("partition_date"):
    print(partition_date, len(group))
```

Expected:
```text
2026-01-10 2
2026-01-11 2
2026-01-12 2
```

Each group contains rows for one day.

## 9. Write partitioned Parquet files

Parquet is a columnar file format: values are stored by column rather than as row-oriented text. It supports efficient column selection and compression in many analytical engines.

Install if needed in the notebook environment:
```text
pip install pyarrow
```

Write one folder per date:
```python
for partition_date, group in sample.groupby("partition_date"):
    folder = "orders_partitioned/order_date=" + partition_date
    group.drop(columns=["partition_date"]).to_parquet(folder + "/part.parquet", index=False)
```

**Note:** In a real notebook, create the directory first using `pathlib.Path(folder).mkdir(parents=True, exist_ok=True)`. The full runnable workbook includes the folder creation and read-back checks.

Typical layout:
```text
orders_partitioned/
  order_date=2026-01-10/part.parquet
  order_date=2026-01-11/part.parquet
  order_date=2026-01-12/part.parquet
```

The `order_date=...` naming convention is commonly recognized by Hive-style partition readers. The partition value may be represented in the path rather than repeated in each data file; readers can reconstruct it when configured to do so.

## 10. Partition pruning

**Partition pruning** means a query engine skips partitions that cannot match the filter.

Example request:
```sql
SELECT order_id, amount
FROM orders
WHERE order_date = '2026-01-11';
```

If the table is correctly registered/laid out by date and the engine recognizes the predicate, it can read the 2026-01-11 partition rather than scan all date partitions.

**Important:** Writing folders alone does not guarantee pruning. The table/catalog, partition metadata, file format, query predicate and engine must support it. Confirm with an execution plan or scan metrics.

## 11. Partition key selection

A useful partition key:
- appears frequently in query filters;
- divides data into meaningful, reasonably sized groups;
- has manageable number of distinct values;
- is stable and available at ingestion.

Poor choices:
- unique order_id → millions of tiny partitions;
- customer_id with millions of customers → excessive directories/files;
- a field with one value for almost all records → skew;
- date partition at minute/second granularity for low volume → tiny files.

## 12. Partition size, small files and skew

### Small-file problem
Thousands of tiny files increase listing, metadata and scheduling overhead. A partitioned dataset should usually target sensible file sizes based on engine/cloud/object-store guidance—not create one file per row or tiny batch.

### Skew
Skew occurs when one partition receives far more data than others. Example: partition by country, but 90% of orders belong to one country. That partition becomes a bottleneck.

Possible approaches:
- choose a better key or multi-level layout;
- use a secondary distribution key for processing (not necessarily storage partition);
- rebalance or handle heavy partitions separately;
- measure actual distribution before deciding.

## 13. Read only a selected partition (local demonstration)

```python
day = pd.read_parquet("orders_partitioned/order_date=2026-01-11/part.parquet")
day
```

Expected rows: O3 and O4. This demonstrates selecting a path. It is not itself proof that a query engine performed automatic partition pruning.

## 14. Reconciliation across batches and partitions

For a batch run, record:
- input rows;
- processed rows;
- accepted rows;
- rejected rows;
- output rows;
- batch/partition ID;
- start/end time;
- status.

At minimum:
```text
Input rows = Accepted rows + Rejected rows
```

If every accepted record is written once:
```text
Accepted rows = Output rows
```

For partitioning:
```text
Total rows = Sum of rows in all partitions
```

These checks catch missing partitions, duplicate writes and incomplete runs.

## 15. Common mistakes

| Mistake | Why it is wrong | Correction |
|---|---|---|
| One batch equals one partition | Processing unit and storage layout are different concepts | Explain both independently |
| Partition by unique ID | Creates too many tiny partitions | Select query-aligned, bounded-cardinality key |
| Assume folder layout guarantees pruning | Engine/catalog may not recognize it | Verify scan plan/metrics |
| Use huge batches without measurement | Memory and retry risk | Tune with observed workload |
| Create one tiny file per batch | Small-file overhead | Compact files where appropriate |
| Ignore skew | One partition can dominate runtime | Profile key distribution |
| Mark success after one partition | Other partitions may fail | Track all partition statuses and reconcile |
| Retry a partition with append blindly | Duplicates may appear | Idempotent overwrite/merge or stable keys |

## 16. Debugging questions

**Only one date appears in output:** inspect date parsing, filter values, partition paths and whether all groups were written.

**Parquet read fails:** check pyarrow installation, path, file existence, permissions and schema.

**Output count is smaller:** compare source count, per-partition counts, accepted/rejected and write results.

**Pipeline is slow despite partitioning:** inspect whether filters enable pruning, number/size of files, skew, selected columns and execution plan. Partitioning is not magic acceleration.

## 17. Production considerations

- Partition layout should follow query patterns and data volume.
- Store partition metadata/catalog and schema version.
- Make writes idempotent; define overwrite/merge behavior per partition.
- Track each partition's status; publish only when required partitions succeed.
- Avoid high-cardinality partition keys.
- Compact small files and monitor file counts.
- Handle late-arriving data by reprocessing affected date partitions safely.
- Keep raw and curated layers separate.
- Secure data and avoid sensitive partition keys in public paths.
- Use execution metrics to prove pruning and performance.

## 18. Revision checklist

- [ ] Explain batch processing and streaming difference.
- [ ] Process a list in batches and explain the last smaller batch.
- [ ] Read CSV with chunksize.
- [ ] Explain partition vs batch.
- [ ] Create date groups.
- [ ] Write/read Parquet partition.
- [ ] Explain pruning and its conditions.
- [ ] Identify small-file and skew risks.
- [ ] Reconcile batch/partition counts.

## 19. What comes next?

Day 37 — **Incremental Loading and Upserts**. We will compare full and incremental loads, understand watermarks and checkpoints, simulate inserts/updates, and safely apply upserts without duplicating records.
