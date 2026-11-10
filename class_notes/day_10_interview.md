# Day 10 Interview Preparation: Exception Handling and Debugging

**Mini-project:** Resilient File Processor

## 1. Core questions

1. **What is an exception?**  
   A runtime event that interrupts normal program execution.

2. **Syntax error vs exception?**  
   A syntax error prevents code from being parsed; an exception occurs while valid code runs.

3. **Why use try/except?**  
   To handle expected failures and take a controlled action.

4. **What is a traceback?**  
   A report showing the call path and the location where an exception occurred.

5. **Why catch specific exceptions?**  
   Different failures need different responses. Broad handlers can hide programming defects.

6. **What does else do in try/except?**  
   It runs when the try block completes without an exception.

7. **What does finally do?**  
   It runs whether an exception occurred or not and is useful for cleanup.

8. **What does raise do?**  
   It signals an error when the current operation cannot safely continue.

9. **What is logging?**  
   Recording application events with levels and useful context.

10. **Name common logging levels.**  
    DEBUG, INFO, WARNING, ERROR and CRITICAL.

11. **What is logging.exception()?**  
    It records an error message and the current exception traceback from inside an exception handler.

12. **Why avoid empty except blocks?**  
    They hide the real failure and make diagnosis difficult.

13. **What is a rejected-record pattern?**  
    A design where invalid records are separated with their identifiers and rejection reasons.

14. **What is the difference between batch failure and row rejection?**  
    Batch failure stops the batch; row rejection quarantines an invalid record while allowing other rows to continue.

## 2. Difference questions

| Concept | Difference |
|---|---|
| Syntax error / runtime exception | Parse-time issue / runtime failure |
| `except` / `finally` | Handles a matching error / executes during cleanup |
| `print` / logging | Simple display / configurable operational event record |
| WARNING / ERROR | Unexpected condition / operation failed |
| Retry / reject | Repeat a transient operation / isolate invalid input |
| FileNotFoundError / PermissionError | Missing path / access denied |

## 3. Practical coding questions

**Q1. Handle invalid integer input.**

```python
try:
    quantity = int("many")
except ValueError:
    print("Invalid quantity")
```

Output: `Invalid quantity`.

**Q2. Raise an error for negative amount.**

```python
amount = -50
if amount < 0:
    raise ValueError("amount cannot be negative")
```

Expected: `ValueError` with the stated message.

**Q3. Log pipeline progress.**

```python
import logging
logging.basicConfig(level=logging.INFO, format="%(levelname)s:%(message)s")
logging.info("Loaded 25 records")
```

Output: `INFO:Loaded 25 records`.

**Q4. Explain a file context manager.**  
`with open(...)` closes the file automatically when the block ends, including when an exception occurs.

## 4. Scenario-based questions

1. **A source file is missing. Should the pipeline continue?**  
   It depends on whether the source is optional. Log the missing path and either stop the batch or record an explicit skipped-source status.

2. **One row has a non-numeric quantity among 10,000 rows. What do you do?**  
   Apply a documented row-level policy: reject or quarantine that row, retain its identifier and reason, and continue only if business rules allow.

3. **An API request times out. What is the correct response?**  
   Record the timeout and apply bounded retries with backoff if the operation is safe to retry. Do not retry forever.

4. **A database authentication error occurs. Should it be retried repeatedly?**  
   Usually not. Invalid credentials are not transient; stop and alert the owner rather than creating repeated failed attempts.

5. **The pipeline reports success despite a failed write. What is wrong?**  
   The error was swallowed or success was logged too early. Only report success after the required write and verification complete.

## 5. Troubleshooting checklist

- Read the final exception type and message.
- Locate the failing line.
- Inspect input values and types.
- Confirm the file path and permissions.
- Reproduce with the smallest input.
- Catch only expected exceptions.
- Log context without secrets.
- Decide whether to retry, reject or fail the batch.

## 6. Follow-up questions

- When should an exception be re-raised?
- Why can retries make an incident worse?
- What is exponential backoff?
- Why should passwords never be logged?
- What information belongs in a rejected-record file?
- How would you distinguish a data error from an infrastructure error?

## 7. Strong revision points

- Error handling should make failure visible, not hide it.
- Catch specific exception types.
- Use context managers for resource cleanup.
- Log source name, batch ID and counts where appropriate.
- Never log credentials or sensitive data.
- Define the batch-failure and record-rejection policy before production.
