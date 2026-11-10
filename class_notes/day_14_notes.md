# Day 14: Pandas for Data Engineering

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** Dataset Transformation Tool  
**Learning time:** 2–3 hours

## 1. Learning outcomes

- Explain Series and DataFrame.
- Load CSV and JSON datasets.
- Inspect rows, columns, data types and missing values.
- Select columns and filter records.
- Sort rows and create calculated columns.
- Handle missing values and duplicates with clear rules.
- Group and aggregate business data.
- Merge related datasets and validate join behavior.
- Export a transformed dataset.

## 2. What is Pandas?

Pandas is a Python library for working with labeled tabular data.

- **Series:** one-dimensional labeled data, similar to one column.
- **DataFrame:** two-dimensional table containing rows and columns.

Pandas is useful for local data inspection and transformation. It is not a replacement for SQL databases or distributed processing when data exceeds available memory.

## 3. Install and import Pandas

```bash
python -m pip install pandas
```

```python
import pandas as pd

print("Pandas version:", pd.__version__)
```

**Expected output:** A Pandas version number, such as `2.x.x`. Exact version depends on the environment.

## 4. Create a realistic retail dataset

We will work with order records from a small online store.

```python
orders = pd.DataFrame({
    "order_id": [1001, 1002, 1003, 1004, 1005, 1006, 1007],
    "customer_id": [201, 202, 201, 203, 204, 202, 205],
    "city": ["Hyderabad", "Chennai", "Hyderabad", "Pune", "Chennai", "Chennai", "Pune"],
    "category": ["Laptop", "Mobile", "Accessories", "Laptop", "Mobile", "Accessories", "Laptop"],
    "amount": [55000, 18000, 2500, 62000, 22000, None, 48000],
    "quantity": [1, 1, 2, 1, 1, 3, 1],
    "status": ["PAID", "PAID", "CANCELLED", "PAID", "PENDING", "PAID", "PAID"]
})

print(orders)
```

**Expected output**

```text
   order_id  customer_id        city     category   amount  quantity     status
0      1001          201   Hyderabad       Laptop  55000.0         1       PAID
1      1002          202     Chennai       Mobile  18000.0         1       PAID
2      1003          201   Hyderabad  Accessories   2500.0         2  CANCELLED
3      1004          203        Pune       Laptop  62000.0         1       PAID
4      1005          204     Chennai       Mobile  22000.0         1    PENDING
5      1006          202     Chennai  Accessories      NaN         3       PAID
6      1007          205        Pune       Laptop  48000.0         1       PAID
```

**Interpretation:** The dataset includes a missing amount and multiple order statuses. This lets us practise realistic checks before aggregation.

## 5. Inspect a DataFrame

### 5.1 First and last rows

```python
print(orders.head(3))
print(orders.tail(2))
```

**Expected output:** The first three rows, followed by the last two rows.

- `head(3)` returns the first three records.
- `tail(2)` returns the last two records.

### 5.2 Shape, columns and data types

```python
print("Shape:", orders.shape)
print("Columns:", list(orders.columns))
print(orders.dtypes)
```

**Expected output**

```text
Shape: (7, 7)
Columns: ['order_id', 'customer_id', 'city', 'category', 'amount', 'quantity', 'status']
order_id         int64
customer_id      int64
city            object
category        object
amount         float64
quantity         int64
status          object
dtype: object
```

**Interpretation:** There are seven rows and seven columns. The amount column is floating point because it contains a missing value.

### 5.3 Summary and missing values

```python
print(orders.info())
print(orders.isna().sum())
print(orders.describe())
```

**Expected missing-value output**

```text
order_id       0
customer_id    0
city           0
category       0
amount         1
quantity       0
status         0
dtype: int64
```

- `info()` reports non-null counts and dtypes.
- `isna()` marks missing values.
- `sum()` counts missing values by column.
- `describe()` summarizes numerical columns.

## 6. Select columns

### 6.1 Select one column

```python
print(orders["city"])
```

**Expected output:** A Series containing the city value for each row.

### 6.2 Select multiple columns

```python
order_view = orders[["order_id", "city", "amount"]]
print(order_view.head(3))
```

**Expected output**

```text
   order_id        city   amount
0      1001   Hyderabad  55000.0
1      1002     Chennai  18000.0
2      1003   Hyderabad   2500.0
```

One column name returns a Series; a list of column names returns a DataFrame.

## 7. Filter rows

### 7.1 Select paid orders

```python
paid_orders = orders[orders["status"] == "PAID"]
print(paid_orders[["order_id", "amount", "status"]])
```

**Expected output**

```text
   order_id   amount status
0      1001  55000.0   PAID
1      1002  18000.0   PAID
3      1004  62000.0   PAID
5      1006      NaN   PAID
6      1007  48000.0   PAID
```

### 7.2 Combine conditions

```python
high_value_paid = orders[
    (orders["status"] == "PAID") &
    (orders["amount"] >= 50000)
]

print(high_value_paid[["order_id", "amount"]])
```

**Expected output**

```text
   order_id   amount
0      1001  55000.0
3      1004  62000.0
```

Use `&` for element-wise AND and `|` for OR. Put each condition in parentheses. Do not use Python's `and` between Series conditions.

## 8. Sort rows

```python
sorted_orders = orders.sort_values(
    by="amount",
    ascending=False,
    na_position="last"
)

print(sorted_orders[["order_id", "amount"]])
```

**Expected output**

```text
   order_id   amount
3      1004  62000.0
0      1001  55000.0
6      1007  48000.0
4      1005  22000.0
1      1002  18000.0
2      1003   2500.0
5      1006      NaN
```

- `by` chooses the sort column.
- `ascending=False` sorts largest to smallest.
- `na_position="last"` places missing values at the end.

## 9. Create calculated columns

### 9.1 Calculate order value

```python
orders["gross_value"] = orders["amount"] * orders["quantity"]

print(orders[["order_id", "amount", "quantity", "gross_value"]])
```

**Expected output:** Each row's amount multiplied by quantity. The row with missing amount remains missing.

### 9.2 Create a simple category label

```python
orders["order_size"] = "STANDARD"
orders.loc[orders["amount"] >= 50000, "order_size"] = "HIGH_VALUE"

print(orders[["order_id", "amount", "order_size"]])
```

**Expected output:** Orders with amount at least 50,000 are labeled HIGH_VALUE; other rows are STANDARD. Missing amount does not satisfy the comparison and remains STANDARD.

**Business-rule caution:** Thresholds must come from business requirements, not assumptions made by the engineer.

## 10. Handle missing values

Missing values require a business decision. Do not fill every missing value with zero automatically.

### 10.1 Identify missing records

```python
missing_amount_rows = orders[orders["amount"].isna()]
print(missing_amount_rows[["order_id", "amount"]])
```

**Expected output**

```text
   order_id  amount
5      1006     NaN
```

### 10.2 Remove rows only when policy allows

```python
orders_without_missing_amount = orders.dropna(subset=["amount"])
print("Rows before:", len(orders))
print("Rows after:", len(orders_without_missing_amount))
```

**Expected output**

```text
Rows before: 7
Rows after: 6
```

### 10.3 Fill with a justified value

For demonstration, fill the missing amount with the median amount. In a real pipeline, confirm that this is a valid business rule.

```python
median_amount = orders["amount"].median()
orders_filled = orders.copy()
orders_filled["amount"] = orders_filled["amount"].fillna(median_amount)

print("Median:", median_amount)
print("Missing after fill:", orders_filled["amount"].isna().sum())
```

**Expected output**

```text
Median: 35000.0
Missing after fill: 0
```

The median is calculated from the six observed amounts. Filling a missing financial transaction amount with a median is usually not appropriate for production accounting; it is only a technique demonstration.

## 11. Remove duplicate records

```python
duplicate_sample = pd.DataFrame({
    "order_id": [1, 2, 2, 3],
    "amount": [100, 200, 200, 300]
})

print("Duplicate rows:", duplicate_sample.duplicated().sum())

deduplicated = duplicate_sample.drop_duplicates()
print(deduplicated)
```

**Expected output**

```text
Duplicate rows: 1
   order_id  amount
0         1     100
1         2     200
3         3     300
```

Use `subset=["order_id"]` when order_id is the defined business key. Do not remove duplicates without deciding which record should survive when duplicate keys contain different values.

## 12. Group and aggregate

### 12.1 Revenue by city

```python
paid_with_amount = orders[
    (orders["status"] == "PAID") &
    (orders["amount"].notna())
]

city_summary = paid_with_amount.groupby("city").agg(
    total_revenue=("amount", "sum"),
    order_count=("order_id", "count"),
    average_order_value=("amount", "mean")
).reset_index()

print(city_summary)
```

**Expected output**

```text
        city  total_revenue  order_count  average_order_value
0     Chennai        18000.0            1              18000.0
1   Hyderabad        55000.0            1              55000.0
2        Pune       110000.0            2              55000.0
```

**Explanation**

- Filter to paid orders with a known amount before aggregation.
- `groupby("city")` creates one group per city.
- `sum` adds amount values.
- `count` counts non-null order IDs.
- `mean` calculates the average amount.
- `reset_index()` turns the group key back into a regular column.

**Interpretation:** Pune has the highest total paid amount in this sample because it has two paid orders. The metric is based on the amount column and excludes the missing amount.

### 12.2 Revenue by category

```python
category_summary = paid_with_amount.groupby("category")["amount"].sum().reset_index()
print(category_summary)
```

**Expected output**

```text
      category   amount
0       Laptop  165000.0
1       Mobile   18000.0
```

The cancelled accessories order is excluded by the paid-order filter.

## 13. Merge related datasets

A common data-engineering task is joining orders to customer attributes.

### 13.1 Create a customer lookup table

```python
customers = pd.DataFrame({
    "customer_id": [201, 202, 203, 204, 205],
    "customer_name": ["Ravi", "Anu", "Kiran", "Meena", "Sana"],
    "segment": ["Gold", "Silver", "Gold", "Bronze", "Silver"]
})

print(customers)
```

### 13.2 Merge orders with customers

```python
orders_with_customer = orders.merge(
    customers,
    on="customer_id",
    how="left",
    validate="many_to_one"
)

print(orders_with_customer[["order_id", "customer_name", "segment"]])
```

**Expected output**

```text
   order_id customer_name segment
0      1001          Ravi    Gold
1      1002           Anu  Silver
2      1003          Ravi    Gold
3      1004         Kiran    Gold
4      1005         Meena  Bronze
5      1006           Anu  Silver
6      1007          Sana  Silver
```

**Parameter explanation**

- `on`: key column used to match rows.
- `how="left"`: retain every order, even if no customer match exists.
- `validate="many_to_one"`: each order can match at most one customer row. It raises an error if the customer lookup has duplicate keys.

### 13.3 Find unmatched customers

```python
unmatched = orders.merge(
    customers,
    on="customer_id",
    how="left",
    indicator=True
)

print(unmatched[unmatched["_merge"] != "both"][["order_id", "customer_id", "_merge"]])
```

**Expected output**

An empty DataFrame, because every order customer_id exists in the customer table.

**Interpretation:** An empty unmatched result indicates every order found a matching customer in this sample. In production, investigate unmatched keys rather than silently dropping them.

## 14. Export transformed data

### 14.1 Write CSV

```python
orders_with_customer.to_csv("orders_enriched.csv", index=False)

print("CSV export completed")
```

**Expected output**

```text
CSV export completed
```

`index=False` avoids exporting the DataFrame row index as an extra column.

### 14.2 Read back and verify

```python
exported_orders = pd.read_csv("orders_enriched.csv")

print("Rows:", len(exported_orders))
print("Columns:", len(exported_orders.columns))
print(exported_orders[["order_id", "customer_name"]].head(2))
```

**Expected output**

```text
Rows: 7
Columns: 10
   order_id customer_name
0      1001          Ravi
1      1002           Anu
```

## 15. Mini-project: Dataset Transformation Tool

### 15.1 Business requirement

The sales team needs an enriched order dataset and a city-level paid revenue report.

Workflow:

```text
Read source -> Inspect -> Check missing values -> Filter valid paid orders
            -> Join customer data -> Aggregate city revenue -> Export
```

### 15.2 Build the transformation dataset

```python
project_orders = orders.copy()

project_orders = project_orders[
    (project_orders["status"] == "PAID") &
    (project_orders["amount"].notna())
].copy()

print("Eligible paid orders:", len(project_orders))
```

**Expected output**

```text
Eligible paid orders: 4
```

### 15.3 Enrich with customer attributes

```python
project_enriched = project_orders.merge(
    customers,
    on="customer_id",
    how="left",
    validate="many_to_one"
)

print(project_enriched[["order_id", "customer_name", "city", "amount"]])
```

**Expected output**

Four rows for orders 1001, 1002, 1004 and 1007, with their customer names, cities and amounts.

### 15.4 Create the business summary

```python
project_city_report = project_enriched.groupby("city").agg(
    paid_revenue=("amount", "sum"),
    paid_orders=("order_id", "count")
).reset_index()

print(project_city_report)
```

**Expected output**

```text
        city  paid_revenue  paid_orders
0     Chennai       18000.0            1
1   Hyderabad       55000.0            1
2        Pune      110000.0            2
```

### 15.5 Export both outputs

```python
project_enriched.to_csv("project_enriched_orders.csv", index=False)
project_city_report.to_csv("project_city_revenue.csv", index=False)

print("Enriched rows:", len(pd.read_csv("project_enriched_orders.csv")))
print("City report rows:", len(pd.read_csv("project_city_revenue.csv")))
```

**Expected output**

```text
Enriched rows: 4
City report rows: 3
```

### 15.6 Explain the project

The transformation filters to paid orders with known amounts, joins customer attributes using a many-to-one validation, aggregates paid revenue by city and exports two datasets. The key engineering decisions are explicit filtering, join validation and output verification.

## 16. Common mistakes

| Mistake | Cause | Better approach |
|---|---|---|
| `KeyError` | Column name is wrong or absent | Inspect `df.columns` |
| Filter returns no rows | Condition or data values differ | Inspect unique values and dtypes |
| Boolean condition error | Used `and` instead of `&` | Use parentheses and element-wise operators |
| Unexpected row multiplication after merge | Duplicate keys on lookup side | Check key uniqueness and use `validate` |
| Missing values become zero | Used fillna without business meaning | Define a valid missing-data policy |
| Wrong revenue total | Included cancelled/pending orders | Confirm status and metric definition |
| Memory exhaustion | Dataset is too large for local Pandas | Filter in source, chunk input or use distributed processing |

## 17. Pandas performance and production guidance

- Select only columns required for the task.
- Filter early to reduce rows carried into later steps.
- Prefer vectorized operations over Python row loops for common calculations.
- Use appropriate numeric and datetime dtypes.
- Validate merge cardinality to catch accidental row multiplication.
- Avoid treating missing business values as zero without agreement.
- For large CSVs, consider `chunksize`; for larger distributed workloads, use an appropriate processing engine.
- Keep raw input separate from transformed output.
- Record row counts before and after important filters.

## 18. Day 14 revision

- A Series is one-dimensional; a DataFrame is two-dimensional.
- Inspect shape, dtypes, nulls and sample rows before transforming.
- Filter with explicit conditions and parentheses.
- Group and aggregate only after defining the business metric.
- Validate joins to prevent unexpected row multiplication.
- Missing-value handling must follow a business rule.
- Export and reload outputs to verify the result.
