# Day 22: SQL Aggregations and Advanced Queries

**Pathway:** TechPhile.ai — Data Engineer Career Pathway  
**Phase:** 3 — SQL and Database Engineering  
**Audience:** Freshers, college students and junior Data Engineer candidates  
**Level:** Beginner to junior | **Class time:** 2–3 hours  
**Approved syllabus:** GROUP BY, HAVING, aggregate functions, subqueries, correlated subqueries, CTEs  
**Mini-project:** Business Data Aggregation  
**Primary database:** PostgreSQL; MySQL differences are identified where relevant.

---

## 1. How to study

Read one concept, predict what the query or program will do, execute the small example, compare the result and explain it aloud. Do not run the entire lesson as one large script.

## 2. Prerequisite and learning outcomes

Use the TechStore training database from Day 20: customers, products, orders and order_items. Confirm you are connected to the correct practice database. If sample data differs, inspect it before comparing outputs. Never experiment on production data.

By the end, you should understand each item in today's syllabus, execute the main examples, explain the output, recognize common mistakes and complete **Business Data Aggregation**.

## 3. Why aggregate data?

A manager does not want every order item. They ask for sales per order, sales per customer, average product price or customers with multiple orders. Aggregation turns detail into business summaries.

## 4. Aggregate functions

```sql
SELECT order_item_id,quantity,unit_price,
       quantity*unit_price AS line_total
FROM order_items ORDER BY order_item_id;
```
Expected line totals: 1200, 1400, 8500, 3000.

```sql
SELECT SUM(quantity*unit_price) AS total_sales FROM order_items;
```
Expected 14100. SUM adds non-NULL values; if there are no non-NULL values it returns NULL.

```sql
SELECT COUNT(*) AS rows,COUNT(product_id) AS non_null_products
FROM order_items;
```
Expected 4 and 4. COUNT(*) counts rows; COUNT(column) counts non-NULL values.

```sql
SELECT AVG(price),MIN(price),MAX(price) FROM products;
```
Expected average 2975, minimum 700, maximum 8500. AVG ignores NULL.

## 5. GROUP BY

```sql
SELECT order_id,SUM(quantity*unit_price) AS order_total
FROM order_items GROUP BY order_id ORDER BY order_id;
```
Expected: 5001=2600, 5002=8500, 5003=3000. GROUP BY forms one group per key; aggregates run inside each group. Selected non-aggregated columns generally must be grouped (PostgreSQL has functional-dependency exceptions).

## 6. WHERE versus HAVING

WHERE filters source rows before grouping; HAVING filters groups after aggregation.
```sql
SELECT order_id,SUM(quantity*unit_price) AS total
FROM order_items GROUP BY order_id
HAVING SUM(quantity*unit_price)>2500 ORDER BY total DESC;
```
Expected all three orders: 8500, 3000, 2600.

```sql
SELECT customer_id,COUNT(*) AS shipped_count
FROM orders WHERE order_status='SHIPPED'
GROUP BY customer_id HAVING COUNT(*)>1;
```
Expected zero rows: only one shipped order exists.

## 7. Join then aggregate

```sql
SELECT c.customer_name,SUM(oi.quantity*oi.unit_price) AS sales
FROM customers c JOIN orders o ON o.customer_id=c.customer_id
JOIN order_items oi ON oi.order_id=o.order_id
GROUP BY c.customer_id,c.customer_name ORDER BY sales DESC;
```
Expected Ravi 8500, Asha 5600. Meena is absent due to INNER JOIN.

Include customers with zero:
```sql
SELECT c.customer_name,
       COALESCE(SUM(oi.quantity*oi.unit_price),0) AS sales
FROM customers c LEFT JOIN orders o ON o.customer_id=c.customer_id
LEFT JOIN order_items oi ON oi.order_id=o.order_id
GROUP BY c.customer_id,c.customer_name ORDER BY c.customer_id;
```
Expected Asha 5600, Ravi 8500, Meena 0. COALESCE returns first non-NULL value.

## 8. Subqueries

A subquery is a query inside another statement. Scalar subquery returns one value:
```sql
SELECT product_name,price FROM products
WHERE price>(SELECT AVG(price) FROM products);
```
Expected Monitor only (8500).

IN checks membership:
```sql
SELECT customer_id,customer_name FROM customers
WHERE customer_id IN (SELECT customer_id FROM orders);
```
Expected Asha and Ravi.

EXISTS tests whether at least one row matches:
```sql
SELECT c.customer_name FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id=c.customer_id);
```
Expected Asha and Ravi. SELECT 1 is conventional because existence matters, not returned values.

## 9. Correlated subquery

The inner query references an outer-query column:
```sql
SELECT c.customer_name FROM customers c
WHERE (SELECT COUNT(*) FROM orders o
       WHERE o.customer_id=c.customer_id)>0;
```
Expected Asha and Ravi. The reference to c.customer_id makes it correlated. The optimizer may transform execution; compare plans and alternatives for large datasets.

## 10. CTE

WITH names a temporary query result for one statement:
```sql
WITH order_totals AS (
 SELECT order_id,SUM(quantity*unit_price) AS total
 FROM order_items GROUP BY order_id
)
SELECT order_id,total FROM order_totals
WHERE total>2500 ORDER BY total DESC;
```
Expected 5002/8500, 5003/3000, 5001/2600. A CTE is not a permanent table. Chain multiple CTEs for readable steps.

## 11. Aggregation errors

- Counting after one-to-many joins without checking duplication.
- Joining multiple detail tables and multiplying totals.
- Using WHERE for aggregate filters instead of HAVING.
- Forgetting COUNT(column), SUM and AVG NULL behavior.
- Grouping by unnecessary columns.
- Treating CTE as permanent storage.

## 12. Mini-project — Business Data Aggregation

Produce total sales, quantity per product, total per order, customer totals including zero-order customers, orders above threshold, products above average price, customers with orders and a multi-step CTE report.

**Deliverables:** SQL file, outputs, WHERE/HAVING explanation, scalar and correlated subqueries, CTE example.

## 13. Self-check

Explain SUM, COUNT(*), COUNT(column), AVG, GROUP BY, HAVING, scalar/correlated subquery, EXISTS, CTE and why joins inflate totals.

## 99. Next lesson

Next: Day 23 — SQL Window Functions.