# Day 21 Interview Preparation: SQL Data Extraction

**Phase:** 3 — SQL and Database Engineering  
**Level:** Beginner to junior Data Engineer  
**Mini-project:** Multi-Table Extraction Queries  
**Database:** PostgreSQL (with portable SQL concepts)

---

## 1. How to use this guide

Do not memorize the answer word for word. For each question:

1. Start with a clear definition.
2. Explain why a Data Engineer needs it.
3. Give a small example.
4. Explain the result.
5. Mention one common mistake or limitation.

The examples use the TechStore customers, products, orders and order_items tables from class.

---

## 2. Core questions with complete answers

### 1. What is SQL?

SQL stands for Structured Query Language. It is used to communicate with relational database systems. A Data Engineer uses SQL to read data, create and modify database structures, load records, transform data and validate results.

Example:

```sql
SELECT customer_name
FROM customers;
```

This asks the database to return the customer_name column from customers.

**Follow-up:** Is SQL a programming language?

**Answer:** SQL is a declarative query language. We describe what result we want. The database optimizer chooses how to execute the query. It is commonly called a language rather than a general-purpose programming language.

### 2. What is a SQL query?

A query is a statement sent to a database to request information or perform an operation. For example, SELECT is used to retrieve data. Queries can be simple, such as reading one column, or complex, such as joining and filtering multiple tables.

**Follow-up:** Does every query change data?

**Answer:** No. SELECT normally reads data. INSERT, UPDATE and DELETE modify rows. CREATE and ALTER modify database structures.

### 3. What is the purpose of SELECT?

SELECT identifies the columns or expressions that should appear in the result.

```sql
SELECT customer_name, city
FROM customers;
```

It returns the customer's name and city. It does not modify the stored table.

**Follow-up:** What does SELECT * mean?

**Answer:** It requests every column from the selected source. It is useful for quick inspection, but production queries should usually specify required columns to avoid unnecessary data transfer and unexpected schema changes.

### 4. What is the purpose of FROM?

FROM identifies the table or source from which rows are read.

```sql
SELECT customer_name
FROM customers;
```

customers is the source table.

**Follow-up:** Can FROM refer to more than one table?

**Answer:** Yes. We can use JOINs or other table expressions to combine multiple sources.

### 5. What is the difference between SELECT and WHERE?

SELECT chooses which columns appear in the result. WHERE filters which rows qualify.

```sql
SELECT customer_name, city
FROM customers
WHERE city = 'Hyderabad';
```

SELECT chooses customer_name and city. WHERE keeps only rows whose city is Hyderabad.

**Follow-up:** Does WHERE change the original table?

**Answer:** No. In a SELECT query it filters the result. It does not delete or update stored rows.

### 6. What is WHERE?

WHERE is a filtering clause. It keeps rows for which its condition evaluates to TRUE.

Example:

```sql
SELECT product_name, price
FROM products
WHERE price > 1000;
```

It returns Keyboard, Monitor and Laptop Stand in the training data.

**Follow-up:** What happens when a condition evaluates to UNKNOWN because of NULL?

**Answer:** WHERE does not keep that row. SQL uses TRUE, FALSE and UNKNOWN logic.

### 7. Explain AND and OR.

AND requires all combined conditions to be TRUE. OR requires at least one condition to be TRUE.

```sql
SELECT product_name
FROM products
WHERE category = 'Accessories'
  AND price > 1000;
```

This requires both conditions, so Keyboard and Laptop Stand qualify.

**Follow-up:** Why use parentheses?

**Answer:** Parentheses make the intended grouping explicit when AND and OR appear together. AND has higher precedence than OR in SQL, but parentheses make code easier to review and reduce logic mistakes.

### 8. What is the difference between = and <>?

= checks equality. <> checks inequality.

```sql
SELECT customer_name
FROM customers
WHERE city <> 'Chennai';
```

This returns customers whose city is not Chennai, excluding rows where city is NULL because the comparison is UNKNOWN.

### 9. What is IN?

IN checks whether a value matches one of the values in a list.

```sql
SELECT customer_name
FROM customers
WHERE city IN ('Hyderabad', 'Chennai');
```

It returns Asha and Ravi.

**Follow-up:** Is IN equivalent to OR?

**Answer:** For ordinary non-NULL equality comparisons, yes. IN is shorter and easier to maintain for a list of allowed values.

### 10. What is NOT IN and what is its NULL risk?

NOT IN excludes values in a list. If the list contains NULL, comparisons can become UNKNOWN and may unexpectedly return no rows.

For anti-join logic against a subquery that may contain NULL, NOT EXISTS is often safer.

### 11. What is BETWEEN?

BETWEEN checks whether a value is within a range, including both endpoints.

```sql
SELECT product_name
FROM products
WHERE price BETWEEN 700 AND 1500;
```

Mouse, Keyboard and Laptop Stand qualify.

**Follow-up:** Is BETWEEN inclusive?

**Answer:** Yes. It includes 700 and 1500 in this example.

### 12. What is LIKE?

LIKE matches text against a pattern. % represents zero or more characters; _ represents exactly one character.

```sql
SELECT customer_name
FROM customers
WHERE customer_name LIKE 'A%';
```

It returns Asha.

**Follow-up:** Is LIKE case-sensitive?

**Answer:** It depends on the database and collation. PostgreSQL LIKE is case-sensitive; PostgreSQL ILIKE is case-insensitive. MySQL behavior depends on collation.

### 13. What is NULL?

NULL represents a missing or unknown value. It is not the same as zero or an empty string.

To test it:

```sql
SELECT customer_name
FROM customers
WHERE city IS NULL;
```

Do not use city = NULL. Comparisons with NULL produce UNKNOWN.

### 14. What is DISTINCT?

DISTINCT removes duplicate combinations from the selected output columns.

```sql
SELECT DISTINCT category
FROM products;
```

It returns Accessories and Electronics.

**Follow-up:** Should you use DISTINCT to remove duplicate rows after a JOIN?

**Answer:** Not automatically. First inspect the join keys and the intended row grain. DISTINCT can hide an incorrect join or unexpected one-to-many relationship.

### 15. What is ORDER BY?

ORDER BY sorts the returned result by one or more expressions.

```sql
SELECT product_name, price
FROM products
ORDER BY price DESC;
```

DESC means highest to lowest. ASC means lowest to highest and is the default.

**Follow-up:** Is row order guaranteed without ORDER BY?

**Answer:** No. SQL does not promise a stable order unless ORDER BY is specified.

### 16. What is LIMIT?

LIMIT restricts the maximum number of rows returned.

```sql
SELECT product_name, price
FROM products
ORDER BY price DESC
LIMIT 2;
```

It returns the two most expensive products.

**Follow-up:** Why is ORDER BY important with LIMIT?

**Answer:** Without ORDER BY, the database may return any qualifying rows. A top-two request needs a defined sort order.

### 17. What is OFFSET?

OFFSET skips a specified number of rows before returning the limited result. It is commonly used for page-based reports.

Large offsets can be inefficient and can lead to inconsistent pages if the underlying data changes between requests.

### 18. What is keyset pagination?

Keyset pagination uses the last-seen sort key to request the next page.

Example:

```sql
SELECT order_id, order_date
FROM orders
WHERE order_id > 5002
ORDER BY order_id
LIMIT 100;
```

It can be more efficient than skipping a large number of rows. The sort key must be stable and unique, or the cursor must include enough columns to define a unique ordering.

### 19. What is a JOIN?

A JOIN combines rows from two or more tables based on a related condition.

In TechStore, orders.customer_id references customers.customer_id. Joining them allows a report to show order IDs with customer names.

### 20. What is the ON clause?

ON defines the matching condition between joined sources.

```sql
FROM customers AS c
JOIN orders AS o
  ON c.customer_id = o.customer_id
```

It matches orders to the customer who placed them.

**Follow-up:** What if the ON condition is missing or incorrect?

**Answer:** The query may create a Cartesian product or incorrect matches, producing too many rows or wrong business results.

### 21. What is INNER JOIN?

INNER JOIN returns only rows with matching values in both joined sources.

In the sample, Meena has no order, so she does not appear in an INNER JOIN between customers and orders.

### 22. What is LEFT JOIN?

LEFT JOIN preserves all rows from the left input and returns matching right-side values where available. If no right-side row matches, right-side columns are NULL.

It is useful when a report must include customers with no orders.

### 23. What is RIGHT JOIN?

RIGHT JOIN preserves all rows from the right input and returns matching left-side values where available.

It can often be rewritten as a LEFT JOIN by reversing the input order. Many teams prefer LEFT JOIN for consistent readability.

### 24. INNER JOIN versus LEFT JOIN?

INNER JOIN removes unmatched rows from both sides. LEFT JOIN preserves every row from the left side and fills right-side columns with NULL when there is no match.

Choose based on the business requirement, not based on which syntax is shorter.

### 25. How do you find customers who never placed an order?

Use LEFT JOIN and test a non-nullable right-side key for NULL.

```sql
SELECT c.customer_name
FROM customers AS c
LEFT JOIN orders AS o
  ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

Expected: Meena.

NOT EXISTS is another clear pattern.

### 26. Why can a LEFT JOIN behave like an INNER JOIN?

If a WHERE condition filters a right-side column, unmatched rows contain NULL and may be removed.

To preserve left rows while limiting matching right rows, put the right-side condition in ON.

### 27. Why does a JOIN increase the number of rows?

A one-to-many relationship allows one left row to match multiple right rows. A customer with three orders produces three joined rows.

This is correct when the output grain is one row per order. It is a problem only if the report expected one row per customer.

### 28. What is a Cartesian product?

A Cartesian product combines every row from one source with every row from another. Three customers crossed with four products produces twelve rows.

CROSS JOIN is intentional syntax for this behavior. Missing join conditions can accidentally produce the same result.

### 29. What is a table alias?

An alias is a temporary name used within a query.

```sql
FROM customers AS c
JOIN orders AS o
```

c refers to customers and o refers to orders. Aliases shorten queries and help distinguish columns with identical names.

### 30. Why qualify column names?

If both joined tables have customer_id, writing customer_id alone can be ambiguous. Use c.customer_id or o.customer_id to identify the intended source.

### 31. What is the difference between a primary key and a foreign key?

A primary key uniquely identifies a row in its table. A foreign key references a key in another table and enforces a relationship.

In TechStore, customers.customer_id is a primary key and orders.customer_id is a foreign key.

### 32. What is the grain of a query result?

Grain describes what one output row represents.

Examples:
- One row per customer.
- One row per order.
- One row per order item.

You should identify the grain before joining tables or aggregating values. Many duplicate-counting errors happen because the grain is not understood.

### 33. How do you validate a multi-table extract?

Check:
- Expected columns and data types.
- Total row count.
- Uniqueness at the expected grain.
- Null values in required fields.
- Unmatched relationships.
- Duplicate business keys.
- Business totals against a trusted source.
- A sample of records against source tables.

### 34. Why avoid SELECT * in production?

It returns unnecessary fields, can expose sensitive columns, transfers extra data and makes downstream schemas vulnerable to unrelated table changes. Explicit columns make the extract contract clear.

### 35. What is the difference between WHERE and ON?

ON defines how rows match during a join. WHERE filters rows from the joined result. For outer joins, moving a condition between ON and WHERE can change whether unmatched rows are preserved.

### 36. Can DISTINCT guarantee one row per customer?

Only if the selected output combination has one unique combination per customer. If other selected columns vary, a customer can still appear more than once. Use the correct grain and aggregation logic.

### 37. How would you extract the latest orders?

Use ORDER BY on the date and a deterministic tie-breaker, then LIMIT.

```sql
SELECT order_id, order_date
FROM orders
ORDER BY order_date DESC, order_id DESC
LIMIT 10;
```

For a large production dataset, ensure the ordering and index strategy support the workload.

### 38. What happens if you use LIMIT without ORDER BY?

The query returns up to that many rows, but which rows are returned is not guaranteed. It is not a reliable top-N or latest-N query.

---

## 3. Scenario-based interview questions

### Scenario 1: A customer is missing from a customer report

**Situation:** The report uses INNER JOIN between customers and orders. A customer with no order does not appear.

**Good answer:** First confirm the report requirement. If all customers must appear, use LEFT JOIN with customers as the left input. Right-side filters must be placed carefully in ON if unmatched customers should remain. Validate the output count against the customers table.

### Scenario 2: Revenue doubles after a JOIN

**Situation:** The sales total was correct before a JOIN but doubles after adding another detail table.

**Good answer:** Check the grain of each table and the relationship cardinality. A one-to-many join may multiply order lines. Count rows before and after the join, inspect key uniqueness, and aggregate each detail source at the correct grain before combining if necessary. Do not hide the issue with DISTINCT.

### Scenario 3: A query returns 10,000 rows instead of 100

**Situation:** The query joins two tables and unexpectedly returns far more rows.

**Good answer:** Check whether the join condition is missing, whether the wrong columns were matched, or whether the relationship is one-to-many. Compare source counts, joined counts and distinct key counts. Inspect sample records.

### Scenario 4: A dashboard needs customers with no orders

**Good answer:** Use customers LEFT JOIN orders and filter where the order primary key is NULL. Confirm that the chosen right-side key cannot be NULL for a real order.

### Scenario 5: The top 10 report changes unpredictably

**Good answer:** Add ORDER BY and a unique tie-breaker. LIMIT without a deterministic ordering does not define which records should be returned.

### Scenario 6: Page 40 of a large report is slow

**Good answer:** Large OFFSET pagination may require scanning/skipping many rows. Consider keyset pagination using a stable indexed key. Confirm the required ordering and ensure the cursor condition handles ties.

### Scenario 7: A downstream pipeline receives duplicate customers

**Good answer:** Determine whether the extract grain is customer, order or order item. Inspect one-to-many joins. If one row per customer is required, aggregate or select records according to an explicit business rule. Do not apply DISTINCT without understanding why duplicates exist.

### Scenario 8: A filter removes customers without orders

**Good answer:** Inspect whether the right table's column is filtered in WHERE after a LEFT JOIN. If all left rows must remain, move the right-side matching condition into ON.

---

## 4. Mini-project defence: Multi-Table Extraction Queries

Be prepared to explain:

1. What business report did you build?
2. Which four tables did you use?
3. What is the primary key of each table?
4. Which foreign keys connect the tables?
5. Why did you use INNER JOIN for order detail?
6. Why did you use LEFT JOIN for the all-customer report?
7. Why does order 5001 appear twice?
8. How did you verify the row count?
9. How did you verify the total line value?
10. What would you change if the data grew to 100 million rows?

A strong answer explains the business grain, joins, validation and performance considerations rather than simply reading the SQL aloud.

---

## 5. Practical SQL challenge

Write queries for these requirements without copying the class solution:

1. Show customer names and cities.
2. Find products above 1,000.
3. Find Accessories priced from 700 to 1,500.
4. Show orders with customer names.
5. Show every customer, including customers with no orders.
6. Find customers who never ordered.
7. Show product details for every order item.
8. Return the two most expensive products.
9. List distinct product categories.
10. Calculate line_total for every order item.

For every query, explain:
- What one output row represents.
- Why you selected each column.
- Why the chosen filter or JOIN is correct.
- How you checked the result.

---

## 6. Common interview mistakes

- Saying WHERE chooses columns; it filters rows.
- Saying LEFT JOIN always returns one row per left row; it may return multiple rows when matches are multiple.
- Forgetting NULL behavior.
- Claiming DISTINCT fixes every duplicate problem.
- Using LIMIT without ORDER BY for top-N requests.
- Ignoring the grain of the result.
- Forgetting to qualify ambiguous columns.
- Assuming a JOIN condition can be omitted safely.
- Moving a right-side filter into WHERE without considering outer-join behavior.
- Claiming query output order is guaranteed without ORDER BY.

---

## 7. Readiness checklist

- [ ] I can explain SELECT, FROM and WHERE.
- [ ] I can use AND, OR, IN, BETWEEN and LIKE.
- [ ] I understand NULL and IS NULL.
- [ ] I can use DISTINCT, ORDER BY, LIMIT and OFFSET.
- [ ] I can explain INNER, LEFT and RIGHT JOIN.
- [ ] I can join three or more tables using the correct keys.
- [ ] I understand one-to-many row multiplication.
- [ ] I can identify a Cartesian product.
- [ ] I can explain the grain of an extract.
- [ ] I can validate row counts and business totals.
- [ ] I can defend the Multi-Table Extraction Queries mini-project.
