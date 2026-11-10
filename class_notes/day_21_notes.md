# Day 21: SQL Data Extraction

**Pathway:** TechPhile.ai — Data Engineer Career Pathway  
**Phase:** 3 — SQL and Database Engineering  
**Audience:** Freshers, college students and junior Data Engineer candidates  
**Level:** Beginner to junior | **Class time:** 2–3 hours  
**Approved syllabus:** SELECT, WHERE, ORDER BY, LIMIT, DISTINCT, filtering, INNER JOIN, LEFT JOIN, RIGHT JOIN  
**Mini-project:** Multi-Table Extraction Queries  
**Primary database:** PostgreSQL; MySQL differences are identified where relevant.

---

## 1. How to study

Read one concept, predict what the query or program will do, execute the small example, compare the result and explain it aloud. Do not run the entire lesson as one large script.

## 2. Prerequisite and learning outcomes

Use the TechStore training database from Day 20: customers, products, orders and order_items. Confirm you are connected to the correct practice database. If sample data differs, inspect it before comparing outputs. Never experiment on production data.

By the end, you should understand each item in today's syllabus, execute the main examples, explain the output, recognize common mistakes and complete **Multi-Table Extraction Queries**.

## 3. SELECT: choose columns

A database may hold millions of rows. SELECT asks for the columns needed by a task.

```sql
SELECT customer_name, city FROM customers;
```
Expected: Asha/Hyderabad, Ravi/Chennai, Meena/Bengaluru. SELECT chooses columns; FROM names the source table. SELECT * returns all columns and is useful for inspection, but production extracts should name required columns.

Use AS for an output label:
```sql
SELECT customer_name AS name, city AS customer_city FROM customers;
```
AS changes the result label, not the stored column.

## 4. WHERE: filter rows

WHERE chooses qualifying rows. Example:
```sql
SELECT product_name, price FROM products WHERE price > 1000;
```
Expected: Keyboard 1200, Monitor 8500, Laptop Stand 1500.

Comparison operators: = equal, <> not equal, > greater, >= greater/equal, < less, <= less/equal. AND requires both conditions; OR requires at least one. Use parentheses when combining them.

```sql
SELECT product_name FROM products
WHERE category = 'Accessories' AND price > 1000;
```
Expected: Keyboard, Laptop Stand.

IN checks membership; BETWEEN includes both endpoints; LIKE matches patterns (% any number of characters, _ one character):
```sql
SELECT customer_name FROM customers WHERE city IN ('Hyderabad','Bengaluru');
SELECT product_name FROM products WHERE price BETWEEN 700 AND 1500;
SELECT customer_name FROM customers WHERE customer_name LIKE 'A%';
```
Expected: Asha/Meena; Mouse/Keyboard/Laptop Stand; Asha.

NULL means missing or unknown. Use IS NULL, never = NULL:
```sql
SELECT customer_name FROM customers WHERE city IS NULL;
```

## 5. DISTINCT, ORDER BY, LIMIT and OFFSET

```sql
SELECT DISTINCT city FROM customers ORDER BY city;
```
Expected: Bengaluru, Chennai, Hyderabad. DISTINCT removes duplicate selected values or combinations.

```sql
SELECT product_name,price FROM products ORDER BY price DESC;
```
DESC sorts largest first; ASC smallest first. Add a stable tie-breaker for deterministic results.

```sql
SELECT product_name,price FROM products
ORDER BY price DESC LIMIT 2;
```
Expected: Monitor, Laptop Stand. LIMIT caps rows; OFFSET skips rows. Large changing datasets often benefit from keyset pagination using the last-seen key instead of large OFFSET values.

## 6. JOIN: combine related tables

orders.customer_id references customers.customer_id.

INNER JOIN returns only matches:
```sql
SELECT o.order_id,c.customer_name,o.order_status
FROM orders o INNER JOIN customers c
ON o.customer_id=c.customer_id ORDER BY o.order_id;
```
Expected: 5001/Asha/PLACED; 5002/Ravi/SHIPPED; 5003/Asha/PLACED.

LEFT JOIN preserves every left row:
```sql
SELECT c.customer_name,o.order_id
FROM customers c LEFT JOIN orders o
ON c.customer_id=o.customer_id
ORDER BY c.customer_id,o.order_id;
```
Expected: Asha/5001, Asha/5003, Ravi/5002, Meena/NULL. Meena remains although she has no order.

RIGHT JOIN preserves every right row:
```sql
SELECT c.customer_name,o.order_id
FROM customers c RIGHT JOIN orders o
ON c.customer_id=o.customer_id ORDER BY o.order_id;
```
Expected: three order rows. RIGHT JOIN can often be rewritten as LEFT JOIN with table order reversed.

Join order items and products:
```sql
SELECT o.order_id,c.customer_name,p.product_name,oi.quantity,oi.unit_price
FROM orders o
JOIN customers c ON c.customer_id=o.customer_id
JOIN order_items oi ON oi.order_id=o.order_id
JOIN products p ON p.product_id=oi.product_id
ORDER BY o.order_id,oi.order_item_id;
```
Expected four lines: order 5001 Keyboard and Mouse; 5002 Monitor; 5003 Laptop Stand. Order 5001 appears twice because it has two items. This is normal one-to-many behavior.

## 7. Join mistakes

- Missing/wrong ON can create a Cartesian product or incorrect matches.
- One-to-many joins increase row counts.
- Filtering a right-side column in WHERE after LEFT JOIN can remove unmatched rows; put the condition in ON if unmatched left rows must remain.
- Qualify repeated names (c.customer_id, o.customer_id).
- Validate row counts and sample records.

Logical teaching order: FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT. The optimizer may execute operations differently.

## 8. Mini-project — Multi-Table Extraction Queries

Create extracts for: products sorted by price; customers in Hyderabad/Bengaluru; orders with customer names; all customers including those without orders; order-line details; top two products by price; distinct categories; orders after a chosen date.

Final report:
```sql
SELECT o.order_id,o.order_date,c.customer_name,p.product_name,
       oi.quantity,oi.unit_price,oi.quantity*oi.unit_price AS line_total
FROM orders o JOIN customers c ON c.customer_id=o.customer_id
JOIN order_items oi ON oi.order_id=o.order_id
JOIN products p ON p.product_id=oi.product_id
ORDER BY o.order_date,o.order_id,oi.order_item_id;
```
Expected line totals: 1200, 1400, 8500, 3000.

**Deliverables:** SQL file, screenshots, join choice explanation, row-count checks.

## 9. Self-check

Explain SELECT versus WHERE, IN/BETWEEN/LIKE, NULL, DISTINCT, ORDER BY/LIMIT, INNER/LEFT/RIGHT JOIN, one-to-many row multiplication, Cartesian product and keyset pagination.

## 99. Next lesson

Next: Day 22 — SQL Aggregations and Advanced Queries.