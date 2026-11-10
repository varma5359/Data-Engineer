# Day 14 — Pandas Transformations, Filtering and Aggregation
**Phase 2: Python for Data Engineering**

## 1. DataFrame
A DataFrame is a labeled table of rows and columns. Pandas supports cleaning, filtering, joining, and aggregation for datasets that fit available memory (or chunked workflows).

```python
import pandas as pd
orders = pd.DataFrame({
    "order_id": [101, 102, 103],
    "region": ["South", "North", "South"],
    "amount": [500, 700, 300]
})
print(orders)
```

## 2. Inspect first
```python
print(orders.head())
print(orders.shape)
print(orders.dtypes)
print(orders.isna().sum())
```
Inspection reveals row/column count, types, and missingness before transformations.

## 3. Filter and select
```python
south_orders = orders[orders["region"] == "South"]
print(south_orders[["order_id", "amount"]])
```
Output: orders 101 and 103.

Use boolean masks; avoid modifying a filtered view ambiguously—assign with `loc` or create an explicit copy when needed.

## 4. Add/transform columns
```python
orders["amount_with_tax"] = orders["amount"] * 1.18
orders["region"] = orders["region"].str.strip().str.upper()
print(orders[["region", "amount_with_tax"]])
```

## 5. Group and aggregate
```python
summary = orders.groupby("region", as_index=False)["amount"].sum()
print(summary)
```
Output: North 700; South 800.

## 6. Join datasets
```python
customers = pd.DataFrame({"customer_id": [1, 2], "name": ["Asha", "Ravi"]})
sales = pd.DataFrame({"customer_id": [1, 2, 1], "amount": [100, 200, 50]})
joined = sales.merge(customers, on="customer_id", how="left", validate="many_to_one")
print(joined)
```
Use `validate` to assert expected relationship and detect unexpected duplicates.

## 7. Missing values and duplicates
```python
print(orders.drop_duplicates(subset=["order_id"]))
print(orders["amount"].isna().sum())
```
Do not drop/fill missing values blindly; decide using business meaning and record rejection policy.

## 8. Mini-project — Dataset Transformation Tool
Load a sample CSV, inspect schema, standardize region, filter invalid amounts, remove duplicates by defined key, aggregate revenue, and write output. Record input/output/rejected counts.

## Scale warning
Pandas generally works in memory. For very large datasets, use chunking, database SQL, or Spark rather than assuming more RAM is always the solution.

## Revision
Inspect → clean → filter → transform → join → aggregate → validate → export.