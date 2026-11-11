# Day 58: Spark Performance Optimization

**Phase:** 6 — Cloud and Distributed Processing  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and junior Data Engineer candidates  
**Technology:** PySpark  
**Topic boundary:** Class notes only. Project workbook and interview guide are separate files.

> A Spark job can produce the correct result and still be too slow or expensive. Today you will learn how to investigate performance rather than guessing. We will inspect execution plans, understand shuffle, cache and persistence, partition pruning, data skew, broadcast joins and file layout. You will also learn to measure before and after changes and validate that optimization has not changed the business result.

## Learning outcomes

- Explain why Spark jobs become slow.
- Establish a repeatable baseline.
- Read a basic Spark execution plan.
- Understand narrow and wide transformations.
- Explain shuffle and spill.
- Use cache and persist carefully.
- Understand partition pruning.
- Recognize data skew and stragglers.
- Understand broadcast joins.
- Avoid unnecessary actions, collect and Python UDFs.
- Explain small-file and output partition problems.
- Validate correctness after optimization.

---

# Part 1 — What is Spark performance?

## 1. Beginner definition

Spark performance describes how efficiently a Spark application processes data.

It can include:
- total job duration;
- stage and task duration;
- CPU and memory use;
- shuffle read and write;
- disk spill;
- data scanned;
- number and size of tasks;
- output file count;
- cloud compute cost.

A job that finishes quickly on a small sample may behave differently on production data.

## 2. Why can Spark jobs become slow?

Common causes:
- too much data is read;
- unnecessary columns are selected;
- large shuffles;
- skewed join keys;
- too many tiny files;
- too few or too many partitions;
- repeated recomputation;
- excessive caching;
- Python UDF overhead;
- driver collects too much data;
- insufficient cluster resources;
- slow external storage or database;
- expensive joins and sorts.

Do not increase the cluster size before identifying the bottleneck.

## 3. Correctness comes before speed

An optimized job must return the intended business result.

Before and after tuning, compare:
- row count;
- distinct business keys;
- important totals;
- null counts;
- duplicate counts;
- filter conditions;
- output schema.

A faster wrong result is a production defect.

---

# Part 2 — Measure a baseline

## 4. What is a baseline?

A baseline is the measured starting performance before a change.

Record:
- Spark version;
- local or cluster mode;
- input data size;
- row count;
- number of partitions;
- query or transformation;
- execution time;
- plan;
- resource configuration;
- cache state.

## 5. Why does one timing not prove improvement?

Execution time can vary because of:
- cold versus warm cache;
- other users;
- CPU availability;
- network conditions;
- file system behavior;
- cluster startup;
- adaptive query execution;
- background processes.

Run comparable tests and record the conditions.

## 6. Measure a simple action

~~~python
import time

start_time = time.time()

row_count = orders.count()

end_time = time.time()

elapsed_seconds = end_time - start_time

print("Rows:", row_count)
print("Elapsed seconds:", round(elapsed_seconds, 3))
~~~

Example output shape:

~~~text
Rows: 5
Elapsed seconds: <measured value>
~~~

The time is environment-dependent. Do not copy another person's timing and claim it is yours.

## 7. Why use the same input?

If the first test reads 1,000 rows and the second reads 1,000,000 rows, the timing comparison is not fair.

Keep the following consistent:
- same input;
- same transformations;
- same output requirement;
- same compute configuration;
- comparable cache conditions.

---

# Part 3 — Inspect the execution plan

## 8. What is an execution plan?

An execution plan describes how Spark intends to execute a DataFrame query.

It can show:
- data sources;
- filters;
- projections;
- joins;
- aggregates;
- exchanges/shuffles;
- sorts;
- physical execution operators.

The plan helps explain what work Spark performs.

## 9. Use explain()

~~~python
orders.filter(
    orders.amount > 300
).select(
    "order_id",
    "amount"
).explain()
~~~

Spark prints a plan. Exact text depends on Spark version and configuration.

## 10. Use explain with extended information

~~~python
orders.filter(
    orders.amount > 300
).select(
    "order_id",
    "amount"
).explain("extended")
~~~

Extended output can include parsed, analyzed, optimized and physical plans.

The physical plan is especially useful for understanding how Spark will execute the work.

## 11. What is a scan?

A scan reads data from a source such as CSV, Parquet or a table.

A query that reads every column from a large dataset may do more work than necessary.

## 12. What is projection?

Projection means selecting the columns needed for the result.

Example:

~~~python
small_orders = orders.select(
    "order_id",
    "amount"
)
~~~

Selecting only required columns can reduce data processed and moved through later stages.

## 13. What is filter pushdown?

Filter pushdown means a filter is applied as close to the data source as possible when the source and format support it.

Parquet and some table formats can support efficient filtering and column selection.

Not every source supports every filter pushdown. Inspect the plan and source capabilities.

---

# Part 4 — Narrow and wide transformations

## 14. What is a narrow transformation?

A narrow transformation can usually process each output partition using data from one input partition, without redistributing data across the cluster.

Examples often include:
- select;
- filter;
- simple withColumn expressions.

These operations can often run within existing partitions.

## 15. What is a wide transformation?

A wide transformation requires data from multiple input partitions to be redistributed or combined.

Examples may include:
- groupBy;
- distinct;
- repartition;
- joins requiring data redistribution;
- global sorting.

Wide transformations often involve shuffle.

The exact execution depends on the plan and optimizations.

## 16. Compare narrow and wide

| Type | Data movement | Example |
|---|---|---|
| Narrow | Usually no cross-partition redistribution | filter |
| Wide | Often requires shuffle | groupBy |

---

# Part 5 — Understand shuffle

## 17. What is shuffle?

Shuffle redistributes data across workers so records that need to be processed together reach the same partition or executor.

Example:
Sales records are distributed across multiple partitions. To calculate total sales by customer, records for each customer must be grouped together.

## 18. Why is shuffle expensive?

Shuffle can require:
- serialization;
- network transfer;
- memory buffers;
- disk writes;
- disk reads;
- sorting or hashing.

A large shuffle can increase execution time and cost.

## 19. How can you reduce unnecessary shuffle?

- filter early when logically valid;
- select only required columns;
- avoid repeated repartition;
- use appropriate join keys;
- aggregate before a join when business semantics allow;
- use broadcast joins for suitable small inputs;
- choose appropriate partition counts.

Never change query logic merely to avoid a shuffle if it changes the result.

---

# Part 6 — Cache and persistence

## 20. What is caching?

Caching stores a computed DataFrame so later actions can reuse it instead of recomputing the full lineage.

It can help when the same expensive DataFrame is reused multiple times.

## 21. When should you cache?

Cache may help when:
- the DataFrame is expensive to compute;
- it is reused by multiple downstream operations;
- it fits available memory/storage resources;
- repeated actions would otherwise recompute it.

Do not cache every DataFrame.

## 22. Cache example

~~~python
filtered_orders = orders.filter(
    orders.amount > 200
)

filtered_orders.cache()

print("Filtered count:", filtered_orders.count())

filtered_orders.groupBy("city").count().show()
~~~

The count action materializes the cached DataFrame. The later groupBy action may reuse cached data.

The actual storage level and execution behavior depend on Spark version and configuration.

## 23. Unpersist

Cached data occupies resources.

When it is no longer needed:

~~~python
filtered_orders.unpersist()

print("Cache released")
~~~

Expected output:

~~~text
Cache released
~~~

unpersist asks Spark to remove the persisted data from cache/storage.

## 24. Cache versus persist

cache() is a convenient default persistence method.

persist() allows an explicit storage level where supported.

Choose a storage level based on memory, disk and recomputation trade-offs. Do not assume cached data always stays entirely in memory.

## 25. Cache mistakes

- caching a DataFrame used only once;
- caching a huge DataFrame that causes memory pressure;
- forgetting to unpersist;
- assuming cache always makes a job faster;
- measuring the first cached action and later actions as if they were identical.

---

# Part 7 — Partition pruning

## 26. What is partition pruning?

Partition pruning means Spark or the query engine skips storage partitions that cannot match a filter.

Example storage layout:

~~~text
sales/
    year=2025/
    year=2026/
~~~

A query filtering for year 2026 may read only that partition if the table and query support pruning.

## 27. Why partition pruning helps

It can reduce:
- files opened;
- bytes scanned;
- input rows;
- compute time;
- cloud storage requests.

The filter should align with the partition column and be expressed in a way the engine can use.

## 28. Read a partitioned dataset

Assume a Parquet dataset is stored with a year partition.

~~~python
sales = spark.read.parquet(
    "spark_day58/data/sales"
)

sales_2026 = sales.filter(
    sales.year == 2026
)

sales_2026.explain("formatted")
~~~

Inspect the plan for partition filters. Exact output depends on Spark version and source layout.

## 29. Do not partition every field

High-cardinality fields such as customer_id can create many tiny partitions.

Common partition candidates for lake datasets include date fields, but only when they match query patterns and file volume.

---

# Part 8 — Data skew

## 30. What is data skew?

Data skew occurs when some keys or partitions contain much more data than others.

Example:
- city A has 1 million records;
- city B has 1 million records;
- city C has 80 million records.

One task may take much longer.

## 31. Why skew slows a job

The job may wait for the largest partition to finish.

Even if most tasks finish quickly, one straggler can keep the stage running.

## 32. How to investigate skew

Inspect:
- task input sizes;
- task duration distribution;
- shuffle read/write;
- keys with unusually high counts;
- partition sizes.

Do not rely only on average task time.

## 33. Possible skew solutions

Depending on the workload:
- use a better partition key;
- pre-aggregate where logically valid;
- enable supported adaptive skew handling;
- isolate very large keys;
- use salting in suitable cases;
- use a broadcast join when one side is sufficiently small.

Every change must be validated for correctness.

---

# Part 9 — Broadcast joins

## 34. What is a broadcast join?

A broadcast join sends a small dataset to executors so it can be joined locally with partitions of a larger dataset.

Example:
- a very large orders dataset;
- a small country-code reference dataset.

Instead of shuffling both sides, Spark may broadcast the small side.

## 35. Broadcast example

~~~python
from pyspark.sql.functions import broadcast

joined = orders.join(
    broadcast(customers),
    on="customer_id",
    how="left"
)

joined.explain()
~~~

This is appropriate only if customers is small enough to be safely broadcast to executors.

## 36. Broadcast risks

A dataset that seems small on one machine may become large after serialization or under cluster memory constraints.

Broadcasting a large table can cause memory pressure or executor failure.

Inspect the plan and size. Do not force broadcast without evidence.

---

# Part 10 — Avoid unnecessary Python overhead

## 37. Prefer built-in Spark functions

Spark understands built-in expressions such as:
- col;
- when;
- sum;
- avg;
- concat;
- date functions.

These can be optimized by Spark's query engine.

## 38. What is a Python UDF?

A user-defined function (UDF) lets you write custom logic in Python and apply it to Spark data.

UDFs can be useful when built-in expressions cannot express the required logic.

However, Python UDFs may add serialization and execution overhead and can limit some query optimizations.

Use built-in Spark functions when possible.

## 39. Avoid row-by-row driver loops

Do not collect a large DataFrame and loop over every row in Python.

That moves distributed data to the driver and loses Spark's distributed execution benefits.

Use DataFrame expressions and aggregations.

---

# Part 11 — File layout and small files

## 40. Why do file sizes matter?

A dataset with thousands of tiny files can require many metadata and file-open operations.

This can slow reads and increase scheduling overhead.

## 41. What is the small-file problem?

The small-file problem occurs when too many tiny files make processing inefficient.

Possible solutions:
- compact small files;
- control output partition counts;
- write appropriately sized files;
- avoid excessive storage partition columns;
- use platform-supported table maintenance.

Do not combine everything into one huge file. Parallelism and recovery also matter.

## 42. Output partition count

A DataFrame with many execution partitions may produce many output part files.

A DataFrame with too few partitions may produce large files and insufficient parallelism.

Tune based on data size, target file size and downstream workload.

---

# Part 12 — A before-and-after experiment

## 43. Prepare a larger training DataFrame

~~~python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, sum

spark = (
    SparkSession.builder
    .appName("Day58PerformanceExperiment")
    .master("local[*]")
    .getOrCreate()
)

sales = spark.range(0, 500000).withColumn(
    "customer_id",
    (col("id") % 10000)
).withColumn(
    "amount",
    (col("id") % 500) + 1
)
~~~

This creates synthetic rows locally. It is a learning dataset, not a benchmark of a cloud cluster.

## 44. Inspect schema and count

~~~python
sales.printSchema()
print("Rows:", sales.count())
~~~

Expected row count:

~~~text
Rows: 500000
~~~

## 45. Inspect the baseline plan

~~~python
baseline = sales.groupBy("customer_id").agg(
    sum("amount").alias("total_sales")
)

baseline.explain("formatted")
~~~

The plan shows how Spark performs the grouping. It may include an exchange/shuffle stage.

## 46. Measure baseline

~~~python
import time

start_time = time.time()

baseline_result = baseline.count()

baseline_time = time.time() - start_time

print("Groups:", baseline_result)
print("Baseline seconds:", round(baseline_time, 3))
~~~

Expected group count:

~~~text
Groups: 10000
Baseline seconds: <measured value>
~~~

Record the actual time from your environment.

## 47. Select only required columns before aggregation

~~~python
sales_required = sales.select(
    "customer_id",
    "amount"
)

optimized_candidate = sales_required.groupBy(
    "customer_id"
).agg(
    sum("amount").alias("total_sales")
)

optimized_candidate.explain("formatted")
~~~

This makes the required columns explicit. Spark may already prune unused columns in the original plan, so the plan and timing may not change. Do not claim an improvement unless measured.

## 48. Compare correctness

~~~python
print(
    "Baseline groups:",
    baseline.count()
)

print(
    "Candidate groups:",
    optimized_candidate.count()
)
~~~

Both should produce 10,000 groups for this dataset.

For stronger validation, compare aggregate totals as well.

~~~python
source_total = sales.agg(
    sum("amount").alias("source_total")
).first()["source_total"]

result_total = optimized_candidate.agg(
    sum("total_sales").alias("result_total")
).first()["result_total"]

print("Source total:", source_total)
print("Grouped total:", result_total)
print("Same total:", source_total == result_total)
~~~

Expected final check:

~~~text
Same total: True
~~~

The numeric totals depend on the sample generation logic and should match between source and grouped result.

## 49. Record the experiment

| Measurement | Baseline | Candidate |
|---|---|---|
| Input rows | 500000 | 500000 |
| Group count | 10000 | 10000 |
| Execution plan | Record actual | Record actual |
| Time | Record actual | Record actual |
| Total validation | Record | Record |

The candidate may not be faster because Spark can already optimize column pruning. The learning goal is to inspect the plan and measure honestly.

---

# Part 13 — Common mistakes

| Mistake | Why it is wrong | Better practice |
|---|---|---|
| Increase workers before investigation | Bottleneck may be elsewhere | Measure and inspect plan |
| Assume every shuffle is bad | Some operations require redistribution | Reduce unnecessary shuffle |
| Cache every DataFrame | Uses memory and storage | Cache only reused expensive data |
| Forget unpersist | Resources remain occupied | Release cached data |
| Ignore skew | One task can dominate | Inspect task distribution |
| Broadcast a large table | Executor memory can fail | Broadcast only suitable small data |
| Use Python UDF for simple logic | Adds overhead and limits optimization | Prefer built-in functions |
| Collect large data | Driver memory pressure | Keep operations distributed |
| Partition by customer ID | Too many small files | Choose workload-aligned partitions |
| Claim speedup without timing | Not evidence | Compare repeatable measurements |
| Change filters while tuning | Business result changes | Validate rows, keys and totals |

---

# Part 14 — Practice questions

## Exercise 1

What does explain() show?

**Answer:** Spark's logical and/or physical execution plan, depending on the selected mode.

## Exercise 2

What is shuffle?

**Answer:** Redistribution of data across partitions or workers.

## Exercise 3

When should you cache?

**Answer:** When an expensive DataFrame is reused and caching is likely to save recomputation without causing resource pressure.

## Exercise 4

What is partition pruning?

**Answer:** Skipping storage partitions that cannot match the query filter.

## Exercise 5

What is data skew?

**Answer:** Uneven data distribution that can make some tasks much slower.

## Exercise 6

What is a broadcast join?

**Answer:** Sending a small dataset to executors so it can be joined with larger partitions locally.

## Exercise 7

Why prefer built-in functions over Python UDFs?

**Answer:** Built-in expressions are generally easier for Spark to optimize and avoid some Python serialization overhead.

## Exercise 8

What must be validated after optimization?

**Answer:** Row counts, keys, totals, null behavior and intended business filters.

---

# Part 15 — Revision sheet

| Term | Remember |
|---|---|
| Baseline | Measured starting performance |
| explain() | Displays execution plan |
| Projection | Select required columns |
| Filter pushdown | Apply filter near source when supported |
| Narrow transformation | Usually works within input partitions |
| Wide transformation | Often requires data redistribution |
| Shuffle | Data movement across workers |
| Cache | Reuse computed data |
| Persist | Store data using a chosen storage level |
| unpersist | Release persisted data |
| Partition pruning | Skip irrelevant storage partitions |
| Data skew | Uneven data distribution |
| Straggler | Slow task |
| Broadcast join | Send small dataset to executors |
| UDF | User-defined function |
| Small-file problem | Overhead from many tiny files |
| Spill | Data moved from memory to disk under resource pressure |

## Final self-check

- [ ] Explain why Spark jobs become slow.
- [ ] Record a baseline.
- [ ] Inspect explain output.
- [ ] Explain narrow and wide transformations.
- [ ] Explain shuffle and spill.
- [ ] Use cache and unpersist carefully.
- [ ] Explain partition pruning.
- [ ] Identify skew and stragglers.
- [ ] Explain broadcast joins.
- [ ] Prefer built-in functions when possible.
- [ ] Explain small-file problems.
- [ ] Compare before/after without inventing speedups.
- [ ] Validate correctness after optimization.

**Next lesson — Day 59: Cloud and Distributed Pipeline Integration.** You will connect the ideas from object storage, cloud databases and Spark into a complete batch pipeline design, then compare it with a streaming architecture and learn the role of Kafka.
