# Day 14 Interview Preparation: Pandas for Data Engineering

**Mini-project:** Dataset Transformation Tool

## 1. Core questions

1. **What is Pandas?**  
   A Python library for working with labeled tabular data.

2. **What is a DataFrame?**  
   A two-dimensional data structure with labeled rows and columns.

3. **What is a Series?**  
   A one-dimensional labeled data structure, often representing one DataFrame column.

4. **How do you inspect a new dataset?**  
   Use head, tail, shape, columns, dtypes, info, describe and null counts.

5. **How do you select one column?**  
   Use `df["column"]`; this returns a Series.

6. **How do you select multiple columns?**  
   Use a list of column names, such as `df[["a", "b"]]`.

7. **How do you filter rows?**  
   Use a boolean condition. Combine Series conditions with `&` or `|` and parentheses.

8. **What does groupby do?**  
   Splits data into groups so calculations such as sum, count and mean can be applied per group.

9. **Why validate a merge?**  
   To detect unexpected one-to-many or many-to-many relationships that can multiply rows.

10. **How do you handle missing values?**  
    Inspect the missingness and apply a business-approved policy: retain, reject, fill or remove as appropriate.

11. **Why use vectorized operations?**  
    They perform operations over whole arrays/columns efficiently and usually avoid slow Python row loops.

12. **Is Pandas suitable for every dataset size?**  
    No. It generally works in local memory. Large datasets may require chunking, SQL pushdown or distributed processing.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| Series vs DataFrame | One-dimensional vs two-dimensional |
| loc vs iloc | Label-based vs position-based selection |
| isna vs notna | Detect missing vs non-missing values |
| dropna vs fillna | Remove missing records/values vs replace missing values |
| merge vs concat | Relational join by keys vs combine along an axis |
| inner vs left join | Keep matching rows vs keep every left-side row |
| apply vs vectorized operation | Function application vs array/column operation |

## 3. Practical implementation questions

13. **How do you read a CSV?**  
    Use `pd.read_csv("file.csv")` and then inspect the result.

14. **How do you filter paid orders above a threshold?**  
    Use `df[(df["status"] == "PAID") & (df["amount"] > threshold)]`.

15. **How do you sort by revenue descending?**  
    Use `sort_values("revenue", ascending=False)`.

16. **How do you calculate revenue by city?**  
    Filter to the required business population, group by city and aggregate amount with sum.

17. **How do you remove duplicate order IDs?**  
    Confirm the business key and use `drop_duplicates(subset=["order_id"])` with an explicit keep policy.

18. **How do you validate a merge?**  
    Use `validate="many_to_one"` or the appropriate cardinality and inspect unmatched rows.

19. **How do you export without the DataFrame index?**  
    Use `to_csv("output.csv", index=False)`.

## 4. Scenario questions

### Scenario 1: Revenue is higher after a merge

**Problem:** A customer lookup has duplicate customer IDs, causing each order to match multiple rows.

**Strong answer:** Check lookup-key uniqueness, validate merge cardinality and investigate duplicates. Do not aggregate inflated revenue as if it were correct.

### Scenario 2: Missing amount values

**Problem:** Some orders have no amount.

**Strong answer:** Determine whether the source omitted the value, whether the record is invalid or whether a business-approved imputation is possible. Do not automatically fill with zero.

### Scenario 3: Dataset exceeds memory

**Problem:** Pandas cannot load the complete CSV.

**Strong answer:** Select required columns, filter at the source, read in chunks, or use a database/distributed engine for the scale.

### Scenario 4: Cancelled orders are included in revenue

**Problem:** The aggregation includes all statuses.

**Strong answer:** Define revenue with the business owner, filter to eligible statuses before aggregation and verify row counts and totals.

### Scenario 5: Duplicate order records disagree

**Problem:** Two rows share an order ID but contain different amounts.

**Strong answer:** Do not arbitrarily keep the first row. Investigate source timestamps/version rules and define a deterministic resolution policy.

## 5. Troubleshooting questions

20. **Why does a filter return zero rows?**  
    Check actual values, whitespace, casing, data types and condition logic.

21. **Why does a boolean filter fail with an ambiguous truth value?**  
    A Series contains many booleans; use element-wise `&` or `|` with parentheses, not Python `and` or `or`.

22. **Why did merge increase row count?**  
    Duplicate keys on one or both sides caused multiple matches.

23. **Why does a numeric column have object dtype?**  
    It may contain text, mixed values or malformed records. Inspect and convert safely.

## 6. Mini-project explanation

"I inspected an order dataset, identified a missing amount, filtered eligible paid orders, enriched them with customer attributes using a validated many-to-one merge, aggregated paid revenue by city and exported the outputs. I checked row counts before and after the key steps."

## 7. Follow-up questions

- How do you handle a dataset with millions of rows?
- What is the difference between count and size in groupby?
- How do you validate totals before and after a transformation?
- How do you avoid chained assignment?
- How do you process multiple CSV files consistently?
- When would you move a transformation from Pandas into SQL?

## 8. Quick revision

- Inspect before transforming.
- Define the business population before calculating metrics.
- Handle nulls based on meaning, not convenience.
- Validate join cardinality.
- Use vectorized operations and filter early.
- Verify exported output with row counts and key checks.
