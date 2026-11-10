# Day 25: Database Indexing and Query Optimization

**Pathway:** TechPhile.ai — Data Engineer Career Pathway  
**Phase:** 3 — SQL and Database Engineering  
**Audience:** Freshers, college students and junior Data Engineer candidates  
**Level:** Beginner to junior | **Class time:** 2–3 hours  
**Approved syllabus:** Indexes, clustered and non-clustered index concepts, query execution plans, EXPLAIN, query performance  
**Mini-project:** Query Performance Investigation  
**Primary database:** PostgreSQL; MySQL differences are identified where relevant.

---

## 1. How to study

Read one concept, predict what the query or program will do, execute the small example, compare the result and explain it aloud. Do not run the entire lesson as one large script.

## 2. Prerequisite and learning outcomes

Use the TechStore training database from Day 20: customers, products, orders and order_items. Confirm you are connected to the correct practice database. If sample data differs, inspect it before comparing outputs. Never experiment on production data.

By the end, you should understand each item in today's syllabus, execute the main examples, explain the output, recognize common mistakes and complete **Query Performance Investigation**.

## 3. What is an index?

An index is a data structure that helps locate rows without scanning every table row. Like a library catalogue, it can speed up selective reads. It uses storage and adds work to INSERT, UPDATE and DELETE.

An index does not guarantee a faster query; optimizer decisions depend on table size, statistics, selectivity and estimated cost.

## 4. EXPLAIN

```sql
EXPLAIN SELECT * FROM orders WHERE customer_id=1;
```
EXPLAIN shows planned operations, not actual elapsed time. A tiny three-row table may use Seq Scan because it is cheaper.

Plan terms:
- Seq Scan: scan table.
- Index Scan: use index to locate rows.
- Index Only Scan: return from index when visibility allows.
- Bitmap Index/Heap Scan: identify matching pages then fetch them.
- Sort, Hash Join, Nested Loop: plan operations.
- cost is planner units, not milliseconds; rows are estimated counts.

## 5. Create an index

```sql
CREATE INDEX idx_orders_customer ON orders(customer_id);
EXPLAIN SELECT * FROM orders WHERE customer_id=1;
```
The tiny table may still use Seq Scan. This is normal. B-tree is a common default for equality/range lookups.

## 6. EXPLAIN ANALYZE

```sql
EXPLAIN (ANALYZE,BUFFERS)
SELECT * FROM orders WHERE customer_id=1;
```
EXPLAIN ANALYZE executes the query and reports actual timing/rows. BUFFERS reports buffer activity. It executes DML too; do not run destructive ANALYZE on production casually. Compare estimated and actual rows, buffers and repeated measurements.

## 7. Larger disposable lab

PostgreSQL example:
```sql
CREATE TABLE index_orders_lab AS
SELECT g AS order_id,(g%1000)+1 AS customer_id,
       DATE '2026-01-01'+(g%300) AS order_date,
       (g%5000)/100.0 AS order_total
FROM generate_series(1,100000) AS g;
SELECT COUNT(*) FROM index_orders_lab;
```
Expected 100000.

Baseline and index:
```sql
EXPLAIN (ANALYZE,BUFFERS)
SELECT * FROM index_orders_lab WHERE customer_id=777;
CREATE INDEX idx_lab_customer ON index_orders_lab(customer_id);
EXPLAIN (ANALYZE,BUFFERS)
SELECT * FROM index_orders_lab WHERE customer_id=777;
```
Record plan, actual rows, buffers and timing. Planner may still choose Seq Scan based on data distribution and cost.

## 8. Composite, covering and partial indexes

```sql
CREATE INDEX idx_lab_customer_date
ON index_orders_lab(customer_id,order_date);
```
A B-tree's leading-column order matters. (customer_id,order_date) is not interchangeable with (order_date,customer_id).

PostgreSQL INCLUDE:
```sql
CREATE INDEX idx_lab_customer_cover
ON index_orders_lab(customer_id) INCLUDE(order_date,order_total);
```
May enable index-only access when visibility permits. Do not include every column.

Partial index:
```sql
CREATE INDEX idx_placed_orders_date ON orders(order_date)
WHERE order_status='PLACED';
```
Indexes only rows matching condition; useful only if workload justifies it. A UNIQUE constraint already creates a unique index in PostgreSQL; avoid duplicating it.

## 9. Clustered and non-clustered concepts

PostgreSQL uses heap-organized tables. CLUSTER can rewrite table in index order, but later writes do not maintain that ordering automatically. MySQL InnoDB organizes table data around primary key (clustered index); secondary indexes carry primary-key values. Terminology differs by engine.

## 10. Optimization workflow

Capture representative query; run safe plan; compare estimated/actual rows; inspect joins, filters and sorts; check selectivity/indexes; change one thing; repeat same workload; verify correctness; document read benefit, write cost and storage. Keep statistics current (PostgreSQL ANALYZE; autovacuum manages routine maintenance).

## 11. Mini-project — Query Performance Investigation

Use index_orders_lab. Record baseline plan, add single-column index, compare, add composite index, compare filter patterns, record estimates/actuals/buffers/time, explain why sequential scan may remain best, and document write/storage costs.

Cleanup only disposable lab:
```sql
DROP TABLE index_orders_lab;
```

**Deliverables:** SQL, before/after plans, performance report, index justification.

## 12. Self-check

Explain index, scan types, EXPLAIN vs EXPLAIN ANALYZE, selectivity, composite order, partial/covering indexes, index write costs and PostgreSQL/MySQL clustering differences.

## 99. Next lesson

Next: Day 26 — Database Normalization.