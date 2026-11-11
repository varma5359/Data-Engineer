# Day 56 Interview Guide — Spark Transformations and Actions

## 1. What is a Spark transformation?

A transformation describes a change to a DataFrame and returns a new DataFrame. Examples include select, filter, withColumn, groupBy and orderBy. Transformations are generally lazy.

## 2. What is a Spark action?

An action triggers execution or returns/writes a result. Examples include show, count, collect and write. An action causes Spark to execute the required plan.

## 3. What is lazy evaluation?

Spark builds a plan for transformations and delays execution until an action requires a result. This allows Spark to optimize the sequence of operations.

## 4. What is the difference between select and filter?

select chooses columns; filter chooses rows that satisfy a condition. A query may use both to reduce the amount of data processed and returned.

## 5. What is the difference between filter and where?

In the Spark DataFrame API, where is an alias for filter. Both select rows based on a condition.

## 6. What does withColumn do?

withColumn adds a new column or replaces an existing column with the same name. The expression is evaluated by Spark as part of the DataFrame plan.

## 7. What is lit?

lit creates a literal column expression, such as a fixed source label or constant value, that can be used in a DataFrame transformation.

## 8. How do you create conditional columns?

Use when and otherwise from pyspark.sql.functions. Conditions are evaluated in order, and otherwise defines the fallback value when no condition matches.

## 9. How do you combine filter conditions?

Use Spark Column expressions with &, | and ~, and wrap individual conditions in parentheses. Do not use Python's and/or operators between Spark Column expressions.

## 10. How do you handle null values?

Use isNull and isNotNull to identify missing values. fillna replaces values and dropna removes rows according to configured rules. The replacement or removal must follow business meaning; null is not automatically zero or empty text.

## 11. What is the difference between distinct and dropDuplicates?

distinct removes fully duplicate rows. dropDuplicates can use selected columns as the duplicate key. If duplicate keys contain conflicting values, define a deterministic winner such as the latest update timestamp.

## 12. What does groupBy do?

groupBy defines grouping keys so aggregates can be calculated for each group. For example, sales can be grouped by city and summed.

## 13. What is agg?

agg applies one or more aggregate expressions such as sum, avg, count, min and max to grouped data. Alias gives output columns meaningful names.

## 14. Why should you use orderBy before limit for top-N?

Distributed output order is not guaranteed without an explicit sort. Sorting first ensures the selected rows represent the intended top-N. Add tie-breaking columns when deterministic ordering is required.

## 15. Why is collect risky?

collect transfers all rows to the driver. Large results can exceed driver memory. Keep data distributed and collect only small results when appropriate.

## 16. What is the difference between count("*") and count("column")?

count("*") counts rows. count("column") counts non-null values in that column. This difference matters when the column can contain nulls.

## 17. Why can fillna(0) be incorrect for a money column?

A missing amount means unknown or unavailable, not necessarily zero. Replacing it with zero can change financial totals and hide source data quality problems. The business owner must define the rule.

## 18. What happens when you write a Spark DataFrame to CSV?

Spark generally writes a directory containing one or more part files, depending on the DataFrame's partitioning and write behavior. It does not necessarily create one single CSV file.

## 19. Scenario: groupBy output rows appear in a different order each run.

That is expected unless an explicit ordering is applied. I would use orderBy on the required output columns and include tie-breakers if deterministic order matters.

## 20. Scenario: a pipeline overwrites a curated output directory.

I would verify the destination and write mode, ensure the output is complete before publishing, validate row counts and business totals, and use a safe run-specific or transactional publish pattern appropriate to the storage platform.

## 21. Explain your Day 56 project.

I selected columns, filtered orders, added derived columns, cast types, handled nulls, compared duplicate operations, grouped by city, calculated sales aggregates, sorted the result and wrote CSV/Parquet outputs. I validated the city totals against the source rows and documented the business assumptions.

## Common interview traps

- Calling every DataFrame operation an action.
- Assuming transformations immediately execute.
- Using Python and/or instead of Spark Column operators.
- Replacing null values without a business rule.
- Removing duplicates without defining the business key.
- Assuming groupBy output is sorted.
- Using limit without orderBy for top-N.
- Collecting large datasets to the driver.

## Readiness checklist

- [ ] Explain transformations and actions.
- [ ] Explain lazy evaluation.
- [ ] Use select, filter, withColumn and cast.
- [ ] Handle nulls and duplicates carefully.
- [ ] Group and aggregate.
- [ ] Sort and limit.
- [ ] Write outputs and validate them.
