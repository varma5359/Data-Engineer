# Day 14: Pandas for Data Engineering

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** Dataset Transformation Tool  
**Learning time:** 2–3 hours

## 1. Learning outcomes
- Understand Series and DataFrame.
- Load CSV and JSON data.
- Inspect and select columns and rows.
- Filter, sort, group and aggregate records.
- Merge related datasets.
- Handle missing values and duplicates carefully.
- Export transformed results.

## 2. What is Pandas?
Pandas is a Python library for working with labeled tabular data. A DataFrame is a two-dimensional table with rows and columns. A Series is a one-dimensional labeled array.

Pandas is useful for small-to-medium data preparation and analysis. It is not a replacement for distributed processing when data exceeds available memory.

Install:

```bash
python -m pip install pandas
```

## 3. Create a DataFrame
```python
import pandas as pd

orders = pd.DataFrame({
    "order_id": [501, 502, 503, 504],
    "customer_id": [1001, 1002, 1001, 1003],
    "amount": [1200.0, 800.0, 450.0, 950.0],
    "status": ["PAID", "PENDING", "PAID", "CANCELLED"]
})

print(orders)
```

## 4. Inspect a dataset
```python
print(orders.head())
print(orders.tail(2))
print(orders.shape)
print(orders.columns)
print(orders.dtypes)
print(orders.info())
print(orders.describe())
```

Use `head` for samples, `shape` for row/column counts, `dtypes` for types and `describe` for numerical summaries. These are initial checks, not complete data-quality validation.

## 5. Select columns
```python
print(orders["amount"])
print(orders[["order_id", "amount"]])
```

One column selection returns a Series; a list of column names returns a DataFrame.

## 6. Filter rows
```python
paid_orders = orders[orders["status"] == "PAID"]
print(paid_orders)
```

Multiple conditions:

```python
high_value_paid = orders[
    (orders["status"] == "PAID") &
    (orders["amount"] >= 500)
]
print(high_value_paid)
```

Use `&` for element-wise AND and `|` for OR; wrap each condition in parentheses.

## 7. Create and transform columns
```python
orders["processing_fee"] = orders["amount"] * 0.02
orders["net_amount"] = orders["amount"] - orders["processing_fee"]

print(orders[["order_id", "amount", "net_amount"]])
```

The fee is a scenario assumption, not a universal business rule. Confirm calculations with the business owner.

## 8. Sort data
```python
sorted_orders = orders.sort_values(
    by="amount",
    ascending=False
)

print(sorted_orders)
```

Use stable tie-breakers when output order must be deterministic.

## 9. Group and aggregate
```python
status_summary = orders.groupby("status").agg(
    order_count=("order_id", "count"),
    total_amount=("amount", "sum"),
    average_amount=("amount", "mean")
).reset_index()

print(status_summary)
```

Choose the correct aggregation semantics. Counting non-null order IDs is not necessarily the same as counting distinct orders if duplicates exist.

## 10. Missing values
```python
sample = pd.DataFrame({
    "order_id": [1, 2, 3],
    "amount": [100.0, None, 300.0],
    "city": ["Hyderabad", None, "Pune"]
})

print(sample.isna().sum())
```

Options depend on meaning:
- Keep missing values when absence is meaningful.
- Fill only when a justified default exists.
- Reject/quarantine records when required fields are missing.
- Ask the data owner when semantics are unclear.

```python
sample["city"] = sample["city"].fillna("UNKNOWN")
print(sample)
```

Use "UNKNOWN" only if that category is approved and distinct from genuinely unknown or not applicable values.

## 11. Duplicates
```python
duplicate_sample = pd.DataFrame({
    "order_id": [1, 2, 2, 3],
    "amount": [100, 200, 200, 300]
})

print(duplicate_sample.duplicated().sum())
print(duplicate_sample.drop_duplicates())
```

Exact duplicate rows and duplicate business keys are different. Define the business key and decide which version wins before removing duplicates.

## 12. Merge datasets
```python
customers = pd.DataFrame({
    "customer_id": [1001, 1002, 1003],
    "city": ["Hyderabad", "Pune", "Chennai"]
})

enriched_orders = orders.merge(
    customers,
    on="customer_id",
    how="left",
    validate="many_to_one"
)

print(enriched_orders)
```

Join types:
- inner: matching keys only.
- left: all left rows plus matches.
- right: all right rows plus matches.
- outer: all keys from both sides.

Use `validate` to assert expected relationship cardinality. A many-to-many join can multiply rows and inflate totals.

## 13. Read CSV and JSON
```python
orders_from_csv = pd.read_csv("orders.csv")
orders_from_json = pd.read_json("orders.json")

print(orders_from_csv.head())
print(orders_from_json.head())
```

Specify data types, date parsing and null conventions deliberately for production ingestion.

## 14. Write output
```python
orders.to_csv("transformed_orders.csv", index=False)
orders.to_json("transformed_orders.json", orient="records", indent=2)
```

Do not export an unintended DataFrame index as a business column.

## 15. Dates and timestamps
```python
events = pd.DataFrame({
    "event_time": ["2026-01-01 10:00:00", "invalid"]
})

events["event_time"] = pd.to_datetime(
    events["event_time"],
    errors="coerce"
)

print(events)
print(events["event_time"].isna().sum())
```

`errors="coerce"` converts invalid values to `NaT`; inspect and report those rows rather than silently dropping them. For production, define timezone handling explicitly.

## 16. Mini-project: Dataset Transformation Tool

Use orders and customers datasets.

Steps:
1. Read both datasets.
2. Inspect shape, types, nulls and duplicate keys.
3. Standardize status values.
4. Convert amount to numeric with invalid-value reporting.
5. Filter records according to a documented business rule.
6. Join customer attributes with a validated relationship.
7. Aggregate order count and amount by city/status.
8. Export curated output and a rejection report.

Example summary:

```python
city_summary = enriched_orders.groupby("city").agg(
    order_count=("order_id", "nunique"),
    order_amount=("amount", "sum")
).reset_index()

print(city_summary)
```

Check whether the business metric should include pending/cancelled/refunded orders before publishing totals.

## 17. Performance and scale
- Select only needed columns with `usecols`.
- Specify efficient data types where safe.
- Read large CSVs with `chunksize`.
- Avoid row-wise Python loops when vectorized operations are appropriate.
- Watch memory use after merges.
- Consider SQL pushdown or Spark for data too large for one machine.

Chunk example:

```python
total_rows = 0

for chunk in pd.read_csv("large_orders.csv", chunksize=50000):
    total_rows += len(chunk)

print(total_rows)
```

## 18. Common mistakes
| Mistake | Better practice |
|---|---|
| Treating every missing value as zero | Apply business-specific missing policy |
| Dropping duplicates without key definition | Define natural/business key |
| Many-to-many join unexpectedly | Check key uniqueness and merge validation |
| Chained assignment ambiguity | Use explicit `.loc` assignment |
| Loading huge files at once | Chunk or use distributed engine |
| Ignoring date timezone | Define UTC/local business time |
| Exporting index accidentally | Set `index=False` when appropriate |

## 19. Day 14 revision
- DataFrame is a labeled table; Series is one-dimensional.
- Inspect data before transformation.
- Filter, group, aggregate and merge with clear semantics.
- Validate join cardinality.
- Missing values and duplicates require business rules.
- Pandas is memory-bound; use chunking or distributed tools when needed.
