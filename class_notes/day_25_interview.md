# Day 25 Interview Preparation: Database Indexing and Query Optimization

**Phase:** 3 — SQL and Database Engineering  
**Level:** Beginner to junior Data Engineer  
**Mini-project:** Query Performance Investigation  
**Database:** PostgreSQL, with MySQL concepts compared

---

## 1. Core questions with complete answers

### 1. What is a database index?

An index is an additional data structure maintained by the database to help locate rows efficiently. It can reduce the amount of table data that must be inspected for suitable queries. It consumes storage and adds work to data modifications.

### 2. Why do we need indexes?

Indexes can speed up selective lookups, joins, range filters and some ordered queries. They are especially useful when a table is large and a query needs a small subset of rows.

### 3. Is an index always faster?

No. A sequential scan may be cheaper for a small table or a query that returns a large percentage of rows. The optimizer compares estimated costs and chooses a plan.

### 4. What is a sequential scan?

A sequential scan reads table pages and checks rows against the query condition. It is often efficient for small tables or broad queries.

### 5. What is a B-tree index?

B-tree is a common ordered index structure. It supports equality lookups, range comparisons and ordered access patterns. It is the default PostgreSQL index type for many common use cases.

### 6. What is a hash index?

A hash index is designed for equality comparisons. It does not provide the same ordered range access as B-tree. The correct index type depends on the operators and workload.

### 7. What is a composite index?

A composite index contains multiple columns, for example (customer_id, order_date). It can help queries that filter by the leading customer_id and optionally by order_date. Column order matters.

### 8. What is the leftmost-prefix concept?

For a B-tree index on (customer_id, order_date), a query filtering by customer_id can generally use the leading index key. A query filtering only by order_date may not use the index as effectively as a leading-key lookup.

### 9. What is a unique index?

A unique index enforces uniqueness of the indexed key values. In PostgreSQL, a PRIMARY KEY or UNIQUE constraint is backed by a unique index. Avoid creating redundant indexes that duplicate an existing constraint index.

### 10. What is a clustered index?

The term describes an index organization tied to how table rows are stored. The exact implementation differs by database. In MySQL InnoDB, the primary key defines the clustered organization. PostgreSQL heap tables are not automatically maintained in clustered order.

### 11. What is a non-clustered index?

Conceptually, it is a separate index structure that points to table rows or their keys. Product terminology and implementation vary. For example, InnoDB secondary indexes contain primary-key values used to find the clustered row.

### 12. What does EXPLAIN do?

EXPLAIN shows the query plan the optimizer expects to use. It includes plan nodes, estimated rows and relative cost. It does not normally execute the SELECT query to collect actual runtime measurements.

### 13. What does EXPLAIN ANALYZE do?

EXPLAIN ANALYZE executes the statement and reports actual runtime information such as actual time, actual rows and loops. It is useful for comparing estimates with reality.

### 14. Why is EXPLAIN ANALYZE dangerous for DML?

It executes the statement. An UPDATE, INSERT or DELETE can modify data. Use a disposable environment or a carefully controlled transaction that is rolled back. Never assume EXPLAIN ANALYZE only previews a modification.

### 15. What is the difference between estimated and actual rows?

Estimated rows are the optimizer's prediction. Actual rows are what the executed plan produced. Large differences can indicate stale statistics, skewed data or assumptions that do not match the workload.

### 16. What does ANALYZE do in PostgreSQL?

ANALYZE collects statistics about table contents and stores them for the query planner. Better statistics can help the optimizer estimate row counts and choose suitable plans.

### 17. What is selectivity?

Selectivity describes how much a filter narrows the data. A condition matching 20 rows from 200,000 is highly selective. A condition matching 150,000 rows is much less selective.

### 18. Why might PostgreSQL ignore an index?

The table may be small, the filter may match many rows, statistics may favor a sequential scan, the index may not match the predicate, or table access through the index may cost more. The optimizer chooses the lowest estimated-cost plan.

### 19. What is an index scan?

An index scan uses an index to find matching row locations and may then fetch table rows. It can be effective for selective conditions.

### 20. What is an index-only scan?

An index-only scan can return required values from the index without fetching every table row, when the index contains the necessary columns and PostgreSQL's visibility map permits it.

### 21. What is a bitmap heap scan?

It uses an index to collect matching row locations and then visits table pages in a grouped way. It can be effective when more rows match than a very selective index scan but a full sequential scan is not ideal.

### 22. What are the disadvantages of indexes?

Indexes consume storage, add work to INSERT/UPDATE/DELETE, require maintenance and can slow write-heavy workloads when too many are created.

### 23. Why does composite index order matter?

The leading columns determine which lookup patterns can efficiently use the index. For an index on (customer_id, order_date), customer_id equality followed by a date range is a common useful pattern.

### 24. What is a covering index?

A covering index contains all columns required by a query's filters and output, potentially allowing the database to satisfy the query from the index alone. PostgreSQL supports INCLUDE columns in B-tree indexes for this purpose.

### 25. What is an expression index?

An expression index stores the result of an expression on a column. It can support frequent queries that filter using that same expression. It adds storage and write cost and should be justified by a real workload.

### 26. Why can a function on a column affect index use?

A normal index on order_date is ordered by the raw date values. A condition such as EXTRACT(YEAR FROM order_date) applies a function. The optimizer may not use the normal index in the same way. A range predicate or expression index may be more suitable.

### 27. What is query optimization?

Query optimization is the process of improving query performance while preserving correctness. It includes understanding the workload, inspecting plans, using appropriate indexes, reducing unnecessary data and improving query structure.

### 28. How would you investigate a slow query?

Capture the exact SQL and parameters, understand the expected output, inspect table sizes and indexes, run EXPLAIN, use EXPLAIN ANALYZE safely, compare estimates with actual rows, identify expensive plan nodes, make one change at a time and validate identical results.

### 29. What is a query planner?

The query planner evaluates possible execution strategies and estimates their cost. It chooses a plan based on query structure, indexes, statistics and configuration.

### 30. What is a query execution plan?

It is the sequence of operations the database intends to use, such as scans, joins, sorts and aggregates. EXPLAIN displays this plan.

### 31. What is the difference between planning time and execution time?

Planning time is the time spent selecting the plan. Execution time is the time spent running it. EXPLAIN ANALYZE reports both in PostgreSQL.

### 32. Does lower EXPLAIN cost always mean lower real execution time?

Not necessarily. Cost is an internal relative estimate, not a time measurement. Compare actual runtime under representative conditions and validate results.

### 33. Why should indexes not be created on every column?

Each index consumes space and adds maintenance work to writes. Unnecessary indexes can slow ingestion and updates. Create indexes based on actual query patterns and measured benefits.

### 34. What is index selectivity?

It refers to how well an index key distinguishes rows for a query. A highly selective condition identifies a small subset, which can make an index access path attractive.

### 35. What is a partial index in PostgreSQL?

A partial index includes only rows satisfying a predicate. For example, an index on recent or active records can be smaller if the query consistently filters on that predicate. The query condition must match the partial-index predicate sufficiently for the planner to use it.

### 36. What is index maintenance?

It includes work required to keep indexes consistent with table changes, plus operations such as REINDEX when appropriate and monitoring index size/bloat. PostgreSQL also relies on vacuuming to manage dead tuples and visibility information.

---

## 2. Scenario-based interview questions

### Scenario 1: A customer lookup is slow on a large orders table

**Good answer:** I would inspect the exact query and its plan, verify the customer_id filter is selective, check existing indexes and statistics, then test a B-tree index on customer_id if justified. I would compare EXPLAIN ANALYZE results before and after and verify the result set is unchanged.

### Scenario 2: A query filters by customer_id and order_date

**Good answer:** I would inspect the query pattern and consider a composite B-tree index on (customer_id, order_date). The column order supports customer equality followed by a date range. I would validate the plan and workload rather than assume the index is always beneficial.

### Scenario 3: The planner chooses a sequential scan despite an index

**Good answer:** I would check table size, filter selectivity, statistics, estimated versus actual rows and whether the predicate matches the index. A sequential scan can be correct when many rows match or the table is small.

### Scenario 4: A query became slow after adding several indexes

**Good answer:** Indexes may have increased write cost, storage pressure and maintenance overhead. I would inspect index usage and workload, identify redundant or unused indexes and remove only after verifying they are not needed by constraints or critical queries.

### Scenario 5: EXPLAIN ANALYZE shows actual rows far above estimates

**Good answer:** I would inspect statistics freshness, data skew, correlated columns and predicate selectivity. I would run ANALYZE and consider extended statistics or query changes where appropriate. Then I would compare the plan again.

### Scenario 6: A report filters on EXTRACT(YEAR FROM order_date)

**Good answer:** I would consider a half-open date range from the start of the year to the start of the next year, which can use a normal B-tree index on order_date. If the expression is a frequent standard access pattern, an expression index may be considered.

### Scenario 7: A nightly pipeline has become slower after index creation

**Good answer:** Indexes add work to inserts and updates. I would measure load throughput, inspect which indexes are necessary for downstream queries and uniqueness constraints, and balance read benefits against ingestion cost. I would not drop a constraint-backed index without redesigning integrity protection.

---

## 3. Mini-project defence

Be ready to explain:

1. Why did you create a 200,000-row practice table?
2. What query did you measure?
3. What did the baseline EXPLAIN plan show?
4. What index did you create and why that column order?
5. Did the plan change after ANALYZE?
6. What were the actual rows and execution time?
7. Why can the optimizer still choose a sequential scan?
8. What is the cost of keeping the index?
9. How would you test the index on a production-like workload?
10. Why must EXPLAIN ANALYZE be used carefully with DML?

---

## 4. Common interview traps

- Claiming an index guarantees faster performance.
- Treating EXPLAIN cost as milliseconds.
- Forgetting EXPLAIN ANALYZE executes the statement.
- Ignoring index write and storage costs.
- Assuming all databases implement clustered indexes identically.
- Creating duplicate indexes for primary keys.
- Ignoring composite index column order.
- Forcing index use without investigating estimates and actual rows.
- Optimizing a query without validating that the results remain correct.

---

## 5. Readiness checklist

- [ ] I can explain indexes and B-tree.
- [ ] I understand composite index column order.
- [ ] I can explain sequential scan and index scan.
- [ ] I can read basic EXPLAIN output.
- [ ] I know EXPLAIN ANALYZE executes the query.
- [ ] I understand selectivity and planner statistics.
- [ ] I can explain why an index may not be used.
- [ ] I understand index storage and write overhead.
- [ ] I can compare PostgreSQL and InnoDB clustered storage concepts.
- [ ] I can describe a measured query optimization investigation.
