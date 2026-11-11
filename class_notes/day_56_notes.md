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
