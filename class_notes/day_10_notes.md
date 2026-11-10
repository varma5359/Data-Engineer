# Day 10: Exception Handling and Debugging

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** Resilient File Processor

## 1. Learning outcomes
- Explain exceptions and tracebacks.
- Use try/except/else/finally correctly.
- Raise meaningful errors for invalid data.
- Create custom exceptions when they improve clarity.
- Use Python logging levels and structured context.
- Debug a failing file-processing workflow.

## 2. Why error handling matters
A pipeline interacts with files, APIs, databases and external services. Failures are normal: a file may be missing, an API may time out, a field may contain invalid text, or a destination may be unavailable.

Good error handling makes failures visible and recoverable. It must not hide data loss by pretending that a failed step succeeded.

## 3. Errors and exceptions
A syntax error prevents code from being parsed. An exception occurs while code executes.

Common exceptions:
- `FileNotFoundError`: requested file does not exist.
- `PermissionError`: process lacks access.
- `ValueError`: value is not valid for an operation.
- `TypeError`: operation receives an unsupported type.
- `KeyError`: dictionary key is absent.
- `ZeroDivisionError`: division by zero.
- `JSONDecodeError`: JSON text is invalid.
- `TimeoutError`: operation exceeded a time limit.

## 4. Read a traceback
A traceback shows the call path and the location where an exception occurred. Read from the bottom upward:
1. Identify the exception type and message.
2. Find the line that raised it.
3. Inspect the variables used on that line.
4. Trace where those values were created.
5. Reproduce with the smallest input.

Do not fix an error by suppressing it before understanding its cause.

## 5. Basic try and except
```python
raw_amount = "1250.50"

try:
    amount = float(raw_amount)
    print(amount)
except ValueError:
    print("Amount is not a valid number")
```

The code in `try` may fail. The matching `except` handles the specified exception.

Prefer specific exceptions. A broad `except Exception` can be appropriate at a pipeline boundary for logging and controlled failure, but should not silently continue.

## 6. Multiple exception types
```python
raw_record = {"order_id": "501", "amount": "bad"}

try:
    order_id = int(raw_record["order_id"])
    amount = float(raw_record["amount"])
except KeyError as error:
    print("Required field missing:", error)
except ValueError as error:
    print("Invalid field value:", error)
```

Different errors may require different actions: missing schema field may stop the batch; an invalid individual record may be quarantined if policy permits.

## 7. else and finally
```python
try:
    amount = float("120.50")
except ValueError:
    print("Conversion failed")
else:
    print("Conversion succeeded:", amount)
finally:
    print("Conversion step finished")
```

- `else` runs only when no exception occurred.
- `finally` runs whether the operation succeeds or fails.
Use `finally` for cleanup, not for pretending a failed operation succeeded.

## 8. Raise an exception
```python
def validate_quantity(quantity):
    if quantity <= 0:
        raise ValueError("Quantity must be greater than zero")

    return quantity

print(validate_quantity(2))
```

Raise errors when a function cannot fulfill its contract. Include enough context to diagnose the issue without exposing sensitive values.

## 9. Custom exceptions
```python
class InvalidOrderError(Exception):
    """Raised when an order violates required business rules."""

try:
    raise InvalidOrderError("Order 501 has an invalid quantity")
except InvalidOrderError as error:
    print(error)
```

Custom exception classes are useful when callers need to distinguish domain failures from ordinary Python errors. Do not create a large hierarchy without a real need.

## 10. Logging vs print
Print is useful for quick exploration. Logging provides levels, timestamps, logger names and configurable destinations.

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(name)s %(message)s"
)

logger = logging.getLogger("order_pipeline")

logger.debug("Detailed diagnostic")
logger.info("Batch processing started")
logger.warning("Optional field is missing")
logger.error("Record validation failed")
```

Levels:
- DEBUG: detailed diagnostic information.
- INFO: normal milestones.
- WARNING: unexpected but recoverable condition.
- ERROR: operation failed.
- CRITICAL: severe failure requiring immediate attention.

In larger applications, configure logging once at the application entry point. Avoid repeatedly calling basicConfig in imported modules.

## 11. Add useful context
```python
run_id = "run_2026_001"
file_name = "orders.csv"
rows_read = 1200

logger.info(
    "File processed run_id=%s file=%s rows_read=%s",
    run_id,
    file_name,
    rows_read
)
```

Useful fields include run ID, source, file/batch ID, stage, row counts, duration and error category. Do not log passwords, tokens, payment details or unnecessary personal information.

## 12. Debugging workflow
1. Reproduce the failure.
2. Read the complete traceback.
3. Inspect the input and types.
4. Check assumptions and boundary conditions.
5. Add temporary or structured diagnostics.
6. Fix the root cause.
7. Rerun the failing case.
8. Run regression tests.
9. Remove unsafe temporary prints and secrets.

A debugger can pause execution, inspect variables and step through lines. Breakpoints are useful when a failure depends on state.

## 13. Mini-project: Resilient File Processor
Create a sample CSV:

```python
from pathlib import Path

Path("orders.csv").write_text(
    "order_id,amount\n501,1200.50\n502,invalid\n503,800\n",
    encoding="utf-8"
)
```

Read and validate rows:

```python
import csv
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger("file_processor")

valid_rows = []
rejected_rows = []

try:
    with open("orders.csv", "r", newline="", encoding="utf-8") as file:
        reader = csv.DictReader(file)

        for line_number, row in enumerate(reader, start=2):
            try:
                order_id = int(row["order_id"])
                amount = float(row["amount"])

                if amount < 0:
                    raise ValueError("Amount cannot be negative")

                valid_rows.append({
                    "order_id": order_id,
                    "amount": amount
                })

            except (ValueError, KeyError) as error:
                rejected_rows.append({
                    "line_number": line_number,
                    "reason": str(error)
                })
                logger.warning(
                    "Rejected row line=%s reason=%s",
                    line_number,
                    error
                )

except FileNotFoundError:
    logger.exception("Input file was not found")
    raise

logger.info(
    "Processing complete valid=%s rejected=%s",
    len(valid_rows),
    len(rejected_rows)
)

print(valid_rows)
print(rejected_rows)
```

Expected behavior: two valid records and one rejected row. In a production pipeline, define whether a rejection threshold permits the batch to continue and where rejected data is stored.

## 14. Common mistakes
| Mistake | Better approach |
|---|---|
| Catching every error and continuing | Catch expected errors; fail visibly on unexpected ones |
| Empty except block | Log and handle or re-raise |
| Logging secrets | Mask or omit sensitive data |
| Using print everywhere | Use configured logging for applications |
| Retrying invalid input | Fix or quarantine permanent data errors |
| Losing traceback | Use logger.exception inside exception handling |
| Reporting success after partial failure | Track rejected counts and batch status |

## 15. Day 10 revision
- Exceptions represent runtime failures.
- Catch specific expected exceptions.
- Use `raise` to enforce function contracts.
- Logging records operational evidence.
- Tracebacks help locate root causes.
- Distinguish recoverable record errors from pipeline-level failures.
