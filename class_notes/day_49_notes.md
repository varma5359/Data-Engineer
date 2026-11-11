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


---

# Part 14 — Your first performance investigation, step by step

The earlier section created a 200,000-row table and an index. Now perform a controlled investigation rather than assuming that an index is always faster.

Keep this work in the same PostgreSQL session so that the temporary table remains available.

## 14.1 Write down the question

Business question:

"Find the order with ID 12345 and show its date and amount."

The query needs only three columns:
- order_id;
- order_date;
- amount.

It does not need customer_id.

## 14.2 Record the baseline plan

Before creating any additional index, run:

~~~sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT order_id, order_date, amount
FROM performance_orders
WHERE order_id = 12345;
~~~

Record:
- scan type;
- estimated rows;
- actual rows;
- execution time;
- shared buffer hits and reads, if displayed.

Your actual output depends on PostgreSQL version, cache state and environment. Do not copy a timing from this document.

If you already created the order_id index in Part 4, this is no longer an unindexed baseline. To repeat the comparison, create a fresh training table or drop the index in the isolated training session and rerun ANALYZE.

## 14.3 Understand BUFFERS at a beginner level

With BUFFERS enabled, PostgreSQL can report buffer activity.

- shared hit: a needed page was already in PostgreSQL shared buffers;
- shared read: PostgreSQL had to read a page into shared buffers.

These values help investigate I/O behavior. They are not a complete measure of physical disk activity or cloud cost.

## 14.4 Compare a query that returns many rows

~~~sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT order_id, order_date, amount
FROM performance_orders
WHERE amount >= 100;
~~~

This predicate may match a large part of the table.

An index on amount is not guaranteed to be useful. If many rows qualify, the optimizer may prefer a sequential scan.

The correct plan depends on actual data distribution and statistics.

## 14.5 Compare a date-range query

~~~sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT
    order_id,
    order_date,
    amount
FROM performance_orders
WHERE order_date >= DATE '2026-03-01'
  AND order_date < DATE '2026-04-01';
~~~

This is a range query. It can be a candidate for a date index or date partitioning in an appropriate production table.

Do not add both automatically. First inspect the plan, table size, workload and write cost.

## 14.6 Build an evidence table

Fill this table with your own execution results:

| Measurement | Before change | After change |
|---|---|---|
| Query text | Record exact query | Same logical query |
| Scan type | Record plan node | Record plan node |
| Estimated rows | Record value | Record value |
| Actual rows | Record value | Record value |
| Execution time | Record actual value | Record actual value |
| Buffer reads/hits | Record if available | Record if available |
| Output count | Validate | Validate |
| Output total | Validate if relevant | Validate if relevant |

Do not write "faster" unless the repeated measurements support that conclusion under comparable conditions.

---

# Part 15 — Understand index choices with query patterns

## 15.1 Equality lookup

Example:

~~~sql
SELECT *
FROM performance_orders
WHERE customer_id = 501;
~~~

This asks for rows belonging to one customer.

An index on customer_id may help if the customer is selective enough and the table is large enough. Inspect the plan rather than assuming.

## 15.2 Range lookup

Example:

~~~sql
SELECT *
FROM performance_orders
WHERE order_date >= DATE '2026-05-01'
  AND order_date < DATE '2026-06-01';
~~~

A B-tree index can support many range predicates. A partitioned table may also prune partitions if its partition key and filter align.

## 15.3 Composite index and leading column

Suppose many queries use both customer_id and order_date.

~~~sql
CREATE INDEX idx_performance_customer_date
ON performance_orders(customer_id, order_date);
~~~

This index orders entries first by customer_id and then by order_date within each customer value.

It can be useful for predicates that constrain the leading column and may also support suitable date conditions.

It is not equivalent to an index on (order_date, customer_id).

After creating the index, inspect plans for the real query patterns and consider the additional storage and write overhead.

## 15.4 Partial index

A partial index stores entries only for rows meeting a predicate.

Example:

~~~sql
CREATE INDEX idx_orders_recent_paid
ON performance_orders(order_date)
WHERE amount > 40000;
~~~

This is a teaching example, not a recommendation for this synthetic table. A partial index is useful only when queries use a predicate compatible with the index condition and the indexed subset is valuable.

Do not create partial indexes without a measured use case.

## 15.5 Index-only scan

An index-only scan can avoid visiting the table heap when the required columns are available from the index and PostgreSQL's visibility information allows it.

For example, an index that includes selected columns may support a narrow lookup:

~~~sql
CREATE INDEX idx_orders_id_covering
ON performance_orders(order_id)
INCLUDE (order_date, amount);
~~~

This is supported by PostgreSQL B-tree indexes. It increases index size and write maintenance.

The optimizer may still choose another plan. Verify with EXPLAIN ANALYZE.

---

# Part 16 — Understand statistics and estimates

The optimizer needs information about data distribution to choose a plan.

## 16.1 What are table statistics?

Statistics summarize properties such as:
- approximate row count;
- value distribution;
- common values;
- distinct value estimates;
- NULL frequency.

PostgreSQL uses statistics when estimating query costs.

## 16.2 Why run ANALYZE?

When a table changes significantly, statistics may no longer describe the current data well.

Run:

~~~sql
ANALYZE performance_orders;
~~~

Expected output:

~~~text
ANALYZE
~~~

ANALYZE updates planner statistics. It does not change the business rows.

## 16.3 Estimated rows versus actual rows

Suppose a plan estimates 100 rows but actually returns 50,000.

The optimizer's assumption is far from the observed result.

Investigate:
- stale statistics;
- uneven distribution;
- correlated columns;
- parameter values;
- filter expressions;
- data changes.

A large estimate error does not automatically tell you which index to create. It tells you where to investigate.

## 16.4 Why a sequential scan can be correct

Imagine a report needs 190,000 of 200,000 rows.

Using an index to find almost every row and then visiting many table pages may cost more than reading the table sequentially.

The optimizer can choose Seq Scan because it estimates that plan to be cheaper.

The goal is not to eliminate sequential scans. The goal is to choose an efficient plan for the workload.

---

# Part 17 — Partitioning lab: inspect which data is read

The earlier section created sales_partitioned with January and February partitions.

## 17.1 Confirm the training rows

~~~sql
SELECT *
FROM sales_partitioned
ORDER BY sale_date;
~~~

Expected result: three rows, two in January and one in February.

## 17.2 Inspect a January-only query

~~~sql
EXPLAIN
SELECT *
FROM sales_partitioned
WHERE sale_date >= DATE '2026-01-01'
  AND sale_date < DATE '2026-02-01';
~~~

Inspect whether the plan references only the January partition.

The exact display varies by PostgreSQL version and planning settings. The important idea is that the date predicate allows PostgreSQL to exclude partitions outside the requested range.

## 17.3 Use a clear half-open date interval

For monthly filters, this pattern is often safer:

~~~sql
WHERE sale_date >= DATE '2026-01-01'
  AND sale_date < DATE '2026-02-01'
~~~

It includes the first day and excludes the next month's first day.

For timestamp columns, this avoids guessing the final time of the month, such as 23:59:59.999.

## 17.4 What if a row has no matching partition?

A partitioned table must have a destination partition for inserted values, unless a suitable DEFAULT partition exists.

If no partition matches, PostgreSQL rejects the insert.

Production partition management must plan future partitions and monitor unexpected date ranges. Do not solve this by creating arbitrary partitions without retention and maintenance rules.

---

# Part 18 — Materialization and freshness decision exercise

A dashboard runs the same expensive daily aggregation many times.

Possible designs:

| Design | Query cost | Freshness | Operational work |
|---|---|---|---|
| View | Recomputes underlying query when read | Current underlying data | Lower storage, query may be expensive |
| Physical table | Reads stored result | Depends on load schedule | Pipeline must rebuild/update |
| Materialized view | Reads stored result | Until refreshed | Refresh schedule and monitoring required |

Ask the business:
- How fresh must the dashboard be?
- How often is the query run?
- How expensive is the aggregation?
- How much storage is acceptable?
- Can users tolerate a refresh window?
- What happens if refresh fails?

Do not materialize a query simply because it is slow. First understand the reason and compare the freshness requirement with the refresh cost.

---

# Part 19 — Performance troubleshooting walkthrough

A dashboard query that used to take 3 seconds now takes 90 seconds.

Do not immediately increase compute or add an index.

Use this sequence.

## Step 1: Confirm the same query and parameters

Check:
- query text;
- date range;
- filter values;
- user parameters;
- dashboard-generated SQL.

A report may be slow because it now requests two years instead of one week.

## Step 2: Confirm data volume

Check:
- row count;
- table growth;
- new partitions;
- file count;
- data skew;
- recent backfill.

The workload may be larger than before.

## Step 3: Inspect the execution plan

Look for:
- sequential scan over a large table;
- unexpected nested loop with many repeated lookups;
- large sort;
- hash operation spilling to disk;
- missing partition pruning;
- estimated versus actual row mismatch;
- expensive function or cast on a filtered column.

These are clues, not automatic diagnoses.

## Step 4: Check joins and grain

A join can multiply rows if the join key is not unique on the expected side.

For example, joining each sales line to multiple active dimension rows can increase row count and distort totals.

Check:
- join key uniqueness;
- table grain;
- filter placement;
- relationship cardinality.

## Step 5: Change one thing

Possible controlled changes:
- add a suitable index;
- rewrite a non-sargable predicate where appropriate;
- add a correct partition filter;
- reduce unnecessary selected columns;
- pre-aggregate only when semantics remain identical;
- refresh statistics;
- materialize a frequently reused result.

## Step 6: Validate the result

Compare:
- output row count;
- distinct business keys;
- total sales;
- date range;
- NULL behavior;
- duplicate count.

A faster query that changes the result is not a successful optimization.

## Step 7: Record and monitor

Record:
- change made;
- plan before and after;
- repeated timings;
- resource/cost observations;
- correctness evidence;
- rollback plan.

Monitor after release. Workloads and data distributions change over time.

---

# Part 20 — Production safety and platform differences

## 20.1 EXPLAIN ANALYZE executes

For SELECT statements, it runs the query and measures it.

For data-changing statements, it can execute the write. Use an isolated environment and understand transaction effects before analyzing DML.

## 20.2 Index creation can affect workloads

On a busy production database, index creation can consume resources and may block or compete with writes depending on the command and platform.

PostgreSQL supports CREATE INDEX CONCURRENTLY for certain production scenarios, but it has its own restrictions and operational behavior. Learn the database version and release procedure before using it.

## 20.3 PostgreSQL is not every cloud warehouse

PostgreSQL concepts such as B-tree indexes and table partitions do not map one-to-one to every cloud analytical platform.

Some cloud warehouses rely more on:
- columnar storage;
- micro-partitions;
- clustering;
- distribution/sort keys;
- automatic statistics;
- elastic compute;
- result caching.

Always read the target platform's execution plan and performance documentation.

## 20.4 Avoid production experiments

Use a development or test dataset for:
- index creation;
- partition changes;
- materialized view refresh experiments;
- query rewrites;
- DDL changes.

Production changes need review, monitoring and a rollback plan.

---

# Part 21 — Additional practice exercises with worked answers

## Exercise 6 — Selective query

A query returns one row from a table containing millions of records. What might help?

**Worked answer:** A suitable index may help locate the row, but inspect the execution plan and confirm that the indexed predicate is selective.

## Exercise 7 — Broad query

A query returns 95% of a table. Must it use an index?

**Worked answer:** No. A sequential scan may be cheaper than many index lookups. Let the optimizer choose based on statistics and verify the plan.

## Exercise 8 — Stale statistics

The table has grown from 10,000 rows to 100 million rows, but estimates remain inaccurate. What should you inspect?

**Worked answer:** Check and refresh planner statistics with ANALYZE, then compare estimated and actual rows. Also inspect data distribution and query predicates.

## Exercise 9 — Partition filter

A table is partitioned by sale_date, but a query filters only by product_id. Will it necessarily prune date partitions?

**Worked answer:** No. The query does not restrict the partition key. It may need to examine many or all partitions unless other constraints or pruning opportunities apply.

## Exercise 10 — Materialized view

The source table receives new rows every minute, but the materialized view refreshes once per day. What is the issue?

**Worked answer:** The result can be stale for most of the day. Confirm whether daily freshness meets the consumer requirement or choose a more frequent/incremental approach.

## Exercise 11 — Wrong result after optimization

A rewritten query runs faster but returns 20% fewer sales rows. Is this success?

**Worked answer:** No. Check filters, joins, NULL behavior and date boundaries. Performance improvement is valid only when the intended result is preserved.

## Exercise 12 — Index overhead

A table receives heavy inserts and updates. What should you consider before adding five indexes?

**Worked answer:** Indexes consume storage and add write/maintenance work. Identify the most valuable query patterns and measure the read benefit against write and storage cost.

## Exercise 13 — Skew

One customer owns 70% of all event records. Why may a distributed aggregation be slow?

**Worked answer:** Work can become concentrated on one partition or worker. Investigate skew and choose a suitable distribution or skew-handling strategy without changing aggregation correctness.

## Exercise 14 — Small files

A lakehouse table has hundreds of thousands of tiny files. What problem can occur?

**Worked answer:** File listing, metadata, scheduling and open costs can dominate. Compaction may help, but choose target file sizes based on the platform and workload.

---

# Part 22 — Day 49 practical completion checklist

### Query plans and measurement
- [ ] Explain what a query plan represents.
- [ ] Run EXPLAIN on a SELECT statement.
- [ ] Run EXPLAIN ANALYZE safely in a training environment.
- [ ] Identify scan and join nodes.
- [ ] Compare estimated and actual rows.
- [ ] Record a repeatable performance baseline.
- [ ] Explain why one timing is not enough.

### Indexes and partitioning
- [ ] Explain B-tree and composite index concepts.
- [ ] Explain why index column order matters.
- [ ] Explain index storage and write overhead.
- [ ] Explain when a sequential scan can be appropriate.
- [ ] Create and inspect a range-partitioned table.
- [ ] Explain partition pruning and half-open date ranges.
- [ ] Explain why partition key selection matters.

### Materialization and distributed concepts
- [ ] Compare a view, table and materialized view.
- [ ] Refresh a materialized view and observe freshness.
- [ ] Explain columnar storage.
- [ ] Explain skew, shuffle and small-file overhead.
- [ ] Explain compute, storage and data movement costs.

### Engineering judgment
- [ ] Change one optimization variable at a time.
- [ ] Compare before-and-after plans and timings.
- [ ] Validate row counts, keys and business totals.
- [ ] Explain why a faster incorrect result is not an optimization.
- [ ] Describe production safety and rollback considerations.

## Final self-assessment

Without looking at the notes, explain:

1. Why is a sequential scan not always bad?
2. What is the difference between EXPLAIN and EXPLAIN ANALYZE?
3. Why can indexes slow down writes?
4. How does partition pruning reduce work?
5. Why must a materialized view be refreshed?
6. What is the difference between a view and a materialized view?
7. What causes data skew and shuffle overhead?
8. What evidence proves an optimization preserved the business result?

If you can answer these questions and perform a basic before-and-after investigation, you have completed the Phase 5 performance fundamentals.

**Phase 5 completion:** You have studied warehouse architecture, dimensional modeling, SCD history, key management, data marts, data quality, reconciliation, metadata, loading strategies and performance fundamentals. The next phase begins with cloud data engineering fundamentals.
