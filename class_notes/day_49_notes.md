# Day 49: Warehouse Performance Optimization

**Phase:** 5 — Data Warehousing and Modeling  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and junior Data Engineer candidates  
**Database:** PostgreSQL for SQL demonstrations  
**Topic boundary:** Class notes only. Project workbook and interview guide are separate files.

> A warehouse query may return the correct answer but take too long or consume too much compute. Performance optimization means improving response time and resource use without changing the business result. Today we learn to inspect query plans, understand indexes and partitioning, compare views and materialized views, and think about warehouse storage and cost.

## Learning outcomes

By the end, you should be able to:
- explain why warehouse queries become slow;
- distinguish query tuning from changing the business result;
- read basic EXPLAIN output;
- understand EXPLAIN ANALYZE safely;
- explain sequential and index scans;
- understand index trade-offs;
- explain partitioning and partition pruning;
- distinguish partitioning from indexing;
- compare views, tables and materialized views;
- understand aggregation and precomputation;
- identify data skew and small-file issues at a conceptual level;
- consider compute, storage and data movement costs;
- perform a basic before/after investigation;
- avoid claiming performance improvement without measurement.

## Today's flow

`Understand workload -> Measure baseline -> Inspect plan -> Identify bottleneck -> Change one thing -> Measure again -> Validate result -> Monitor cost`

---

# Part 1 — What is warehouse performance?

## 1.1 Beginner definition

Warehouse performance describes how efficiently the analytical platform executes queries and data workloads.

It can include:
- query response time;
- throughput;
- concurrency;
- compute usage;
- memory usage;
- disk or storage reads;
- data scanned;
- shuffle or data movement;
- cost.

A query that is fast for one user may become slow when hundreds of users run it together.

## 1.2 Why do queries become slow?

Common causes:
- scanning too many rows;
- reading too many columns;
- missing or unsuitable indexes in row-based databases;
- poor partition filtering;
- joining large datasets unnecessarily;
- incorrect join keys;
- data skew;
- repeated expensive transformations;
- stale statistics;
- too much concurrency;
- inefficient file layout;
- insufficient or poorly configured compute.

Do not assume every slow query needs an index. First inspect the workload and execution plan.

## 1.3 Performance must preserve correctness

A query that returns fewer rows because a filter was accidentally added is not optimized correctly.

The goal is:

**Same intended business result, using fewer resources or less time.**

Before and after tuning, validate:
- row count;
- key set;
- important totals;
- null behavior;
- filter semantics.

---

# Part 2 — Measure before optimizing

## 2.1 Establish a baseline

Record:
- query text and parameters;
- execution time;
- rows returned;
- rows scanned if available;
- bytes scanned if available;
- compute usage;
- concurrency conditions;
- data size;
- date range;
- query plan.

Do not compare two timings if the data volume, cache state or concurrency differs substantially without noting it.

## 2.2 Why one timing is not enough

Query time can vary due to:
- warm or cold cache;
- other database activity;
- network delay;
- background maintenance;
- platform scaling;
- query compilation.

Run repeatable tests and compare more than one execution where appropriate.

## 2.3 Avoid invented benchmark claims

Do not write "the index made the query 90% faster" unless you measured that result in the target environment.

The plan and timing depend on:
- PostgreSQL version;
- table size;
- data distribution;
- statistics;
- cache;
- hardware;
- query predicate.

This lesson teaches the measurement process rather than promising a universal speedup.

---

# Part 3 — Read EXPLAIN

## 3.1 What is a query execution plan?

A query execution plan describes how the database intends to retrieve and combine data.

It may show:
- scan methods;
- join methods;
- sort operations;
- aggregate operations;
- estimated rows;
- estimated cost;
- filters.

The database optimizer chooses a plan based on query structure, statistics and configuration.

## 3.2 EXPLAIN

In PostgreSQL:

```sql
EXPLAIN
SELECT *
FROM orders
WHERE order_id = 1001;
```

This displays the planned operations without executing the SELECT.

The exact plan text depends on the table, indexes and statistics.

## 3.3 EXPLAIN ANALYZE

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE order_id = 1001;
```

This executes the query and reports actual execution information as well as plan estimates.

Important:
- EXPLAIN ANALYZE executes the statement.
- It is safe to demonstrate with a read-only SELECT.
- Do not casually run it on INSERT, UPDATE or DELETE in production.
- For data-changing statements, understand side effects and use a safe transaction/test environment.

## 3.4 Estimated versus actual rows

The plan may show:
- estimated rows;
- actual rows.

Large differences can indicate stale or insufficient statistics, skew or a mismatch between the optimizer's assumptions and the data.

## 3.5 Basic scan types

| Plan node | Meaning |
|---|---|
| Seq Scan | Reads table rows sequentially |
| Index Scan | Uses an index to locate rows and may visit table pages |
| Index Only Scan | May satisfy query from index data when visibility conditions allow |
| Bitmap Index Scan | Builds a bitmap of matching row locations |
| Bitmap Heap Scan | Visits table pages identified by the bitmap |

A sequential scan is not automatically bad. For a query that needs most of a table, reading sequentially may be cheaper than many index lookups.

## 3.6 Join methods

| Join node | Beginner explanation |
|---|---|
| Nested Loop | For each row from one input, searches the other input |
| Hash Join | Builds a hash structure for one input and matches the other |
| Merge Join | Joins sorted inputs using ordered key values |

The best join method depends on row counts, indexes, sorting and data distribution.

---

# Part 4 — Build a training table for plan inspection

The following uses PostgreSQL's generate_series function.

## 4.1 Create a large-enough training table

```sql
CREATE TEMP TABLE performance_orders AS
SELECT
    value AS order_id,
    (value % 10000) + 1 AS customer_id,
    DATE '2026-01-01' + (value % 365) AS order_date,
    (value % 50000) + 100 AS amount
FROM generate_series(1, 200000) AS gs(value);
```

Expected output:

```text
SELECT 200000
```

This creates 200,000 synthetic rows. It does not represent a real benchmark.

## 4.2 Inspect the row count

```sql
SELECT COUNT(*) AS row_count
FROM performance_orders;
```

Expected result:

| row_count |
|---:|
| 200000 |

## 4.3 Inspect a selective query plan

```sql
EXPLAIN
SELECT *
FROM performance_orders
WHERE order_id = 12345;
```

The output will show a plan selected by your PostgreSQL instance.

Before creating an index, note:
- scan type;
- estimated rows;
- estimated cost;
- filter condition.

The temporary table has no explicit index on order_id in this exercise. The optimizer may choose a sequential scan.

## 4.4 Create an index

```sql
CREATE INDEX idx_performance_orders_order_id
ON performance_orders(order_id);
```

Expected output:

```text
CREATE INDEX
```

An index is an additional structure that helps locate rows by the indexed column.

## 4.5 Inspect the plan again

```sql
EXPLAIN
SELECT *
FROM performance_orders
WHERE order_id = 12345;
```

The optimizer may now choose an index-based plan.

Do not assume it must choose an index. The optimizer decides based on estimated cost and available statistics.

## 4.6 Run ANALYZE statistics

```sql
ANALYZE performance_orders;
```

Expected output:

```text
ANALYZE
```

ANALYZE collects table statistics used by the optimizer.

## 4.7 Inspect actual execution

```sql
EXPLAIN ANALYZE
SELECT *
FROM performance_orders
WHERE order_id = 12345;
```

The output includes actual timing and rows for your environment.

Record your actual output. Do not copy a timing from another student's computer as if it were your measurement.

## 4.8 Why did we use a selective filter?

The query returns one row out of 200,000.

An index can be useful for locating a small subset of rows.

If the query returns most rows, a sequential scan may still be cheaper.

## 4.9 Query only required columns

Compare:

```sql
SELECT *
FROM performance_orders
WHERE order_id = 12345;
```

with:

```sql
SELECT order_id, order_date, amount
FROM performance_orders
WHERE order_id = 12345;
```

Selecting only needed columns can reduce data transfer and memory use, especially in columnar warehouse platforms.

In PostgreSQL, the optimizer may also use an index-only strategy when the index and visibility conditions permit it.

---

# Part 5 — Understand indexes

## 5.1 What is an index?

An index is a data structure that helps the database locate rows using selected column values.

Think of a book index:
- without an index, you may scan every page;
- with an index, you can locate the page range for a topic.

A database index helps avoid reading every table row for some queries.

## 5.2 Index trade-offs

Indexes can improve reads, but they have costs:
- extra storage;
- extra work on INSERT;
- extra work on UPDATE;
- extra work on DELETE;
- maintenance and vacuum considerations;
- possible cache pressure.

Do not create an index on every column.

## 5.3 Common B-tree index

PostgreSQL's default index method is B-tree.

It is useful for many equality and range predicates, such as:
- order_id = 1001;
- order_date >= a date;
- customer_id between two values.

Other index methods support specialized use cases.

## 5.4 Composite index

A composite index includes more than one column.

Example:

```sql
CREATE INDEX idx_orders_customer_date
ON orders(customer_id, order_date);
```

This can support queries filtering by customer and date.

Column order matters. An index on (customer_id, order_date) is not identical to one on (order_date, customer_id).

Choose based on actual query predicates and access patterns.

## 5.5 Low-selectivity columns

A column with only a few distinct values, such as a boolean flag, may not benefit from a simple B-tree index for many queries.

But it depends on distribution, table size, partial indexes and query patterns.

Do not apply a universal rule without examining the plan.

## 5.6 Index maintenance

Indexes consume storage and require maintenance.

Unused indexes can slow writes without helping important queries.

Review actual usage and query plans before adding or removing indexes.

---

# Part 6 — Partitioning

## 6.1 What is partitioning?

Partitioning divides a large logical table into smaller physical pieces according to a partition key.

Example:
- sales by month;
- events by day;
- orders by year.

The user queries one logical table, while the database can access only relevant partitions when partition pruning applies.

## 6.2 Why partition?

Possible benefits:
- reduce data scanned for filtered queries;
- simplify retention management;
- make bulk loading or archival easier;
- isolate data by time or another key.

Partitioning is not a replacement for a good model or index.

## 6.3 Range partitioning example

Create a partitioned parent table:

```sql
CREATE TABLE sales_partitioned (
    sale_id INTEGER NOT NULL,
    sale_date DATE NOT NULL,
    amount NUMERIC(12, 2) NOT NULL
) PARTITION BY RANGE (sale_date);
```

Expected output:

```text
CREATE TABLE
```

Create January partition:

```sql
CREATE TABLE sales_2026_01
PARTITION OF sales_partitioned
FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');
```

Expected output:

```text
CREATE TABLE
```

Create February partition:

```sql
CREATE TABLE sales_2026_02
PARTITION OF sales_partitioned
FOR VALUES FROM ('2026-02-01') TO ('2026-03-01');
```

Expected output:

```text
CREATE TABLE
```

The lower bound is inclusive and the upper bound is exclusive.

## 6.4 Insert rows

```sql
INSERT INTO sales_partitioned VALUES
(1, DATE '2026-01-05', 1000.00),
(2, DATE '2026-01-20', 1500.00),
(3, DATE '2026-02-10', 700.00);
```

Expected output:

```text
INSERT 0 3
```

PostgreSQL routes each row to the matching partition.

## 6.5 Query one month

```sql
SELECT *
FROM sales_partitioned
WHERE sale_date >= DATE '2026-01-01'
  AND sale_date < DATE '2026-02-01';
```

Expected result:

| sale_id | sale_date | amount |
|---:|---|---:|
| 1 | 2026-01-05 | 1000.00 |
| 2 | 2026-01-20 | 1500.00 |

## 6.6 Understand partition pruning

Partition pruning means the database avoids scanning partitions that cannot contain matching rows.

Inspect:

```sql
EXPLAIN
SELECT *
FROM sales_partitioned
WHERE sale_date >= DATE '2026-01-01'
  AND sale_date < DATE '2026-02-01';
```

Your plan should show which partitions are considered. Exact plan output depends on PostgreSQL version and settings.

## 6.7 Choose a partition key carefully

A useful partition key often matches common filters and lifecycle operations.

Examples:
- event_date;
- transaction_month;
- tenant_id in some multi-tenant systems.

Avoid creating too many tiny partitions. Partition management and planning overhead can become significant.

## 6.8 Partitioning mistakes

- partitioning by a column that reports rarely filter;
- creating one partition for every tiny group;
- forgetting future partitions;
- not handling rows outside configured ranges;
- assuming partitioning automatically speeds every query;
- using functions on the partition column in a way that prevents pruning.

---

# Part 7 — Views, tables and materialized views

## 7.1 View

A view stores a query definition. When queried, the database executes the view's underlying query.

Useful for:
- reusable SQL logic;
- access abstraction;
- simple presentation models.

A view does not normally store a separate copy of the query result.

## 7.2 Table materialization

A table stores the result physically.

Useful when:
- the result is expensive to calculate;
- many queries reuse it;
- refresh timing is controlled.

It requires storage and a refresh/load process.

## 7.3 Materialized view

A materialized view stores the result of a query and can be refreshed.

PostgreSQL example:

This example uses permanent practice objects because a PostgreSQL materialized view is a persistent database object. Run it in an isolated training database. If you rerun the lesson, remove the previous practice objects first.

```sql
DROP MATERIALIZED VIEW IF EXISTS daily_sales_summary;
DROP TABLE IF EXISTS daily_sales_source;
```

Expected output: PostgreSQL reports that the objects were dropped when they existed. Notices are normal when they did not exist.

Create the source table:

```sql
CREATE TABLE daily_sales_source (
    sale_id INTEGER PRIMARY KEY,
    sale_date DATE NOT NULL,
    amount NUMERIC(10, 2) NOT NULL
);
```

Expected output:

```text
CREATE TABLE
```

Insert the training rows:

```sql
INSERT INTO daily_sales_source VALUES
(1, DATE '2026-01-01', 1000.00),
(2, DATE '2026-01-01', 500.00),
(3, DATE '2026-01-02', 700.00);
```

Expected output:

```text
INSERT 0 3
```

Create a materialized view:

```sql
CREATE MATERIALIZED VIEW daily_sales_summary AS
SELECT
    sale_date,
    SUM(amount) AS total_sales
FROM daily_sales_source
GROUP BY sale_date;
```

Expected output:

```text
SELECT 2
```

Query it:

```sql
SELECT *
FROM daily_sales_summary
ORDER BY sale_date;
```

Expected result:

| sale_date | total_sales |
|---|---:|
| 2026-01-01 | 1500.00 |
| 2026-01-02 | 700.00 |

Now insert a new source row:

```sql
INSERT INTO daily_sales_source VALUES
(4, DATE '2026-01-02', 300.00);
```

Expected output:

```text
INSERT 0 1
```

Query the materialized view again before refreshing. It still shows the earlier result because a materialized view stores its previous query result.

Refresh it:

```sql
REFRESH MATERIALIZED VIEW daily_sales_summary;
```

Expected output:

```text
REFRESH MATERIALIZED VIEW
```

Query again:

```sql
SELECT *
FROM daily_sales_summary
ORDER BY sale_date;
```

Expected result after refresh:

| sale_date | total_sales |
|---|---:|
| 2026-01-01 | 1500.00 |
| 2026-01-02 | 1000.00 |

This demonstrates the freshness trade-off: the stored result does not change until it is refreshed. Production systems must define refresh ownership, schedule, locking/concurrency behavior and failure monitoring.

## 7.4 Materialization trade-off

| Type | Stores result? | Refresh behavior |
|---|---|---|
| View | No separate result copy | Reads current underlying data |
| Table | Yes | Pipeline rebuilds or updates it |
| Materialized view | Yes | Explicit refresh or managed refresh process |

The right choice depends on freshness, query cost, storage, concurrency and platform support.

---

# Part 8 — Warehouse-specific performance ideas

## 8.1 Columnar storage

Many analytical warehouses store data in column-oriented formats.

Instead of reading every field for every row, a query may read only the columns it needs.

This can reduce I/O for analytical scans.

Columnar storage is one reason warehouse platforms may optimize differently from row-oriented PostgreSQL.

## 8.2 Partition pruning

A query filtered by partition column can avoid reading unrelated partitions.

The filter must be compatible with the partition design.

## 8.3 Clustering and sorting

Some warehouse platforms support clustering, sorting or data organization features.

They can improve locality for common filters and joins.

The exact feature and behavior are platform-specific.

## 8.4 Data skew

Data skew occurs when some partition or key contains much more data than others.

Example:
- one city has 80% of all orders;
- other cities have very few.

Skew can make distributed processing uneven.

Possible mitigations depend on the engine and workload:
- better partition keys;
- skew-aware join strategies;
- pre-aggregation;
- salting in suitable distributed workloads.

Do not apply salting without understanding the query and correctness requirements.

## 8.5 Small-file problem

In file-based warehouses and lakehouse systems, thousands of tiny files can create overhead:
- listing files;
- opening files;
- scheduling tasks;
- reading metadata.

Compaction can combine small files into larger efficient files.

But very large files or partitions can also create parallelism and recovery problems. Choose sizes based on platform guidance and workload.

## 8.6 Data movement and shuffle

Distributed engines may move data across workers for joins and aggregations.

This is often called shuffle.

Large shuffles can increase:
- network traffic;
- disk spill;
- execution time;
- compute cost.

Reduce unnecessary columns and rows before expensive joins, and inspect the execution plan.

---

# Part 9 — Cost awareness

## 9.1 Performance and cost are connected

A query may be fast because it uses a large compute cluster, but that may cost more.

A good engineer considers:
- compute size;
- query duration;
- bytes scanned;
- storage;
- concurrency;
- refresh frequency;
- data movement;
- idle resources.

## 9.2 Common cost controls

- filter early where semantically valid;
- select only needed columns;
- avoid repeated full scans;
- reuse curated models;
- choose appropriate materialization;
- partition by useful access patterns;
- avoid excessive tiny files;
- scale compute to workload;
- stop idle resources where platform supports it;
- monitor expensive queries.

## 9.3 Do not optimize only for one query

A new index or materialized table may speed one report but slow writes or increase storage.

Measure the workload as a whole.

---

# Part 10 — Before-and-after optimization workflow

Use this workflow:

1. Capture the original query.
2. Record the business result and expected scope.
3. Record baseline time and plan.
4. Identify the expensive operation.
5. Change one thing.
6. Run the same query on comparable data.
7. Record the new plan and time.
8. Validate row count and important totals.
9. Compare storage/write costs.
10. Document the trade-off.

Example optimization candidates:
- add a useful index;
- add a partition filter;
- select fewer columns;
- aggregate before a large join when logically valid;
- materialize a repeatedly used expensive result;
- update table statistics.

---

# Part 11 — Common mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Add index to every column | Storage and write costs increase | Use workload and plan evidence |
| Assume Seq Scan is bad | Full scan may be cheapest | Compare rows needed and cost |
| Run EXPLAIN ANALYZE on production DML casually | It executes the statement | Use read-only SELECT or safe isolated environment |
| Claim speedup without measuring | Results depend on environment | Record repeatable baseline and after result |
| Partition by arbitrary field | Queries may scan all partitions | Match access and lifecycle patterns |
| Create too many partitions | Planning and maintenance overhead | Choose sensible partition granularity |
| Materialize everything | Storage and refresh costs grow | Materialize expensive reused models |
| Ignore stale materialized results | Reports can be outdated | Define and monitor refresh |
| Optimize one query only | Other workloads may regress | Evaluate workload and cost together |
| Change SQL and not validate output | Faster wrong result is still wrong | Compare keys, rows and metrics |

---

# Part 12 — Practice exercises

## Exercise 1

A query filters one customer out of ten million rows. What could help?

**Answer:** A suitable index may help in a row-oriented database, but inspect the plan and data distribution first.

## Exercise 2

A monthly report reads a five-year fact table but filters to one month. What can help?

**Answer:** Date partitioning with a predicate that enables partition pruning, if the platform and workload justify it.

## Exercise 3

A report repeatedly calculates the same expensive aggregate. What could help?

**Answer:** A table or materialized view with a controlled refresh process may help, subject to freshness requirements.

## Exercise 4

Why is a sequential scan not always bad?

**Answer:** If a query needs a large percentage of rows, reading the table sequentially can be cheaper than many random index lookups.

## Exercise 5

What must be checked after optimization?

**Answer:** Same business scope, row counts, keys, totals and output semantics, along with execution time, plan and resource/cost impact.

---

# Part 13 — Revision map

| Term | Remember |
|---|---|
| Query plan | Database's execution strategy |
| EXPLAIN | Shows planned operations |
| EXPLAIN ANALYZE | Executes and reports actual plan information |
| Sequential scan | Reads table rows sequentially |
| Index | Structure to locate matching rows |
| Composite index | Index over multiple columns |
| Partitioning | Divides a logical table into physical partitions |
| Partition pruning | Avoids irrelevant partitions |
| View | Stored query definition |
| Materialized view | Stored query result that must be refreshed |
| Columnar storage | Stores data by column for analytical access |
| Data skew | Uneven distribution of data |
| Shuffle | Data movement across workers |
| Small-file problem | Overhead caused by too many tiny files |
| Baseline | Measured starting performance |
| Cost awareness | Balance performance and resource consumption |

## Final competency checklist

- [ ] Explain why queries become slow.
- [ ] Read basic EXPLAIN output.
- [ ] Explain EXPLAIN ANALYZE and its execution risk.
- [ ] Compare sequential and index scans.
- [ ] Explain index trade-offs.
- [ ] Explain partitioning and pruning.
- [ ] Compare view, table and materialized view.
- [ ] Explain skew, shuffle and small-file overhead.
- [ ] Record a before-and-after performance test.
- [ ] Validate correctness after optimization.
- [ ] Explain compute and storage cost trade-offs.

**Phase 5 completion:** You have learned warehouse architecture, dimensional modeling, SCD history, key management, data marts, quality, reconciliation, metadata, loading strategies and performance fundamentals. The next phase begins with cloud data engineering fundamentals.
