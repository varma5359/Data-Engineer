# Day 56: Spark Transformations and Actions

**Phase:** 6 — Cloud and Distributed Processing  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and junior Data Engineer candidates  
**Technology:** PySpark DataFrames  
**Topic boundary:** Class notes only. Project workbook and interview guide are separate files.

> Yesterday you created a Spark DataFrame. Today you will learn how to work with it. We will select columns, filter rows, create new columns, change data types, group records, calculate totals and sort results. Most importantly, you will understand the difference between transformations and actions, and why Spark uses lazy evaluation.

## Learning outcomes

- Explain transformations and actions.
- Understand lazy evaluation.
- Use select, filter and where.
- Use withColumn and lit.
- Rename and drop columns.
- Cast data types.
- Handle null values.
- Use distinct and dropDuplicates.
- Group data and calculate aggregates.
- Sort and limit results.
- Read the expected output of each operation.
- Avoid collecting large datasets to the driver.

---

# Part 1 — Prepare the Spark DataFrame

## 1. Start Spark

Use the same local PySpark environment from Day 55.

~~~python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("Day56Transformations")
    .master("local[*]")
    .getOrCreate()
)
~~~

## 2. Read the orders file

Use the orders.csv file created on Day 55.

~~~python
orders = spark.read.csv(
    "spark_day55/orders.csv",
    header=True,
    inferSchema=True
)

orders.show()
~~~

Expected rows:

| order_id | customer_id | city | amount |
|---:|---|---|---:|
| 101 | C001 | Hyderabad | 250 |
| 102 | C002 | Chennai | 499 |
| 103 | C001 | Hyderabad | 100 |
| 104 | C003 | Bengaluru | 750 |
| 105 | C004 | Chennai | 300 |

If your working directory differs, use the correct path.

## 3. Why do we transform data?

Raw source data may contain:
- unnecessary columns;
- inconsistent types;
- null values;
- duplicate records;
- business values that need calculation;
- fields that are not ready for reporting.

Transformations shape data into a useful form.

---

# Part 2 — Transformations and actions

## 4. What is a transformation?

A transformation describes a change to a DataFrame.

Examples:
- select columns;
- filter rows;
- add a column;
- group records;
- sort records.

A transformation generally returns a new DataFrame. It does not modify the original DataFrame in place.

## 5. What is an action?

An action asks Spark to execute work or return a result.

Examples:
- show();
- count();
- collect();
- write();

An action may trigger the execution plan created by earlier transformations.

## 6. Compare transformation and action

| Operation | Category | Purpose |
|---|---|---|
| select | Transformation | Choose columns |
| filter | Transformation | Keep matching rows |
| where | Transformation | Filter rows |
| withColumn | Transformation | Add or replace a column |
| drop | Transformation | Remove columns |
| groupBy | Transformation-like grouped plan | Define grouping before aggregation |
| orderBy | Transformation | Define sorting |
| show | Action | Display rows |
| count | Action | Count rows |
| collect | Action | Return all rows to driver |
| write | Action | Execute and save output |

The exact internal execution behavior is planned by Spark. Remember the beginner rule: transformations build the plan; actions trigger it.

## 7. Why does Spark use lazy evaluation?

Imagine you write:

~~~python
large_orders = orders.filter(orders.amount > 200)
~~~

Spark creates a new DataFrame describing the filter.

It does not necessarily scan all rows immediately.

When you run:

~~~python
large_orders.show()
~~~

Spark executes the required plan to produce the output.

Lazy evaluation allows Spark to optimize the sequence of operations before execution.

---

# Part 3 — Select columns

## 8. What does select() do?

select() chooses the columns you want to keep in the output DataFrame.

Example:

~~~python
selected_orders = orders.select("order_id", "amount")

selected_orders.show()
~~~

Expected output:

~~~text
+--------+------+
|order_id|amount|
+--------+------+
|     101|   250|
|     102|   499|
|     103|   100|
|     104|   750|
|     105|   300|
+--------+------+
~~~

Explanation:
- orders is the original DataFrame;
- select chooses only order_id and amount;
- selected_orders is a new DataFrame;
- show triggers execution and displays rows.

## 9. Select one column

~~~python
orders.select("city").show()
~~~

Expected output contains the city column for each row.

Repeated city values remain. select() does not automatically remove duplicates.

## 10. Select with a column expression

~~~python
from pyspark.sql.functions import col

orders.select(
    col("order_id"),
    col("customer_id")
).show()
~~~

col() creates a column expression that Spark can use in DataFrame operations.

Both string column names and col() expressions are common. Use a consistent style.

---

# Part 4 — Filter rows

## 11. What does filter() do?

filter() keeps rows that satisfy a condition.

Example: orders greater than 300.

~~~python
high_value_orders = orders.filter(orders.amount > 300)

high_value_orders.show()
~~~

Expected output:

~~~text
+--------+-----------+---------+------+
|order_id|customer_id|     city|amount|
+--------+-----------+---------+------+
|     102|       C002|  Chennai|   499|
|     104|       C003|Bengaluru|   750|
+--------+-----------+---------+------+
~~~

The condition amount > 300 is evaluated for each row.

## 12. Filter using col()

~~~python
high_value_orders = orders.filter(col("amount") > 300)

high_value_orders.show()
~~~

This produces the same result.

## 13. Filter using where()

where() is an alias for filter() in the DataFrame API.

~~~python
chennai_orders = orders.where(col("city") == "Chennai")

chennai_orders.show()
~~~

Expected rows:

| order_id | customer_id | city | amount |
|---:|---|---|---:|
| 102 | C002 | Chennai | 499 |
| 105 | C004 | Chennai | 300 |

## 14. Combine conditions

Use parentheses around each condition.

~~~python
hyderabad_high_orders = orders.filter(
    (col("city") == "Hyderabad") &
    (col("amount") > 150)
)

hyderabad_high_orders.show()
~~~

Expected result:

| order_id | customer_id | city | amount |
|---:|---|---|---:|
| 101 | C001 | Hyderabad | 250 |

In PySpark column expressions:
- & means AND;
- | means OR;
- ~ means NOT.

Do not use Python's and/or operators between Spark Column expressions.

## 15. OR condition

~~~python
selected_cities = orders.filter(
    (col("city") == "Chennai") |
    (col("city") == "Bengaluru")
)

selected_cities.show()
~~~

Expected order IDs: 102, 104 and 105.

## 16. NOT condition

~~~python
not_chennai = orders.filter(
    ~(col("city") == "Chennai")
)

not_chennai.show()
~~~

Expected order IDs: 101, 103 and 104.

## 17. Range condition

~~~python
medium_orders = orders.filter(
    col("amount").between(200, 500)
)

medium_orders.show()
~~~

Expected order IDs: 101, 102 and 105.

between() includes both boundary values.

## 18. Filter using isin()

~~~python
selected_cities = orders.filter(
    col("city").isin("Hyderabad", "Bengaluru")
)

selected_cities.show()
~~~

Expected order IDs: 101, 103 and 104.

isin() checks whether a value belongs to the supplied set of values.

---

# Part 5 — Add and modify columns

## 19. What does withColumn() do?

withColumn() creates a new column or replaces a column with the same name.

Example: add a fixed source label.

~~~python
from pyspark.sql.functions import lit

orders_with_source = orders.withColumn(
    "source_system",
    lit("order_app")
)

orders_with_source.show()
~~~

Expected output includes a new source_system column containing order_app for every row.

lit() creates a literal value that Spark can use as a column expression.

## 20. Calculate a new column

Suppose the business wants a 10% service fee for this training example.

~~~python
orders_with_fee = orders.withColumn(
    "service_fee",
    col("amount") * 0.10
)

orders_with_fee.select(
    "order_id",
    "amount",
    "service_fee"
).show()
~~~

Expected values:

| order_id | amount | service_fee |
|---:|---:|---:|
| 101 | 250 | 25.0 |
| 102 | 499 | 49.9 |
| 103 | 100 | 10.0 |
| 104 | 750 | 75.0 |
| 105 | 300 | 30.0 |

This is a teaching business rule, not a real tax or fee policy.

## 21. Create a business category

~~~python
from pyspark.sql.functions import when

orders_with_category = orders.withColumn(
    "order_category",
    when(col("amount") >= 500, "High")
    .when(col("amount") >= 250, "Medium")
    .otherwise("Low")
)

orders_with_category.select(
    "order_id",
    "amount",
    "order_category"
).show()
~~~

Expected result:

| order_id | amount | order_category |
|---:|---:|---|
| 101 | 250 | Medium |
| 102 | 499 | Medium |
| 103 | 100 | Low |
| 104 | 750 | High |
| 105 | 300 | Medium |

when() creates conditional expressions. otherwise() defines the result when no previous condition matches.

## 22. Replace an existing column

~~~python
orders_updated = orders.withColumn(
    "amount",
    col("amount") + 10
)

orders_updated.show()
~~~

The amount column is replaced in the new DataFrame. The original orders DataFrame is not modified.

Be careful when replacing business fields. Keep the original value if audit or reconciliation requires it.

## 23. Cast a column type

~~~python
from pyspark.sql.types import StringType

orders_text_id = orders.withColumn(
    "order_id",
    col("order_id").cast(StringType())
)

orders_text_id.printSchema()
~~~

Expected schema shows order_id as string.

Casting can fail or produce null values for incompatible input depending on the type, Spark version and ANSI behavior. Validate after casting.

---

# Part 6 — Rename and drop columns

## 24. Rename a column

~~~python
renamed_orders = orders.withColumnRenamed(
    "customer_id",
    "buyer_id"
)

renamed_orders.show()
~~~

The output column is buyer_id instead of customer_id.

## 25. Drop a column

~~~python
orders_without_city = orders.drop("city")

orders_without_city.show()
~~~

The new DataFrame no longer includes city.

Do not drop a field simply because it is not needed in today's report. Confirm that downstream consumers and audit requirements do not need it.

---

# Part 7 — Handle null values

## 26. What is a null value?

A null means a value is missing or unknown.

It is not the same as:
- zero;
- empty string;
- text "NULL";
- false.

Null handling affects filters, aggregations and joins.

## 27. Create a DataFrame with null values

~~~python
sample_data = [
    (1, "C001", 250.0),
    (2, "C002", None),
    (3, None, 100.0),
    (4, "C004", 300.0)
]

sample_columns = ["order_id", "customer_id", "amount"]

sample_orders = spark.createDataFrame(
    sample_data,
    sample_columns
)

sample_orders.show()
~~~

Expected output contains null values in customer_id and amount.

## 28. Find rows with null values

~~~python
sample_orders.filter(
    col("customer_id").isNull()
).show()
~~~

Expected result:

| order_id | customer_id | amount |
|---:|---|---:|
| 3 | null | 100.0 |

## 29. Find non-null rows

~~~python
sample_orders.filter(
    col("customer_id").isNotNull()
).show()
~~~

This returns rows 1, 2 and 4.

## 30. Fill missing values

~~~python
filled_orders = sample_orders.fillna(
    {"customer_id": "UNKNOWN", "amount": 0.0}
)

filled_orders.show()
~~~

Expected result:

| order_id | customer_id | amount |
|---:|---|---:|
| 1 | C001 | 250.0 |
| 2 | C002 | 0.0 |
| 3 | UNKNOWN | 100.0 |
| 4 | C004 | 300.0 |

Only use these replacement values if they match the business rule. Replacing an unknown amount with zero can create misleading sales totals.

## 31. Drop rows with nulls

~~~python
complete_orders = sample_orders.dropna()

complete_orders.show()
~~~

Expected output contains only row 1.

dropna() removes rows according to its null-handling options. Understand the default behavior and choose the required columns/threshold explicitly for production.

---

# Part 8 — Distinct and duplicate handling

## 32. What does distinct() do?

distinct() removes fully duplicate rows.

~~~python
duplicate_data = [
    (101, "C001", 250),
    (102, "C002", 499),
    (101, "C001", 250)
]

duplicate_orders = spark.createDataFrame(
    duplicate_data,
    ["order_id", "customer_id", "amount"]
)

unique_rows = duplicate_orders.distinct()

unique_rows.show()
~~~

Expected result contains two unique rows.

## 33. What does dropDuplicates() do?

dropDuplicates() removes duplicate rows based on all columns or selected columns.

~~~python
unique_order_ids = duplicate_orders.dropDuplicates(
    ["order_id"]
)

unique_order_ids.show()
~~~

This keeps one row for each order_id.

If duplicate rows contain conflicting values, do not assume the retained row is the correct business record. Define a rule such as latest update timestamp or trusted source priority.

---

# Part 9 — Group and aggregate

## 34. What is groupBy()?

groupBy() groups rows using one or more columns.

Example: calculate total sales by city.

~~~python
city_sales = orders.groupBy("city").sum("amount")

city_sales.show()
~~~

Expected values:

| city | sum(amount) |
|---|---:|
| Bengaluru | 750 |
| Chennai | 799 |
| Hyderabad | 350 |

The order of output rows is not guaranteed unless you sort them.

## 35. Use named aggregation

~~~python
from pyspark.sql.functions import sum, avg, count, min, max

city_summary = orders.groupBy("city").agg(
    sum("amount").alias("total_sales"),
    avg("amount").alias("average_order"),
    count("*").alias("order_count"),
    min("amount").alias("minimum_order"),
    max("amount").alias("maximum_order")
)

city_summary.show()
~~~

Expected values:

| city | total_sales | average_order | order_count | minimum_order | maximum_order |
|---|---:|---:|---:|---:|---:|
| Bengaluru | 750 | 750.0 | 1 | 750 | 750 |
| Chennai | 799 | 399.5 | 2 | 300 | 499 |
| Hyderabad | 350 | 175.0 | 2 | 100 | 250 |

## 36. Understand common aggregation functions

| Function | Meaning |
|---|---|
| sum() | Adds numeric values |
| avg() | Calculates average |
| count() | Counts rows or non-null values depending on expression |
| min() | Finds smallest value |
| max() | Finds largest value |

Be careful with count:
- count("*") counts rows;
- count("customer_id") counts non-null customer_id values.

## 37. Count by city

~~~python
city_counts = orders.groupBy("city").count()

city_counts.show()
~~~

Expected counts:
- Bengaluru: 1
- Chennai: 2
- Hyderabad: 2

## 38. Group by multiple columns

~~~python
customer_city_sales = orders.groupBy(
    "customer_id",
    "city"
).sum("amount")

customer_city_sales.show()
~~~

The grouping key is the combination of customer_id and city.

Use only the grouping columns required by the business question. More grouping columns create more detailed groups.

---

# Part 10 — Sort and limit

## 39. What does orderBy() do?

orderBy() sorts rows by one or more columns.

~~~python
orders.orderBy("amount").show()
~~~

Expected order IDs: 103, 101, 105, 102, 104.

## 40. Sort descending

~~~python
from pyspark.sql.functions import desc

orders.orderBy(
    desc("amount")
).show()
~~~

Expected order IDs: 104, 102, 105, 101, 103.

## 41. Sort by multiple columns

~~~python
orders.orderBy(
    col("city").asc(),
    col("amount").desc()
).show()
~~~

Rows are first grouped alphabetically by city, then amount is sorted descending inside each city.

## 42. Limit rows

~~~python
orders.orderBy(
    desc("amount")
).limit(3).show()
~~~

Expected top three order IDs: 104, 102 and 105.

limit() restricts the output row count. If you need deterministic top-N results, apply a deterministic ordering before limit() and consider tie-breaking columns.

---

# Part 11 — Combine operations into a business transformation

## 43. Business requirement

TechStore wants a city-level sales summary for orders with amount greater than or equal to 200.

The output must include:
- city;
- number of qualifying orders;
- total sales;
- average order value.

## 44. Build the transformation step by step

First filter:

~~~python
valid_orders = orders.filter(
    col("amount") >= 200
)
~~~

Then group:

~~~python
city_sales = valid_orders.groupBy("city").agg(
    count("*").alias("order_count"),
    sum("amount").alias("total_sales"),
    avg("amount").alias("average_order_value")
)
~~~

Then sort:

~~~python
final_city_sales = city_sales.orderBy(
    desc("total_sales")
)

final_city_sales.show()
~~~

Expected result:

| city | order_count | total_sales | average_order_value |
|---|---:|---:|---:|
| Chennai | 2 | 799 | 399.5 |
| Bengaluru | 1 | 750 | 750.0 |
| Hyderabad | 1 | 250 | 250.0 |

Why is Hyderabad total 250 rather than 350? The order with amount 100 was removed by the filter.

This is a business rule. If the rule changes, the output changes.

---

# Part 12 — Write output

## 45. Write CSV output

~~~python
final_city_sales.write.mode("overwrite").option(
    "header",
    True
).csv("spark_day56/output/city_sales")
~~~

Spark writes output as a directory containing one or more part files and metadata files, not necessarily one single CSV file.

The number of part files depends on the DataFrame's partitions.

## 46. Write Parquet output

~~~python
final_city_sales.write.mode("overwrite").parquet(
    "spark_day56/output/city_sales_parquet"
)
~~~

Parquet is commonly used for analytical output.

## 47. Understand write modes

| Mode | Meaning |
|---|---|
| append | Add output to existing destination where supported |
| overwrite | Replace existing output |
| error or errorifexists | Fail if destination exists |
| ignore | Do nothing if destination exists |

Be careful with overwrite. Confirm the exact destination before writing.

---

# Part 13 — Common mistakes

| Mistake | Why it is wrong | Better practice |
|---|---|---|
| Expect transformation to execute immediately | Spark is lazy | Trigger an action |
| Use Python and/or with Column expressions | Not valid Spark boolean composition | Use &, | and ~ with parentheses |
| Assume select removes duplicates | It only selects columns | Use distinct when appropriate |
| Replace null with zero automatically | Can change business meaning | Define a business rule |
| Drop duplicates on key without tie-breaker | Conflicting records may remain arbitrarily | Define which record wins |
| Forget to alias aggregate columns | Output names become awkward | Use alias |
| Assume groupBy output order | Distributed output is not guaranteed sorted | Apply orderBy |
| Use limit without ordering for top-N | Results may be nondeterministic | Sort with tie-breaker |
| Overwrite production output casually | Existing data can be replaced | Use reviewed destination and safe publish process |
| Collect large results | Driver memory risk | Keep work distributed |

---

# Part 14 — Practice questions

## Exercise 1

What is a transformation?

**Answer:** An operation that returns a DataFrame describing a change, such as selecting or filtering rows.

## Exercise 2

What is an action?

**Answer:** An operation that triggers execution or returns/writes a result, such as show, count or write.

## Exercise 3

Why is filter lazy?

**Answer:** It builds a plan. Spark executes the required work when an action needs the result.

## Exercise 4

What is the difference between select and filter?

**Answer:** select chooses columns; filter chooses rows.

## Exercise 5

Why use withColumn?

**Answer:** To add or replace a column using a Spark expression.

## Exercise 6

Why can fillna(0) be dangerous for amount?

**Answer:** A missing amount is unknown, not necessarily zero. Replacing it may understate or distort business totals.

## Exercise 7

What is the difference between distinct and dropDuplicates with selected columns?

**Answer:** distinct removes fully duplicate rows. dropDuplicates can define duplicate identity using selected columns, but a tie-breaking rule may be needed for conflicting records.

## Exercise 8

Why sort before limit for top-N?

**Answer:** Without ordering, Spark is not required to return the highest values.

---

# Part 15 — Revision sheet

| Term | Remember |
|---|---|
| Transformation | Describes a DataFrame change |
| Action | Triggers execution/result |
| Lazy evaluation | Spark delays work until required |
| select | Choose columns |
| filter/where | Choose rows |
| withColumn | Add or replace a column |
| lit | Create a literal expression |
| when/otherwise | Conditional column logic |
| cast | Convert data type |
| drop | Remove a column |
| isNull/isNotNull | Test missing values |
| fillna | Replace nulls |
| dropna | Remove rows with nulls |
| distinct | Remove fully duplicate rows |
| dropDuplicates | Remove duplicates using selected identity columns |
| groupBy | Group records |
| agg | Calculate grouped aggregates |
| orderBy | Sort rows |
| limit | Restrict row count |
| write | Save DataFrame output |

## Final self-check

- [ ] Explain transformations, actions and lazy evaluation.
- [ ] Use select, filter and where.
- [ ] Combine conditions correctly.
- [ ] Add and replace columns with withColumn.
- [ ] Rename, cast and drop columns.
- [ ] Handle nulls based on business rules.
- [ ] Explain distinct and dropDuplicates.
- [ ] Group and aggregate data.
- [ ] Sort and limit deterministically.
- [ ] Write CSV and Parquet outputs safely.
- [ ] Explain why collect and overwrite require care.

**Next lesson — Day 57: Spark Joins and Partitioning.** You will join orders, customers and products, understand join types, inspect unmatched records, and learn repartition and coalesce.


---

# Part 16 — Prepare a complete beginner notebook

The previous sections explained individual operations. This lab combines them in one notebook and checks each result.

## 48. Create the practice folder

In Windows CMD:

~~~bat
mkdir spark_day56
cd spark_day56
mkdir data
mkdir output
~~~

Create data/orders.csv:

~~~csv
order_id,customer_id,city,amount,status
201,C001,Hyderabad,250.00,PAID
202,C002,Chennai,499.50,PAID
203,C001,Hyderabad,100.00,PENDING
204,C003,Bengaluru,750.25,PAID
205,C004,Chennai,300.00,PAID
206,C005,Hyderabad,0.00,CANCELLED
207,C006,Bengaluru,125.00,PAID
208,C002,Chennai,499.50,PAID
~~~

Save and close the file. Check:

~~~bat
dir data
~~~

You should see orders.csv. If the name ends with .csv.txt, correct the file extension.

## 49. Cell 1 — Create or reuse Spark

~~~python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("Day56CompleteLab")
    .master("local[*]")
    .getOrCreate()
)

print("Spark ready:", spark.version)
~~~

Expected output: Spark ready followed by your installed Spark version.

## 50. Cell 2 — Import the expressions used in the lesson

~~~python
from pyspark.sql.functions import (
    col,
    lit,
    when,
    sum,
    avg,
    count,
    min,
    max,
    desc,
    asc
)
~~~

Expected output: no printed output if the imports succeed.

These are Spark column-expression functions. They are not ordinary Python calculations over a local list.

## 51. Cell 3 — Read the source file

~~~python
orders = spark.read.csv(
    "data/orders.csv",
    header=True,
    inferSchema=True
)

orders.show(truncate=False)
~~~

Expected rows:

| order_id | customer_id | city | amount | status |
|---:|---|---|---:|---|
| 201 | C001 | Hyderabad | 250.00 | PAID |
| 202 | C002 | Chennai | 499.50 | PAID |
| 203 | C001 | Hyderabad | 100.00 | PENDING |
| 204 | C003 | Bengaluru | 750.25 | PAID |
| 205 | C004 | Chennai | 300.00 | PAID |
| 206 | C005 | Hyderabad | 0.00 | CANCELLED |
| 207 | C006 | Bengaluru | 125.00 | PAID |
| 208 | C002 | Chennai | 499.50 | PAID |

## 52. Cell 4 — Inspect the source

~~~python
print("Rows:", orders.count())
print("Columns:", len(orders.columns))
orders.printSchema()
~~~

Expected row count: 8.

The schema types depend on inference. Inspect the actual schema rather than assuming a particular numeric type.

## 53. Cell 5 — Select only required columns

~~~python
order_view = orders.select(
    "order_id",
    "city",
    "amount",
    "status"
)

order_view.show()
~~~

Expected result has four columns and eight rows.

Why select early? Removing unused columns can reduce the amount of data carried through later operations. The optimizer may also perform column pruning.

## 54. Cell 6 — Filter paid orders

~~~python
paid_orders = orders.filter(
    col("status") == "PAID"
)

paid_orders.show()
~~~

Expected order IDs: 201, 202, 204, 205, 207 and 208.

Expected count:

~~~python
print("Paid order rows:", paid_orders.count())
~~~

Expected output:

~~~text
Paid order rows: 6
~~~

## 55. Cell 7 — Filter using two conditions

Find paid orders with amount at least 200:

~~~python
large_paid_orders = orders.filter(
    (col("status") == "PAID") &
    (col("amount") >= 200)
)

large_paid_orders.select(
    "order_id",
    "city",
    "amount"
).show()
~~~

Expected order IDs: 201, 202, 204, 205 and 208.

Order 207 is excluded because its amount is 125.

## 56. Cell 8 — Add a source label

~~~python
orders_labeled = orders.withColumn(
    "source_system",
    lit("techstore_orders")
)

orders_labeled.select(
    "order_id",
    "source_system"
).show()
~~~

Every row receives the same source label.

## 57. Cell 9 — Add a conditional order band

~~~python
orders_banded = orders.withColumn(
    "order_band",
    when(col("amount") >= 500, "HIGH")
    .when(col("amount") >= 200, "MEDIUM")
    .otherwise("LOW")
)

orders_banded.select(
    "order_id",
    "amount",
    "order_band"
).orderBy("order_id").show()
~~~

Expected classifications:

| order_id | amount | order_band |
|---:|---:|---|
| 201 | 250.00 | MEDIUM |
| 202 | 499.50 | MEDIUM |
| 203 | 100.00 | LOW |
| 204 | 750.25 | HIGH |
| 205 | 300.00 | MEDIUM |
| 206 | 0.00 | LOW |
| 207 | 125.00 | LOW |
| 208 | 499.50 | MEDIUM |

The category is based on amount only. It does not mean the order is paid.

## 58. Cell 10 — Rename a column

~~~python
orders_named = orders.withColumnRenamed(
    "customer_id",
    "buyer_id"
)

orders_named.select(
    "order_id",
    "buyer_id"
).show(3)
~~~

The output uses buyer_id instead of customer_id. The original DataFrame remains unchanged.

## 59. Cell 11 — Cast amount to a decimal type

~~~python
from pyspark.sql.types import DecimalType

orders_decimal = orders.withColumn(
    "amount",
    col("amount").cast(DecimalType(12, 2))
)

orders_decimal.printSchema()
~~~

The amount field should now be decimal(12,2).

Always validate conversion results. A cast is not proof that every source value was valid.

## 60. Cell 12 — Count nulls

~~~python
null_summary = orders.select(
    [
        count("*").alias("total_rows"),
        sum(col("customer_id").isNull().cast("int")).alias("null_customer_ids"),
        sum(col("amount").isNull().cast("int")).alias("null_amounts"),
        sum(col("status").isNull().cast("int")).alias("null_statuses")
    ]
)

null_summary.show()
~~~

Expected result for the supplied file:

| total_rows | null_customer_ids | null_amounts | null_statuses |
|---:|---:|---:|---:|
| 8 | 0 | 0 | 0 |

This checks missing values in three important fields. Add checks for every required field in a real pipeline.

## 61. Cell 13 — Find duplicate order IDs

~~~python
duplicate_ids = (
    orders
    .groupBy("order_id")
    .count()
    .filter(col("count") > 1)
)

duplicate_ids.show()
~~~

Expected output: no rows.

This example assumes one row per order. If the source contains one row per order line, use the actual line-level business key.

## 62. Cell 14 — Group by city

~~~python
city_totals = orders.groupBy("city").agg(
    sum("amount").alias("total_amount"),
    count("*").alias("row_count"),
    avg("amount").alias("average_amount")
)

city_totals.orderBy("city").show()
~~~

Expected results:

| city | total_amount | row_count | average_amount |
|---|---:|---:|---:|
| Bengaluru | 875.25 | 2 | 437.625 |
| Chennai | 1299.00 | 3 | 433.00 |
| Hyderabad | 350.00 | 3 | 116.6667 |

These totals include PAID, PENDING and CANCELLED records because this cell groups all source rows. That is intentional: the next cell applies a business eligibility rule.

## 63. Cell 15 — Calculate paid sales only

~~~python
paid_city_totals = (
    orders
    .filter(col("status") == "PAID")
    .groupBy("city")
    .agg(
        sum("amount").alias("paid_sales"),
        count("*").alias("paid_order_count")
    )
    .orderBy("city")
)

paid_city_totals.show()
~~~

Expected results:

| city | paid_sales | paid_order_count |
|---|---:|---:|
| Bengaluru | 875.25 | 2 |
| Chennai | 1299.00 | 3 |
| Hyderabad | 250.00 | 1 |

Hyderabad has only one paid order. Its pending order and cancelled order are excluded.

## 64. Cell 16 — Sort the largest paid-sales city first

~~~python
paid_city_totals.orderBy(
    desc("paid_sales"),
    asc("city")
).show()
~~~

Expected order:
1. Chennai — 1298.00
2. Bengaluru — 875.25
3. Hyderabad — 250.00

The second sort column provides a deterministic tie-breaker if two cities have the same paid_sales.

## 65. Cell 17 — Select the top two cities

~~~python
top_two_cities = (
    paid_city_totals
    .orderBy(desc("paid_sales"), asc("city"))
    .limit(2)
)

top_two_cities.show()
~~~

Expected cities: Chennai and Bengaluru.

Always order before limit when the requirement is top-N.

## 66. Cell 18 — Validate total paid sales

~~~python
source_paid_total = (
    orders
    .filter(col("status") == "PAID")
    .agg(sum("amount").alias("source_paid_total"))
)

curated_paid_total = paid_city_totals.agg(
    sum("paid_sales").alias("curated_paid_total")
)

source_paid_total.show()
curated_paid_total.show()
~~~

Expected source and curated totals are both 2424.25.

Arithmetic:
- 250.00 + 499.50 + 750.25 + 300.00 + 125.00 + 499.50 = 2424.25.

Use the arithmetic from the actual input rows to verify the Spark output. If the displayed aggregate differs, investigate the source values and filter. Do not force the expected number into the result.

## 67. Cell 19 — Inspect the execution plan

~~~python
paid_city_totals.explain()
~~~

Spark displays a logical/physical plan. The exact plan varies by Spark version and optimizer decisions.

Look for:
- file scan;
- filter;
- aggregate;
- exchange or shuffle where required;
- sort.

This is an inspection exercise, not a requirement to memorize every plan node.

## 68. Cell 20 — Write Parquet output

~~~python
paid_city_totals.write.mode("overwrite").parquet(
    "output/paid_city_totals"
)
~~~

Expected behavior: Spark creates an output directory containing Parquet part files and metadata.

Do not assume the output is one file. Spark writes data according to its partitions.

## 69. Cell 21 — Read output back

~~~python
saved_paid_city_totals = spark.read.parquet(
    "output/paid_city_totals"
)

saved_paid_city_totals.orderBy("city").show()
~~~

The rows and values should match the paid_city_totals DataFrame.

Read-after-write confirms that the saved output can be read. Also compare row counts and totals.

## 70. Cell 22 — Compare before and after

~~~python
print("Before write rows:", paid_city_totals.count())
print("After read rows:", saved_paid_city_totals.count())
~~~

Expected row count: 3 before write and 3 after read.

## 71. Cell 23 — Stop Spark at the end

~~~python
spark.stop()
print("Spark stopped")
~~~

Expected output:

~~~text
Spark stopped
~~~

Stop the session only after all notebook cells that need Spark have completed.

---

# Part 17 — Understand null behavior in filters

## 72. Why null comparisons surprise beginners

A comparison with null does not behave like a normal comparison with a number or string.

For example, a condition such as amount > 100 is not true when amount is null. The result is unknown, and filter keeps only rows where the condition is true.

## 73. Explicitly include or exclude nulls

To keep only known amounts greater than 100:

~~~python
known_large_amounts = orders.filter(
    col("amount").isNotNull() &
    (col("amount") > 100)
)
~~~

To inspect rows with unknown amounts:

~~~python
unknown_amounts = orders.filter(
    col("amount").isNull()
)

unknown_amounts.show()
~~~

Do not replace nulls just to make a filter return more rows. Define the correct business treatment.

---

# Part 18 — Understand distinct versus business deduplication

## 74. Fully duplicate rows

If every column is identical, distinct() can remove repeated copies.

~~~python
all_columns_unique = orders.distinct()
~~~

This checks the full row, not only order_id.

## 75. Duplicate business keys with different values

Imagine the source sends:

| order_id | amount | updated_at |
|---:|---:|---|
| 9001 | 100.00 | 09:00 |
| 9001 | 120.00 | 10:00 |

These are not identical rows. distinct() will not decide which one is the correct version.

The business/source contract must define the winning record. It may be the latest trusted update, a source sequence number or an approved source priority.

Do not use dropDuplicates(order_id) as a substitute for a documented tie-breaker.

---

# Part 19 — Common Spark transformation mistakes

| Mistake | Why it is a problem | Better practice |
|---|---|---|
| Using and/or with Spark columns | Python boolean operators do not compose Spark expressions | Use &, | and ~ with parentheses |
| Filtering before understanding nulls | Unknown values may be excluded | Add explicit null checks |
| Aggregating before applying business filters | Pending/cancelled rows may be included | Confirm eligibility rule |
| Calling distinct to solve conflicting keys | It removes identical rows only | Define deterministic deduplication |
| Sorting only by a non-unique value before limit | Ties may return in varying order | Add stable tie-breaker |
| Replacing missing amount with zero | Changes unknown into a real numeric value | Use approved missing-value policy |
| Writing with overwrite to an important path | Can replace prior output | Use controlled run path and publish |
| Assuming one output file | Spark writes partitions | Inspect output directory |
| Trusting a successful write alone | Data may still be wrong | Read back and reconcile |

---

# Part 20 — Additional practice exercises with worked answers

## Exercise 9 — Select or filter

You need only order_id and amount, but you need every row. Which operation?

**Worked answer:** select(), because the requirement is to choose columns, not remove rows.

## Exercise 10 — Filter and null

A row has amount = null. Will filter(col("amount") > 100) keep it?

**Worked answer:** No. The comparison is unknown, not true, so the filter does not keep that row.

## Exercise 11 — Business status

A sales report should include PAID orders only. Why is grouping all rows first incorrect?

**Worked answer:** Pending and cancelled rows may be included in the total. Apply the approved status filter before the aggregation.

## Exercise 12 — Conditional column

What does otherwise() do in a when() expression?

**Worked answer:** It defines the value returned when none of the preceding conditions match.

## Exercise 13 — Duplicate keys

Two rows share order_id but have different amounts and update timestamps. Is distinct() enough?

**Worked answer:** No. The rows are different. Define a deterministic rule for which source version is valid.

## Exercise 14 — Aggregation output

Why might groupBy().sum() show an awkward column name such as sum(amount)?

**Worked answer:** The aggregate expression has a generated name. Use alias() to give it a clear business name.

## Exercise 15 — Top-N

Why should a DataFrame be sorted before limit(10) when selecting the ten highest sales?

**Worked answer:** limit alone does not define which ten rows are highest. Sort by sales descending and add a stable tie-breaker.

## Exercise 16 — Overwrite

A job writes yesterday's output to a shared curated path using overwrite. What should be checked first?

**Worked answer:** Confirm the exact destination, downstream consumers, partition replacement semantics and recovery plan. Avoid replacing unrelated dates or incomplete output.

## Exercise 17 — Output validation

The Parquet write succeeded. What checks should follow?

**Worked answer:** Read the output back, compare row count, keys, date range, nulls and business totals against the expected result.

---

# Part 21 — Day 56 practical completion checklist

### DataFrame transformations
- [ ] Use select and filter.
- [ ] Combine AND, OR and NOT expressions with parentheses.
- [ ] Use between and isin.
- [ ] Add columns with lit and withColumn.
- [ ] Create conditional values with when and otherwise.
- [ ] Rename, cast and drop columns.
- [ ] Handle nulls with explicit business rules.

### Deduplication and aggregation
- [ ] Distinguish distinct from business-key deduplication.
- [ ] Group by one and multiple columns.
- [ ] Calculate sum, average, count, minimum and maximum.
- [ ] Filter before aggregation when required by the business definition.
- [ ] Sort with deterministic tie-breakers.
- [ ] Use limit for a deliberate top-N query.

### Execution and output
- [ ] Explain transformations, actions and lazy evaluation.
- [ ] Inspect an execution plan.
- [ ] Write CSV or Parquet output.
- [ ] Read output back and compare it.
- [ ] Explain why overwrite and collect require care.

### Engineering judgment
- [ ] Explain why missing amount is not automatically zero.
- [ ] Explain why successful execution does not prove business correctness.
- [ ] Validate row counts, keys and totals.
- [ ] Keep large datasets distributed.

## Final self-assessment

Without looking at the notes, explain:

1. What is the difference between select and filter?
2. Why does Spark delay many transformations?
3. How do you combine multiple conditions in PySpark?
4. Why is withColumn useful?
5. What is the risk of replacing null amounts with zero?
6. How is distinct different from business-key deduplication?
7. Why should business filters be applied before aggregation?
8. Why use aliases for aggregate output columns?
9. Why sort before limit for top-N?
10. How do you validate a Parquet output after writing?

If you can complete the notebook and explain each output, you are ready for Day 57.

**Next lesson — Day 57: Spark Joins and Partitioning.** You will join orders, customers and products, understand join types, inspect unmatched records, and learn repartition and coalesce.
