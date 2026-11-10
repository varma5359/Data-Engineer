# Day 25: Database Indexing and Query Optimization
**Phase:** 3 — SQL and Database Engineering  
**Mini-project:** Query Performance Investigation  
**Learning time:** 2–3 hours  
**Audience:** Beginners progressing to junior Data Engineer level  
**SQL dialect:** PostgreSQL for the practical lab; MySQL differences are identified.

---

## 1. Why performance matters

A query that works on 10 rows may become slow when a table grows to 10 million rows.

Imagine TechStore has:
- 10 million orders.
- 50 million order items.
- A dashboard that filters orders by customer and date.
- A nightly pipeline that reads recent orders.

If the database repeatedly checks every row for a small lookup, the query may take longer and use more resources than necessary.

Database indexing and query optimization help us understand and improve these workloads.

### Learning outcomes

By the end, you should be able to:
1. Explain what an index is.
2. Explain how an index helps find rows.
3. Understand the cost of indexes.
4. Create and remove a PostgreSQL index.
5. Understand B-tree indexes at a beginner level.
6. Explain composite and unique indexes.
7. Understand clustered and non-clustered index concepts.
8. Read the basic parts of EXPLAIN output.
9. Distinguish a sequential scan from an index scan.
10. Understand EXPLAIN ANALYZE and its safety.
11. Recognize why an index may not be used.
12. Apply a basic query performance investigation process.

---

## 2. Start with a real-world analogy

Imagine a library with 500,000 books.

Without a catalogue, a librarian may have to inspect many shelves to find one title. A catalogue gives a way to locate the shelf and book faster.

A database index plays a similar role. It is an additional data structure that helps the database locate rows using selected column values.

Important:
- An index is not the table itself.
- An index uses additional storage.
- The database must maintain indexes when rows change.
- An index does not guarantee every query will become faster.

---

## 3. What is a database index?

An index is a data structure maintained by the database to help find rows efficiently.

Example business query:

```sql
SELECT *
FROM orders
WHERE customer_id = 1;
```

If orders contains millions of rows, the database needs a way to find the rows where customer_id is 1.

Without a suitable index, it may scan the table and test each row.

With a suitable index, it may use the index to locate matching row references and fetch the required data.

The optimizer decides which plan is best based on table size, statistics, query conditions and estimated costs.

---

## 4. Understand a sequential scan

A sequential scan reads the table's pages and checks rows against the filter condition.

For a small table, this may be perfectly efficient. Reading a few pages sequentially can be cheaper than using an index and then fetching scattered table rows.

An index is not always better.

### 4.1 Ask the database to show a plan

```sql
EXPLAIN
SELECT *
FROM orders
WHERE customer_id = 1;
```

EXPLAIN displays the plan PostgreSQL expects to use. The exact output depends on table size, indexes, statistics, PostgreSQL version and settings.

You may see a plan node such as Seq Scan on orders. This means PostgreSQL expects to scan the table sequentially.

Do not memorize one exact cost number from a screenshot. Cost estimates are environment-specific.

---

## 5. Create a safe performance lab table

We need enough rows to make performance behavior easier to observe. The existing TechStore training tables contain only a few rows, so a sequential scan may remain the best plan even after an index is created.

Create a separate practice table:

```sql
DROP TABLE IF EXISTS demo_orders;

CREATE TABLE demo_orders (
    order_id BIGINT PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    order_date DATE NOT NULL,
    order_status VARCHAR(20) NOT NULL,
    amount NUMERIC(12, 2) NOT NULL
);
```

This creates an empty table for the performance experiment.

Insert a larger deterministic dataset using PostgreSQL's generate_series:

```sql
INSERT INTO demo_orders (
    order_id,
    customer_id,
    order_date,
    order_status,
    amount
)
SELECT
    n,
    ((n - 1) % 10000) + 1,
    DATE '2026-01-01' + ((n - 1) % 300),
    CASE
        WHEN n % 5 = 0 THEN 'CANCELLED'
        WHEN n % 5 = 1 THEN 'PENDING'
        ELSE 'PAID'
    END,
    ((n % 50000) + 100)::NUMERIC(12, 2)
FROM generate_series(1, 200000) AS n;
```

Expected: INSERT 0 200000.

**Important:** generate_series is PostgreSQL-specific. It generates a sequence of values for this lab. It is not a MySQL function.

### 5.1 Understand the generated values

- n is a number from 1 to 200,000.
- order_id uses n, so every generated order has a unique ID.
- customer_id cycles through 1 to 10,000.
- order_date cycles through 300 dates starting January 1, 2026.
- order_status is selected using n modulo 5.
- amount is a deterministic numeric value based on n.

This data is synthetic training data, not real customer activity.

### 5.2 Verify the row count

```sql
SELECT COUNT(*) AS total_rows
FROM demo_orders;
```

Expected: 200000.

### 5.3 Inspect sample rows

```sql
SELECT *
FROM demo_orders
ORDER BY order_id
LIMIT 5;
```

The first five order IDs should be 1, 2, 3, 4 and 5. Inspect the generated customer IDs, dates and statuses to understand the pattern.

---

## 6. Understand index creation

Create an index on customer_id:

```sql
CREATE INDEX idx_demo_orders_customer_id
ON demo_orders (customer_id);
```

Expected: CREATE INDEX.

Break down the command:
- CREATE INDEX asks PostgreSQL to build an index.
- idx_demo_orders_customer_id is the index name.
- ON demo_orders identifies the table.
- (customer_id) is the indexed column.

The index is separate from the table's rows. It stores an ordered access structure that helps locate rows by customer_id.

### 6.1 Inspect the query plan after creating the index

```sql
EXPLAIN
SELECT *
FROM demo_orders
WHERE customer_id = 4321;
```

The plan may use an index scan or bitmap scan, but it may still choose a sequential scan depending on estimates, cache state, table statistics and settings. Do not force a plan just to obtain a preferred-looking output.

### 6.2 Update statistics

PostgreSQL's planner uses statistics to estimate how many rows match a condition.

```sql
ANALYZE demo_orders;
```

ANALYZE collects statistics for the planner. It does not change business records.

Run EXPLAIN again and compare the plan. The result may still differ by environment.

---

## 7. Understand common index types

### 7.1 B-tree index

B-tree is the common default index type in PostgreSQL. It supports equality comparisons and range comparisons, and can support ordered retrieval.

Examples:
- customer_id = 4321
- order_date >= DATE '2026-03-01'
- amount BETWEEN 1000 AND 5000

B-tree is a strong general-purpose starting point for many transactional lookups.

### 7.2 Hash index

Hash indexes are designed for equality comparisons. They do not provide the same ordering and range-search behavior as B-tree indexes.

For most beginner use cases, learn B-tree first.

### 7.3 Other index types

PostgreSQL also supports specialized index types such as GiST, SP-GiST, GIN and BRIN. They solve particular search problems, such as geometric data, arrays, full-text search or large naturally ordered tables. They are beyond today's main lab.

---

## 8. Composite index: index more than one column

A composite index contains multiple columns.

Example:

```sql
CREATE INDEX idx_demo_orders_customer_date
ON demo_orders (customer_id, order_date);
```

This may help a query such as:

```sql
SELECT order_id, order_date, amount
FROM demo_orders
WHERE customer_id = 4321
  AND order_date >= DATE '2026-03-01'
ORDER BY order_date;
```

Column order matters. A B-tree index on (customer_id, order_date) is generally most useful when queries constrain the leading customer_id column, optionally followed by order_date.

It is not equivalent to two unrelated single-column indexes in every situation.

### 8.1 Leftmost-prefix idea

For an index on (customer_id, order_date):
- customer_id filter: potentially useful.
- customer_id plus order_date filter: potentially useful.
- order_date only: may not efficiently use the index as a normal leading-key lookup.

Always inspect the actual plan and workload.

---

## 9. Unique index

A unique index prevents duplicate indexed key values.

```sql
CREATE UNIQUE INDEX idx_demo_orders_order_id_unique
ON demo_orders (order_id);
```

In this lab, order_id is already a primary key, and PostgreSQL automatically creates a unique index to enforce it. Therefore, creating this extra index is unnecessary and would be redundant. This is an example of why you should inspect existing constraints and indexes before creating new ones.

A UNIQUE constraint is a declarative data-integrity rule. PostgreSQL enforces it using a unique index.

---

## 10. Clustered and non-clustered index concepts

Database products use different terminology and storage structures.

At a conceptual level:
- A clustered organization determines or influences how table data is physically organized around an index key.
- A non-clustered index is a separate structure that points to table rows.

### 10.1 PostgreSQL note

PostgreSQL tables are heap-organized by default. PostgreSQL supports the CLUSTER command to reorder a table once according to an index, but it does not automatically maintain that physical order as rows change. It is not the same as SQL Server's continuously maintained clustered index.

### 10.2 MySQL note

In InnoDB, the table data is organized by the primary key's clustered index. Secondary indexes contain primary-key values used to locate the corresponding clustered row.

Do not assume the terms mean exactly the same implementation across PostgreSQL, MySQL and SQL Server.

---

## 11. EXPLAIN: read the execution plan

EXPLAIN shows the plan the optimizer expects to use.

```sql
EXPLAIN
SELECT *
FROM demo_orders
WHERE customer_id = 4321;
```

Learn to identify:

- Scan type: Seq Scan, Index Scan, Index Only Scan or Bitmap Heap Scan.
- Relation/table: which table is being read.
- Filter: condition applied to rows.
- Estimated rows: number of rows the planner expects.
- Cost: planner's relative cost estimate, not milliseconds.
- Join strategy: nested loop, hash join or merge join for joins.

### 11.1 Sequential Scan

A sequential scan reads table pages and tests rows. It can be the best plan when:
- The table is small.
- The query needs a large percentage of rows.
- The index is not selective.
- The table is cached and sequential reads are cheap.

### 11.2 Index Scan

An index scan uses an index to find row locations and may fetch table rows.

It can be useful for selective filters, but random table access may be expensive when many rows match.

### 11.3 Index Only Scan

An index-only scan can return required values from the index without fetching the table for every row, when the index contains the needed columns and PostgreSQL's visibility information allows it.

### 11.4 Bitmap scan

A bitmap scan can collect matching row locations from an index and then visit table pages more efficiently. It is useful for some medium-selectivity conditions.

Do not decide that one scan type is always best. Compare estimated and actual work.

---

## 12. EXPLAIN ANALYZE: execute and measure

EXPLAIN ANALYZE actually executes the statement and reports measured execution information.

```sql
EXPLAIN ANALYZE
SELECT *
FROM demo_orders
WHERE customer_id = 4321;
```

Look for:
- Actual time.
- Actual rows.
- Loops.
- Rows removed by filter.
- Planning time.
- Execution time.

The exact numbers change across machines, database versions, cache state and concurrent load.

**Safety warning:** EXPLAIN ANALYZE executes the statement. For SELECT, it runs the read query. For INSERT, UPDATE or DELETE, it performs the modification unless you wrap it in a transaction and roll it back. Do not casually run EXPLAIN ANALYZE on production DML.

Safe demonstration for a harmless update in a disposable practice table:

```sql
BEGIN;

EXPLAIN ANALYZE
UPDATE demo_orders
SET order_status = order_status
WHERE order_id = -1;

ROLLBACK;
```

This targets no rows in our generated data, but the key lesson remains: EXPLAIN ANALYZE executes. Use a disposable environment and understand the statement before running it.

---

## 13. Selectivity: why some indexes are useful and others are not

Selectivity describes how narrowly a condition filters the data.

Example:
- customer_id = 4321 may return around 20 rows in this synthetic dataset.
- order_status = 'PAID' returns a large share of the table.

A highly selective condition returns a small fraction of rows. An index is often more useful for selective lookups.

A low-selectivity filter that returns many rows may be cheaper with a sequential scan.

The optimizer estimates selectivity using statistics. It does not choose an index simply because one exists.

---

## 14. Why an index may not be used

Common reasons:
1. The table is small.
2. The condition returns a large percentage of rows.
3. Statistics are stale.
4. The query applies a function or cast that prevents use of a normal index in the expected way.
5. The indexed column is not the leading column in a composite index.
6. The query returns many columns and table access would be expensive.
7. The planner estimates a sequential scan is cheaper.
8. The data distribution is skewed.
9. The query uses a pattern or operator not supported by that index type.

Never create indexes only because a column appears in a WHERE clause. Measure the real workload.

---

## 15. Functions on indexed columns

Consider:

```sql
SELECT *
FROM demo_orders
WHERE EXTRACT(YEAR FROM order_date) = 2026;
```

A normal index on order_date may not be used as effectively because the query applies a function to the column.

A range condition is often more index-friendly:

```sql
SELECT *
FROM demo_orders
WHERE order_date >= DATE '2026-01-01'
  AND order_date < DATE '2027-01-01';
```

This expresses the same calendar-year range for a DATE column and can use a B-tree index on order_date.

PostgreSQL also supports expression indexes when a function-based lookup is a frequent workload:

```sql
CREATE INDEX idx_demo_orders_order_year
ON demo_orders ((EXTRACT(YEAR FROM order_date)));
```

Do not add expression indexes without a measured need. They consume space and add write overhead.

---

## 16. Index costs and trade-offs

Indexes are not free.

Benefits:
- Faster selective reads.
- Faster lookups and joins in suitable workloads.
- Support for uniqueness constraints.
- Can help ORDER BY and range conditions.

Costs:
- Additional disk storage.
- More work during INSERT.
- More work during DELETE.
- More work when indexed columns are updated.
- Maintenance and vacuum considerations.
- Too many indexes can slow write-heavy workloads.

A Data Engineer should balance read speed, write throughput, storage and maintenance.

---

## 17. How to investigate a slow query

Follow this process:

1. Understand the business requirement.
2. Capture the exact SQL query and representative parameters.
3. Confirm table row counts and data distribution.
4. Check existing indexes and constraints.
5. Run EXPLAIN.
6. Run EXPLAIN ANALYZE only in a safe environment.
7. Compare estimated rows with actual rows.
8. Identify expensive scans, joins, sorts or repeated loops.
9. Check whether predicates are selective and index-friendly.
10. Make one change at a time.
11. Compare results and performance.
12. Confirm the query still returns correct data.
13. Document the before/after evidence.

Do not optimize by guessing or by forcing an index without understanding the plan.

---

## 18. Mini-project: Query Performance Investigation

### Business requirement

TechStore's order lookup is slow. Investigate a query filtering demo_orders by customer_id and order_date.

### Step 1: baseline query

```sql
EXPLAIN ANALYZE
SELECT order_id, order_date, amount
FROM demo_orders
WHERE customer_id = 4321
  AND order_date >= DATE '2026-03-01'
ORDER BY order_date;
```

Save the plan output. Record actual execution time and actual rows. Your values will differ from another machine.

### Step 2: inspect existing indexes

In PostgreSQL, you can inspect indexes in psql with:

```text
\d demo_orders
```

In pgAdmin, inspect the table's Indexes section. Alternatively, query PostgreSQL catalog views such as pg_indexes.

### Step 3: create a composite index

```sql
CREATE INDEX idx_demo_orders_customer_date
ON demo_orders (customer_id, order_date);
```

### Step 4: refresh statistics

```sql
ANALYZE demo_orders;
```

### Step 5: run the same query plan again

```sql
EXPLAIN ANALYZE
SELECT order_id, order_date, amount
FROM demo_orders
WHERE customer_id = 4321
  AND order_date >= DATE '2026-03-01'
ORDER BY order_date;
```

Compare:
- Did the scan type change?
- Are actual rows similar?
- Did execution time change?
- Did planning time change?
- Is the result identical?

The planner may choose a different strategy depending on the environment. If it continues to use a sequential scan, that does not automatically mean the index is broken.

### Step 6: test a broad filter

```sql
EXPLAIN ANALYZE
SELECT *
FROM demo_orders
WHERE order_status = 'PAID';
```

A sequential scan may be reasonable because a large fraction of rows can match.

### Step 7: document findings

Create a small report:

| Investigation item | Your finding |
|---|---|
| Query purpose | Orders for one customer after a date |
| Baseline scan type | Record from EXPLAIN |
| Baseline actual rows | Record from EXPLAIN ANALYZE |
| Index created | customer_id, order_date |
| Post-index scan type | Record from EXPLAIN |
| Post-index actual rows | Record from EXPLAIN ANALYZE |
| Result correctness | Same rows before and after? |
| Recommendation | Keep index only if workload benefit justifies its cost |

### Project checklist

- [ ] I created the disposable 200,000-row table.
- [ ] I verified the row count.
- [ ] I ran EXPLAIN before creating the composite index.
- [ ] I created the index.
- [ ] I ran ANALYZE.
- [ ] I ran EXPLAIN ANALYZE after the index.
- [ ] I compared actual rows and execution time.
- [ ] I tested a broad filter.
- [ ] I understand why the optimizer may choose a sequential scan.
- [ ] I documented the index's read benefit and write/storage cost.

---

## 19. Common beginner mistakes

1. Assuming every index makes every query faster.
2. Creating indexes without checking existing primary-key or unique indexes.
3. Ignoring the order of columns in a composite index.
4. Assuming a sequential scan is always bad.
5. Comparing EXPLAIN cost numbers as if they were milliseconds.
6. Treating EXPLAIN and EXPLAIN ANALYZE as identical.
7. Forgetting that EXPLAIN ANALYZE executes the statement.
8. Running EXPLAIN ANALYZE for production UPDATE or DELETE without a safety plan.
9. Ignoring stale statistics.
10. Creating too many indexes on a write-heavy table.
11. Assuming PostgreSQL clustered behavior is the same as InnoDB.
12. Forcing a plan before understanding the workload.

---

## 20. Practice questions

1. What is an index?
2. How does an index help a query?
3. What is a sequential scan?
4. What is a B-tree index?
5. What is a composite index?
6. Why does column order matter in a composite index?
7. What is selectivity?
8. Why might a database ignore an index?
9. What does EXPLAIN show?
10. What does EXPLAIN ANALYZE do?
11. Why is EXPLAIN ANALYZE risky for DML?
12. What is the difference between estimated rows and actual rows?
13. What does ANALYZE do in PostgreSQL?
14. What are the costs of maintaining indexes?
15. How do PostgreSQL and MySQL differ in clustered storage concepts?
16. How would you investigate a slow customer/date query?

### Answer hints

1. An auxiliary data structure to locate rows.
2. It can avoid scanning every row for selective lookups.
3. A scan of table pages and rows.
4. A common ordered index structure supporting equality and range searches.
5. An index containing more than one key column.
6. The leading column determines many efficient lookup patterns.
7. How narrowly a condition filters data.
8. Cost estimates may favor another plan, or the index may not match the predicate.
9. The optimizer's estimated execution plan.
10. Executes the query and reports actual measurements.
11. It performs the modification unless safely rolled back.
12. Planner estimate versus observed execution.
13. Collects statistics for the planner.
14. Storage and additional write/maintenance work.
15. InnoDB uses a primary-key clustered organization; PostgreSQL heap tables are not automatically maintained in clustered order.
16. Capture query, inspect plan, statistics, indexes, predicates, compare measured results and validate correctness.

---

## 21. Day 25 revision

- An index is an additional data structure for locating rows.
- B-tree is a common general-purpose index.
- Composite index column order matters.
- Indexes use storage and add write cost.
- Sequential scans can be the correct plan.
- EXPLAIN shows estimated plans.
- EXPLAIN ANALYZE executes and measures.
- ANALYZE refreshes PostgreSQL planner statistics.
- Selectivity affects plan choice.
- Query functions and casts can affect index use.
- PostgreSQL and MySQL storage/index implementations differ.
- Measure a real workload before changing indexes.

**You are ready for Day 26 when you can explain why an index may help, read a basic plan and describe the trade-off between faster reads and slower writes.**
