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
