# Day 10 — Exception Handling, Logging and Debugging
**Phase 2: Python for Data Engineering**

## 1. Exceptions
An exception signals an error during execution: missing file, invalid conversion, network timeout, or missing key.

```python
amount_text = "500"
try:
    amount = float(amount_text)
    print(amount)
except ValueError:
    print("Invalid amount")
```
Output: `500.0`

Catch specific exceptions. Broad `except Exception` is appropriate only at a deliberate boundary where you log context and re-raise/mark failure.

## 2. else and finally
```python
try:
    count = int("25")
except ValueError:
    print("Bad count")
else:
    print("Count:", count)
finally:
    print("Conversion step completed")
```
Output: Count: 25; Conversion step completed.

## 3. Logging
Logging provides timestamped severity and context. Levels: DEBUG, INFO, WARNING, ERROR, CRITICAL.
```python
import logging
logging.basicConfig(level=logging.INFO, format="%(levelname)s:%(message)s")
logging.info("Starting orders ingestion")
logging.warning("3 records rejected")
```
Output:
`INFO:Starting orders ingestion`
`WARNING:3 records rejected`

Log batch ID, source, counts, duration, and error category; never log secrets or unnecessary personal data.

## 4. Debugging approach
1. Reproduce with a small input.
2. Read traceback from bottom upward to locate exception and line.
3. Inspect types/values at boundary.
4. Check assumptions (path, schema, permissions, network).
5. Fix root cause and add a test.
6. Confirm valid and invalid cases.

## 5. Mini-project — Resilient File Processor
For each input file, log start/end, row count, and failures. A malformed file should be reported and retained; do not silently skip errors.

## Mistakes
- Catching all errors and continuing with corrupt output.
- Using print-only debugging in production.
- Logging credentials/tokens.
- Retrying permanent validation errors.
- Losing traceback/context.
- Marking job success despite rejected data beyond agreed threshold.

## Revision
Exceptions control failure paths; logging records evidence; debugging isolates root cause. Error handling must preserve data correctness.