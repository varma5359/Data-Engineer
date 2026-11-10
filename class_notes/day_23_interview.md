# Day 23 Interview Preparation: SQL Window Functions

**Phase:** 3 — SQL and Database Engineering  
**Level:** Beginner to junior Data Engineer  
**Mini-project:** Historical Trend Query Engine  
**Core coverage:** ROW_NUMBER, RANK, DENSE_RANK, LAG, LEAD, PARTITION BY, ORDER BY

---

## 1. How to answer

For a window-function question:
1. Explain the function in plain English.
2. Explain what rows it considers.
3. Show a small SQL example.
4. Explain the output, including ties and NULLs.
5. Give a practical use case.

---

## 2. Core questions with complete answers

### 1. What is a window function?

A window function calculates a value using a set of rows related to the current row, while keeping the individual rows visible. It is useful for rankings, previous and next values, running totals and row sequencing.

### 2. How is a window function different from GROUP BY?

GROUP BY combines rows into groups and normally returns one result row per group. A window function calculates across a group of rows but retains the original detail rows.

### 3. What does OVER do?

OVER defines the window used by the function. It can specify partitioning, ordering and, for applicable functions, a frame.

```sql
ROW_NUMBER() OVER (ORDER BY order_date)
```

The function numbers rows in order_date sequence.

### 4. What is PARTITION BY?

PARTITION BY divides the input into independent groups for the window calculation. For example, PARTITION BY customer_id restarts row numbering for each customer.

### 5. Does PARTITION BY sort rows?

No. PARTITION BY defines calculation groups. ORDER BY defines the sequence inside each window.

### 6. What does ORDER BY inside OVER control?

It controls the order in which a window function evaluates rows. For ROW_NUMBER, it determines the numbering sequence. For LAG and LEAD, it determines what counts as the previous or next row.

### 7. Does window ORDER BY sort the final output?

No. Add an outer ORDER BY if the final result must be displayed in a particular order.

### 8. What does ROW_NUMBER do?

ROW_NUMBER assigns a unique sequential integer to each row in a window, starting at 1.

```sql
ROW_NUMBER() OVER (
    PARTITION BY customer_id
    ORDER BY order_date, order_id
)
```

This numbers each customer's orders chronologically. Add a unique tie-breaker to make the sequence deterministic.

### 9. Does ROW_NUMBER give tied values the same number?

No. Every row receives a different number, even when the ordering values are tied. If tied values should share a rank, use RANK or DENSE_RANK.

### 10. What is RANK?

RANK assigns the same rank to tied rows and leaves gaps after ties.

For values 900, 800, 800, 700, the ranks are 1, 2, 2, 4.

### 11. What is DENSE_RANK?

DENSE_RANK assigns the same rank to tied rows but does not leave gaps. For 900, 800, 800, 700, the ranks are 1, 2, 2, 3.

### 12. RANK versus DENSE_RANK?

Both treat equal ordering values as ties. RANK leaves gaps after a tie; DENSE_RANK assigns the next consecutive rank to the next distinct value.

### 13. ROW_NUMBER versus RANK?

ROW_NUMBER always gives each row a unique sequence number. RANK gives equal values the same rank and can leave gaps. Use ROW_NUMBER for deduplication or selecting one latest record; use RANK when tied positions should be shared.

### 14. How do you rank customers by total spend?

First aggregate order-item values to one row per customer. Then apply RANK over total_spend descending. Do not rank individual order-item rows if the business asks for a customer ranking.

### 15. What does LAG do?

LAG accesses a value from a preceding row in the window order. It is useful for comparing current sales with previous-period sales or finding the previous event date.

### 16. What does LEAD do?

LEAD accesses a value from a following row in the window order. It is useful for finding the next event, next order or next effective date.

### 17. LAG versus LEAD?

LAG looks backward in the ordered window. LEAD looks forward. Both use the ORDER BY inside OVER to define previous and next.

### 18. What does LAG return for the first row?

By default it returns NULL because no preceding row exists in that partition. You can provide an offset and a default value, but the default must make business sense.

### 19. Can LAG access more than one row back?

Yes. The second argument is the offset. LAG(value, 2) accesses the value two rows before the current row, if that row exists.

### 20. What is the default offset for LAG?

The default offset is 1. It accesses the immediately preceding row.

### 21. Why is PARTITION BY important with LAG?

Without partitioning by customer, one customer's previous order could become another customer's previous row. Partitioning ensures comparisons stay within the same customer.

### 22. How do you calculate days between orders?

Use LAG(order_date) over each customer's chronological order sequence, then subtract the previous date from the current date. The first order in each partition returns NULL for the gap.

### 23. How do you find the latest record per customer?

Use ROW_NUMBER partitioned by customer_id and ordered by the timestamp descending, with a unique tie-breaker. Put the window result in a CTE or subquery and filter row number 1 in the outer query.

### 24. Why can't you normally filter ROW_NUMBER in WHERE at the same query level?

WHERE is logically evaluated before the SELECT list's window result is produced. Use a subquery or CTE to calculate the row number, then filter it outside.

### 25. How do you find the top three products in each category?

Calculate ROW_NUMBER or RANK partitioned by category and ordered by the business metric descending. Filter the ranking in an outer query. Use ROW_NUMBER for at most three rows per category; use RANK or DENSE_RANK when ties must be included according to the requirement.

### 26. What happens when two rows tie in RANK?

They receive the same rank. The next rank skips positions. If two rows tie at rank 2, the next rank is 4.

### 27. Should you add a unique ID to RANK's ORDER BY?

Only if you intend to break ties. If the business says equal sales totals share a rank, do not add a unique ID to the ranking expression. You can still use a separate outer ORDER BY to display tied rows deterministically.

### 28. What is a window frame?

A frame defines which rows around the current row are considered for frame-sensitive window calculations, such as running totals or moving averages. It can be specified with ROWS or RANGE boundaries. Ranking functions do not require a custom frame for the basic behavior taught today.

### 29. What is a running total?

A running total is a cumulative sum from the beginning of an ordered sequence through the current row. It is commonly calculated with SUM(...) OVER (ORDER BY ... ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW).

### 30. Does a window function modify table data?

No. A SELECT query with window functions returns calculated values. It does not permanently update the source table.

### 31. Can we use multiple window functions in one SELECT?

Yes. For example, one query can calculate ROW_NUMBER, RANK and LAG using separate OVER definitions. Each function can have its own partition and ordering requirements.

### 32. What is a named window?

Some SQL dialects support a WINDOW clause that names a window definition so it can be reused by multiple window functions. It improves readability when several functions use the same partition and ordering.

### 33. What if ORDER BY values are not unique?

Rows tied on the ordering expressions do not have a fully defined order. ROW_NUMBER may assign tied rows in an unspecified sequence. Add a stable unique tie-breaker when deterministic sequencing is required.

### 34. What is the difference between a window ORDER BY and final ORDER BY?

Window ORDER BY determines calculation order. The outer query ORDER BY determines presentation order. Both may be needed.

### 35. Can window functions be used with aggregate queries?

Yes. A common pattern is to aggregate to a business grain in a CTE or subquery, then apply a window function to rank or compare those summary rows.

---

## 3. Scenario-based questions

### Scenario 1: Find the latest customer record from a daily file

Use ROW_NUMBER partitioned by the customer business key and ordered by update timestamp descending. Include a deterministic tie-breaker such as source sequence or ingestion ID. Filter row number 1. Validate that the key is unique in the result.

### Scenario 2: Calculate month-over-month sales change

First calculate total sales per month. Then use LAG(month_sales) over ORDER BY month_start. Compare current month sales with the previous month. Define how to handle the first month, missing months and zero previous sales.

### Scenario 3: Rank employees with equal performance scores

Use RANK if tied scores should share a position and the next rank should have a gap. Use DENSE_RANK if the next distinct score should receive the next consecutive rank. Use ROW_NUMBER only if every person must have a unique position.

### Scenario 4: Compare each customer's current order with their previous order

Use LAG(order_date) or LAG(order_total) partitioned by customer_id and ordered by order date and order ID. The first order has no previous order, so the comparison is NULL.

### Scenario 5: The latest-row query returns inconsistent results

Check whether the window ORDER BY has a unique tie-breaker. If timestamps tie, add a stable unique source key. Also verify the partition key is the correct business key.

### Scenario 6: A ranking report shows unexpected ties

Inspect the ranking expression. If it ranks only total_spend, equal totals tie. If a unique identifier was added to the window ORDER BY, it may have broken ties unintentionally.

---

## 4. Mini-project defence

Be ready to answer:

1. What does one row represent in your order sequence output?
2. Why does row numbering restart for each customer?
3. Why does the first LAG result for a customer return NULL?
4. How do ROW_NUMBER, RANK and DENSE_RANK differ?
5. How did you calculate customer spend before ranking?
6. Why did you use a CTE before filtering the latest order?
7. How did you handle tied timestamps?
8. Why is final ORDER BY separate from window ORDER BY?
9. How would you extend the project to month-over-month sales?
10. How did you validate the results?

---

## 5. Common interview traps

- Saying window functions reduce the result to one row per group.
- Saying PARTITION BY sorts data.
- Saying window ORDER BY guarantees output order.
- Saying RANK and DENSE_RANK are identical.
- Saying ROW_NUMBER assigns the same number to tied values.
- Forgetting to partition LAG by the entity key.
- Filtering a window result in the same-level WHERE clause.
- Ranking detail rows instead of pre-aggregated business totals.
- Ignoring ties and nondeterministic ordering.
- Using a default value for LAG that invents false historical data.

---

## 6. Readiness checklist

- [ ] I can explain GROUP BY versus a window function.
- [ ] I can explain OVER, PARTITION BY and ORDER BY.
- [ ] I can write ROW_NUMBER.
- [ ] I can write RANK and DENSE_RANK and explain ties.
- [ ] I can write LAG and LEAD.
- [ ] I can partition by customer.
- [ ] I can calculate a date gap using LAG.
- [ ] I can rank aggregated customer totals.
- [ ] I can select the latest row per customer.
- [ ] I can explain deterministic tie-breaking.
