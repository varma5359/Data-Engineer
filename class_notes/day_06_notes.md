# Day 6: Python Introduction and Environment

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** Data Processing Utility  
**Learning time:** 2–3 hours

## 1. Learning outcomes

- Explain Python's role in data engineering.
- Set up and verify a Python environment.
- Understand interpreter, script, terminal and notebook.
- Use variables, built-in data types and collections.
- Write simple code to inspect and summarize source records.

## 2. Why Python in data engineering?

Python is widely used to connect to APIs and databases, process files, validate records, automate jobs and coordinate transformations. SQL is still essential for relational querying; Python complements it when a workflow needs application logic or integration.

Typical workflow: `Read source -> Validate -> Transform -> Write destination`.

## 3. Environment concepts

| Term | Meaning |
|---|---|
| Python interpreter | Program that executes Python code |
| Script | Saved Python source file, commonly `.py` |
| Terminal | Text interface for running commands |
| IDE/editor | Tool for editing and running code |
| Virtual environment | Isolated project package environment |
| Jupyter notebook | Interactive document with code and output cells |
| Package | Reusable Python code installed into an environment |

Use the same interpreter for installing packages and running the project. A frequent beginner issue is installing a package into one Python environment and executing code with another.

## 4. Verify installation

Open Command Prompt or terminal.

```bash
python --version
```

If Windows does not recognize `python`, try:

```bash
py --version
```

Check the executable:

```bash
python -c "import sys; print(sys.executable)"
```

Expected output: a Python version and the path to the interpreter. The exact version and path depend on the machine.

Create a project folder and virtual environment:

```bash
mkdir data_processing_utility
cd data_processing_utility
python -m venv .venv
```

Activate on Windows CMD:

```bat
.venv\Scripts\activate
```

Activate on PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Deactivate:

```bash
deactivate
```

## 5. First Python program

Create `hello.py`.

```python
print("Data Engineering with Python")
print("Reading source data")
print("Preparing a pipeline")
```

Run:

```bash
python hello.py
```

Expected output:

```text
Data Engineering with Python
Reading source data
Preparing a pipeline
```

`print()` displays a value. Python executes statements from top to bottom unless control flow changes the order.

## 6. Comments and indentation

```python
# This is a comment. Python does not execute it.
source_name = "orders.csv"

if source_name.endswith(".csv"):
    print("CSV source detected")
```

Indentation defines a code block. Use consistent four-space indentation. Incorrect indentation raises an error or changes the block's meaning.

## 7. Variables and naming

A variable is a name referring to a value.

```python
source_name = "orders.csv"
total_orders = 1250
pipeline_active = True
average_order_value = 845.75

print(source_name)
print(total_orders)
print(pipeline_active)
print(average_order_value)
```

Expected output: the four assigned values, each on a separate line.

Naming rules:
- Use meaningful names such as `total_orders`.
- Use lowercase words separated by underscores.
- Do not start a name with a number.
- Avoid Python keywords such as `class` and `for`.
- Constants are conventionally uppercase, e.g. `MAX_RETRIES = 3`.

## 8. Core data types

```python
order_id = 501
amount = 1250.50
status = "PAID"
is_valid = True
missing_value = None

print(type(order_id))
print(type(amount))
print(type(status))
print(type(is_valid))
print(type(missing_value))
```

| Type | Example | Use |
|---|---|---|
| int | `501` | Counts and whole numbers |
| float | `1250.50` | Decimal measurements |
| str | `"PAID"` | Text and identifiers |
| bool | `True` | Conditions |
| NoneType | `None` | No value / missing value marker |

Important: identifiers such as postal codes and customer IDs are often strings, even if they contain digits. They are labels, not quantities.

## 9. Type conversion

```python
quantity_text = "3"
price_text = "199.50"

quantity = int(quantity_text)
price = float(price_text)
line_total = quantity * price

print(quantity)
print(price)
print(line_total)
```

Expected output:

```text
3
199.5
598.5
```

Conversion can fail when text is not valid for the target type. Validate input before converting.

```python
raw_quantity = "three"

print(raw_quantity.isdigit())
```

Expected output: `False`.

## 10. Operators

```python
quantity = 4
unit_price = 250

line_amount = quantity * unit_price
is_large_order = line_amount >= 1000
has_quantity = quantity > 0

print(line_amount)
print(is_large_order)
print(has_quantity)
```

Operators used often:
- Arithmetic: `+`, `-`, `*`, `/`, `//`, `%`, `**`.
- Comparison: `==`, `!=`, `>`, `<`, `>=`, `<=`.
- Logical: `and`, `or`, `not`.
- Membership: `in`, `not in`.

## 11. Strings

```python
customer_name = "Asha"
city = "Hyderabad"

message = f"Customer {customer_name} is from {city}"
print(message)
print(customer_name.upper())
print(customer_name.lower())
print(customer_name.strip())
print("A" in customer_name)
```

Strings are immutable: methods return new strings rather than changing the original value.

## 12. Lists

A list is an ordered, mutable collection.

```python
order_amounts = [250.0, 500.0, 125.0]

print(order_amounts[0])
print(len(order_amounts))

order_amounts.append(750.0)
order_amounts[1] = 550.0

print(order_amounts)
```

Expected final list: `[250.0, 550.0, 125.0, 750.0]`.

Indexes start at zero. Use lists when order matters and values may change.

## 13. Tuples

A tuple is ordered and immutable.

```python
source_config = ("orders_db", "postgresql", 5432)
print(source_config[0])
print(source_config[1])
```

Tuples are useful for fixed groups of values. Do not use them when the collection must be modified.

## 14. Dictionaries

A dictionary stores key-value pairs.

```python
order = {
    "order_id": 501,
    "customer_id": 1001,
    "amount": 1250.50,
    "status": "PAID"
}

print(order["order_id"])
print(order.get("status"))
print(order.get("payment_method", "UNKNOWN"))
```

Use `.get()` when a key may be absent. Direct indexing raises `KeyError` if the key does not exist.

## 15. Sets

A set stores unique values and does not preserve a meaningful positional index.

```python
statuses = ["PAID", "PENDING", "PAID", "CANCELLED"]
unique_statuses = set(statuses)

print(unique_statuses)
print(len(unique_statuses))
```

Set display order is not guaranteed. Use sets for membership checks and uniqueness, not for ordered reporting.

## 16. Conditional logic

```python
amount = 1500

if amount >= 1000:
    category = "HIGH_VALUE"
elif amount >= 500:
    category = "MEDIUM_VALUE"
else:
    category = "LOW_VALUE"

print(category)
```

Conditions let a pipeline apply business rules. Keep thresholds documented and confirmed by the business owner.

## 17. Loops

```python
order_ids = [501, 502, 503]

for order_id in order_ids:
    print("Processing order:", order_id)
```

A loop processes each item. Avoid changing a collection while iterating over it unless the behavior is deliberate.

## 18. Mini-project: Data Processing Utility

Create a small in-memory order dataset.

```python
orders = [
    {"order_id": 501, "quantity": 2, "unit_price": 300, "status": "PAID"},
    {"order_id": 502, "quantity": 1, "unit_price": 800, "status": "PENDING"},
    {"order_id": 503, "quantity": 3, "unit_price": 150, "status": "PAID"}
]

print(len(orders))
print(orders[0])
```

Calculate line amounts:

```python
for order in orders:
    order["line_amount"] = order["quantity"] * order["unit_price"]

print(orders)
```

Filter paid orders:

```python
paid_orders = []

for order in orders:
    if order["status"] == "PAID":
        paid_orders.append(order)

print(paid_orders)
```

Calculate paid revenue:

```python
paid_revenue = 0

for order in paid_orders:
    paid_revenue = paid_revenue + order["line_amount"]

print(paid_revenue)
```

Expected result: `1950`. This is a learning example; a real pipeline must handle currency, refunds, duplicate orders, nulls and approved revenue definitions.

## 19. Common errors

| Error | Cause | Fix |
|---|---|---|
| NameError | Variable name is undefined | Check spelling and execution order |
| TypeError | Operation not supported for types | Convert or validate types |
| ValueError | Value cannot be converted | Inspect input before conversion |
| KeyError | Dictionary key missing | Check schema or use `.get()` |
| IndexError | Index outside collection | Check length and index |
| IndentationError | Invalid indentation | Align code blocks |

## 20. Day 6 revision and practice

- Python runs through an interpreter.
- A virtual environment isolates project packages.
- Variables refer to values; types determine supported operations.
- Lists are mutable; tuples are immutable; dictionaries map keys to values; sets hold unique values.
- Validate types and business assumptions before processing records.

Practice: add a `currency` field, count paid and pending orders separately, and reject an order with a non-positive quantity.
