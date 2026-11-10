# Day 23: SQL Window Functions

**Pathway:** TechPhile.ai — Data Engineer Career Pathway  
**Phase:** 3 — SQL and Database Engineering  
**Audience:** Freshers, college students and junior Data Engineer candidates  
**Level:** Beginner to junior | **Class time:** 2–3 hours  
**Approved syllabus:** ROW_NUMBER, RANK, DENSE_RANK, LAG, LEAD, PARTITION BY, ORDER BY, analytical queries  
**Mini-project:** Historical Trend Query Engine  
**Primary database:** PostgreSQL; MySQL differences are identified where relevant.

---

## 1. How to study

Read one concept, predict what the query or program will do, execute the small example, compare the result and explain it aloud. Do not run the entire lesson as one large script.

## 2. Prerequisite and learning outcomes

Use the TechStore training database from Day 20: customers, products, orders and order_items. Confirm you are connected to the correct practice database. If sample data differs, inspect it before comparing outputs. Never experiment on production data.

By the end, you should understand each item in today's syllabus, execute the main examples, explain the output, recognize common mistakes and complete **Historical Trend Query Engine**.

## 3. Why window functions?

GROUP BY collapses detail into groups. A window function calculates across related rows but keeps each input row visible. Use it for rankings, previous/next values, running totals and customer-specific sequences.

Prepare order totals:
```sql
WITH order_totals AS (
 SELECT o.order_id,o.customer_id,o.order_date,
        SUM(oi.quantity*oi.unit_price) AS total
 FROM orders o JOIN order_items oi ON oi.order_id=o.order_id
 GROUP BY o.order_id,o.customer_id,o.order_date
)
SELECT * FROM order_totals ORDER BY order_date,order_id;
```
Expected: 5001/2600, 5002/8500, 5003/3000.

## 4. OVER and PARTITION BY

OVER defines the window. PARTITION BY divides rows into independent groups; ORDER BY defines order inside the window; a frame can narrow rows around the current row. Window ORDER BY does not guarantee final output order—add an outer ORDER BY.

```sql
WITH totals AS (
 SELECT o.order_id,o.customer_id,o.order_date,
        SUM(oi.quantity*oi.unit_price) AS total
 FROM orders o JOIN order_items oi ON oi.order_id=o.order_id
 GROUP BY o.order_id,o.customer_id,o.order_date
)
SELECT customer_id,order_id,
 ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY order_date,order_id) AS seq
FROM totals ORDER BY customer_id,order_date;
```
Expected Asha 5001=1, Asha 5003=2, Ravi 5002=1. Numbering restarts per customer. PARTITION BY does not sort final output.

## 5. ROW_NUMBER, RANK, DENSE_RANK

ROW_NUMBER gives unique sequence numbers. RANK gives tied rows the same rank and leaves gaps (1,1,3). DENSE_RANK gives tied rows the same rank without gaps (1,1,2).

```sql
WITH demo(product_name,sales) AS (
 VALUES ('Keyboard',100),('Mouse',100),('Cable',80)
)
SELECT product_name,sales,
 ROW_NUMBER() OVER(ORDER BY sales DESC,product_name) AS rn,
 RANK() OVER(ORDER BY sales DESC) AS rnk,
 DENSE_RANK() OVER(ORDER BY sales DESC) AS drnk
FROM demo ORDER BY sales DESC,product_name;
```
Expected Keyboard 1/1/1, Mouse 2/1/1, Cable 3/3/2. Ranking functions intentionally order only by sales so equal sales tie. The row number adds product_name as tie-breaker.

## 6. LAG and LEAD

LAG reads a previous row; LEAD reads a following row:
```sql
WITH totals AS (
 SELECT o.order_id,o.order_date,
        SUM(oi.quantity*oi.unit_price) AS total
 FROM orders o JOIN order_items oi ON oi.order_id=o.order_id
 GROUP BY o.order_id,o.order_date
)
SELECT order_id,total,
 LAG(total) OVER(ORDER BY order_date,order_id) AS previous_total,
 LEAD(total) OVER(ORDER BY order_date,order_id) AS next_total
FROM totals ORDER BY order_date,order_id;
```
Expected previous: NULL, 2600, 8500. Next: 8500, 3000, NULL. First row has no previous; last has no next.

Use PARTITION BY customer_id with LAG to compare each customer's own order history.

## 7. Running total and window frame

```sql
WITH totals AS (
 SELECT o.order_id,o.order_date,
        SUM(oi.quantity*oi.unit_price) AS total
 FROM orders o JOIN order_items oi ON oi.order_id=o.order_id
 GROUP BY o.order_id,o.order_date
)
SELECT order_id,total,
 SUM(total) OVER(ORDER BY order_date,order_id
 ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_sales
FROM totals ORDER BY order_date,order_id;
```
Expected running sales: 2600, 11100, 14100. Frame begins at first row and ends at current row. Explicit ROWS framing clarifies row-by-row accumulation.

## 8. GROUP BY versus window functions

GROUP BY produces summary rows; window functions preserve detail and add calculations. Use GROUP BY for one row per group, windows when you need each original row plus ranking or neighboring-row values.

## 9. Mini-project — Historical Trend Query Engine

Create queries to number customer orders chronologically, rank order totals, demonstrate tied ranks, calculate LAG/LEAD, create running totals and explain boundary NULL values.

**Deliverables:** SQL, outputs and explanations of OVER, PARTITION BY, ROW_NUMBER, RANK, DENSE_RANK, LAG and LEAD.

## 10. Mistakes and self-check

Avoid forgetting ORDER BY inside OVER, assuming PARTITION BY sorts results, confusing ranking functions, or filtering window output in WHERE at the same query level. Put window calculations in a CTE/subquery, then filter outside.

Explain window versus GROUP BY, ties, LAG/LEAD, frame and final ORDER BY.

## 99. Next lesson

Next: Day 24 — Transactions and ACID Principles.