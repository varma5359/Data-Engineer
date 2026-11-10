# Day 17: Unit Testing and Reusable Pipeline Functions

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** Transformation Test Suite  
**Learning time:** 2–3 hours

## 1. Learning outcomes
- Explain unit tests and why pipelines need them.
- Write tests using Python's unittest framework.
- Test normal, boundary and invalid inputs.
- Understand assertions, test isolation and fixtures.
- Test transformations separately from external systems.
- Recognize integration and end-to-end testing.

## 2. Why test pipeline code?
A transformation may work for one sample but fail when it receives nulls, duplicates, empty input or a new status. Tests protect business rules as code changes.

Testing does not prove that every production input is correct. It provides repeatable evidence that defined behavior works.

## 3. What is a unit test?
A unit test checks a small unit of code, usually a function, in isolation.

Example function:

```python
def calculate_line_amount(quantity, unit_price):
    if quantity <= 0:
        raise ValueError("quantity must be positive")

    if unit_price < 0:
        raise ValueError("unit_price cannot be negative")

    return quantity * unit_price
```

## 4. Assertions
An assertion compares actual behavior with expected behavior.

```python
assert calculate_line_amount(2, 100) == 200
```

If the condition is false, Python raises AssertionError. Assertions are useful for tests, not for enforcing production validation because optimized execution can disable them.

## 5. unittest test file
Project structure:

```text
transformation_test_suite/
    transformations.py
    test_transformations.py
```

In `transformations.py`:

```python
def normalize_status(status):
    return status.strip().upper()

def calculate_line_amount(quantity, unit_price):
    if quantity <= 0:
        raise ValueError("quantity must be positive")
    if unit_price < 0:
        raise ValueError("unit_price cannot be negative")
    return quantity * unit_price
```

In `test_transformations.py`:

```python
import unittest
from transformations import normalize_status, calculate_line_amount

class TestTransformations(unittest.TestCase):

    def test_normalize_status(self):
        self.assertEqual(normalize_status(" paid "), "PAID")

    def test_calculate_line_amount(self):
        self.assertEqual(calculate_line_amount(2, 100), 200)

    def test_quantity_must_be_positive(self):
        with self.assertRaises(ValueError):
            calculate_line_amount(0, 100)

if __name__ == "__main__":
    unittest.main()
```

Run:

```bash
python -m unittest -v
```

Expected: three tests pass. Exact output formatting depends on Python version.

## 6. Common assertions

| Assertion | Purpose |
|---|---|
| assertEqual | Expected values equal |
| assertNotEqual | Values differ |
| assertTrue | Condition is true |
| assertFalse | Condition is false |
| assertIsNone | Value is None |
| assertIn | Item is contained |
| assertRaises | Code raises expected exception |

## 7. Test categories for a transformation
- Happy path: valid normal input.
- Boundary: minimum/maximum allowed values.
- Invalid: wrong type or prohibited value.
- Empty: no records.
- Null: missing required/optional field.
- Duplicate: repeated key.
- Regression: a previously fixed bug remains fixed.

## 8. Test a filter
```python
def get_paid_orders(orders):
    return [order for order in orders if order["status"] == "PAID"]
```

```python
class TestPaidOrders(unittest.TestCase):

    def test_only_paid_orders_returned(self):
        orders = [
            {"order_id": 1, "status": "PAID"},
            {"order_id": 2, "status": "PENDING"}
        ]

        result = get_paid_orders(orders)

        self.assertEqual(len(result), 1)
        self.assertEqual(result[0]["order_id"], 1)
```

Keep test inputs small and explicit so failures are easy to understand.

## 9. Test data isolation
Tests should not depend on production data or shared mutable state. Create controlled inputs for each test. A test should pass regardless of the order in which tests run.

## 10. Mocking external dependencies
A unit test should not depend on a live API or production database. Replace external calls with controlled test doubles/mocks when testing the local decision logic. Test real integrations separately in a controlled environment.

## 11. Test pipeline behavior
For a pipeline, test:
- Transformation outputs.
- Validation decisions.
- Rejected-record reasons.
- Duplicate handling.
- Retry policy decisions.
- Checkpoint advancement rules.
- Failure propagation.

For database writes and API extraction, integration tests should verify the actual interaction against test services or isolated databases.

## 12. Mini-project: Transformation Test Suite
Create transformations for:
1. Status normalization.
2. Line amount calculation.
3. Required-field validation.
4. Paid-order filtering.
5. Duplicate-key detection.

Write tests for normal, boundary and invalid inputs. Run tests from the project root. Fix the code rather than weakening expected results to make tests pass.

## 13. Test quality
Good tests are:
- Deterministic.
- Independent.
- Readable.
- Fast at unit level.
- Focused on behavior.
- Useful when they fail.

Avoid testing implementation details that may change while behavior remains correct.

## 14. Common mistakes
| Mistake | Better practice |
|---|---|
| Testing only happy path | Add boundary and invalid cases |
| Calling live production API | Mock or use controlled test service |
| Tests share mutable data | Create isolated fixtures |
| Assertions too vague | Assert exact meaningful outcomes |
| Ignoring failing tests | Investigate root cause |
| Test only helper function | Add integration tests for pipeline boundaries |

## 15. Day 17 revision
- Unit tests protect small pieces of behavior.
- Assertions compare actual and expected results.
- Test normal, boundary and invalid inputs.
- Keep tests isolated and deterministic.
- Unit, integration and end-to-end tests serve different purposes.
