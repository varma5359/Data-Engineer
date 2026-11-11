# Day 49 Interview Guide — Warehouse Performance Optimization

## 1. What is query performance optimization?

It is improving query response time and resource use while preserving the intended business result. It includes understanding workload, measuring a baseline, inspecting execution plans, changing one thing at a time and validating correctness.

## 2. What is EXPLAIN?

EXPLAIN shows the database's planned execution operations, such as scans, joins, sorts and estimated rows/cost. In PostgreSQL, plain EXPLAIN does not execute the SELECT query.

## 3. What is EXPLAIN ANALYZE?

EXPLAIN ANALYZE executes the query and reports actual execution information. It is useful for comparing estimates with reality. Because it executes the statement, do not casually use it on production INSERT, UPDATE or DELETE operations.

## 4. What is a sequential scan?

A sequential scan reads table rows sequentially. It is not automatically inefficient. If a query needs most of the table, a sequential scan may be cheaper than many index lookups.

## 5. What is an index?

An index is a data structure that helps locate rows using selected column values. It can improve selective reads but consumes storage and adds maintenance work to inserts, updates and deletes.

## 6. What is a composite index?

A composite index contains multiple columns. Its column order matters and should reflect actual filter and join patterns. An index on (customer_id, order_date) is not identical to one on (order_date, customer_id).

## 7. What is partitioning?

Partitioning divides a logical table into smaller physical partitions based on a partition key such as date. Queries that filter appropriately may benefit from partition pruning, which avoids scanning irrelevant partitions.

## 8. Is partitioning a replacement for indexing?

No. Partitioning divides data into partitions. Indexing helps locate rows within data. They solve related but different problems and can be used together depending on platform and workload.

## 9. What is partition pruning?

Partition pruning is the process of excluding partitions that cannot contain rows matching the query predicate. The partition key and query filter must align for pruning to work.

## 10. What is a materialized view?

A materialized view stores the result of a query. It can speed repeated expensive reads but must be refreshed, uses storage and may be stale between refreshes.

## 11. View versus materialized view?

A regular view stores a query definition and reads current underlying data when queried. A materialized view stores query results and requires refresh. The choice depends on query cost, freshness and storage.

## 12. What is data skew?

Data skew occurs when data is unevenly distributed across partitions or keys. Some workers may process much more data than others, slowing distributed operations. The remedy depends on the platform and workload.

## 13. What is shuffle?

Shuffle is data movement across workers, often for distributed joins or aggregations. Large shuffles can increase network traffic, disk spill and execution time.

## 14. What is the small-file problem?

Many tiny files create metadata, listing, scheduling and open-file overhead in lake and lakehouse systems. Compaction can combine files, but file sizing should follow platform and workload guidance.

## 15. How do you optimize a slow query?

I first confirm the business scope and capture a baseline. I inspect the plan, row estimates, scan and join operations, data distribution and filters. I change one factor, measure again under comparable conditions, and validate that row counts, keys and totals remain correct. I also consider write, storage and compute costs.

## 16. Why not create indexes on every column?

Indexes consume storage and add write and maintenance costs. Some indexes are not useful for the actual query patterns. I create indexes based on measured workload and execution plans.

## 17. Scenario: index exists but query still uses a sequential scan.

That can be correct. The query may return a large portion of the table, the index may not match the predicate, statistics may be stale, or the optimizer may estimate a sequential scan as cheaper. I would inspect the plan and data distribution rather than force an index blindly.

## 18. Scenario: a monthly report reads five years of data.

I would inspect whether date partitioning and partition pruning fit the workload, ensure the query filters on the partition key, and consider pre-aggregated or materialized models if the same aggregation is repeatedly used. I would measure storage, refresh and compute trade-offs.

## 19. Scenario: query is faster but total sales changed.

The optimization is not acceptable until the difference is explained. I would compare filters, joins, grain, null behavior and output keys, then reconcile totals. A faster incorrect query is a defect.

## 20. Explain your Day 49 performance audit.

I created a synthetic training table, captured a baseline query plan, added an index, inspected the new plan, tested date partition pruning and compared view/table/materialized-view trade-offs. I recorded actual environment results and validated query correctness rather than claiming a universal speedup.

## Interview traps

- Saying every sequential scan is bad.
- Claiming an index always improves performance.
- Running EXPLAIN ANALYZE on production DML without considering execution.
- Assuming partitioning makes every query faster.
- Forgetting materialized-view freshness.
- Claiming a speedup without a baseline.
- Ignoring storage and write costs.
- Changing query semantics while calling it optimization.

## Readiness checklist

- [ ] Explain plans and scan types.
- [ ] Explain index and partition trade-offs.
- [ ] Explain materialized view refresh.
- [ ] Describe skew, shuffle and small files.
- [ ] Explain a measured before/after investigation.
- [ ] Defend correctness and cost validation.
