
# Day 36 Project Workbook — TechStore Partitioned Batch Processor

**Goal:** Process a CSV in bounded chunks, transform each chunk, write date-based Parquet partitions, read one date independently and reconcile the result.  
**Prerequisites:** Pandas and file handling. **Time:** 90–120 minutes.

## 1. Understand the problem

A large source file may not fit comfortably in memory. A batch is a group processed together. A partition is a storage organization that groups records by a chosen key. Batch size and partition size solve different problems.

## 2. Setup

CMD:
~~~bat
mkdir day36_batch_lab
cd day36_batch_lab
mkdir data output
python -m pip install pandas pyarrow
~~~

Create day36_batch.ipynb. If installation fails, use python -m pip show pandas pyarrow and confirm Jupyter uses the same Python environment.

## 3. Create the source CSV

~~~python
import pandas as pd
from pathlib import Path

data_dir = Path("data")
output_dir = Path("output") / "orders"
data_dir.mkdir(exist_ok=True, parents=True)
output_dir.mkdir(exist_ok=True, parents=True)

orders = pd.DataFrame({
    "order_id": ["O1", "O2", "O3", "O4", "O5", "O6", "O7"],
    "order_date": ["2026-01-10", "2026-01-10", "2026-01-11",
                   "2026-01-11", "2026-01-12", "2026-01-12", "2026-01-12"],
    "amount": [100, 200, 150, 80, 300, 120, 90]
})
source_path = data_dir / "orders.csv"
orders.to_csv(source_path, index=False)
print("Created", source_path, "rows =", len(orders))
~~~

Expected: data/orders.csv with 7 rows.

## 4. Read bounded chunks

~~~python
chunk_size = 3
rows_seen = 0
batch_number = 0

for chunk in pd.read_csv(source_path, chunksize=chunk_size):
    batch_number += 1
    rows_seen += len(chunk)
    print("Batch", batch_number, "rows", len(chunk),
          "first ID", chunk.iloc[0]["order_id"])
print("Total rows:", rows_seen)
~~~

Expected:
~~~text
Batch 1 rows 3 first ID O1
Batch 2 rows 3 first ID O4
Batch 3 rows 1 first ID O7
Total rows: 7
~~~

Seven records become batches of 3 + 3 + 1. The final batch can be smaller.

## 5. Write date-partitioned Parquet

~~~python
rows_written = 0
for chunk in pd.read_csv(source_path, chunksize=chunk_size):
    chunk["order_date"] = pd.to_datetime(chunk["order_date"])
    chunk["order_year"] = chunk["order_date"].dt.year.astype("string")
    chunk["order_month"] = chunk["order_date"].dt.month.astype("string").str.zfill(2)
    chunk["order_day"] = chunk["order_date"].dt.day.astype("string").str.zfill(2)
    chunk.to_parquet(
        output_dir,
        engine="pyarrow",
        partition_cols=["order_year", "order_month", "order_day"],
        index=False,
        existing_data_behavior="overwrite_or_ignore"
    )
    rows_written += len(chunk)
print("Rows written:", rows_written)
~~~

Expected: Rows written: 7. The directory contains Hive-style folders such as order_year=2026/order_month=01/order_day=10. It is a Parquet dataset directory, not one file.

## 6. Read the complete dataset

~~~python
loaded = pd.read_parquet(output_dir, engine="pyarrow")
print("Rows read:", len(loaded))
print(loaded[["order_id", "amount"]].sort_values("order_id").to_string(index=False))
~~~

Expected count: 7. File order is not guaranteed, so sort only to make classroom output predictable.

## 7. Read one date partition

~~~python
one_day = pd.read_parquet(
    output_dir,
    filters=[("order_year", "==", "2026"),
             ("order_month", "==", "01"),
             ("order_day", "==", "12")]
)
print(one_day[["order_id", "amount"]].sort_values("order_id").to_string(index=False))
print("Rows for 12 Jan:", len(one_day))
~~~

Expected IDs: O5, O6, O7. Expected count: 3. Partition pruning depends on the storage layout and query engine; a filtered result alone does not prove that irrelevant files were skipped. Production engines expose scan metrics or query plans.

## 8. Reconcile

~~~python
source_check = pd.read_csv(source_path)
output_check = pd.read_parquet(output_dir)
assert len(source_check) == 7
assert len(output_check) == len(source_check)
assert output_check["order_id"].nunique() == len(source_check)
assert set(source_check["order_id"]) == set(output_check["order_id"])
print("Source rows:", len(source_check))
print("Output rows:", len(output_check))
print("Unique output IDs:", output_check["order_id"].nunique())
print("Reconciliation passed")
~~~

Expected: source 7, output 7, unique IDs 7, reconciliation passed.

## 9. Choose partition keys intelligently

Date is useful when consumers filter by date. Partitioning by order_id usually creates too many tiny folders. Region may be useful only when query patterns justify it. Too many tiny files create listing and scheduling overhead. Uneven data distribution creates skew. Measure real query patterns and volume before choosing a key.

## 10. Rerun and failure drills

For a repeatable classroom run, remove only the lab output folder before starting:

~~~python
import shutil
if output_dir.exists():
    shutil.rmtree(output_dir)
output_dir.mkdir(parents=True, exist_ok=True)
print("Clean lab output ready")
~~~

1. Change chunk_size to 2; explain why batch count changes but total rows remain 7.
2. Add an invalid date and validate it before writing.
3. Run without clearing output; inspect duplicate risk.
4. Create one partition per order ID; explain the small-file problem.
5. Put most rows on one date; explain skew.

Never delete production partitions without a controlled replacement plan and backup.

## 11. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Missing pyarrow | Package absent in active environment | Install in the same interpreter used by Jupyter |
| Duplicate rows after rerun | Old files appended again | Use run-scoped staging or safe partition replacement |
| Too many folders | High-cardinality partition key | Select key from query patterns |
| One partition is huge | Skew | Inspect distribution and revise layout |
| Memory still high | Chunk too large or transformation copies data | Measure peak memory and reduce batch size |

## 12. Acceptance checklist

- [ ] Read with chunksize.
- [ ] Every batch processed.
- [ ] Parquet written into date partitions.
- [ ] One date read independently.
- [ ] Source and output counts match.
- [ ] Unique keys match.
- [ ] Rerun behavior tested.
- [ ] Can explain batch size, partition key, pruning, small files and skew.

**Project explanation:** "I processed a CSV in bounded Pandas chunks, derived date partition columns and wrote Parquet partitions. I read one date independently and reconciled source/output counts and keys. I also tested reruns and considered small-file and skew risks."
