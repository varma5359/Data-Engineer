# Day 7: Functions and Modules

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** Reusable Transformation Library  
**Learning time:** 2–3 hours

## 1. Learning outcomes

- Explain why reusable code matters in data pipelines.
- Define, call and return values from functions.
- Distinguish parameters from arguments.
- Use positional, keyword and default arguments.
- Understand local variables and scope.
- Import built-in modules and create a simple project module.
- Build a small reusable order-transformation library.

## 2. Why functions are important

Imagine three pipeline steps calculate an order amount. If the same calculation is copied into three places, a business-rule change must be applied three times. One copy may be missed.

A function gives a named operation one implementation:

`Input -> Function -> Output`

Functions help engineers:

- Reuse logic instead of copying it.
- Test one transformation independently.
- Give business operations meaningful names.
- Keep pipeline steps small and understandable.
- Fix a rule in one place.

A function is not automatically better for every two-line calculation. Use one when a block represents a meaningful operation or needs reuse.

## 3. Define and call a function

```python
def calculate_line_amount(quantity, unit_price):
    line_amount = quantity * unit_price
    return line_amount

result = calculate_line_amount(3, 250)
print(result)
```

Expected output:

```text
750
```

Explanation:

- `def` begins a function definition.
- `calculate_line_amount` is the function name.
- `quantity` and `unit_price` are parameters.
- The indented statements form the function body.
- `return` sends a value back to the caller.
- `calculate_line_amount(3, 250)` calls the function with arguments.

Defining a function does not run its body. The body executes when the function is called.

## 4. Parameters and arguments

A **parameter** is a variable in the function definition. An **argument** is the actual value passed during a call.

```python
def create_status_message(order_id, status):
    return f"Order {order_id}: {status}"

message = create_status_message(501, "SHIPPED")
print(message)
```

Expected output:

```text
Order 501: SHIPPED
```

Here, `order_id` and `status` are parameters. The values `501` and `"SHIPPED"` are arguments.

## 5. Positional arguments

Positional arguments are matched to parameters by their order.

```python
def calculate_discount(amount, discount):
    return amount - discount

print(calculate_discount(1000, 100))
```

Expected output:

```text
900
```

The first argument is assigned to `amount`; the second is assigned to `discount`. Passing them in the wrong order may produce a logically incorrect result even when Python does not raise an error.

## 6. Keyword arguments

Keyword arguments identify parameters by name.

```python
def build_customer_label(customer_name, city):
    return f"{customer_name} - {city}"

label = build_customer_label(city="Hyderabad", customer_name="Asha")
print(label)
```

Expected output:

```text
Asha - Hyderabad
```

Keyword arguments improve readability when a function has several parameters.

## 7. Default parameters

A default value is used when the caller does not provide that argument.

```python
def calculate_tax(amount, tax_rate=0.05):
    return amount * tax_rate

print(calculate_tax(1000))
print(calculate_tax(1000, 0.18))
```

Expected output:

```text
50.0
180.0
```

- First call uses the default rate of 5%.
- Second call overrides it with 18%.

For real tax processing, rates must come from the applicable business rules and jurisdiction; the example values are only for learning.

## 8. Return versus print

`print()` displays a value. `return` gives a value back to the caller so another step can use it.

```python
def calculate_total(quantity, price):
    total = quantity * price
    return total

order_total = calculate_total(2, 300)
tax = order_total * 0.05

print(order_total)
print(tax)
```

Expected output:

```text
600
30.0
```

If a function only prints its result, another function cannot directly use that displayed value. Data transformation functions usually return results.

## 9. What happens when there is no return?

A function without an explicit return statement returns `None`.

```python
def show_pipeline_name():
    print("Retail ETL")

result = show_pipeline_name()
print(result)
```

Expected output:

```text
Retail ETL
None
```

The first line is printed inside the function. The second line prints the returned value, which is `None`.

## 10. Local variables and scope

A variable created inside a function is normally local to that function.

```python
def calculate_order():
    order_amount = 500
    return order_amount

result = calculate_order()
print(result)
```

Expected output:

```text
500
```

The name `order_amount` is created inside the function. Use the returned value outside it rather than depending on an internal variable.

## 11. Validate function inputs

Functions can reject invalid values instead of producing misleading results.

```python
def calculate_line_amount(quantity, unit_price):
    if quantity <= 0:
        raise ValueError("quantity must be greater than zero")

    if unit_price < 0:
        raise ValueError("unit_price cannot be negative")

    return quantity * unit_price

print(calculate_line_amount(2, 150))
```

Expected output:

```text
300
```

The function checks its assumptions before calculating. If an invalid value is passed, Python raises a `ValueError`. Exception handling is covered in Day 10.

Try an invalid call separately:

```python
calculate_line_amount(0, 150)
```

Expected result:

```text
ValueError: quantity must be greater than zero
```

## 12. Functions can return multiple values

Python can return a tuple containing multiple results.

```python
def summarize_amounts(amounts):
    total = sum(amounts)
    count = len(amounts)
    return total, count

total_sales, order_count = summarize_amounts([100, 250, 150])

print(total_sales)
print(order_count)
```

Expected output:

```text
500
3
```

The caller unpacks the returned pair into two variables.

## 13. Built-in modules

A module is a Python file containing reusable definitions. Python also provides a standard library of modules.

### Use the math module

```python
import math

print(math.ceil(12.3))
print(math.floor(12.8))
```

Expected output:

```text
13
12
```

- `import math` makes the module available.
- `math.ceil()` rounds upward to the next integer.
- `math.floor()` rounds downward to the previous integer.

### Use the datetime module

```python
from datetime import date

pipeline_date = date(2026, 11, 10)
print(pipeline_date)
print(pipeline_date.year)
```

Expected output:

```text
2026-11-10
2026
```

The example uses an explicit date so the output is stable when students rerun the notebook.

## 14. Create your own module

Create a project folder:

```text
reusable_transformation_library/
    main.py
    transformations.py
```

In `transformations.py`:

```python
def calculate_line_amount(quantity, unit_price):
    if quantity <= 0:
        raise ValueError("quantity must be positive")
    if unit_price < 0:
        raise ValueError("unit price cannot be negative")
    return quantity * unit_price


def normalize_status(status):
    return status.strip().upper()
```

In `main.py`:

```python
from transformations import calculate_line_amount, normalize_status

amount = calculate_line_amount(3, 200)
status = normalize_status(" shipped ")

print(amount)
print(status)
```

Run from the project folder:

```bash
python main.py
```

Expected output:

```text
600
SHIPPED
```

Python finds `transformations.py` because it is in the same project directory as `main.py`. The import statement makes the selected functions available to the main script.

## 15. Packages and pip

A package is a way to organize and distribute Python modules. Many third-party packages are installed using `pip`.

Check pip:

```bash
python -m pip --version
```

Install a package:

```bash
python -m pip install pandas
```

Check installation:

```bash
python -m pip show pandas
```

Use `python -m pip` so pip is associated with the Python interpreter named by `python`.

## 16. Virtual environments

A virtual environment isolates project dependencies.

Create it:

```bash
python -m venv .venv
```

Activate on Windows CMD:

```bash
.venv\Scripts\activate
```

Activate on PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Install a dependency inside the active environment:

```bash
python -m pip install pandas
```

Save dependency versions:

```bash
python -m pip freeze > requirements.txt
```

Install dependencies later:

```bash
python -m pip install -r requirements.txt
```

A virtual environment is normally not committed to Git. Commit the dependency file instead.

## 17. Mini-project: Reusable Transformation Library

**Business scenario:** A retail pipeline receives order records. Several steps need the same calculations and status normalization. We will build a small module and call it from a main script.

### Step 1 — Create the project structure

```text
reusable_transformation_library/
    main.py
    transformations.py
```

### Step 2 — Write the reusable transformations

In `transformations.py`:

```python
def normalize_customer_name(name):
    return name.strip().title()


def normalize_order_status(status):
    return status.strip().upper()


def calculate_order_amount(quantity, unit_price):
    if quantity <= 0:
        raise ValueError("quantity must be positive")
    if unit_price < 0:
        raise ValueError("unit price cannot be negative")
    return quantity * unit_price
```

Explanation:

- `strip()` removes spaces at the beginning and end.
- `title()` capitalizes words in a name.
- `upper()` standardizes status values.
- The calculation validates its inputs before multiplying.
- Each function does one clearly named task.

### Step 3 — Prepare the source records

In `main.py`:

```python
from transformations import (
    normalize_customer_name,
    normalize_order_status,
    calculate_order_amount
)

orders = [
    {"order_id": 501, "customer": " asha ", "status": " paid ", "quantity": 2, "unit_price": 250},
    {"order_id": 502, "customer": "RAVI", "status": "shipped", "quantity": 1, "unit_price": 800},
    {"order_id": 503, "customer": " meena", "status": " pending ", "quantity": 3, "unit_price": 150}
]

print(len(orders))
```

Expected output:

```text
3
```

There are three source records. The values intentionally contain inconsistent spaces and letter casing.

### Step 4 — Normalize customer and status fields

```python
for order in orders:
    order["customer"] = normalize_customer_name(order["customer"])
    order["status"] = normalize_order_status(order["status"])

for order in orders:
    print(order["customer"], order["status"])
```

Expected output:

```text
Asha PAID
Ravi SHIPPED
Meena PENDING
```

The same reusable rules are applied to every record. If the normalization rule changes, update the module once.

### Step 5 — Calculate order amounts

```python
for order in orders:
    order["order_amount"] = calculate_order_amount(
        order["quantity"],
        order["unit_price"]
    )

for order in orders:
    print(order["order_id"], order["order_amount"])
```

Expected output:

```text
501 500
502 800
503 450
```

The main script controls the workflow; the module contains reusable transformation logic.

### Step 6 — Summarize sales by status

```python
status_totals = {}

for order in orders:
    status = order["status"]
    amount = order["order_amount"]

    if status not in status_totals:
        status_totals[status] = 0

    status_totals[status] += amount

print(status_totals)
```

Expected output:

```text
{'PAID': 500, 'SHIPPED': 800, 'PENDING': 450}
```

The dictionary stores one running total for each status. This is a basic aggregation pattern.

### Step 7 — Run the project

From the project directory:

```bash
python main.py
```

The terminal should display the outputs from the previous steps in order. If Python reports `ModuleNotFoundError: No module named 'transformations'`, check that both files are in the same folder and that the command is run from that project directory.

## 18. Common mistakes

| Mistake | Cause | Fix |
|---|---|---|
| Function not called | Function was defined but never invoked | Call it with arguments |
| Output is `None` | Function printed but did not return a value | Return the result |
| Wrong argument order | Positional values mapped incorrectly | Use clear names or keyword arguments |
| NameError | Variable is outside its scope or misspelled | Pass values as parameters and return results |
| ModuleNotFoundError | Python cannot locate the module | Check folder structure and working directory |
| Different package versions | Environment dependencies differ | Use a virtual environment and requirements file |

## 19. Practical exercises

1. Write a function that converts a status to uppercase.
2. Write a function that calculates quantity multiplied by price.
3. Add a default tax-rate parameter to a calculation.
4. Return both total sales and record count from one function.
5. Move two functions into a separate module and import them.
6. Create a virtual environment and install one package.

## 20. Interview revision

- A function is a named reusable block of code.
- A parameter appears in the function definition; an argument is passed during a call.
- `return` sends a result to the caller; `print` only displays it.
- A default parameter value is used when the caller omits that argument.
- Local variables belong to the function scope in which they are created.
- A module is a Python file; packages organize modules.
- A virtual environment isolates dependencies for a project.
- Reusable transformations are easier to test and maintain than duplicated logic.

**Day 7 checkpoint:** You should be able to write a function, validate its inputs, return a result, import it from another Python file and explain how a virtual environment keeps project dependencies separate.
