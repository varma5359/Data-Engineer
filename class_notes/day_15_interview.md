# Day 15 Interview Preparation: NumPy and Numerical Processing

**Mini-project:** Numerical Transformation Lab

## 1. Core questions

1. **What is NumPy?**  
   A Python library for efficient numerical arrays and vectorized operations.

2. **What is ndarray?**  
   NumPy's multidimensional array object.

3. **What does shape describe?**  
   The length along each array dimension.

4. **What does ndim describe?**  
   The number of dimensions.

5. **What is dtype?**  
   The data type stored in an array.

6. **What is vectorization?**  
   Applying an operation to array values without writing an explicit Python loop for each element.

7. **What is a Boolean mask?**  
   A Boolean array used to select elements that meet a condition.

8. **What is broadcasting?**  
   NumPy's mechanism for applying operations to arrays with compatible shapes.

9. **What is axis=0 for a 2D array?**  
   Reduce across rows to produce one result per column.

10. **What is axis=1?**  
    Reduce across columns to produce one result per row.

11. **What is NaN?**  
    A special floating-point value commonly used to represent missing or undefined numerical data.

12. **What is the difference between mean and nanmean?**  
    `mean` propagates NaN; `nanmean` ignores NaN values.

13. **Does NumPy guarantee exact decimal currency arithmetic?**  
    No. Binary floating-point cannot represent every decimal exactly. Use decimal arithmetic or database decimal types where exact monetary precision is required.

14. **NumPy vs Pandas?**  
    NumPy focuses on numerical arrays; Pandas adds labeled Series and DataFrame structures for tabular data.

## 2. Difference questions

| Concept | Difference |
|---|---|
| Python list / ndarray | General-purpose collection / numerical array with typed elements and vectorized operations |
| ndim / shape | Number of dimensions / size of each dimension |
| size / shape | Total element count / dimensions |
| mean / nanmean | Includes NaN propagation / ignores NaN values |
| axis=0 / axis=1 | Reduce rows / reduce columns |
| vectorized / Python loop | Array-level operation / explicit iteration in Python |
| NumPy / Pandas | Numerical arrays / labeled tabular data |

## 3. Practical coding questions

**Q1. Multiply quantities and prices.**

```python
import numpy as np
q = np.array([2, 3])
p = np.array([100, 50])
print(q * p)
```

Output: `[200 150]`.

**Q2. Filter high sales.**

```python
amounts = np.array([100, 500, 800])
print(amounts[amounts >= 500])
```

Output: `[500 800]`.

**Q3. Sum each column.**

```python
sales = np.array([[1, 2], [3, 4]])
print(np.sum(sales, axis=0))
```

Output: `[4 6]`.

**Q4. Ignore missing values.**

```python
values = np.array([10.0, np.nan, 20.0])
print(np.nanmean(values))
```

Output: `15.0`.

## 4. Scenario-based questions

1. **Two arrays cannot be multiplied due to a shape error. What do you check?**  
   Inspect both shapes and confirm whether NumPy broadcasting rules make them compatible.

2. **A monthly average is NaN. Why?**  
   The array may contain NaN and ordinary mean propagates it. Inspect missingness and select an approved missing-value policy.

3. **A sales report has two stores and three months. How do you calculate monthly totals?**  
   Represent rows as stores and columns as months, then sum with `axis=0`.

4. **A currency total differs by a few tiny decimal places. What is the concern?**  
   Floating-point representation and rounding. Use a defined decimal precision strategy for financial calculations.

5. **Why not use NumPy for every dataset?**  
   It is focused on in-memory numerical arrays. Relational querying, labeled tables, large-scale distributed processing and storage need other tools.

## 5. Follow-up questions

- What is the difference between `arange` and `linspace`?
- What does `astype()` do?
- Why should Boolean conditions be parenthesized when combined with `&`?
- What is a broadcasting-compatible shape?
- What is population versus sample standard deviation?
- When would you use Pandas instead of NumPy?

## 6. Strong revision points

- Inspect shapes before numerical operations.
- Use vectorization for numerical transformations.
- Understand the axis direction on a small example.
- Treat missing values deliberately.
- Do not assume floating-point values are exact currency values.


---

## 7. Follow-up interview questions

16. What is an ndarray? NumPy's multidimensional array structure.
17. What does ndim mean? Number of array dimensions.
18. What does shape mean? The length along each dimension.
19. What does size mean? Total number of array elements.
20. What is vectorization? Applying operations to whole arrays rather than writing explicit Python loops for each element.
21. What is broadcasting? NumPy's rules for applying operations to arrays with compatible shapes.
22. What is a Boolean mask? A Boolean array used to select matching elements.
23. What does axis=0 mean for a 2D sum? Reduce down rows, producing one sum per column.
24. What does axis=1 mean? Reduce across columns, producing one sum per row.
25. What is NaN? A special floating-point value representing not-a-number, often used for missing numerical values.
26. What does nanmean do? Calculates a mean while ignoring NaN values.
27. Does NumPy replace Spark? No. NumPy is generally in-memory on one machine; Spark distributes processing across a cluster.
28. Why inspect dtype? To detect unexpected numeric types, object arrays or conversion problems.
29. Why can broadcasting fail? The dimensions may not satisfy NumPy's compatibility rules.
30. Why be careful with financial floats? Binary floating point cannot represent every decimal fraction exactly; financial calculations need explicit precision and rounding rules.

## 8. Scenario questions

A daily revenue array contains NaN. Decide whether the metric should exclude, impute or reject the missing value and document the business rule.

Two arrays cannot be multiplied. Inspect their shapes and align them intentionally rather than reshaping blindly.

A dataset is too large for one machine's memory. Consider database-side processing or a distributed tool such as Spark instead of assuming NumPy can scale without limit.

## 9. Mini-project defense

- What shape did your arrays have?
- Which operation was vectorized?
- How did you interpret axis?
- What happened to NaN values?
- How did you reconcile the computed total?

## 10. Readiness rubric

| Skill | Ready when the student can… |
|---|---|
| Structure | Explain ndim, shape, size and dtype |
| Indexing | Select array values and slices |
| Computation | Use vectorized operations |
| Filtering | Apply Boolean masks |
| Aggregation | Explain axis behavior |
| Missing values | Identify NaN and justify a handling rule |
