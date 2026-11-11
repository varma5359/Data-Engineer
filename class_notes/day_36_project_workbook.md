# Day 36 Project Workbook — TechStore Partitioned Batch Processor

## Goal
Read a CSV in bounded chunks, write date-based Parquet partitions, read one partition independently and reconcile all written rows.

## 1. Setup and input
```python
import pandas as pd
from pathlib import Path

raw_dir = Path("data/raw")
output_dir = Path("data/curated/orders")
raw_dir.mkdir(parents=True, exist_ok=True)
sample = pd.DataFrame({
    "order_id": ["O1","O2","O3","O4","O5","O6"],
    "order_date": ["2026-01-10","2026-01-10","2026-01-11","2026-01-11","2026-01-12","2026-01-12"],
    "amount": [100,200,50,75,120,80]
})
sample.to_csv(raw_dir / "orders.csv", index=False)
```

## 2. Read in chunks
```python
for chunk in pd.read_csv(raw_dir / "orders.csv", chunksize=2):
    print("Batch:", chunk["order_id"].tolist())
```
Expected batches: O1/O2, O3/O4, O5/O6. chunksize limits rows per DataFrame read, not necessarily total memory of later code.

## 3. Prepare partition key and write
Install `pyarrow` in the notebook kernel if Parquet support is unavailable.
```python
sample["order_date"] = pd.to_datetime(sample["order_date"])
sample["partition_date"] = sample["order_date"].dt.strftime("%Y-%m-%d")

for day, group in sample.groupby("partition_date"):
    folder = output_dir / ("order_date=" + day)
    folder.mkdir(parents=True, exist_ok=True)
    group.drop(columns=["partition_date"]).to_parquet(folder / "part.parquet", index=False)
    print(day, len(group))
```
Expected: three dates, two rows each.

## 4. Read one partition
```python
jan11 = pd.read_parquet(output_dir / "order_date=2026-01-11" / "part.parquet")
print(jan11["order_id"].tolist())
```
Expected: O3 and O4.

## 5. Reconcile all partition files
```python
total = 0
for file_path in output_dir.glob("order_date=*/part.parquet"):
    part = pd.read_parquet(file_path)
    print(file_path.parent.name, len(part))
    total += len(part)
print("Total:", total)
print("Source:", len(sample))
print("Reconciled:", total == len(sample))
```
Expected Total 6, Source 6, Reconciled True.

## 6. Failure drills
Delete one partition and rerun reconciliation. Write a date twice and decide overwrite vs append. Create an uneven date distribution and describe skew. Partition by order_id and explain why high cardinality creates too many small files.

## Acceptance
Raw source preserved; chunks bounded; Parquet readable; date partitions exist; counts reconcile; one partition is independently readable; pruning conditions and retry/write policy explained.
