# Day 57: Spark Joins and Partitioning

**Phase:** 6 — Cloud and Distributed Processing  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and junior Data Engineer candidates  
**Technology:** PySpark DataFrames  
**Topic boundary:** Class notes only. Project workbook and interview guide are separate files.

> A sales dataset rarely contains every business detail in one file. Orders may contain customer IDs, while customer details live in another dataset. Today you will join orders, customers and products, understand join types, detect unmatched records and learn how Spark partitions data using repartition, coalesce and partitioned file output.

## Learning outcomes

- Explain why data engineers join datasets.
- Understand join keys and matching rows.
- Use inner, left, right and full outer joins.
- Understand left semi and left anti joins.
- Handle duplicate keys and null join keys.
- Avoid ambiguous column names.
- Understand repartition and coalesce.
- Explain shuffle during repartition.
- Write partitioned output using partitionBy.
- Validate join results and partition output.

---

# Part 1 — Why do we join datasets?

## 1. Business scenario

TechStore stores order records separately from customer details.

Orders:

| order_id | customer_id | amount |
|---:|---|---:|
| 101 | C001 | 250 |
| 102 | C002 | 499 |
| 103 | C001 | 100 |
| 104 | C003 | 750 |
| 105 | C005 | 300 |

Customers:

| customer_id | customer_name | city |
|---|---|---|
| C001 | Ravi | Hyderabad |
| C002 | Meena | Chennai |
| C003 | Arjun | Bengaluru |
| C004 | Sana | Pune |

The order table does not contain customer_name or city.

The customer table does not contain order amount.

To create a report with order and customer information, we join the datasets using customer_id.

## 2. What is a join?

A join combines rows from two DataFrames based on a matching condition.

The matching column is called a join key.

In this example:
- left DataFrame: orders;
- right DataFrame: customers;
- join key: customer_id.

A join can return:
- only matching rows;
- all rows from the left side;
- all rows from the right side;
- all rows from both sides.

The join type determines the result.

## 3. Why does the join key matter?

A join key should represent the intended relationship.

Examples:
- orders.customer_id = customers.customer_id;
- orders.product_id = products.product_id;
- payments.order_id = orders.order_id.

Joining on the wrong column can produce incorrect results or a huge row count.

Never choose a join key merely because two columns have similar names.

---

# Part 2 — Prepare practice DataFrames

## 4. Start Spark

~~~python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("Day57JoinsAndPartitions")
    .master("local[*]")
    .getOrCreate()
)
~~~

## 5. Create orders

~~~python
orders_data = [
    (101, "C001", 250),
    (102, "C002", 499),
    (103, "C001", 100),
    (104, "C003", 750),
    (105, "C005", 300)
]

orders = spark.createDataFrame(
    orders_data,
    ["order_id", "customer_id", "amount"]
)

orders.show()
~~~

## 6. Create customers

~~~python
customers_data = [
    ("C001", "Ravi", "Hyderabad"),
    ("C002", "Meena", "Chennai"),
    ("C003", "Arjun", "Bengaluru"),
    ("C004", "Sana", "Pune")
]

customers = spark.createDataFrame(
    customers_data,
    ["customer_id", "customer_name", "city"]
)

customers.show()
~~~

Expected customer rows: C001, C002, C003 and C004.

## 7. Inspect keys before joining

~~~python
orders.select("customer_id").distinct().show()
customers.select("customer_id").distinct().show()
~~~

Order customer IDs: C001, C002, C003 and C005.

Customer IDs: C001, C002, C003 and C004.

Therefore:
- C001, C002 and C003 match;
- C005 exists in orders but not customers;
- C004 exists in customers but has no order in this sample.

---

# Part 3 — Inner join

## 8. What is an inner join?

An inner join returns only rows with matching keys on both sides.

~~~python
inner_result = orders.join(
    customers,
    on="customer_id",
    how="inner"
)

inner_result.show()
~~~

Expected result:

| order_id | customer_id | amount | customer_name | city |
|---:|---|---:|---|---|
| 101 | C001 | 250 | Ravi | Hyderabad |
| 103 | C001 | 100 | Ravi | Hyderabad |
| 102 | C002 | 499 | Meena | Chennai |
| 104 | C003 | 750 | Arjun | Bengaluru |

Spark may display rows in a different order unless you sort them.

C005 is excluded because there is no matching customer row.

C004 is excluded because there is no matching order row.

## 9. When should you use inner join?

Use inner join when the business result should contain only records with valid matches on both sides.

Example:
A report that includes only orders linked to a known customer.

Do not use inner join if unmatched source records must be retained for investigation.

---

# Part 4 — Left join

## 10. What is a left join?

A left join keeps every row from the left DataFrame.

If the right side has no matching key, right-side columns become null.

~~~python
left_result = orders.join(
    customers,
    on="customer_id",
    how="left"
)

left_result.orderBy("order_id").show()
~~~

Expected result:

| order_id | customer_id | amount | customer_name | city |
|---:|---|---:|---|---|
| 101 | C001 | 250 | Ravi | Hyderabad |
| 102 | C002 | 499 | Meena | Chennai |
| 103 | C001 | 100 | Ravi | Hyderabad |
| 104 | C003 | 750 | Arjun | Bengaluru |
| 105 | C005 | 300 | null | null |

C005 is retained because it exists on the left side.

## 11. When should you use left join?

Use it when all rows from the main dataset must remain.

Examples:
- retain every order even if customer details are missing;
- retain every event even if product metadata is not available;
- identify missing dimension records.

---

# Part 5 — Right join

## 12. What is a right join?

A right join keeps every row from the right DataFrame.

~~~python
right_result = orders.join(
    customers,
    on="customer_id",
    how="right"
)

right_result.orderBy("customer_id").show()
~~~

Expected customer keys include C001, C002, C003 and C004.

C004 remains even though it has no matching order. Its order columns are null.

Right joins are valid, but many teams prefer to reorder the inputs and use a left join for readability.

---

# Part 6 — Full outer join

## 13. What is a full outer join?

A full outer join keeps matching and unmatched rows from both sides.

~~~python
full_result = orders.join(
    customers,
    on="customer_id",
    how="full"
)

full_result.orderBy("customer_id", "order_id").show()
~~~

Expected key coverage:
- C001, C002 and C003 have matches;
- C004 is a customer without an order;
- C005 is an order without a customer.

The unmatched side contains null values.

## 14. When is full outer join useful?

It can help with:
- reconciliation;
- source comparison;
- identifying records missing from one system;
- comparing two versions of a dataset.

It is not automatically the right join for a reporting query. Choose based on business requirements.

---

# Part 7 — Left semi and left anti joins

## 15. What is a left semi join?

A left semi join returns rows from the left DataFrame that have a match on the right.

It returns left-side columns only.

~~~python
matched_orders = orders.join(
    customers,
    on="customer_id",
    how="left_semi"
)

matched_orders.orderBy("order_id").show()
~~~

Expected order IDs: 101, 102, 103 and 104.

## 16. What is a left anti join?

A left anti join returns rows from the left DataFrame that do not have a match on the right.

~~~python
unmatched_orders = orders.join(
    customers,
    on="customer_id",
    how="left_anti"
)

unmatched_orders.show()
~~~

Expected result:

| order_id | customer_id | amount |
|---:|---|---:|
| 105 | C005 | 300 |

This is useful for finding missing dimension records or failed reference matches.

## 17. Compare join types

| Join type | Rows retained |
|---|---|
| inner | Matching rows only |
| left | All left rows plus matching right values |
| right | All right rows plus matching left values |
| full | All rows from both sides |
| left_semi | Left rows that have a match |
| left_anti | Left rows that do not have a match |

---

# Part 8 — Join on different column names

## 18. Join using an explicit condition

Sometimes the key columns have different names.

Example:
- orders.customer_id;
- customer_master.id.

Use a condition:

~~~python
joined_different_names = orders.join(
    customer_master,
    orders.customer_id == customer_master.id,
    "inner"
)
~~~

This example assumes customer_master exists and has an id column.

When the names differ, both key columns may appear in the output. Select the columns you need and rename them clearly.

## 19. Avoid ambiguous columns

If both DataFrames contain a column called status, a later reference to status may be ambiguous.

Use aliases:

~~~python
o = orders.alias("o")
c = customers.alias("c")

joined = o.join(
    c,
    col("o.customer_id") == col("c.customer_id"),
    "left"
)
~~~

Then select explicit columns:

~~~python
joined.select(
    col("o.order_id"),
    col("o.customer_id"),
    col("o.amount"),
    col("c.customer_name"),
    col("c.city")
).show()
~~~

Aliases make the source of each column clear.

---

# Part 9 — Duplicate keys and join multiplication

## 20. Why duplicate keys matter

Suppose the customer table contains two rows for C001.

An order for C001 may match both rows.

This can multiply output rows.

Example:
- one order row for C001;
- two customer rows for C001;
- join output contains two rows for that order.

This may inflate sales totals.

## 21. Demonstrate duplicate keys

~~~python
customers_duplicate_data = [
    ("C001", "Ravi", "Hyderabad"),
    ("C001", "Ravi Kumar", "Hyderabad"),
    ("C002", "Meena", "Chennai")
]

customers_duplicate = spark.createDataFrame(
    customers_duplicate_data,
    ["customer_id", "customer_name", "city"]
)

duplicate_join = orders.join(
    customers_duplicate,
    on="customer_id",
    how="inner"
)

duplicate_join.filter(
    col("customer_id") == "C001"
).show()
~~~

The two orders for C001 each match two customer rows. Four output rows are produced for C001.

## 22. Validate dimension key uniqueness

~~~python
customer_key_counts = customers_duplicate.groupBy(
    "customer_id"
).count()

customer_key_counts.filter(
    col("count") > 1
).show()
~~~

Expected result identifies C001 with count 2.

Before joining a fact dataset to a dimension, understand whether the dimension key is expected to be unique at the relevant point in time.

## 23. Do not remove duplicates blindly

If C001 has two customer records, determine why:
- accidental duplicate;
- source system conflict;
- historical customer versions;
- multiple addresses;
- slowly changing dimension rows.

The correct handling depends on the model and effective dates.

---

# Part 10 — Null join keys

## 24. Do null keys match?

In standard equality comparisons, null does not equal null.

A join using ordinary equality will not match null keys.

If the business requires special null matching behavior, use an explicitly chosen null-safe comparison supported by the Spark API and validate its consequences.

Do not convert every null key to the same placeholder without checking whether that creates false matches.

## 25. Find null join keys before joining

~~~python
orders.filter(
    col("customer_id").isNull()
).show()
~~~

Missing join keys should be measured and handled according to data quality rules.

---

# Part 11 — Understand Spark partitions

## 26. What is a Spark partition?

A Spark partition is a logical chunk of a DataFrame's distributed data.

Spark tasks process partitions.

The number and size of partitions affect:
- parallelism;
- task scheduling;
- memory use;
- shuffle;
- output file count.

## 27. Inspect partition count

~~~python
print("Partitions:", orders.rdd.getNumPartitions())
~~~

The exact number depends on how the DataFrame was created and the Spark environment.

Do not copy another student's partition count as your own result.

## 28. What is repartition()?

repartition() changes the number or distribution of partitions.

It generally causes a shuffle so Spark can redistribute records.

~~~python
orders_repartitioned = orders.repartition(4)

print("Partitions after repartition:", orders_repartitioned.rdd.getNumPartitions())
~~~

Expected partition count:

~~~text
Partitions after repartition: 4
~~~

The exact count is four because the requested partition count is explicit. The amount of data in each partition may not be perfectly equal.

## 29. Repartition by a column

~~~python
orders_by_city = orders.repartition(
    4,
    "city"
)

print("Partitions:", orders_by_city.rdd.getNumPartitions())
~~~

This redistributes rows using city as a partitioning expression.

Rows with the same city value are directed to the same hash partition for this repartition operation, but different cities can share a partition.

Do not assume each city gets a separate partition.

## 30. What is coalesce()?

coalesce() is commonly used to reduce the number of partitions while avoiding a full shuffle in typical use.

~~~python
orders_coalesced = orders_repartitioned.coalesce(2)

print("Partitions after coalesce:", orders_coalesced.rdd.getNumPartitions())
~~~

Expected partition count:

~~~text
Partitions after coalesce: 2
~~~

coalesce() is useful when reducing output partition count. It does not generally redistribute data evenly like repartition.

## 31. Compare repartition and coalesce

| Operation | Typical behavior | Shuffle |
|---|---|---|
| repartition(n) | Increase or decrease and redistribute partitions | Full shuffle generally occurs |
| coalesce(n) | Reduce partition count efficiently | Avoids full shuffle in typical use |

If you need a more even distribution or must increase partitions, use repartition. If you only need to reduce partitions, coalesce may be appropriate.

---

# Part 12 — Write partitioned files

## 32. What does partitionBy() do?

partitionBy() organizes output files into directory prefixes based on selected column values.

This is storage partitioning for output layout. It is related to, but not identical to, Spark's in-memory execution partitions.

## 33. Write sales data partitioned by city

~~~python
orders.write.mode("overwrite").partitionBy(
    "city"
).parquet("spark_day57/output/orders_by_city")
~~~

The output may look like:

~~~text
orders_by_city/
    city=Bengaluru/
        part-....parquet
    city=Chennai/
        part-....parquet
    city=Hyderabad/
        part-....parquet
~~~

The exact part-file count depends on the DataFrame's execution partitions.

## 34. Why partition output?

A query that filters for one city may be able to read only the relevant partition when the query engine supports partition pruning.

However, partitioning by a column with many unique values can create many small folders and files.

Choose partition columns based on query patterns and data volume.

## 35. Do not confuse three ideas

| Term | Meaning |
|---|---|
| Spark execution partition | Chunk processed by a task |
| DataFrame repartition/coalesce | Change execution partition layout |
| Storage partitionBy | Organize output files by column values |

They are related but not interchangeable.

---

# Part 13 — Join and partition validation

## 36. Compare row counts

Before and after a join, record row counts.

An inner join may reduce rows.
A left join should preserve left-side rows only when the right key is unique; duplicate right keys can multiply rows.
A full join may contain more rows than either input.

Never assume a join preserves the original row count.

## 37. Validate unmatched records

Use left_anti to identify left-side records without a match.

Use a reverse left_anti or full outer join to identify records missing from the other side.

## 38. Validate totals

If joining order rows to customer attributes, compare total amount before and after the join.

If the amount increases unexpectedly, check:
- duplicate customer keys;
- incorrect join condition;
- many-to-many relationships;
- historical dimension rows;
- unintended row multiplication.

## 39. Validate partitioned output

Check:
- expected partition columns;
- folder names;
- file count;
- row counts;
- null partition values;
- whether partitions are too small;
- whether downstream tools can read the output.

---

# Part 14 — Common mistakes

| Mistake | Why it is wrong | Better practice |
|---|---|---|
| Join on similarly named but unrelated columns | Produces incorrect matches | Confirm business key |
| Use inner join for all cases | Unmatched records disappear | Choose join type from requirement |
| Ignore duplicate dimension keys | Join can multiply facts | Validate key uniqueness |
| Ignore null join keys | Records may fail to match | Measure and handle nulls |
| Use ambiguous column names | Queries become unclear or fail | Alias and select explicitly |
| Assume join preserves row count | One-to-many matches multiply rows | Validate cardinality |
| Repartition constantly | Adds shuffle cost | Repartition only when needed |
| Use coalesce to increase partitions | It is intended primarily to reduce | Use repartition |
| Partition storage by high-cardinality field | Creates too many tiny directories/files | Choose sensible partition keys |
| Confuse execution partitions and storage partitions | Different concepts | Explain each separately |

---

# Part 15 — Practice questions

## Exercise 1

What does an inner join return?

**Answer:** Rows with matching keys on both sides.

## Exercise 2

Which join keeps all rows from the left DataFrame?

**Answer:** Left join.

## Exercise 3

Which join identifies orders without a matching customer?

**Answer:** Left anti join.

## Exercise 4

Why can duplicate customer keys increase order totals?

**Answer:** Each order may match multiple customer rows, multiplying the joined records.

## Exercise 5

What is repartition?

**Answer:** An operation that redistributes data into a requested number of Spark partitions, generally using a shuffle.

## Exercise 6

What is coalesce?

**Answer:** An operation commonly used to reduce partition count with less data movement than a full repartition.

## Exercise 7

What does partitionBy do?

**Answer:** Organizes written files into storage directories/prefixes based on selected column values.

## Exercise 8

Why should you validate totals after a join?

**Answer:** To detect duplicate keys, wrong join conditions and unintended row multiplication.

---

# Part 16 — Revision sheet

| Term | Remember |
|---|---|
| Join | Combines rows based on a condition |
| Join key | Column(s) used to match rows |
| Inner join | Matching rows only |
| Left join | All left rows |
| Right join | All right rows |
| Full outer join | All rows from both sides |
| Left semi | Left rows with a match |
| Left anti | Left rows without a match |
| Duplicate key | Key appears more than once |
| Join multiplication | One row matches multiple rows |
| Spark partition | Chunk of distributed data |
| repartition | Redistributes data and generally shuffles |
| coalesce | Commonly reduces partition count with less shuffle |
| partitionBy | Organizes output files by column values |

## Final self-check

- [ ] Explain why datasets are joined.
- [ ] Use inner, left, right and full joins.
- [ ] Use left semi and left anti joins.
- [ ] Identify unmatched and null keys.
- [ ] Detect duplicate keys and join multiplication.
- [ ] Use aliases to avoid ambiguous columns.
- [ ] Explain Spark execution partitions.
- [ ] Compare repartition and coalesce.
- [ ] Write output partitioned by a column.
- [ ] Validate row counts and totals after joins.

**Next lesson — Day 58: Spark Performance Optimization.** You will inspect execution plans, understand shuffle and caching, learn partition pruning and data skew, and compare performance using a repeatable experiment.
