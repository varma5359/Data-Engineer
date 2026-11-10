# Day 14 Interview Preparation: Pandas for Data Engineering

**Mini-project:** Dataset Transformation Tool

## 1. Core questions

1. **What is a DataFrame?** A two-dimensional labeled tabular data structure.
2. **What is a Series?** A one-dimensional labeled array.
3. **How do you inspect a dataset?** Use head, shape, columns, dtypes, info, describe and null counts.
4. **How do you group data?** Use groupby with explicit aggregation functions.
5. **Why validate a merge?** To detect unexpected one-to-many or many-to-many relationships that can multiply rows.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| Series vs DataFrame | One-dimensional vs two-dimensional |
| loc vs iloc | Label-based vs integer-position-based selection |
| inner vs left merge | Matching keys only vs all left rows |
| isna vs duplicated | Detect missing values vs duplicate rows |
| apply vs vectorized operation | Function applied along axis vs optimized array/column operation |

## 3. Practical questions

6. **How do you process a large CSV?** Use chunksize, select required columns and avoid retaining every chunk; use Spark or SQL when the data exceeds single-machine capacity.
7. **How do you handle invalid numeric strings?** Convert with explicit error policy, identify coerced failures and quarantine/report them.
8. **How do you prevent join row multiplication?** Check key uniqueness, validate relationship cardinality and inspect unmatched keys.
9. **How do you count unique orders?** Use nunique on the defined order key, if that matches the business grain.

## 4. Scenario questions

### Scenario A: Revenue doubles after merge
Inspect duplicate customer/product keys and join cardinality. Compare row counts and totals before and after joining.

### Scenario B: Null amount values appear
Determine whether null means missing, not applicable or invalid. Do not replace with zero without an approved rule.

### Scenario C: Pandas process runs out of memory
Profile memory, select columns, set appropriate dtypes, chunk input, reduce copies or move processing to a database/Spark.

## 5. Troubleshooting
- KeyError: column missing or name differs.
- SettingWithCopyWarning: use explicit .loc or create a deliberate copy.
- ParserError: inspect delimiter, quoting and malformed rows.
- Unexpected NaT: inspect date format and timezone assumptions.
- Wrong group totals: check null keys, duplicate grain and included statuses.

## 6. Follow-up questions
- What is vectorization?
- What is a categorical dtype?
- How does chunksize work?
- What is join cardinality?
- What is the difference between count and nunique?
- When would SQL be better than Pandas?

## 7. Mini-project defence
Walk through input profiling, cleaning policy, join validation, aggregation grain and output reconciliation. Explain why the transformation is safe to rerun and when Pandas is no longer the right processing engine.
