# Day 57 Interview Guide — Spark Joins and Partitioning

## 1. What is a join?

A join combines rows from two DataFrames using a matching condition. The join key should represent the intended business relationship, such as orders.customer_id matching customers.customer_id.

## 2. What is an inner join?

An inner join returns rows with matching keys on both sides. Unmatched rows are excluded.

## 3. What is a left join?

A left join retains all rows from the left DataFrame and adds matching right-side columns. When there is no right-side match, right-side values are null.

## 4. What is a full outer join?

A full outer join retains matched and unmatched rows from both DataFrames. It is useful for reconciliation and identifying records missing from either side.

## 5. What is a left semi join?

A left semi join returns only left-side rows that have a match on the right. It is useful for filtering a dataset to keys known in a reference dataset.

## 6. What is a left anti join?

A left anti join returns left-side rows that do not have a matching right-side key. It is useful for finding orphan facts or missing reference records.

## 7. Why are join keys important?

A join key defines which records are related. A wrong key can produce incorrect matches or many-to-many multiplication. Engineers should confirm the business relationship, data type and uniqueness expectations.

## 8. How can duplicate keys affect a join?

If one order matches two customer rows, the joined result contains two rows for that order. This can inflate row counts and sales totals. Check key uniqueness and understand whether duplicates represent errors or historical versions.

## 9. How do you identify unmatched orders?

Use a left anti join from orders to customers. It returns orders whose customer_id does not exist in the customer dataset.

## 10. How do null join keys behave?

Ordinary equality does not match null to null. Rows with null join keys generally do not match in an equality join. The pipeline should measure missing keys and apply an explicit business rule.

## 11. Why use DataFrame aliases?

Aliases identify which input DataFrame a column belongs to. They help avoid ambiguous column references when both DataFrames contain columns with the same name.

## 12. What is a Spark partition?

A Spark partition is a logical chunk of a DataFrame that can be processed by a task. Partition count and size influence parallelism, scheduling, memory and output file count.

## 13. What does repartition do?

repartition redistributes data into a requested number of partitions and generally causes a shuffle. It can increase or decrease partition count and can repartition by expressions or columns.

## 14. What does coalesce do?

coalesce is commonly used to reduce partition count with less data movement than a full repartition. It is not the preferred method for increasing partitions or evenly redistributing data.

## 15. What is the difference between repartition and coalesce?

repartition generally shuffles data to redistribute it and can increase or decrease partition count. coalesce is generally used to reduce partition count more efficiently, but may leave uneven partition sizes.

## 16. What does partitionBy do?

partitionBy organizes output files into directory/prefix structures based on selected column values. It is a storage layout decision and is not the same as changing Spark's in-memory execution partitions.

## 17. Why avoid partitioning by a high-cardinality column?

A high-cardinality field can create a very large number of small directories or files, increasing metadata and file management overhead. Choose partition columns based on query filters, volume and retention.

## 18. Scenario: total sales doubles after joining orders to customers.

I would check whether customer_id is unique in the customer dataset, inspect duplicate keys, verify the join condition and examine one-to-many relationships or historical dimension records. I would compare order IDs and totals before and after the join.

## 19. Scenario: orders without customer matches must be investigated.

I would use a left anti join to isolate those orders, capture the missing keys and source details, and route them to a data-quality process. I would not silently drop them if the business needs reconciliation.

## 20. Explain your Day 57 project.

I created order and customer DataFrames, ran multiple join types, identified unmatched orders using left anti, introduced duplicate customer keys to demonstrate join multiplication, checked null-key behavior, inspected Spark partition counts, compared repartition and coalesce, and wrote partitioned output by city. I validated row counts and totals.

## Common interview traps

- Using inner join when unmatched records must remain.
- Assuming a left join always preserves the left row count despite duplicate right keys.
- Ignoring null join keys.
- Removing duplicate dimension records without understanding history.
- Confusing repartition with coalesce.
- Confusing Spark execution partitions with storage partitionBy.
- Partitioning by a high-cardinality field without a size analysis.

## Readiness checklist

- [ ] Explain all common join types.
- [ ] Use semi and anti joins.
- [ ] Identify unmatched and duplicate keys.
- [ ] Explain join multiplication.
- [ ] Explain repartition and coalesce.
- [ ] Explain partitioned file output.
- [ ] Validate row counts and business totals.
