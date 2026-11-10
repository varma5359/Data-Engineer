# Day 7 — Functions, Modules and Reusable Code
**Phase 2: Python for Data Engineering**

## Outcomes
Write functions with parameters/return values, split code into modules, and reuse transformations without duplicating logic.

## 1. Why functions?
A pipeline repeats operations: normalize a date, validate an ID, calculate a total. Functions name a responsibility, reduce duplication, and make behavior testable.

## 2. First function
```python
def normalize_status(status):
    """Return a lowercase, trimmed order status."""
    return status.strip().lower()

print(normalize_status("  DELIVERED "))
```
Output: `delivered`

- `def` defines a function.
- Parameter `status` receives input.
- `return` sends a result back.
- Docstring explains purpose.

## 3. Parameters and return
```python
def calculate_total(amount, tax):
    """Calculate amount including tax."""
    return amount + (amount * tax)

total = calculate_total(1000, 0.18)
print(total)
```
Output: `1180.0`

Prefer returning values over printing inside reusable logic. Printing belongs to the caller/reporting layer.

## 4. Validate before transforming
```python
def is_valid_order(order):
    """Check minimum required order fields."""
    return order.get("order_id") is not None and order.get("amount") is not None

orders = [{"order_id": 1, "amount": 200}, {"order_id": None, "amount": 50}]
valid_orders = []
for order in orders:
    if is_valid_order(order):
        valid_orders.append(order)
print(valid_orders)
```
Output: `[{'order_id': 1, 'amount': 200}]`

## 5. Modules
Create `transformations.py`:
```python
def clean_text(value):
    """Trim text and convert it to lowercase."""
    return value.strip().lower()
```
In `main.py`:
```python
from transformations import clean_text
print(clean_text(" SOUTH "))
```
Output: `south`

Keep module names lowercase; avoid naming files after standard modules (e.g., `json.py`, `logging.py`).

## 6. Mini-project — Reusable Transformation Library
Create modules for text cleaning, status normalization, and required-field validation. Keep source reading, transformations, and summary output separated.

## Common mistakes
- Function has no return, so caller receives None.
- Mutating global variables unexpectedly.
- Too many responsibilities in one function.
- Repeating logic instead of reusing it.
- Catching every exception inside a function and hiding errors.

## Revision
Functions define reusable behavior; parameters supply inputs; return values enable composition; modules organize related code.