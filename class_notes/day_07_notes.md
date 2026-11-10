# Day 7: Functions and Modules

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** Reusable Transformation Library  
**Learning time:** 2–3 hours

## 1. Learning outcomes

- Explain why functions improve maintainability.
- Define and call functions.
- Understand parameters, arguments, return values and scope.
- Use default, keyword and positional arguments.
- Import built-in and project modules.
- Organize reusable transformation code.

## 2. Why functions matter

Suppose three pipeline steps calculate order totals. If the same logic is copied into three places, a business-rule change must be applied three times. One copy may be missed.

A function gives a named operation one implementation:

`Input -> Function -> Output`

Benefits:
- Reuse logic.
- Reduce duplication.
- Make code easier to test.
- Give a business operation a clear name.
- Separate pipeline stages.

## 3. Define a function

```python
def calculate_line_amount(quantity, unit_price):
    line_amount = quantity * unit_price
    return line_amount

result = calculate_line_amount(3, 250)
print(result)
```

Expected output: `750`.

- `def` starts a function definition.
- `calculate_line_amount` is the function name.
- `quantity` and `unit_price` are parameters.
- The indented block is the function body.
- `return` sends a result to the caller.
- The call passes arguments.

A function definition does not execute its body until it is called.

## 4. Parameters and arguments

Parameter: name in the function definition.  
Argument: actual value supplied during the call.

```python
def format_order_status(order_id, status):
    return f"Order {order_id}: {status}"

message = format_order_status(501, "PAID")
print(message)
```

Positional arguments are matched by position. Keyword arguments are matched by parameter name:

```python
message = format_order_status(status="PENDING", order_id=502)
print(message)
```

Keyword arguments improve readability when a function has several parameters.

## 5. Return vs print

```python
def add_tax(amount, tax_rate):
    return amount + (amount * tax_rate)

final_amount = add_tax(1000, 0.18)
print(final_amount)
```

`return` makes a value available to the calling code. `print` only displays a value. A reusable transformation should generally return its result rather than only print it.

## 6. Default parameters

```python
def calculate_discount(amount, discount_rate=0.10):
    return amount - (amount * discount_rate)

print(calculate_discount(1000))
print(calculate_discount(1000, 0.20))
```

Expected outputs: `900.0` and `800.0`.

Use defaults only when the default is a valid business assumption. Do not hide a business decision such as tax rate inside code without approval.

## 7. Validate function inputs

```python
def calculate_line_amount(quantity, unit_price):
    if quantity <= 0:
        raise ValueError("quantity must be positive")

    if unit_price < 0:
        raise ValueError("unit_price cannot be negative")

    return quantity * unit_price

print(calculate_line_amount(2, 150))
```

Validation makes function contracts explicit. Production functions should also consider types, null values, decimal precision and currency.

## 8. Scope

Variables created inside a function are local unless declared otherwise.

```python
tax_rate = 0.18

def calculate_tax(amount):
    tax_amount = amount * tax_rate
    return tax_amount

print(calculate_tax(1000))
```

Prefer passing important configuration explicitly when it may differ between environments. Hidden global dependencies make code harder to test.

## 9. Functions with collections

```python
def get_paid_orders(orders):
    paid_orders = []

    for order in orders:
        if order["status"] == "PAID":
            paid_orders.append(order)

    return paid_orders

orders = [
    {"order_id": 1, "status": "PAID"},
    {"order_id": 2, "status": "PENDING"},
    {"order_id": 3, "status": "PAID"}
]

print(get_paid_orders(orders))
```

The function receives a list of dictionaries and returns a new list. It does not write to a database or modify the source system.

## 10. Reusable transformation examples

```python
def normalize_status(status):
    return status.strip().upper()

print(normalize_status(" paid "))
```

```python
def is_valid_quantity(quantity):
    return quantity is not None and quantity > 0

print(is_valid_quantity(3))
print(is_valid_quantity(0))
```

```python
def calculate_order_total(items):
    total = 0

    for item in items:
        total += item["quantity"] * item["unit_price"]

    return total

items = [
    {"quantity": 2, "unit_price": 100},
    {"quantity": 1, "unit_price": 250}
]

print(calculate_order_total(items))
```

Expected output: `450`.

## 11. Built-in modules

A module is a Python file or library that groups related code.

```python
import math
import json
from pathlib import Path

print(math.ceil(12.2))
print(Path("data/orders.csv").suffix)
```

- `import math` imports the module.
- `from pathlib import Path` imports one name.
- `Path` helps work with file paths across operating systems.
- `json` provides JSON serialization and parsing.

## 12. Create a project module

Project structure:

```text
reusable_transformation_library/
    main.py
    transformations.py
    data/
        orders.json
```

In `transformations.py`:

```python
def normalize_status(status):
    return status.strip().upper()

def calculate_line_amount(quantity, unit_price):
    return quantity * unit_price
```

In `main.py`:

```python
from transformations import normalize_status, calculate_line_amount

status = normalize_status(" paid ")
amount = calculate_line_amount(2, 300)

print(status)
print(amount)
```

Run from the project directory:

```bash
python main.py
```

Expected output:

```text
PAID
600
```

Run the script from the correct project directory so Python can resolve the module.

## 13. Main guard

```python
def main():
    print("Transformation library ready")

if __name__ == "__main__":
    main()
```

The main guard runs the entry-point code when the file is executed directly, but not when imported as a module.

## 14. Docstrings

```python
def calculate_line_amount(quantity, unit_price):
    """Return quantity multiplied by unit price."""
    return quantity * unit_price

help(calculate_line_amount)
```

A useful docstring explains purpose, inputs, output and important assumptions. Do not write a docstring that merely repeats the function name.

## 15. Mini-project: Reusable Transformation Library

Create these files:

```text
reusable_transformation_library/
    main.py
    transformations.py
```

Add functions for:
- Normalizing status text.
- Validating a positive quantity.
- Calculating line amount.
- Filtering paid orders.
- Summarizing order count and paid revenue.

Example input:

```python
orders = [
    {"order_id": 101, "quantity": 2, "unit_price": 100, "status": "paid"},
    {"order_id": 102, "quantity": 1, "unit_price": 500, "status": "pending"},
    {"order_id": 103, "quantity": 3, "unit_price": 50, "status": "PAID"}
]
```

Expected paid revenue: `350`.

Verify each function with normal, boundary and invalid inputs. Keep functions focused: one function should perform one understandable transformation.

## 16. Common mistakes

| Mistake | Correction |
|---|---|
| Printing instead of returning | Return values for reuse |
| Defining but never calling | Invoke the function |
| Too many unrelated responsibilities | Split into focused functions |
| Hidden global configuration | Pass dependencies explicitly |
| Unclear parameter names | Use business-meaningful names |
| Mutable default list/dict | Use `None` and create a new collection inside |
| Import error | Check module name and working directory |

Mutable default example to avoid:

```python
def add_record(record, records=[]):
    records.append(record)
    return records
```

Safer pattern:

```python
def add_record(record, records=None):
    if records is None:
        records = []

    records.append(record)
    return records
```

## 17. Day 7 revision

- A function is a reusable named block of code.
- Parameters define inputs; arguments supply values.
- Return values allow downstream steps to use results.
- Modules organize reusable code.
- Docstrings document function contracts.
- Focused functions are easier to test and maintain.
