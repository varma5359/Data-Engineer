# Day 15 Interview Preparation: NumPy for Data Engineering

**Mini-project:** Numerical Transformation Lab

## 1. Core questions

1. **What is NumPy?** A Python library for efficient numerical arrays and vectorized operations.
2. **What is ndarray?** NumPy's multidimensional array object.
3. **What is vectorization?** Applying operations to whole arrays instead of explicit element-by-element Python loops.
4. **What is broadcasting?** Rules that allow compatible array shapes to work together.
5. **What does axis=0 mean?** Aggregate along rows, producing a result for each column in a 2-D array.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| list vs ndarray | General Python collection vs typed numerical array |
| shape vs size | Dimension lengths vs total element count |
| arange vs linspace | Step-based values vs fixed number of evenly spaced values |
| NaN vs zero | Missing/invalid marker vs valid numeric value |
| axis 0 vs axis 1 | Down rows/column-wise result vs across columns/row-wise result |

## 3. Practical questions

6. **How do you filter values above a threshold?** Use a boolean mask such as `values[values > threshold]`.
7. **Why inspect dtype?** It controls representation and supported operations; unexpected coercion can affect precision and memory.
8. **Why be careful with money in float?** Binary floating-point cannot represent every decimal fraction exactly; use decimal-aware types for exact financial rules.

## 4. Scenario questions

### Scenario A: Aggregation returns unexpected shape
Inspect array dimensions and axis. Test with a tiny matrix whose row and column totals are known.

### Scenario B: NaN values distort a metric
Count and inspect missing values, then apply a documented exclusion/imputation/rejection policy. Do not replace all missing values with zero automatically.

### Scenario C: Broadcasting error
Print both shapes and align dimensions deliberately. Reshape only when the intended relationship is clear.

## 5. Troubleshooting
- ValueError during reshape: total element count does not match.
- TypeError: inspect dtype and input values.
- Unexpected float output: check mixed input types and division.
- Wrong total: verify axis and selected rows.

## 6. Follow-up questions
- What is a contiguous array?
- What is dtype coercion?
- How does NumPy handle NaN?
- When is Pandas preferable?
- When would Spark be more suitable than NumPy?

## 7. Mini-project defence
Explain the array shapes, vectorized line-sales calculation, boolean filter and aggregation. Reconcile the result against an independent calculation and explain any precision limitations.
