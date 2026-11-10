# Day 23: SQL Window Functions
**Phase:** 3 — SQL and Database Engineering  
**Mini-project:** Historical Trend Query Engine  
**Learning time:** 2–3 hours  
**Audience:** Beginners progressing to junior Data Engineer level  
**SQL dialect:** PostgreSQL; the core window concepts also exist in modern MySQL.

---

## 1. Why are window functions important?

A business report often needs more than a total.

A sales manager may ask:
- Rank customers by their total spend.
- Show the previous order date for each customer.
- Compare this month's sales with the previous month.
- Number each customer's orders from oldest to newest.
- Find the top three products in every category.
- Calculate a running total without removing individual records.

You already learned GROUP BY in Day 22. GROUP BY combines many rows into fewer summary rows. But sometimes we need a calculation across related rows while keeping every original row visible. Window functions solve this problem.

**Today's topic remains SQL Window Functions.** We will study ROW_NUMBER, RANK, DENSE_RANK, LAG and LEAD, along with OVER, PARTITION BY and ORDER BY.

### Learning outcomes

By the end, you can:
1. Explain what a window function does.
2. Explain OVER and the window definition.
3. Distinguish PARTITION BY from GROUP BY.
4. Explain ORDER BY inside and outside OVER.
5. Use ROW_NUMBER for unique sequential numbering.
6. Use RANK and understand gaps after ties.
7. Use DENSE_RANK and understand no gaps after ties.
8. Compare the three ranking functions using the same data.
9. Use LAG to access a previous row's value.
10. Use LEAD to access a following row's value.
11. Partition calculations by customer or product category.
12. Build business analysis queries from small steps.
13. Validate ranking and trend outputs.

---

## 2. Refresh the TechStore order data

Orders:

| order_id | customer_id | order_date |
|---:|---:|---|
| 5001 | 1 | 2026-11-01 |
| 5002 | 2 | 2026-11-02 |
| 5003 | 1 | 2026-11-03 |

Order items:

| order_item_id | order_id | product_id | quantity | unit_price |
|---:|---:|---:|---:|---:|
| 1 | 5001 | 101 | 1 | 1200 |
| 2 | 5001 | 102 | 2 | 700 |
| 3 | 5002 | 103 | 1 | 8500 |
| 4 | 5003 | 104 | 2 | 1500 |

The order totals are:
- Order 5001 = 2,600
- Order 5002 = 8,500
- Order 5003 = 3,000

For a useful ranking demonstration, these are the values we will analyze.

---

## 3. First understand GROUP BY versus window functions

### 3.1 GROUP BY reduces rows

```sql
SELECT customer_id,
       COUNT(*) AS order_count
FROM orders
GROUP BY customer_id;
```

Expected:

| customer_id | order_count |
|---:|---:|
| 1 | 2 |
| 2 | 1 |

There are two output rows because there are two customer groups. Individual order records are no longer shown.

### 3.2 A window function keeps detail rows

```sql
SELECT
    order_id,
    customer_id,
    COUNT(*) OVER (PARTITION BY customer_id) AS customer_order_count
FROM orders
ORDER BY order_id;
```

Expected:

| order_id | customer_id | customer_order_count |
|---:|---:|---:|
| 5001 | 1 | 2 |
| 5002 | 2 | 1 |
| 5003 | 1 | 2 |

Notice all three order rows remain visible. The count is calculated across each customer's partition and repeated on that customer's rows.

**Remember:**
- GROUP BY: one output row per group (unless more grouping columns are used).
- Window function: calculates across a related set of rows while retaining the input row detail.

---

## 4. Understand the OVER clause

A window function is followed by OVER(...).

Example:

```sql
SELECT
    order_id,
    ROW_NUMBER() OVER (ORDER BY order_date) AS sequence_number
FROM orders;
```

Expected:

| order_id | sequence_number |
|---:|---:|
| 5001 | 1 |
| 5002 | 2 |
| 5003 | 3 |

OVER tells SQL which rows belong to the calculation window and how they should be ordered or divided.

The parts you will learn:

- OVER: begins the window definition.
- PARTITION BY: divides rows into independent groups.
- ORDER BY: defines row order inside the window.
- Frame clauses: define a subset of rows around the current row for certain window calculations. We will introduce them later.

A window definition can use PARTITION BY, ORDER BY, both, or neither, depending on the function and requirement.

---

## 5. ROW_NUMBER: give every row a unique sequence

Imagine a customer has many orders. The manager wants to number those orders from earliest to latest.

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    ROW_NUMBER() OVER (
        ORDER BY order_date
    ) AS row_num
FROM orders;
```

Expected:

| order_id | customer_id | order_date | row_num |
|---:|---:|---|---:|
| 5001 | 1 | 2026-11-01 | 1 |
| 5002 | 2 | 2026-11-02 | 2 |
| 5003 | 1 | 2026-11-03 | 3 |

ROW_NUMBER assigns 1, 2, 3... in the window order.

### 5.1 Why add a tie-breaker?

If two orders have the same date, SQL may assign either one first unless the ORDER BY fully determines the order.

Use a unique key as a tie-breaker:

```sql
ROW_NUMBER() OVER (
    ORDER BY order_date, order_id
)
```

This makes the sequence deterministic when order_id is unique.

### 5.2 ROW_NUMBER per customer

Task: Number each customer's orders separately.

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    ROW_NUMBER() OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
    ) AS customer_order_number
FROM orders
ORDER BY customer_id, order_date;
```

Expected:

| order_id | customer_id | order_date | customer_order_number |
|---:|---:|---|---:|
| 5001 | 1 | 2026-11-01 | 1 |
| 5003 | 1 | 2026-11-03 | 2 |
| 5002 | 2 | 2026-11-02 | 1 |

Customer 1 has their own sequence 1, 2. Customer 2 starts again at 1.

### 5.3 A common use: latest row per customer

A common Data Engineering requirement is to keep the most recent record for each business key.

First understand the numbering:

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    ROW_NUMBER() OVER (
        PARTITION BY customer_id
        ORDER BY order_date DESC, order_id DESC
    ) AS latest_rank
FROM orders;
```

The latest order for each customer receives row number 1.

To return only those rows, use a subquery:

```sql
SELECT order_id, customer_id, order_date
FROM (
    SELECT
        order_id,
        customer_id,
        order_date,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date DESC, order_id DESC
        ) AS latest_rank
    FROM orders
) AS ranked_orders
WHERE latest_rank = 1
ORDER BY customer_id;
```

Expected: order 5003 for customer 1 and order 5002 for customer 2.

Why a subquery? A window function result is not available in the WHERE clause at the same query level. Calculate it in the inner query, then filter it in the outer query.

---

## 6. RANK: ranking with gaps after ties

Imagine a product competition:

| Product | Sales |
|---|---:|
| A | 900 |
| B | 800 |
| C | 800 |
| D | 700 |

Two products share second place. RANK gives tied rows the same rank and leaves a gap after the tie.

### 6.1 Create a small practice input

```sql
WITH product_sales(product_name, sales_amount) AS (
    VALUES
        ('A', 900),
        ('B', 800),
        ('C', 800),
        ('D', 700)
)
SELECT
    product_name,
    sales_amount,
    RANK() OVER (
        ORDER BY sales_amount DESC
    ) AS sales_rank
FROM product_sales
ORDER BY sales_amount DESC, product_name;
```

Expected:

| product_name | sales_amount | sales_rank |
|---|---:|---:|
| A | 900 | 1 |
| B | 800 | 2 |
| C | 800 | 2 |
| D | 700 | 4 |

D receives rank 4, not 3, because two rows occupy rank 2.

### 6.2 Why do ties happen?

Two customers can have the same sales total. If the ranking expression is sales total only, they are tied.

Do not add customer_id to the RANK window ordering if you want equal sales values to remain tied. Adding a unique ID breaks the tie.

---

## 7. DENSE_RANK: ranking without gaps

DENSE_RANK also gives equal values the same rank, but it does not leave gaps after ties.

```sql
WITH product_sales(product_name, sales_amount) AS (
    VALUES
        ('A', 900),
        ('B', 800),
        ('C', 800),
        ('D', 700)
)
SELECT
    product_name,
    sales_amount,
    DENSE_RANK() OVER (
        ORDER BY sales_amount DESC
    ) AS dense_sales_rank
FROM product_sales
ORDER BY sales_amount DESC, product_name;
```

Expected:

| product_name | sales_amount | dense_sales_rank |
|---|---:|---:|
| A | 900 | 1 |
| B | 800 | 2 |
| C | 800 | 2 |
| D | 700 | 3 |

D receives rank 3 because DENSE_RANK counts distinct ranking values without gaps.

---

## 8. Compare ROW_NUMBER, RANK and DENSE_RANK

Use exactly the same sample:

| Product | Sales |
|---|---:|
| A | 900 |
| B | 800 |
| C | 800 |
| D | 700 |

```sql
WITH product_sales(product_name, sales_amount) AS (
    VALUES
        ('A', 900),
        ('B', 800),
        ('C', 800),
        ('D', 700)
)
SELECT
    product_name,
    sales_amount,
    ROW_NUMBER() OVER (ORDER BY sales_amount DESC, product_name) AS row_num,
    RANK() OVER (ORDER BY sales_amount DESC) AS rank_num,
    DENSE_RANK() OVER (ORDER BY sales_amount DESC) AS dense_rank_num
FROM product_sales
ORDER BY sales_amount DESC, product_name;
```

Expected:

| Product | Sales | ROW_NUMBER | RANK | DENSE_RANK |
|---|---:|---:|---:|---:|
| A | 900 | 1 | 1 | 1 |
| B | 800 | 2 | 2 | 2 |
| C | 800 | 3 | 2 | 2 |
| D | 700 | 4 | 4 | 3 |

Use:
- ROW_NUMBER when every row needs a unique sequence.
- RANK when ties share a position and gaps are meaningful.
- DENSE_RANK when ties share a position but the next distinct value should receive the next consecutive rank.

---

## 9. PARTITION BY: restart the calculation for each group

Without PARTITION BY, the entire result set is one window.

With PARTITION BY customer_id, each customer gets a separate window.

```sql
SELECT
    order_id,
    customer_id,
    ROW_NUMBER() OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
    ) AS customer_sequence
FROM orders
ORDER BY customer_id, order_date;
```

Customer 1's sequence is 1, 2. Customer 2's sequence is 1.

PARTITION BY does not physically split the table or permanently group records. It defines independent calculation groups for the window function.

### 9.1 Partition by more than one column

You can partition by multiple expressions:

```sql
ROW_NUMBER() OVER (
    PARTITION BY customer_id, order_status
    ORDER BY order_date
)
```

The sequence restarts for each unique customer_id and order_status combination.

Use this only when the business requirement needs that grouping.

---

## 10. LAG: read a value from a previous row

A manager wants to compare each customer's order date with their previous order date.

LAG accesses a value from a preceding row in the ordered window.

### 10.1 First LAG example

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    LAG(order_date) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
    ) AS previous_order_date
FROM orders
ORDER BY customer_id, order_date;
```

Expected:

| order_id | customer_id | order_date | previous_order_date |
|---:|---:|---|---|
| 5001 | 1 | 2026-11-01 | NULL |
| 5003 | 1 | 2026-11-03 | 2026-11-01 |
| 5002 | 2 | 2026-11-02 | NULL |

For the first order of each customer, there is no earlier row, so LAG returns NULL by default.

### 10.2 LAG with an offset

LAG(value, 2) accesses the value two rows before the current row, if available.

```sql
LAG(order_date, 2) OVER (
    PARTITION BY customer_id
    ORDER BY order_date
)
```

If a customer has fewer than three orders, the result is NULL for that row.

### 10.3 LAG with a default value

```sql
LAG(order_date, 1, DATE '1900-01-01') OVER (
    PARTITION BY customer_id
    ORDER BY order_date
)
```

The third argument is used when the requested previous row does not exist. Choose a default carefully. A fake historical date can be misleading in business reports; NULL is often more honest.

---

## 11. LEAD: read a value from a following row

LEAD is the opposite direction of LAG. It accesses a value from a later row in the ordered window.

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    LEAD(order_date) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
    ) AS next_order_date
FROM orders
ORDER BY customer_id, order_date;
```

Expected:

| order_id | customer_id | order_date | next_order_date |
|---:|---:|---|---|
| 5001 | 1 | 2026-11-01 | 2026-11-03 |
| 5003 | 1 | 2026-11-03 | NULL |
| 5002 | 2 | 2026-11-02 | NULL |

For the last order of each customer, there is no following row, so LEAD returns NULL.

### 11.1 LAG versus LEAD

| Function | Direction | Typical use |
|---|---|---|
| LAG | Previous row | Compare with prior month/order |
| LEAD | Following row | Look ahead to next event/order |

Both need a meaningful ORDER BY if the previous or next row must have business meaning.

---

## 12. Compare current and previous values

Suppose the manager wants the change in order date sequence. Date subtraction can calculate the number of days between dates in PostgreSQL.

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    LAG(order_date) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
    ) AS previous_order_date,
    order_date - LAG(order_date) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
    ) AS days_since_previous_order
FROM orders
ORDER BY customer_id, order_date;
```

Expected:
- Asha's first order: previous date NULL, difference NULL.
- Asha's second order: previous date 2026-11-01, difference 2 days.
- Ravi's only order: previous date NULL, difference NULL.

The calculation is meaningful only after partitioning by customer. Without partitioning, Ravi's previous row might incorrectly become Asha's order.

---

## 13. Ranking business totals

Now use the actual TechStore order data.

### 13.1 Calculate each customer's spend first

```sql
WITH customer_sales AS (
    SELECT
        o.customer_id,
        SUM(oi.quantity * oi.unit_price) AS total_spend
    FROM orders AS o
    JOIN order_items AS oi
      ON o.order_id = oi.order_id
    GROUP BY o.customer_id
)
SELECT *
FROM customer_sales;
```

Expected:
- Customer 1: 5,600
- Customer 2: 8,500

### 13.2 Rank customers by spend

```sql
WITH customer_sales AS (
    SELECT
        o.customer_id,
        SUM(oi.quantity * oi.unit_price) AS total_spend
    FROM orders AS o
    JOIN order_items AS oi
      ON o.order_id = oi.order_id
    GROUP BY o.customer_id
)
SELECT
    customer_id,
    total_spend,
    RANK() OVER (
        ORDER BY total_spend DESC
    ) AS spend_rank
FROM customer_sales
ORDER BY spend_rank;
```

Expected:
- Customer 2: 8,500, rank 1
- Customer 1: 5,600, rank 2

The CTE prepares one row per customer. The window function ranks those summary rows.

### 13.3 Include customers with zero spend

```sql
WITH customer_sales AS (
    SELECT
        c.customer_id,
        c.customer_name,
        COALESCE(SUM(oi.quantity * oi.unit_price), 0) AS total_spend
    FROM customers AS c
    LEFT JOIN orders AS o
      ON c.customer_id = o.customer_id
    LEFT JOIN order_items AS oi
      ON o.order_id = oi.order_id
    GROUP BY c.customer_id, c.customer_name
)
SELECT
    customer_id,
    customer_name,
    total_spend,
    DENSE_RANK() OVER (
        ORDER BY total_spend DESC
    ) AS spend_rank
FROM customer_sales
ORDER BY spend_rank, customer_id;
```

Expected:
- Ravi: 8,500, rank 1
- Asha: 5,600, rank 2
- Meena: 0, rank 3

Meena is included because the customer summary begins with customers and uses LEFT JOIN.

---

## 14. Top N records per group

Business question: Show the latest order for every customer.

This is a common pattern in ETL pipelines, deduplication and current-state tables.

```sql
WITH ranked_orders AS (
    SELECT
        order_id,
        customer_id,
        order_date,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date DESC, order_id DESC
        ) AS row_num
    FROM orders
)
SELECT order_id, customer_id, order_date
FROM ranked_orders
WHERE row_num = 1
ORDER BY customer_id;
```

Expected:
- Customer 1: order 5003
- Customer 2: order 5002

### 14.1 Top two order items per order

```sql
WITH ranked_items AS (
    SELECT
        order_item_id,
        order_id,
        product_id,
        quantity,
        ROW_NUMBER() OVER (
            PARTITION BY order_id
            ORDER BY quantity DESC, order_item_id
        ) AS item_rank
    FROM order_items
)
SELECT *
FROM ranked_items
WHERE item_rank <= 2
ORDER BY order_id, item_rank;
```

This example returns up to two item rows per order. Order 5001 has two lines; orders 5002 and 5003 each have one.

---

## 15. Window ORDER BY versus final ORDER BY

These are not the same.

Inside OVER:

```sql
ROW_NUMBER() OVER (ORDER BY order_date)
```

This controls the order used to calculate row numbers.

Outside OVER:

```sql
ORDER BY customer_id
```

This controls the order in which the final result is displayed.

A window ORDER BY does not guarantee the final output order. If the report needs sorted output, add a final ORDER BY.

---

## 16. Window frame: an introduction

Some window functions use a frame, which is the subset of rows considered relative to the current row.

A running total is a common example.

```sql
SELECT
    order_id,
    order_date,
    SUM(order_id) OVER (
        ORDER BY order_date, order_id
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_order_id_total
FROM orders
ORDER BY order_date, order_id;
```

This example is only to illustrate frame syntax; summing order IDs is not a meaningful business metric.

Understand the frame words:
- UNBOUNDED PRECEDING: start at the first row in the partition.
- CURRENT ROW: stop at the current row.
- ROWS: count physical rows in the ordered frame.

For a real running sales total, first calculate one row per day or order and sum the sales amount over that ordered sequence.

---

## 17. Common mistakes

### Mistake 1: Confusing GROUP BY with a window function

GROUP BY reduces detail rows. A window function keeps them.

### Mistake 2: Forgetting PARTITION BY

Without it, a calculation may run across all customers instead of restarting per customer.

### Mistake 3: Assuming PARTITION BY sorts rows

PARTITION BY defines groups. ORDER BY defines order inside the window.

### Mistake 4: Thinking window ORDER BY sorts the final result

Add an outer ORDER BY for final presentation.

### Mistake 5: Adding a unique tie-breaker to RANK unintentionally

If you add customer_id to the RANK ordering, equal sales totals may no longer tie. Use only the business ranking measure when ties should share rank.

### Mistake 6: Using ROW_NUMBER when ties should share rank

ROW_NUMBER always assigns unique numbers. Use RANK or DENSE_RANK when the business requires tied positions.

### Mistake 7: Filtering a window result in WHERE at the same level

Calculate the window result in a CTE or subquery, then filter it in the outer query.

### Mistake 8: Forgetting NULL at the first or last row

LAG has no previous row for the first row in a partition. LEAD has no next row for the last row.

### Mistake 9: Using an incomplete ordering

If multiple rows tie on the window ORDER BY values, previous/next and row numbering may be nondeterministic. Add a stable tie-breaker where needed.

### Mistake 10: Ranking raw detail rows instead of business totals

If the requirement is rank customers by spend, calculate spend per customer first, then rank the customer summary.

---

## 18. Mini-project: Historical Trend Query Engine

### Business request

TechStore wants an analysis query that can:
1. Number each customer's orders chronologically.
2. Show each customer's previous order date.
3. Show the days between orders.
4. Calculate each customer's total spend.
5. Rank customers by spend.
6. Identify the latest order per customer.

### Task 1: chronological order sequence

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    ROW_NUMBER() OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
    ) AS customer_order_number
FROM orders
ORDER BY customer_id, order_date;
```

Expected: Asha's orders numbered 1 and 2; Ravi's order numbered 1.

### Task 2: previous order date

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    LAG(order_date) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
    ) AS previous_order_date
FROM orders
ORDER BY customer_id, order_date;
```

Expected: Asha's second order shows 2026-11-01 as the previous date.

### Task 3: days between orders

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    order_date - LAG(order_date) OVER (
        PARTITION BY customer_id
        ORDER BY order_date, order_id
    ) AS days_between_orders
FROM orders
ORDER BY customer_id, order_date;
```

Expected: Asha's second order has 2 days. First orders have NULL.

### Task 4: rank customer spend

Use a customer-level aggregate CTE, then RANK over total_spend. Expected: Ravi rank 1, Asha rank 2, and Meena rank 3 if customers without orders are included with zero spend.

### Task 5: latest order per customer

Use ROW_NUMBER partitioned by customer_id and ordered by order_date descending, then filter row_num = 1 in an outer query. Expected: 5003 for customer 1 and 5002 for customer 2.

### Project validation

- [ ] The output has one row per order for the order sequence.
- [ ] Row numbering restarts for each customer.
- [ ] The first order per customer has no previous date.
- [ ] The gap for Asha's two orders is two days.
- [ ] Customer spend totals reconcile to 14,100.
- [ ] The ranking is based on customer spend, not individual order-item rows.
- [ ] The latest order query returns one row per customer with at least one order.
- [ ] Ties have a documented ranking rule.

---

## 19. Practice exercises

1. Number all orders by date.
2. Number orders separately for each customer.
3. Rank products by price.
4. Create a sample where two products tie and compare RANK with DENSE_RANK.
5. Use LAG to show the previous order date for each customer.
6. Use LEAD to show the next order date for each customer.
7. Calculate the gap in days between consecutive orders.
8. Rank customers by total spend.
9. Return the latest order for every customer.
10. Explain why the first LAG result in each partition is NULL.
11. Explain why ROW_NUMBER always produces unique numbers.
12. Explain why a window ORDER BY does not guarantee final result ordering.

### Answer hints

1. ROW_NUMBER() OVER (ORDER BY order_date, order_id).
2. Add PARTITION BY customer_id.
3. RANK() OVER (ORDER BY price DESC).
4. Use tied sales values such as 900, 800, 800, 700.
5. LAG(order_date) with customer partition.
6. LEAD(order_date) with customer partition.
7. Subtract LAG(order_date) from order_date.
8. Aggregate customer spend first, then rank it.
9. ROW_NUMBER descending within each customer, filter 1 outside.
10. No earlier row exists in that partition.
11. It assigns a sequential number to each row, including tied values.
12. The window order is for calculation; outer ORDER BY controls presentation.

---

## 20. Day 23 revision

- A window function calculates across related rows while retaining row-level detail.
- OVER defines the window.
- PARTITION BY divides rows into independent calculation groups.
- ORDER BY defines the sequence inside the window.
- ROW_NUMBER assigns a unique sequence.
- RANK gives ties the same rank and leaves gaps.
- DENSE_RANK gives ties the same rank without gaps.
- LAG reads a previous row's value.
- LEAD reads a following row's value.
- Window ORDER BY is not final output ORDER BY.
- A window result generally needs an outer query or CTE before it can be filtered.
- Identify the business grain before ranking.
- Add deterministic tie-breakers when a unique row sequence is required.

**You are ready for Day 24 when you can explain and write all five required window functions and show why PARTITION BY changes the calculation.**
