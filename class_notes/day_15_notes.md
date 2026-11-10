# Day 15: NumPy for Data Engineering

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** Numerical Transformation Lab  
**Learning time:** 2–3 hours

## 1. Learning outcomes
- Understand NumPy arrays and their advantages.
- Create and inspect arrays.
- Understand shape, dimensions and data types.
- Use indexing, slicing and boolean masks.
- Perform vectorized calculations and aggregations.
- Recognize broadcasting and missing-value behavior.

## 2. Why NumPy?
NumPy provides efficient numerical arrays and operations. Pandas uses NumPy concepts extensively. Data engineers encounter NumPy when preparing numerical data, performing calculations and integrating with scientific Python libraries.

Install:

```bash
python -m pip install numpy
```

## 3. Create arrays
```python
import numpy as np

quantities = np.array([2, 3, 1, 5])
prices = np.array([100.0, 250.0, 500.0, 80.0])

print(quantities)
print(prices)
print(type(quantities))
```

A NumPy ndarray stores elements in an array with a defined shape and dtype.

## 4. Inspect array properties
```python
print(quantities.ndim)
print(quantities.shape)
print(quantities.size)
print(quantities.dtype)
```

- ndim: number of dimensions.
- shape: size along each dimension.
- size: total number of elements.
- dtype: element data type.

## 5. Create common arrays
```python
print(np.zeros(4))
print(np.ones(3))
print(np.arange(1, 6))
print(np.linspace(0, 1, 5))
```

`arange` uses a step interval; `linspace` creates a specified number of evenly spaced values.

## 6. Indexing and slicing
```python
amounts = np.array([100, 250, 400, 150, 900])

print(amounts[0])
print(amounts[-1])
print(amounts[1:4])
```

Slicing selects a range. The ending index is excluded.

## 7. Vectorized arithmetic
```python
quantity = np.array([2, 3, 4])
unit_price = np.array([100, 200, 50])

line_amount = quantity * unit_price

print(line_amount)
```

Expected: `[200 600 200]`.

Vectorized operations apply to arrays without writing an explicit Python loop for each element.

## 8. Compare vectorized and loop logic
```python
amounts = np.array([100, 200, 300])
taxed_amounts = amounts * 1.18

print(taxed_amounts)
```

The operation is concise and usually efficient for numerical arrays. It does not remove the need to verify numeric types, precision and business rules.

## 9. Boolean filtering
```python
amounts = np.array([100, 250, 800, 1200, 400])
high_value = amounts[amounts >= 500]

print(high_value)
```

Combine conditions with parentheses and bitwise operators:

```python
selected = amounts[(amounts >= 200) & (amounts < 1000)]
print(selected)
```

## 10. Aggregations
```python
daily_sales = np.array([1200, 1500, 900, 1800])

print(np.sum(daily_sales))
print(np.mean(daily_sales))
print(np.min(daily_sales))
print(np.max(daily_sales))
print(np.std(daily_sales))
```

Understand what population/sample standard deviation means before interpreting a metric. NumPy defaults may differ from another tool's statistical convention.

## 11. Two-dimensional arrays
```python
sales = np.array([
    [100, 120, 90],
    [200, 180, 220]
])

print(sales.shape)
print(sales[0, 1])
print(sales.sum(axis=0))
print(sales.sum(axis=1))
```

For a two-dimensional array:
- axis=0 aggregates down rows, producing one result per column.
- axis=1 aggregates across columns, producing one result per row.

## 12. Broadcasting
Broadcasting allows compatible shapes to participate in an operation.

```python
prices = np.array([100, 200, 300])
tax_rate = 0.18

final_prices = prices * (1 + tax_rate)
print(final_prices)
```

The scalar is applied to each array element. More complex broadcasting requires compatible dimensions; inspect shapes when results are surprising.

## 13. Missing and invalid values
```python
measurements = np.array([10.0, 12.0, np.nan, 15.0])

print(np.isnan(measurements))
print(np.nanmean(measurements))
```

NaN is a floating-point missing/invalid marker. It is not the same as zero. Decide whether missing values should be rejected, imputed or excluded for a particular calculation.

## 14. Data types and precision
```python
values = np.array([1, 2, 3])
print(values.dtype)

decimal_values = np.array([1.25, 2.50, 3.75])
print(decimal_values.dtype)
```

For financial calculations, floating-point representation may introduce rounding effects. Use decimal-aware approaches or database decimal types when exact monetary arithmetic is required.

## 15. Reshape arrays
```python
values = np.arange(1, 7)
matrix = values.reshape(2, 3)

print(matrix)
print(matrix.shape)
```

The new shape must contain the same total number of elements.

## 16. Mini-project: Numerical Transformation Lab
Scenario: daily product quantities and unit prices.

```python
import numpy as np

quantities = np.array([2, 1, 5, 3, 4])
unit_prices = np.array([100, 500, 80, 150, 200])

line_sales = quantities * unit_prices
print(line_sales)
```

Find lines above 500:

```python
high_sales = line_sales[line_sales > 500]
print(high_sales)
```

Calculate total and average:

```python
total_sales = np.sum(line_sales)
average_sales = np.mean(line_sales)

print(total_sales)
print(average_sales)
```

Compare NumPy result with a small Pandas DataFrame result. The two should agree for the same input and calculation.

## 17. Common mistakes
| Mistake | Better practice |
|---|---|
| Confusing shape and size | Shape gives dimensions; size gives element count |
| Unexpected broadcasting | Print shapes and understand alignment |
| Treating NaN as zero | Apply an explicit missing-data rule |
| Floating point for exact currency | Use decimal-aware representation |
| Wrong axis | Draw rows/columns and test a small array |
| Assuming arrays accept mixed types efficiently | Choose suitable dtype |

## 18. Day 15 revision
- ndarray is NumPy's core array.
- Vectorization applies operations to arrays efficiently.
- Boolean masks filter values.
- Axis controls aggregation direction.
- Broadcasting applies compatible shapes.
- Missing values and numerical precision require deliberate handling.
