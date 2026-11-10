# Day 10: Exception Handling and Debugging

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** Resilient File Processor  
**Learning time:** 2–3 hours

## 1. Learning outcomes

- Explain the difference between syntax errors and runtime exceptions.
- Read a traceback and locate the failing statement.
- Use `try`, `except`, `else` and `finally`.
- Catch specific exception types instead of hiding every failure.
- Raise meaningful errors when input data violates a rule.
- Use Python logging to record pipeline activity.
- Build a file processor that reports success and failure clearly.

## 2. Why error handling matters

A data pipeline depends on files, APIs, databases and external services. Any of them can fail.

Examples:
- The expected source file was not delivered.
- A CSV contains a quantity such as `"many"`.
- A JSON file is incomplete.
- The process does not have permission to read a folder.
- A destination database is temporarily unavailable.

A reliable pipeline does not pretend a failed step succeeded. It explains what failed, preserves useful evidence and allows an operator to decide whether to retry, skip or stop.

## 3. Errors and exceptions

A syntax error prevents Python from understanding the program. An exception happens while a syntactically valid program is running.

| Exception | Typical cause |
|---|---|
| `FileNotFoundError` | File path does not exist |
| `PermissionError` | Process cannot access a file |
| `ValueError` | Value is not valid for the requested conversion |
| `TypeError` | An operation receives an incompatible type |
| `KeyError` | Dictionary key is absent |
| `IndexError` | List index is outside its valid range |
| `ZeroDivisionError` | Divisor is zero |
| `JSONDecodeError` | Text is not valid JSON |

## 4. Read a traceback

Run this deliberately failing example:

```python
quantity = int("three")
print(quantity)
```

Expected result: Python displays a traceback ending with a `ValueError` similar to:

```text
ValueError: invalid literal for int() with base 10: 'three'
```

How to investigate:

1. Read the final exception type and message.
2. Find the line number where the error occurred.
3. Inspect the value passed to the operation.
4. Trace where that value came from.
5. Reproduce the issue with the smallest possible input.

Do not start by changing unrelated code. The message tells us the conversion to integer failed because the text was not numeric.

## 5. Handle one known exception

```python
quantity_text = "three"

try:
    quantity = int(quantity_text)
    print(quantity)
except ValueError:
    print("Quantity is not a valid whole number")
```

Expected output:

```text
Quantity is not a valid whole number
```

- The `try` block contains code that may fail.
- The `except ValueError` block handles this specific conversion problem.
- The program continues after the handler.

Use a specific exception whenever possible. Catching every exception can hide programming defects.

## 6. Multiple exception types

```python
source_path = "missing_orders.csv"

try:
    file = open(source_path, "r", encoding="utf-8")
    print(file.read())
    file.close()
except FileNotFoundError:
    print("Source file was not found")
except PermissionError:
    print("Permission denied while reading the source")
```

If the file does not exist, expected output:

```text
Source file was not found
```

The first matching handler runs. These two problems have different operational causes, so separate messages help troubleshooting.

## 7. The else block

The `else` block runs only when the `try` block completes without an exception.

```python
quantity_text = "5"

try:
    quantity = int(quantity_text)
except ValueError:
    print("Invalid quantity")
else:
    print("Valid quantity:", quantity)
```

Expected output:

```text
Valid quantity: 5
```

Keep success-only logic in `else` when that makes the control flow easier to understand.

## 8. The finally block

The `finally` block runs whether an exception occurred or not. It is often used for cleanup.

```python
try:
    print("Starting file operation")
    result = 10 / 2
    print(result)
except ZeroDivisionError:
    print("Cannot divide by zero")
finally:
    print("File operation finished")
```

Expected output:

```text
Starting file operation
5.0
File operation finished
```

For file handling, prefer a `with open(...)` context manager because it closes the file automatically, including when an exception occurs.

## 9. Use a context manager for files

```python
with open("sample.txt", "w", encoding="utf-8") as file:
    file.write("pipeline started")

print("Write completed")
```

Expected output:

```text
Write completed
```

The `with` statement manages the file resource. Once the indented block ends, the file is closed automatically.

## 10. Raise an exception for invalid business data

A conversion can succeed while a business value is still invalid. For example, quantity `-3` is an integer but may be disallowed for a normal sales order.

```python
quantity = -3

if quantity <= 0:
    raise ValueError("quantity must be greater than zero")
```

Expected result:

```text
ValueError: quantity must be greater than zero
```

Use `raise` when the current operation cannot safely continue with the supplied value.

## 11. Validate and convert one record

```python
order = {
    "order_id": "501",
    "quantity": "2",
    "unit_price": "250.00"
}

try:
    order_id = int(order["order_id"])
    quantity = int(order["quantity"])
    unit_price = float(order["unit_price"])

    if quantity <= 0:
        raise ValueError("quantity must be positive")

    if unit_price < 0:
        raise ValueError("unit_price cannot be negative")

    order_amount = quantity * unit_price
    print(order_id, order_amount)

except (ValueError, KeyError) as error:
    print("Order rejected:", error)
```

Expected output:

```text
501 500.0
```

This example handles conversion and missing-field failures. In production, record-level rejection should also preserve the order identifier and the reason so the source can be corrected.

## 12. Why broad exception handling is risky

Avoid this pattern:

```python
try:
    total = 100 / 0
except:
    print("Something happened")
```

It catches exceptions too broadly and hides their type. It may also intercept exceptions that should stop the program.

Prefer:

```python
try:
    total = 100 / 0
except ZeroDivisionError as error:
    print("Calculation failed:", error)
```

Expected output:

```text
Calculation failed: division by zero
```

## 13. Logging fundamentals

Logging records events while a program runs. It is more useful than scattered print statements for repeatable applications.

Common levels:

| Level | Purpose |
|---|---|
| DEBUG | Detailed diagnostic information |
| INFO | Normal progress |
| WARNING | Unexpected situation; process may continue |
| ERROR | An operation failed |
| CRITICAL | Serious failure that may stop the application |

## 14. First logging example

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(levelname)s:%(message)s"
)

logging.info("Pipeline started")
logging.warning("One optional field is missing")
logging.error("Could not load source file")
```

Expected output:

```text
INFO:Pipeline started
WARNING:One optional field is missing
ERROR:Could not load source file
```

- `level=logging.INFO` displays INFO and more severe messages.
- `format` controls the displayed message structure.
- Logging levels help separate normal progress from failures.

## 15. Log useful context

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(levelname)s:%(message)s"
)

source_file = "orders_2026_11_10.csv"
row_count = 250

logging.info("Reading source file: %s", source_file)
logging.info("Rows received: %s", row_count)
```

Expected output:

```text
INFO:Reading source file: orders_2026_11_10.csv
INFO:Rows received: 250
```

Context such as file name, batch identifier and record count helps an engineer investigate a failed run. Never log passwords, access tokens or sensitive personal data.

## 16. Log an exception with its traceback

```python
import logging

logging.basicConfig(level=logging.ERROR)

try:
    amount = int("unknown")
except ValueError:
    logging.exception("Could not convert order amount")
```

Expected result: an ERROR log message and traceback ending in `ValueError`. The exact timestamp, file path and line number depend on the environment.

`logging.exception()` should be called inside an exception handler. It includes traceback information.

## 17. Debugging with small checks

When a transformation returns an unexpected result, inspect assumptions in order.

```python
raw_quantity = " 4 "

print("Raw value:", repr(raw_quantity))
print("Raw type:", type(raw_quantity))

clean_quantity = raw_quantity.strip()
print("Clean value:", repr(clean_quantity))

quantity = int(clean_quantity)
print("Converted:", quantity)
```

Expected output:

```text
Raw value: ' 4 '
Raw type: <class 'str'>
Clean value: '4'
Converted: 4
```

The `repr()` display makes leading and trailing spaces visible. Debugging often means inspecting intermediate values, not guessing.

## 18. Mini-project: Resilient File Processor

**Scenario:** A retail team receives a CSV file of orders. The processor should read the file, convert numeric fields, reject invalid rows with reasons, and report a summary.

### Step 1 — Create the project folder and sample file

Create `day10_resilient_processor` with a file named `orders.csv`.

Contents of `orders.csv`:

```csv
order_id,quantity,unit_price
1001,2,250
1002,1,800
1003,three,150
1004,-1,200
1005,3,100
```

The file intentionally includes one non-numeric quantity and one negative quantity.

### Step 2 — Read the file

```python
import csv

with open("orders.csv", "r", newline="", encoding="utf-8") as file:
    reader = csv.DictReader(file)
    rows = list(reader)

print(len(rows))
print(rows[0])
```

Expected output:

```text
5
{'order_id': '1001', 'quantity': '2', 'unit_price': '250'}
```

CSV fields are initially text. Conversion and validation are separate steps.

### Step 3 — Prepare accepted and rejected collections

```python
accepted_orders = []
rejected_orders = []

print(type(accepted_orders))
print(type(rejected_orders))
```

Expected output:

```text
<class 'list'>
<class 'list'>
```

We keep valid rows separate from rejected rows so invalid input does not silently enter the output dataset.

### Step 4 — Process and validate each row

```python
for row in rows:
    try:
        order_id = int(row["order_id"])
        quantity = int(row["quantity"])
        unit_price = float(row["unit_price"])

        if quantity <= 0:
            raise ValueError("quantity must be greater than zero")

        if unit_price < 0:
            raise ValueError("unit_price cannot be negative")

        order_amount = quantity * unit_price

        accepted_orders.append({
            "order_id": order_id,
            "quantity": quantity,
            "unit_price": unit_price,
            "order_amount": order_amount
        })

    except (ValueError, KeyError) as error:
        rejected_orders.append({
            "order_id": row.get("order_id", "UNKNOWN"),
            "reason": str(error)
        })

print("Accepted:", len(accepted_orders))
print("Rejected:", len(rejected_orders))
```

Expected output:

```text
Accepted: 3
Rejected: 2
```

Each row is processed independently. One bad row does not stop the remaining rows. This is a deliberate record-level error policy; some pipelines should fail the entire batch instead.

### Step 5 — Inspect rejected records

```python
for rejected in rejected_orders:
    print(rejected)
```

Expected output:

```text
{'order_id': '1003', 'reason': "invalid literal for int() with base 10: 'three'"}
{'order_id': '1004', 'reason': 'quantity must be greater than zero'}
```

A rejection reason allows a source owner to correct the data. Avoid storing only a generic message such as "bad row".

### Step 6 — Calculate a summary

```python
total_sales = 0

for order in accepted_orders:
    total_sales += order["order_amount"]

print("Accepted rows:", len(accepted_orders))
print("Rejected rows:", len(rejected_orders))
print("Accepted sales:", total_sales)
```

Expected output:

```text
Accepted rows: 3
Rejected rows: 2
Accepted sales: 1550.0
```

The total is 500 + 800 + 300. Rejected records are excluded from the accepted sales total.

### Step 7 — Write accepted records to a new CSV

```python
import csv

with open("accepted_orders.csv", "w", newline="", encoding="utf-8") as file:
    fieldnames = ["order_id", "quantity", "unit_price", "order_amount"]
    writer = csv.DictWriter(file, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(accepted_orders)

print("Accepted output file created")
```

Expected output:

```text
Accepted output file created
```

The new file contains only validated rows. In a production workflow, also write rejected rows to a separate quarantine file and include a batch ID and processing timestamp.

## 19. Common mistakes and troubleshooting

| Symptom | Likely cause | Action |
|---|---|---|
| FileNotFoundError | Wrong working directory or filename | Print/check current directory and path |
| ValueError | Invalid conversion or business value | Inspect raw value and validate before conversion |
| KeyError | Column missing or misspelled | Inspect CSV headers and schema |
| Empty output | Filter rejected every row | Check conditions and source values |
| Duplicate logs | Logging configured repeatedly | Configure logging once near application startup |
| Error hidden | Broad empty `except` | Catch specific exceptions and log context |

## 20. Practical exercises

1. Handle a missing input file without crashing silently.
2. Convert a text price into a float and handle invalid values.
3. Add a rule that rejects zero or negative unit prices.
4. Log the number of rows read and accepted.
5. Write rejected rows to `rejected_orders.csv`.
6. Explain when a pipeline should stop the batch instead of rejecting one row.

## 21. Interview revision

- An exception is a runtime event that interrupts normal execution.
- A traceback shows the call path and failing location.
- `try/except` handles known failures; `else` runs on success; `finally` runs during cleanup.
- Catch specific exceptions so programming errors are not hidden.
- `raise` communicates that an operation cannot safely continue.
- Logging provides durable, level-based operational evidence.
- A rejected-record strategy must preserve the record identifier and reason.
- A pipeline must define whether errors fail the whole batch or quarantine individual records.

**Day 10 checkpoint:** You should be able to diagnose a traceback, handle expected file and data errors, log useful context and build a processor that separates accepted and rejected records.


---

# Beginner Teaching Extension — Day 10

## A. Read an error as a message

A syntax error means Python cannot parse the code. A runtime exception happens while valid code is executing. A traceback shows the exception type, message and line where execution failed. Read the final lines first, then inspect the named source line and its inputs.

## B. Catch a known conversion error

```python
raw_quantity = "unknown"
try:
    quantity = int(raw_quantity)
    print("Quantity:", quantity)
except ValueError:
    print("Rejected: quantity is not an integer")
```

Expected output:

```text
Rejected: quantity is not an integer
```

The try block contains a statement that may fail. The except block handles ValueError specifically. It does not hide unrelated errors such as a misspelled variable name.

## C. Understand else and finally

```python
try:
    quantity = int("4")
except ValueError:
    print("Invalid quantity")
else:
    print("Valid quantity:", quantity)
finally:
    print("Validation attempt finished")
```

Expected output:

```text
Valid quantity: 4
Validation attempt finished
```

else runs only when try completes without an exception. finally runs during normal completion and exception handling, making it useful for cleanup.

## D. Separate invalid data from business-rule failures

```python
raw_values = ["10", "unknown", "0", "25"]
accepted = []
rejected = []

for raw_value in raw_values:
    try:
        number = int(raw_value)
        if number <= 0:
            rejected.append((raw_value, "Must be positive"))
        else:
            accepted.append(number)
    except ValueError:
        rejected.append((raw_value, "Not an integer"))

print("Accepted:", accepted)
print("Rejected:", rejected)
```

Expected output:

```text
Accepted: [10, 25]
Rejected: [('unknown', 'Not an integer'), ('0', 'Must be positive')]
```

The conversion error and business rule are different. Retain the raw value and rejection reason so a source owner can investigate.

## E. Raise a clear error

```python
def validate_quantity(quantity):
    if quantity <= 0:
        raise ValueError("Quantity must be greater than zero")
    return quantity

print(validate_quantity(3))
```

Expected output: 3. Passing zero raises ValueError. Raising an exception makes a failure explicit to the caller; expected bad rows may instead be collected as rejected records.

## F. Logging records operational events

```python
import logging
logging.basicConfig(level=logging.INFO, format="%(levelname)s: %(message)s")
logging.info("File processing started")
logging.warning("One row was rejected")
logging.error("Destination load failed")
```

Expected output:

```text
INFO: File processing started
WARNING: One row was rejected
ERROR: Destination load failed
```

INFO is a normal operational event; WARNING is an unexpected condition; ERROR is a failed operation. Do not log passwords, tokens or unnecessary personal data. Existing logging configuration in a notebook may affect formatting.

## G. Retry only failures that may recover

A temporary network interruption may recover on retry. An invalid integer string will not become valid merely by retrying. A permission error usually requires configuration correction. A retry policy should be bounded, delayed, logged and limited to transient failures.

## H. Debugging sequence

1. Reproduce the issue with the smallest input.
2. Read the exception type and traceback line.
3. Inspect the input value and its type.
4. Check the assumption made by the failing statement.
5. Correct one cause and rerun the same test.
6. Add a test so the failure is not reintroduced.

## I. Readiness check

Students should explain ValueError versus a business-rule rejection, use a specific except clause, explain else/finally, identify a transient failure, and state why a pipeline must not report success after a required step failed.
