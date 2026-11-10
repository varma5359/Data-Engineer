# Day 22: SQL Aggregations and Advanced Queries
**Phase:** 3 — SQL and Database Engineering  
**Mini-project:** Business Data Aggregation  
**Learning time:** 2–3 hours  
**Audience:** Beginners progressing to junior Data Engineer level  
**SQL dialect:** PostgreSQL; portable SQL is used unless a dialect difference is mentioned.

---

## 1. Start with a business question

Yesterday you learned to extract rows from one or more tables. Today you will learn to turn many detail rows into meaningful business summaries.

Imagine TechStore's sales manager asks:

- How many orders did we receive?
- How much value did each order contain?
- How much did each customer spend?
- Which customers placed more than one order?
- Which products generated the highest sales value?
- Which customers spent more than the average customer?
- Can we calculate a summary first and then query that summary?

These questions require aggregation and advanced query patterns.

A Data Engineer frequently transforms operational records into reporting datasets. This lesson teaches you how to do that safely.

### Learning outcomes

By the end, you should be able to:

1. Explain why aggregation is required.
2. Use COUNT, SUM, AVG, MIN and MAX.
3. Understand the difference between COUNT(*) and COUNT(column).
4. Understand how NULL affects aggregate functions.
5. Calculate row-level expressions before aggregating.
6. Group records with GROUP BY.
7. Filter groups with HAVING.
8. Understand the order and purpose of WHERE, GROUP BY and HAVING.
9. Write subqueries in WHERE and FROM.
10. Use correlated subqueries.
11. Understand EXISTS and NOT EXISTS.
12. Build and read Common Table Expressions (CTEs).
13. Break a large business query into small understandable steps.
14. Validate aggregate results against detail records.

---

## 2. Refresh the TechStore training data

We use the database created in Day 20.

### Customers

| customer_id | customer_name | city |
|---:|---|---|
| 1 | Asha | Hyderabad |
| 2 | Ravi | Chennai |
| 3 | Meena | Bengaluru |

### Products

| product_id | product_name | category | price |
|---:|---|---|---:|
| 101 | Keyboard | Accessories | 1200 |
| 102 | Mouse | Accessories | 700 |
| 103 | Monitor | Electronics | 8500 |
| 104 | Laptop Stand | Accessories | 1500 |

### Orders

| order_id | customer_id | order_date | order_status |
|---:|---:|---|---|
| 5001 | 1 | 2026-11-01 | PLACED |
| 5002 | 2 | 2026-11-02 | SHIPPED |
| 5003 | 1 | 2026-11-03 | PLACED |

### Order items

| order_item_id | order_id | product_id | quantity | unit_price |
|---:|---:|---:|---:|---:|
| 1 | 5001 | 101 | 1 | 1200 |
| 2 | 5001 | 102 | 2 | 700 |
| 3 | 5002 | 103 | 1 | 8500 |
| 4 | 5003 | 104 | 2 | 1500 |

Our sales amount is calculated from the order item: quantity multiplied by unit_price.

Expected line values:
- Order item 1: 1 × 1200 = 1200
- Order item 2: 2 × 700 = 1400
- Order item 3: 1 × 8500 = 8500
- Order item 4: 2 × 1500 = 3000

Total line value = 14,100.

**Important business definition:** This training total is the sum of order line values. It is not necessarily recognized revenue. A real company may exclude cancelled orders, returns, taxes, discounts or unpaid orders according to its finance rules.

---

## 3. What is aggregation?

Aggregation means summarizing multiple records into a smaller result.

Suppose a shop has these order amounts:

| Order | Amount |
|---|---:|
| A | 100 |
| B | 200 |
| C | 300 |

A manager may not need all three records. They may ask for the total amount, average amount, highest amount or number of orders.

- Total = 100 + 200 + 300 = 600
- Average = 600 divided by 3 = 200
- Highest = 300
- Lowest = 100
- Count = 3 orders

SQL aggregate functions calculate these summaries.

---

## 4. First understand aggregate functions

The five core aggregate functions are:

| Function | What it calculates | Business question |
|---|---|---|
| COUNT | Number of rows or non-NULL values | How many orders? |
| SUM | Total of numeric values | What is the total sales value? |
| AVG | Average of non-NULL numeric values | What is the average order value? |
| MIN | Smallest value | What is the lowest product price? |
| MAX | Largest value | What is the highest product price? |

These functions usually return one result for the selected input set. When combined with GROUP BY, they return one result per group.

---

## 5. COUNT: count records

### 5.1 Count all order records

Task: The manager asks how many orders exist.

```sql
SELECT COUNT(*) AS total_orders
FROM orders;
```

Expected result:

| total_orders |
|---:|
| 3 |

Explanation:

- COUNT is an aggregate function.
- * means count every row.
- FROM orders identifies the source.
- AS total_orders gives the output a readable heading.

The database sees three rows in orders, so the result is 3.

### 5.2 Count a specific column

```sql
SELECT COUNT(customer_id) AS customer_id_values
FROM orders;
```

Expected result: 3.

COUNT(column) counts only rows where that column is not NULL.

In this sample, every order has a customer_id, so COUNT(*) and COUNT(customer_id) both return 3.

### 5.3 COUNT(*) versus COUNT(column)

Imagine a table containing:

| feedback_id | rating |
|---:|---:|
| 1 | 5 |
| 2 | NULL |
| 3 | 4 |
| 4 | NULL |

- COUNT(*) = 4 because there are four rows.
- COUNT(rating) = 2 because only two rating values are non-NULL.

A safe demonstration query:

```sql
SELECT
    COUNT(*) AS all_rows,
    COUNT(rating) AS rows_with_rating
FROM (
    VALUES (1, 5), (2, NULL), (3, 4), (4, NULL)
) AS feedback(feedback_id, rating);
```

Expected: all_rows = 4, rows_with_rating = 2.

This VALUES syntax is supported by PostgreSQL. The table expression is a small temporary input for the query; it does not create a permanent table.

### 5.4 COUNT(DISTINCT column)

Task: How many different customers placed orders?

```sql
SELECT COUNT(DISTINCT customer_id) AS unique_customers
FROM orders;
```

Expected: 2.

Asha placed two orders, but she is counted once. Ravi is counted once. Meena is not counted because she has no order.

---

## 6. SUM: calculate a total

### 6.1 Add all product prices

```sql
SELECT SUM(price) AS catalogue_price_total
FROM products;
```

Expected: 11,900.

Calculation:
- 1,200 + 700 = 1,900
- 1,900 + 8,500 = 10,400
- 10,400 + 1,500 = 11,900

This is the sum of catalogue prices, not sales.

### 6.2 Calculate total order line value

```sql
SELECT SUM(quantity * unit_price) AS total_line_value
FROM order_items;
```

Expected: 14,100.

The multiplication is evaluated for each input row:
- 1 × 1,200 = 1,200
- 2 × 700 = 1,400
- 1 × 8,500 = 8,500
- 2 × 1,500 = 3,000

Then SUM adds those four values.

### 6.3 Why is multiplication inside SUM?

SUM(quantity * unit_price) means:
1. Calculate quantity multiplied by unit_price for each row.
2. Add all resulting row values.
3. Return one total.

Do not confuse this with SUM(quantity) * SUM(unit_price). That expression multiplies two separate totals and is not the same business calculation.

### 6.4 NULL behavior with SUM

Most aggregate functions, including SUM, ignore NULL input values. If all values are NULL, SUM returns NULL rather than zero.

If a business rule says missing amounts should be treated as zero, make that decision explicitly:

```sql
SELECT SUM(COALESCE(amount, 0)) AS total_amount
FROM some_table;
```

COALESCE returns the first non-NULL argument. Do not replace NULL with zero automatically unless the business meaning supports it.

---

## 7. AVG: calculate an average

### 7.1 Average product price

```sql
SELECT AVG(price) AS average_product_price
FROM products;
```

Expected: 2975.

Calculation:
- Sum of prices = 11,900
- Number of non-NULL prices = 4
- Average = 11,900 / 4 = 2,975

### 7.2 Average order line value

```sql
SELECT AVG(quantity * unit_price) AS average_line_value
FROM order_items;
```

Expected: 3,525.

Calculation:
14,100 divided by 4 order-item rows = 3,525.

This is average order-item line value, not average order value. The names matter because they describe different business metrics.

### 7.3 Average ignores NULL

If a column contains 10, NULL and 20, AVG(column) is 15, because the denominator counts the two non-NULL values.

It does not divide by three.

---

## 8. MIN and MAX: find boundaries

### 8.1 Lowest and highest product price

```sql
SELECT
    MIN(price) AS lowest_price,
    MAX(price) AS highest_price
FROM products;
```

Expected: lowest_price = 700, highest_price = 8500.

### 8.2 Find the earliest and latest order dates

```sql
SELECT
    MIN(order_date) AS first_order_date,
    MAX(order_date) AS latest_order_date
FROM orders;
```

Expected: 2026-11-01 and 2026-11-03.

MIN and MAX work on dates and text as well as numbers. Text ordering follows the database's collation rules.

---

## 9. Use multiple aggregate functions in one query

A manager wants a quick catalogue summary.

```sql
SELECT
    COUNT(*) AS product_count,
    SUM(price) AS total_catalogue_price,
    AVG(price) AS average_price,
    MIN(price) AS cheapest_price,
    MAX(price) AS most_expensive_price
FROM products;
```

Expected:

| product_count | total_catalogue_price | average_price | cheapest_price | most_expensive_price |
|---:|---:|---:|---:|---:|
| 4 | 11900 | 2975 | 700 | 8500 |

All five metrics summarize the same four product rows.

---

## 10. GROUP BY: summarize each group separately

So far, aggregate functions returned one summary for the entire table.

A manager may ask for sales by order. We need one summary per order. GROUP BY divides rows into groups.

### 10.1 Count orders by customer

```sql
SELECT
    customer_id,
    COUNT(*) AS number_of_orders
FROM orders
GROUP BY customer_id
ORDER BY customer_id;
```

Expected:

| customer_id | number_of_orders |
|---:|---:|
| 1 | 2 |
| 2 | 1 |

Explanation:

1. SQL reads the three order rows.
2. It groups rows with the same customer_id.
3. Customer 1 has two rows.
4. Customer 2 has one row.
5. It returns one result row per customer_id group.

Meena is absent because no order row exists for customer 3. GROUP BY cannot invent a group for a customer with no matching orders.

### 10.2 Sum sales by order

```sql
SELECT
    order_id,
    SUM(quantity * unit_price) AS order_total
FROM order_items
GROUP BY order_id
ORDER BY order_id;
```

Expected:

| order_id | order_total |
|---:|---:|
| 5001 | 2600 |
| 5002 | 8500 |
| 5003 | 3000 |

Walkthrough:
- Order 5001 has 1,200 + 1,400 = 2,600.
- Order 5002 has 8,500.
- Order 5003 has 3,000.

### 10.3 Count products by category

```sql
SELECT
    category,
    COUNT(*) AS products_in_category
FROM products
GROUP BY category
ORDER BY category;
```

Expected:

| category | products_in_category |
|---|---:|
| Accessories | 3 |
| Electronics | 1 |

Three products belong to Accessories. One product belongs to Electronics.

### 10.4 Group by more than one column

```sql
SELECT
    category,
    product_name,
    COUNT(*) AS product_rows
FROM products
GROUP BY category, product_name
ORDER BY category, product_name;
```

Each unique combination of category and product_name becomes a group. Because product_name is unique in this small catalogue, every group has one row.

Use multiple grouping columns only when the report needs that level of detail.

---

## 11. A critical GROUP BY rule

In a grouped query, every selected expression must either:
- be part of the GROUP BY expressions, or
- be aggregated, or
- be allowed by a database-specific functional dependency rule.

For beginner-safe SQL, follow the first two rules.

Correct:

```sql
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id;
```

Incorrect:

```sql
SELECT customer_id, order_status, COUNT(*)
FROM orders
GROUP BY customer_id;
```

Why is the second query problematic? A customer may have multiple order_status values. Which one should SQL display for the customer group? The query has not specified a rule.

Correct if you want counts by both customer and status:

```sql
SELECT customer_id, order_status, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id, order_status;
```

This returns one row per customer/status combination.

---

## 12. WHERE versus HAVING

These clauses are different.

- WHERE filters individual input rows before grouping.
- HAVING filters groups after aggregation.

### 12.1 WHERE example

Task: Count orders only for customer 1.

```sql
SELECT customer_id, COUNT(*) AS order_count
FROM orders
WHERE customer_id = 1
GROUP BY customer_id;
```

Expected: customer_id 1, order_count 2.

WHERE removes other customers' order rows before the grouping step.

### 12.2 HAVING example

Task: Show customers who have placed more than one order.

```sql
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

Expected: customer_id 1, order_count 2.

HAVING evaluates the count for each group and keeps only groups where the count is greater than one.

### 12.3 Use WHERE and HAVING together

Task: Among orders placed from November 1 onward, show customers with more than one order.

```sql
SELECT customer_id, COUNT(*) AS order_count
FROM orders
WHERE order_date >= DATE '2026-11-01'
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

Expected: customer 1 with 2 orders.

The row filter happens before grouping; the group filter happens after the count is calculated.

### 12.4 Common mistake: using aggregate in WHERE

This is not valid:

```sql
SELECT customer_id, COUNT(*)
FROM orders
WHERE COUNT(*) > 1
GROUP BY customer_id;
```

WHERE is evaluated before grouping, so the grouped COUNT is not available there. Use HAVING.

---

## 13. Understand the logical processing order

A useful mental model for a grouped SELECT query:

1. FROM and JOIN identify source rows.
2. WHERE filters individual rows.
3. GROUP BY forms groups.
4. Aggregate functions calculate group summaries.
5. HAVING filters groups.
6. SELECT produces the requested output.
7. ORDER BY sorts the result.
8. LIMIT restricts the returned rows.

This is the logical model for understanding query meaning. The optimizer may choose a different physical execution plan.

---

## 14. Aggregate after joining tables

Business question: Show each customer's total order line value.

We need customer details, orders and order items.

```sql
SELECT
    c.customer_id,
    c.customer_name,
    SUM(oi.quantity * oi.unit_price) AS customer_spend
FROM customers AS c
JOIN orders AS o
    ON c.customer_id = o.customer_id
JOIN order_items AS oi
    ON o.order_id = oi.order_id
GROUP BY c.customer_id, c.customer_name
ORDER BY c.customer_id;
```

Expected:

| customer_id | customer_name | customer_spend |
|---:|---|---:|
| 1 | Asha | 5600 |
| 2 | Ravi | 8500 |

Asha: 2,600 + 3,000 = 5,600.  
Ravi: 8,500.  
Meena has no orders and is not included by the INNER JOIN.

### 14.1 Include customers with no orders

```sql
SELECT
    c.customer_id,
    c.customer_name,
    COALESCE(SUM(oi.quantity * oi.unit_price), 0) AS customer_spend
FROM customers AS c
LEFT JOIN orders AS o
    ON c.customer_id = o.customer_id
LEFT JOIN order_items AS oi
    ON o.order_id = oi.order_id
GROUP BY c.customer_id, c.customer_name
ORDER BY c.customer_id;
```

Expected:

| customer_id | customer_name | customer_spend |
|---:|---|---:|
| 1 | Asha | 5600 |
| 2 | Ravi | 8500 |
| 3 | Meena | 0 |

The LEFT JOIN preserves Meena. SUM returns NULL for her missing order-item rows, and COALESCE converts that result to zero.

This is correct only if the business defines no orders as zero spend. If the amount is unknown rather than absent, do not force zero.

---

## 15. Subqueries: a query inside another query

A subquery is a query nested inside another SQL statement.

Why use one? Sometimes we need to calculate a value first and use it to answer a larger question.

Example business question: Which products cost more than the average product price?

We need two steps:
1. Calculate average product price.
2. Return products above that average.

### 15.1 First run the inner query alone

```sql
SELECT AVG(price)
FROM products;
```

Expected: 2975.

Understand this result before nesting it.

### 15.2 Use the query as a subquery

```sql
SELECT product_name, price
FROM products
WHERE price > (
    SELECT AVG(price)
    FROM products
);
```

Expected: Monitor only.

The inner query calculates 2,975. The outer query compares every product price with 2,975. Only Monitor at 8,500 is greater.

### 15.3 Read the query from the inside out

- Inner SELECT calculates one average value.
- Parentheses make it a subquery.
- Outer WHERE compares each product price with that value.
- Outer SELECT returns the qualifying product name and price.

When a subquery returns one value, it is called a scalar subquery.

---

## 16. Subquery with IN

Business question: Find customers who have placed at least one order.

```sql
SELECT customer_id, customer_name
FROM customers
WHERE customer_id IN (
    SELECT customer_id
    FROM orders
);
```

Expected: Asha and Ravi.

The inner query returns customer IDs that appear in orders. The outer query selects customers whose ID is in that result.

### 16.1 Understand the two queries separately

Inner query:

```sql
SELECT customer_id
FROM orders;
```

Expected values: 1, 2, 1.

Outer query uses IN, so customer 1 and customer 2 qualify. Repeated 1 does not create duplicate customer rows in the outer table.

### 16.2 NOT IN warning

This can be risky if the subquery returns NULL:

```sql
SELECT customer_id, customer_name
FROM customers
WHERE customer_id NOT IN (
    SELECT customer_id
    FROM orders
);
```

In this particular training data, it returns Meena. But if the inner result includes NULL, NOT IN may return no rows due to UNKNOWN comparisons. Prefer NOT EXISTS for robust anti-match logic.

---

## 17. EXISTS and NOT EXISTS

EXISTS checks whether a subquery returns at least one row. It does not need to return a particular column value.

### 17.1 Customers who have orders

```sql
SELECT c.customer_id, c.customer_name
FROM customers AS c
WHERE EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customer_id = c.customer_id
);
```

Expected: Asha and Ravi.

The subquery is correlated with the current customer through c.customer_id.

### 17.2 Customers who have no orders

```sql
SELECT c.customer_id, c.customer_name
FROM customers AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customer_id = c.customer_id
);
```

Expected: Meena.

NOT EXISTS keeps a customer only when the matching subquery finds no order.

### 17.3 Why SELECT 1?

EXISTS only cares whether at least one row exists. The selected value is not used. SELECT 1 is a common convention that communicates this clearly.

---

## 18. Correlated subqueries

A normal subquery can run independently of the outer query. A correlated subquery refers to a value from the current outer row.

Think of it as asking a related question for each customer.

Task: Find customers whose number of orders is greater than one.

```sql
SELECT c.customer_id, c.customer_name
FROM customers AS c
WHERE (
    SELECT COUNT(*)
    FROM orders AS o
    WHERE o.customer_id = c.customer_id
) > 1;
```

Expected: Asha.

Walkthrough:
- For Asha, the subquery counts 2 orders. 2 > 1 is true.
- For Ravi, it counts 1. 1 > 1 is false.
- For Meena, it counts 0. 0 > 1 is false.

The inner query refers to c.customer_id from the outer query. That makes it correlated.

### 18.1 Correlated subquery versus GROUP BY

The same business result can be written with GROUP BY and HAVING:

```sql
SELECT c.customer_id, c.customer_name
FROM customers AS c
JOIN orders AS o
  ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name
HAVING COUNT(*) > 1;
```

Both can express the requirement. Choose the version that is easiest to understand and performs well for the real data and database engine.

---

## 19. Subquery in the FROM clause

A query in FROM can be treated as a derived table.

Task: Calculate order totals first, then return orders greater than 3,000.

```sql
SELECT order_summary.order_id,
       order_summary.order_total
FROM (
    SELECT order_id,
           SUM(quantity * unit_price) AS order_total
    FROM order_items
    GROUP BY order_id
) AS order_summary
WHERE order_summary.order_total > 3000;
```

Expected: order 5002 with 8500.

The inner query creates a temporary result with one row per order. The outer query filters that result.

A derived table must have an alias in PostgreSQL. Here the alias is order_summary.

---

## 20. Common Table Expressions (CTEs)

A CTE is a named temporary result defined using WITH. It exists for the duration of one SQL statement.

Why use a CTE?

- Break a complex query into understandable steps.
- Give a calculation a meaningful name.
- Reuse a named result within the statement.
- Make the query easier to review and debug.

### 20.1 First CTE

```sql
WITH order_totals AS (
    SELECT
        order_id,
        SUM(quantity * unit_price) AS order_total
    FROM order_items
    GROUP BY order_id
)
SELECT *
FROM order_totals;
```

Expected:

| order_id | order_total |
|---:|---:|
| 5001 | 2600 |
| 5002 | 8500 |
| 5003 | 3000 |

Explanation:
1. WITH begins the CTE definition.
2. order_totals is its name.
3. The parentheses contain the query that builds the result.
4. The final SELECT reads from that named result.

The CTE is not automatically a permanent table.

### 20.2 Filter the CTE result

```sql
WITH order_totals AS (
    SELECT
        order_id,
        SUM(quantity * unit_price) AS order_total
    FROM order_items
    GROUP BY order_id
)
SELECT order_id, order_total
FROM order_totals
WHERE order_total > 3000
ORDER BY order_total DESC;
```

Expected: order 5002, total 8500.

The first step calculates order totals. The final query filters the calculated values.

### 20.3 Two CTEs

Task: Show customer names and their total spend.

```sql
WITH order_totals AS (
    SELECT
        o.order_id,
        o.customer_id,
        SUM(oi.quantity * oi.unit_price) AS order_total
    FROM orders AS o
    JOIN order_items AS oi
      ON o.order_id = oi.order_id
    GROUP BY o.order_id, o.customer_id
),
customer_totals AS (
    SELECT
        customer_id,
        SUM(order_total) AS customer_spend
    FROM order_totals
    GROUP BY customer_id
)
SELECT
    c.customer_name,
    COALESCE(ct.customer_spend, 0) AS customer_spend
FROM customers AS c
LEFT JOIN customer_totals AS ct
  ON c.customer_id = ct.customer_id
ORDER BY c.customer_id;
```

Expected:

| customer_name | customer_spend |
|---|---:|
| Asha | 5600 |
| Ravi | 8500 |
| Meena | 0 |

The first CTE calculates one row per order. The second calculates one row per customer. The final query attaches customer names and preserves customers with no orders.

### 20.4 CTE versus subquery

Both can express many of the same calculations.

| CTE | Subquery |
|---|---|
| Named using WITH | Nested directly inside another query |
| Often easier to read in multi-step logic | Often compact for one small calculation |
| Can define multiple named steps | Nested levels can become hard to read |
| Not automatically stored permanently | Not automatically stored permanently |

A CTE is not automatically faster than a subquery. Query planning and CTE materialization behavior depend on the database and query. In PostgreSQL, modern versions can inline many CTEs; MATERIALIZED and NOT MATERIALIZED can influence planning in supported cases.

---

## 21. Build a business summary step by step

Business request: "Prepare a customer sales summary for our manager."

The report must show:
- Customer ID
- Customer name
- Number of orders
- Total line value
- Average order total

First, decide the grain: one row per customer.

### Step 1: calculate each order total

```sql
SELECT
    o.order_id,
    o.customer_id,
    SUM(oi.quantity * oi.unit_price) AS order_total
FROM orders AS o
JOIN order_items AS oi
  ON o.order_id = oi.order_id
GROUP BY o.order_id, o.customer_id
ORDER BY o.order_id;
```

Expected: 5001 = 2600, 5002 = 8500, 5003 = 3000.

### Step 2: summarize those orders by customer

```sql
WITH order_totals AS (
    SELECT
        o.order_id,
        o.customer_id,
        SUM(oi.quantity * oi.unit_price) AS order_total
    FROM orders AS o
    JOIN order_items AS oi
      ON o.order_id = oi.order_id
    GROUP BY o.order_id, o.customer_id
)
SELECT
    customer_id,
    COUNT(*) AS number_of_orders,
    SUM(order_total) AS total_spend,
    AVG(order_total) AS average_order_value
FROM order_totals
GROUP BY customer_id
ORDER BY customer_id;
```

Expected:

| customer_id | number_of_orders | total_spend | average_order_value |
|---:|---:|---:|---:|
| 1 | 2 | 5600 | 2800 |
| 2 | 1 | 8500 | 8500 |

Asha's average order value is (2600 + 3000) / 2 = 2800.

### Step 3: attach customer names and include customers with no orders

```sql
WITH order_totals AS (
    SELECT
        o.order_id,
        o.customer_id,
        SUM(oi.quantity * oi.unit_price) AS order_total
    FROM orders AS o
    JOIN order_items AS oi
      ON o.order_id = oi.order_id
    GROUP BY o.order_id, o.customer_id
),
customer_summary AS (
    SELECT
        customer_id,
        COUNT(*) AS number_of_orders,
        SUM(order_total) AS total_spend,
        AVG(order_total) AS average_order_value
    FROM order_totals
    GROUP BY customer_id
)
SELECT
    c.customer_id,
    c.customer_name,
    COALESCE(cs.number_of_orders, 0) AS number_of_orders,
    COALESCE(cs.total_spend, 0) AS total_spend,
    cs.average_order_value
FROM customers AS c
LEFT JOIN customer_summary AS cs
  ON c.customer_id = cs.customer_id
ORDER BY c.customer_id;
```

Expected:

| customer_id | customer_name | number_of_orders | total_spend | average_order_value |
|---:|---|---:|---:|---:|
| 1 | Asha | 2 | 5600 | 2800 |
| 2 | Ravi | 1 | 8500 | 8500 |
| 3 | Meena | 0 | 0 | NULL |

Why is Meena's average NULL? She has no orders. An average of no observations is undefined. It is usually better to keep that as NULL rather than invent an average of zero.

---

## 22. Avoid double-counting when joining

This is a critical Data Engineering skill.

Suppose an order has:
- two order-item rows
- three payment rows

Joining order_items directly to payments by order_id can create 2 × 3 = 6 rows for that order. Summing item values after this join would count each item three times.

The correct approach is often to aggregate each detail table to the order grain first, then join the summaries.

Example pattern:

```sql
WITH item_totals AS (
    SELECT order_id,
           SUM(quantity * unit_price) AS item_total
    FROM order_items
    GROUP BY order_id
),
payment_totals AS (
    SELECT order_id,
           SUM(payment_amount) AS paid_total
    FROM payments
    GROUP BY order_id
)
SELECT
    o.order_id,
    it.item_total,
    pt.paid_total
FROM orders AS o
LEFT JOIN item_totals AS it
  ON o.order_id = it.order_id
LEFT JOIN payment_totals AS pt
  ON o.order_id = pt.order_id;
```

The example assumes a payments table with payment_amount. It is a design pattern, not a query to run against the current Day 20 schema.

**Rule:** Before joining detail tables, check their grain and cardinality. Aggregate to a common grain when the business calculation requires it.

---

## 23. Common beginner mistakes

### Mistake 1: Confusing COUNT(*) and COUNT(column)

COUNT(*) counts rows. COUNT(column) counts non-NULL values in that column.

### Mistake 2: Expecting COUNT(column) to count NULL

It does not. Use COUNT(*) if the task is to count rows.

### Mistake 3: Using SUM(column) and expecting NULL to mean zero

SUM ignores NULL values. If all values are NULL, the result is NULL. Decide whether COALESCE is appropriate.

### Mistake 4: Grouping at the wrong level

Grouping by order_id gives one row per order. Grouping by customer_id gives one row per customer. Identify the output grain first.

### Mistake 5: Selecting a non-grouped column

If a column is not grouped or aggregated, SQL cannot generally choose a meaningful value for the group.

### Mistake 6: Putting aggregate conditions in WHERE

Use HAVING to filter groups based on aggregate results.

### Mistake 7: Using HAVING for ordinary row filters

If a condition does not depend on an aggregate, WHERE is usually the correct place.

### Mistake 8: Averaging line values and calling it average order value

Average line value and average order value are different metrics. Aggregate at the order level first, then average order totals.

### Mistake 9: Joining multiple one-to-many tables before aggregating

This can multiply rows and inflate sums. Aggregate each source to the required common grain first.

### Mistake 10: Assuming CTE means permanent table

A normal CTE exists only for one SQL statement.

### Mistake 11: Assuming CTE always improves performance

CTEs improve organization, but performance depends on the query plan and database version.

### Mistake 12: Using NOT IN with a nullable subquery

A NULL in the subquery can change the result. Consider NOT EXISTS.

---

## 24. Mini-project: Business Data Aggregation

### Business requirement

TechStore wants a small business summary from the operational database. Deliver:
1. Total order count.
2. Total line value.
3. Value per order.
4. Spend per customer.
5. Customers with more than one order.
6. Products with price above average.
7. Customers who have never ordered.

### Task 1: total number of orders

```sql
SELECT COUNT(*) AS total_orders
FROM orders;
```

Expected: 3.

### Task 2: total line value

```sql
SELECT SUM(quantity * unit_price) AS total_line_value
FROM order_items;
```

Expected: 14100.

### Task 3: total per order

```sql
SELECT order_id,
       SUM(quantity * unit_price) AS order_total
FROM order_items
GROUP BY order_id
ORDER BY order_id;
```

Expected: 2600, 8500 and 3000.

### Task 4: customer spend

Use the customer summary query from Section 21. Confirm Asha = 5600 and Ravi = 8500.

### Task 5: customers with more than one order

```sql
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

Expected: customer 1 with 2 orders.

### Task 6: products above average price

```sql
SELECT product_name, price
FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

Expected: Monitor.

### Task 7: customers without orders

```sql
SELECT c.customer_id, c.customer_name
FROM customers AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.customer_id = c.customer_id
);
```

Expected: Meena.

### Task 8: create a reusable summary query using CTEs

```sql
WITH order_totals AS (
    SELECT
        o.order_id,
        o.customer_id,
        SUM(oi.quantity * oi.unit_price) AS order_total
    FROM orders AS o
    JOIN order_items AS oi
      ON o.order_id = oi.order_id
    GROUP BY o.order_id, o.customer_id
),
customer_totals AS (
    SELECT
        customer_id,
        COUNT(*) AS order_count,
        SUM(order_total) AS spend
    FROM order_totals
    GROUP BY customer_id
)
SELECT
    c.customer_id,
    c.customer_name,
    COALESCE(ct.order_count, 0) AS order_count,
    COALESCE(ct.spend, 0) AS spend
FROM customers AS c
LEFT JOIN customer_totals AS ct
  ON c.customer_id = ct.customer_id
ORDER BY c.customer_id;
```

Expected:

| customer_id | customer_name | order_count | spend |
|---:|---|---:|---:|
| 1 | Asha | 2 | 5600 |
| 2 | Ravi | 1 | 8500 |
| 3 | Meena | 0 | 0 |

### Project validation

- [ ] Total order count is 3.
- [ ] Total line value is 14,100.
- [ ] Order totals are 2,600, 8,500 and 3,000.
- [ ] Customer spend reconciles to 14,100.
- [ ] Asha has two orders.
- [ ] Monitor is the only product above average catalogue price.
- [ ] Meena is identified as a customer without orders.
- [ ] Every output has a clearly defined grain.
- [ ] No total is inflated by a one-to-many join.

---

## 25. Practice questions

Try these before checking the answer hints.

1. Count all customers.
2. Count non-NULL customer emails.
3. Find the total price of all catalogue products.
4. Find the average product price.
5. Find the cheapest and most expensive products.
6. Count orders per customer.
7. Calculate the total value per order.
8. Find customers with more than one order.
9. Find products priced above the average.
10. Find customers who have placed an order using EXISTS.
11. Find customers who have never placed an order using NOT EXISTS.
12. Build an order_totals CTE.
13. Explain why average order value is not the same as average order-item value.
14. Explain how joining two one-to-many tables can inflate a SUM.
15. Explain the difference between WHERE and HAVING.

### Answer hints

1. COUNT(*) on customers.
2. COUNT(email) on customers.
3. SUM(price) on products.
4. AVG(price) on products.
5. MIN(price) and MAX(price).
6. GROUP BY customer_id in orders.
7. SUM(quantity * unit_price), grouped by order_id.
8. GROUP BY customer_id with HAVING COUNT(*) > 1.
9. Compare price with a scalar AVG(price) subquery.
10. EXISTS with a correlated customer_id condition.
11. NOT EXISTS with the same relationship.
12. WITH order_totals AS (...) followed by SELECT.
13. First aggregate lines into orders, then average the order totals.
14. A row from one detail table may match several rows in the other detail table, multiplying combinations.
15. WHERE filters rows before grouping; HAVING filters groups after aggregation.

---

## 26. Day 22 revision

- COUNT(*) counts rows.
- COUNT(column) counts non-NULL values.
- COUNT(DISTINCT column) counts distinct non-NULL values.
- SUM adds values.
- AVG calculates the average of non-NULL values.
- MIN and MAX find boundaries.
- GROUP BY creates one result group per unique grouping combination.
- WHERE filters source rows.
- HAVING filters grouped results.
- A scalar subquery returns one value.
- IN compares against a set of values.
- EXISTS checks whether a matching row exists.
- A correlated subquery refers to the outer query.
- A CTE names a temporary query result for one statement.
- Identify the grain before aggregating.
- Avoid joining multiple detail tables in a way that multiplies amounts.
- Reconcile summary totals against trusted detail-level totals.

**You are ready for Day 23 when you can calculate totals per order and customer, explain WHERE versus HAVING, and build a readable CTE-based summary without inflating the sales total.**
