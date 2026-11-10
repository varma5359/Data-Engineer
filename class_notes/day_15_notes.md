# Day 15 — NumPy and Vectorized Processing Concepts
**Phase 2: Python for Data Engineering**

## 1. NumPy
NumPy provides arrays and efficient numerical operations. It is a foundation for scientific/data libraries and can avoid slow Python loops for numeric transformations.

## 2. Array basics
```python
import numpy as np
amounts = np.array([100, 200, 300])
print(amounts)
print(amounts.sum(), amounts.mean())
```
Output: `[100 200 300]`; `600 200.0`.

Arrays generally hold compatible element types and support shape-based operations.

## 3. Vectorization
```python
amounts = np.array([100, 200, 300])
taxed = amounts * 1.18
print(taxed)
```
Output: `[118. 236. 354.]`
One operation applies to the array; no explicit Python loop needed.

## 4. Boolean masks
```python
amounts = np.array([100, 200, 300, 50])
large = amounts[amounts >= 200]
print(large)
```
Output: `[200 300]`

## 5. Missing and numeric caveats
NumPy has values such as `np.nan`; comparisons and aggregation need deliberate handling (`np.nanmean`, masks, or conversion). Do not treat NaN as ordinary null equality.

## 6. Mini-project — Numerical Transformation Lab
Compare a simple loop and vectorized calculation on sample amounts; verify identical results and inspect execution time only on meaningful larger input. Avoid claiming speedups from tiny samples.

## Common mistakes
- Confusing array shape and dimensions.
- Assuming all data types are preserved exactly.
- Ignoring NaN behavior.
- Using NumPy for text-heavy records unnecessarily.
- Assuming vectorization automatically solves memory limits.

## Revision
Array = typed numeric collection; vectorization = array-level operations; masks select values; validate shape and missing-value behavior.