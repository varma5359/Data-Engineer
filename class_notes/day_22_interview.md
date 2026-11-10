# Day 22 Interview Preparation: SQL Aggregations and Advanced Queries

**Phase:** 3 — SQL and Database Engineering  
**Level:** Beginner to junior Data Engineer  
**Mini-project:** Business Data Aggregation  
**Database:** PostgreSQL

---

## 1. How to answer aggregation questions

For each answer, explain:
1. What the SQL feature does.
2. Which business question it solves.
3. A short query.
4. How to interpret the output.
5. A common edge case.

The TechStore training data has three orders and four order-item rows. Total line value is 14,100.

---

## 2. Core questions and complete answers

### 1. What is an aggregate function?

An aggregate function summarizes values from multiple rows and returns a summary value. Examples include COUNT, SUM, AVG, MIN and MAX.

For example:

```sql
SELECT COUNT(*) FROM orders;
```

It returns the number of order rows.

### 2. What is COUNT(*)?

COUNT(*) counts all rows in the input set, including rows where some columns contain NULL.

```sql
SELECT COUNT(*) AS total_orders
FROM orders;
```

In TechStore it returns 3.

### 3. COUNT(*) versus COUNT(column)?

COUNT(*) counts rows. COUNT(column) counts only rows where that column is not NULL.

If a four-row table has two non-NULL ratings and two NULL ratings, COUNT(*) returns 4 while COUNT(rating) returns 2.

### 4. What is COUNT(DISTINCT column)?

It counts distinct non-NULL values.

```sql
SELECT COUNT(DISTINCT customer_id)
FROM orders;
```

It returns 2 because customers 1 and 2 placed orders. Customer 1 appears twice but is counted once.

### 5. What does SUM do?

SUM adds numeric values across the input rows.

```sql
SELECT SUM(quantity * unit_price)
FROM order_items;
```

It returns 14,100. Each row's quantity is multiplied by its unit price, then the results are added.

### 6. What does AVG do?

AVG calculates the arithmetic average of non-NULL numeric values.

```sql
SELECT AVG(price)
FROM products;
```

It returns 2,975 because the catalogue prices total 11,900 across four products.

### 7. Does AVG include NULL values in its denominator?

No. AVG ignores NULL input values. If the values are 10, NULL and 20, the average is 15, not 10.

### 8. What do MIN and MAX do?

MIN returns the smallest value and MAX returns the largest value in the input set. They can be used with numeric values, dates and text, with text behavior determined by collation.

### 9. What happens when SUM receives NULL?

SUM ignores NULL inputs. If every input is NULL, the result is NULL. COALESCE can provide a fallback value, but converting missing values to zero must match the business meaning.

### 10. What is GROUP BY?

GROUP BY divides rows into groups based on one or more expressions. Aggregate functions then calculate a summary for each group.

```sql
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id;
```

It returns two groups: customer 1 has two orders and customer 2 has one.

### 11. Why do we use GROUP BY?

It answers questions such as sales by customer, orders by month, products by category or error counts by pipeline. It changes the result grain from individual records to groups.

### 12. What is the grain of an aggregate result?

Grain describes what one output row represents. A query grouped by order_id has one row per order. A query grouped by customer_id has one row per customer with qualifying orders.

Identifying grain prevents double counting and incorrect report interpretation.

### 13. What is the difference between WHERE and HAVING?

WHERE filters individual input rows before grouping. HAVING filters groups after aggregation.

WHERE example:

```sql
WHERE order_date >= DATE '2026-11-01'
```

HAVING example:

```sql
HAVING COUNT(*) > 1
```

Use HAVING when the condition depends on an aggregate result.

### 14. Can we use COUNT(*) in WHERE?

Not to filter grouped counts in the same query. WHERE is evaluated before grouping. Use HAVING COUNT(*) > ... for that purpose.

### 15. Can WHERE and HAVING be used together?

Yes. WHERE can select the source rows, GROUP BY forms groups, and HAVING keeps only groups meeting an aggregate condition.

### 16. What is a scalar subquery?

A scalar subquery returns one value, such as one number or one date.

```sql
SELECT product_name, price
FROM products
WHERE price > (
    SELECT AVG(price)
    FROM products
);
```

The inner query returns one average price; the outer query returns products above it.

### 17. What happens if a scalar subquery returns more than one row?

A scalar comparison expects one value. If the subquery returns multiple rows, many database systems raise a cardinality error. Use IN, EXISTS or a query that correctly reduces the result when multiple values are intended.

### 18. What is a subquery?

A subquery is a SELECT statement nested inside another SQL statement. It can appear in WHERE, FROM or the SELECT list, depending on the query.

### 19. What is a subquery in WHERE?

It lets the outer query filter rows based on another query's result. For example, select customers whose IDs appear in the orders table.

### 20. What is IN?

IN checks whether a value matches one of the values returned by a list or subquery. It is useful for membership tests.

### 21. What is the NULL risk with NOT IN?

If a NOT IN subquery returns NULL, comparisons may become UNKNOWN, causing rows that a beginner expects to keep to be excluded. NOT EXISTS is often safer for anti-match logic.

### 22. What is EXISTS?

EXISTS tests whether a subquery returns at least one row. It is commonly used to test whether a related record exists.

### 23. What is NOT EXISTS?

NOT EXISTS returns TRUE when the subquery returns no rows. It is useful for finding customers without orders, products never sold or parent records without child records.

### 24. What is a correlated subquery?

A correlated subquery refers to a column from the outer query. For each outer row, it evaluates a condition using that row's values.

Example: count orders for the current customer and return customers whose count is greater than one.

### 25. Correlated versus non-correlated subquery?

A non-correlated subquery can be evaluated independently of the outer query. A correlated subquery refers to the current outer row. The optimizer may transform either query, so do not assume the correlated version always executes literally once per row.

### 26. What is a derived table?

A derived table is a subquery in the FROM clause. It acts as a table-like result for the outer query and usually requires an alias.

### 27. What is a CTE?

A Common Table Expression is a named query result introduced with WITH. It exists for one SQL statement and can make multi-step queries easier to understand.

### 28. Is a CTE a permanent table?

No. A normal CTE is not a permanent database object. It is scoped to the statement in which it is defined.

### 29. CTE versus subquery?

Both can express many of the same calculations. A CTE gives a named step and is often easier to read when a query has several stages. A subquery can be more compact for a small one-off calculation. Performance depends on the database optimizer and query.

### 30. Can one query have multiple CTEs?

Yes. Multiple CTEs are separated by commas after WITH. A later CTE can refer to an earlier CTE.

### 31. What is a recursive CTE?

A recursive CTE refers to its own result and is useful for hierarchical or graph-like traversal, such as employee reporting trees or category hierarchies. Recursive CTEs are beyond today's core practice but are a useful next SQL topic.

### 32. What is COALESCE?

COALESCE returns the first non-NULL expression in its argument list.

```sql
COALESCE(customer_spend, 0)
```

It returns zero when customer_spend is NULL. Use it only when zero is the correct business interpretation.

### 33. How do you calculate order value?

For an order with multiple item rows, calculate quantity multiplied by unit_price for each row and sum those values grouped by order_id.

```sql
SELECT order_id,
       SUM(quantity * unit_price) AS order_total
FROM order_items
GROUP BY order_id;
```

### 34. Average line value versus average order value?

Average line value averages individual order-item values. Average order value first calculates a total per order and then averages those order totals. They are different because an order may contain multiple lines.

### 35. How can joins inflate aggregate values?

Joining two one-to-many detail tables at their parent key can multiply combinations. If an order has two items and three payments, the combined join may produce six rows. Summing item amounts after that join can count each item three times.

### 36. How do you prevent double counting?

Identify each table's grain and relationship cardinality. Aggregate each detail table to the required common grain before joining, or use a query design that preserves the intended row count. Reconcile totals against the source detail.

### 37. Is GROUP BY the same as DISTINCT?

No. DISTINCT removes duplicate selected combinations. GROUP BY forms groups and is commonly used with aggregate functions. A query may use GROUP BY without an aggregate, but its main purpose is grouping.

### 38. Can aggregate functions be nested?

Many SQL engines do not allow directly nesting aggregate functions at the same query level, such as AVG(SUM(amount)). Use a subquery or CTE to calculate the first aggregation, then aggregate that result in an outer query.

### 39. How do you find the second highest price?

One approach is a subquery using the maximum price below the overall maximum:

```sql
SELECT MAX(price) AS second_highest_distinct_price
FROM products
WHERE price < (SELECT MAX(price) FROM products);
```

This returns the second-highest distinct price. If ties or ranking semantics matter, use a ranking approach and define the expected behavior.

### 40. How do you validate a sales summary?

Check source row counts, output grain, duplicate keys, NULL behavior, date filters, business status rules and total reconciliation. Compare the summary total with a trusted detail-level calculation.

---

## 3. Scenario-based questions

### Scenario 1: The sales total doubles after adding a payments table

**Good answer:** I would inspect the grain and cardinality of order_items and payments. Both can contain multiple rows per order. Joining them directly can multiply item rows by payment rows. I would aggregate each table to one row per order before joining, then compare the final totals with the original detail totals.

### Scenario 2: The manager asks for customers who spent more than 5,000

**Good answer:** I would join customers to orders and order_items, calculate line totals, group by customer, and use HAVING SUM(quantity * unit_price) > 5000. I would confirm whether cancelled, refunded or unpaid orders should be excluded.

### Scenario 3: The average order value looks too high

**Good answer:** I would check whether the query averages line totals instead of order totals, whether joins duplicate rows, whether NULLs are excluded, and whether the business definition includes taxes, refunds or cancelled orders. I would calculate one row per order first and then average that result.

### Scenario 4: A report must include customers who have never ordered

**Good answer:** Start from customers and LEFT JOIN an aggregated order summary. Use COALESCE for the order count and spend only if the business wants zero for no activity. Keep average order value NULL because no order exists to average.

### Scenario 5: A correlated subquery is slow on a large table

**Good answer:** I would inspect the execution plan and indexes, then compare the correlated query with a grouped join or CTE. The optimizer may decorrelate some queries, so I would measure rather than assume. I would also check filtering, cardinality and statistics.

### Scenario 6: The report total does not match finance

**Good answer:** First clarify the metric definition. Then compare included statuses, date boundaries, refunds, discounts, taxes, currency, NULL handling and duplicate rows. Reconcile from transaction detail to order totals to customer totals, documenting each step.

---

## 4. Mini-project defence questions

Be ready to explain:

1. What business report did you build?
2. What is the grain of each intermediate result?
3. Why did you calculate order totals before customer totals?
4. Why is Asha's spend 5,600?
5. Why is Meena's spend zero but her average order value NULL?
6. What is the difference between WHERE and HAVING in your project?
7. Why did you use a CTE?
8. How did you avoid a double-counting join?
9. How did you validate the total 14,100?
10. What business rules would you confirm before calling this revenue?

A strong answer makes clear that SQL correctness includes business definitions, not only syntax.

---

## 5. Short practical challenge

Write queries to:
- Count orders.
- Sum line values.
- Calculate order totals.
- Count orders by customer.
- Find customers with more than one order.
- Find products above average price.
- Find customers with orders using EXISTS.
- Find customers without orders using NOT EXISTS.
- Build a two-CTE customer summary.
- Explain why the summary total reconciles to 14,100.

---

## 6. Common interview traps

- Saying COUNT(column) counts NULL values.
- Treating NULL as zero without a business rule.
- Using WHERE for aggregate conditions.
- Grouping at the wrong grain.
- Calling average line value average order value.
- Using DISTINCT to hide a multiplied join.
- Assuming CTEs always improve performance.
- Assuming NOT IN is safe when the subquery can return NULL.
- Forgetting that a customer with no orders will not appear in an INNER JOIN.
- Reporting sales as revenue without confirming status, refunds, discounts and tax rules.

---

## 7. Readiness checklist

- [ ] I can explain COUNT, SUM, AVG, MIN and MAX.
- [ ] I understand COUNT(*) versus COUNT(column).
- [ ] I can group records by customer and order.
- [ ] I can explain WHERE versus HAVING.
- [ ] I can write a scalar subquery.
- [ ] I can use IN, EXISTS and NOT EXISTS.
- [ ] I can explain a correlated subquery.
- [ ] I can write and read one or more CTEs.
- [ ] I understand the grain of every intermediate result.
- [ ] I can prevent double-counting after joins.
- [ ] I can reconcile a business summary with source records.
