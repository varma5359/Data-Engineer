# Day 17: Testing Data Processing Code

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** Transformation Test Suite  
**Learning time:** 2–3 hours

## 1. Learning outcomes

- Explain why data transformations need tests.
- Distinguish unit, integration and end-to-end tests.
- Write tests using Python's built-in `unittest` framework.
- Use assertions to compare actual and expected behavior.
- Test normal, boundary and invalid inputs.
- Run tests from the terminal and interpret the result.
- Organize a small reusable transformation project.

## 2. Why test pipeline code?

A transformation may work for one sample and fail when it receives:
- Empty input.
- Missing fields.
- Duplicate records.
- Unexpected status values.
- Negative quantities.
- A new business rule.

Tests provide repeatable evidence that defined behavior works. They do not prove every possible input is correct, but they catch regressions before code is deployed.

## 3. What is a unit test?

A unit test checks one small unit of code, usually a function, in isolation.

For example, if a function calculates order value, test the calculation without connecting to a database or API.

## 4. Assertions

An assertion compares actual behavior with an expected result.

```python
actual = 2 + 3
expected = 5

assert actual == expected
print("Assertion passed")
```

Expected output:

```text
Assertion passed
```

If the condition is false, Python raises `AssertionError`. Assertions are useful for tests and development checks, but production input validation should use explicit validation and exceptions.

## 5. Create a project structure

Create this folder:

```text
transformation_test_suite/
    transformations.py
    test_transformations.py
```

We will keep transformation code separate from test code.

## 6. Write the transformation module

In `transformations.py`:

```python
def calculate_line_amount(quantity, unit_price):
    if quantity <= 0:
        raise ValueError("quantity must be positive")

    if unit_price < 0:
        raise ValueError("unit_price cannot be negative")

    return quantity * unit_price


def normalize_status(status):
    return status.strip().upper()


def remove_duplicate_order_ids(orders):
    unique_orders = []
    seen_order_ids = set()

    for order in orders:
        order_id = order["order_id"]

        if order_id not in seen_order_ids:
            unique_orders.append(order)
            seen_order_ids.add(order_id)

    return unique_orders
```

Each function has a focused responsibility. The duplicate-removal rule keeps the first occurrence; a real pipeline must define which record wins when duplicates differ.

## 7. Test one function manually first

Run this from the project folder:

```bash
python -c "from transformations import calculate_line_amount; print(calculate_line_amount(2, 100))"
```

Expected output:

```text
200
```

This confirms the module can be imported and the function can run. It is a manual smoke check, not yet a repeatable test suite.

## 8. Create the first unittest

In `test_transformations.py`:

```python
import unittest
from transformations import calculate_line_amount


class TestCalculateLineAmount(unittest.TestCase):

    def test_calculates_amount(self):
        result = calculate_line_amount(2, 100)
        self.assertEqual(result, 200)


if __name__ == "__main__":
    unittest.main()
```

Run:

```bash
python -m unittest -v
```

Expected output:

```text
test_calculates_amount (test_transformations.TestCalculateLineAmount) ... ok

----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
```

The execution time may differ. The important result is that the test is marked `ok` and the test run ends with `OK`.

## 9. Understand the test structure

- `import unittest` loads Python's standard testing framework.
- `TestCalculateLineAmount` groups related tests.
- Test classes inherit from `unittest.TestCase`.
- Test method names begin with `test_`.
- `self.assertEqual(actual, expected)` checks equality.
- `unittest.main()` starts the test runner when the file is run directly.

## 10. Test boundary and invalid inputs

Add these methods to the same test class:

```python
def test_rejects_zero_quantity(self):
    with self.assertRaises(ValueError):
        calculate_line_amount(0, 100)


def test_rejects_negative_price(self):
    with self.assertRaises(ValueError):
        calculate_line_amount(2, -10)
```

Run:

```bash
python -m unittest -v
```

Expected summary:

```text
Ran 3 tests

OK
```

The two new tests verify that invalid values raise the expected exception.

## 11. Common assertion methods

| Assertion | What it checks |
|---|---|
| `assertEqual(a, b)` | Values are equal |
| `assertNotEqual(a, b)` | Values differ |
| `assertTrue(value)` | Value is true |
| `assertFalse(value)` | Value is false |
| `assertIsNone(value)` | Value is None |
| `assertIn(item, collection)` | Item belongs to collection |
| `assertRaises(Error)` | Code raises the specified exception |

## 12. Test status normalization

Add to `test_transformations.py`:

```python
from transformations import normalize_status


class TestNormalizeStatus(unittest.TestCase):

    def test_removes_spaces_and_changes_case(self):
        result = normalize_status(" shipped ")
        self.assertEqual(result, "SHIPPED")
```

Expected result after running the suite: the new test is marked `ok`.

This test checks two parts of the rule: leading/trailing spaces are removed and letters are converted to uppercase.

## 13. Test duplicate removal

Add:

```python
from transformations import remove_duplicate_order_ids


class TestRemoveDuplicateOrderIds(unittest.TestCase):

    def test_keeps_first_order_id(self):
        orders = [
            {"order_id": 1, "amount": 100},
            {"order_id": 1, "amount": 120},
            {"order_id": 2, "amount": 200}
        ]

        result = remove_duplicate_order_ids(orders)

        self.assertEqual(len(result), 2)
        self.assertEqual(result[0]["amount"], 100)
```

The expected result contains order IDs 1 and 2. The first version of order ID 1 is retained. This is only one possible deduplication policy.

## 14. Full test file

For a clean final project, replace `test_transformations.py` with:

```python
import unittest

from transformations import (
    calculate_line_amount,
    normalize_status,
    remove_duplicate_order_ids
)


class TestCalculateLineAmount(unittest.TestCase):

    def test_calculates_amount(self):
        self.assertEqual(calculate_line_amount(2, 100), 200)

    def test_rejects_zero_quantity(self):
        with self.assertRaises(ValueError):
            calculate_line_amount(0, 100)

    def test_rejects_negative_price(self):
        with self.assertRaises(ValueError):
            calculate_line_amount(2, -10)


class TestNormalizeStatus(unittest.TestCase):

    def test_removes_spaces_and_changes_case(self):
        self.assertEqual(normalize_status(" shipped "), "SHIPPED")


class TestRemoveDuplicateOrderIds(unittest.TestCase):

    def test_keeps_first_order_id(self):
        orders = [
            {"order_id": 1, "amount": 100},
            {"order_id": 1, "amount": 120},
            {"order_id": 2, "amount": 200}
        ]

        result = remove_duplicate_order_ids(orders)

        self.assertEqual(len(result), 2)
        self.assertEqual(result[0]["amount"], 100)


if __name__ == "__main__":
    unittest.main()
```

Run:

```bash
python -m unittest -v
```

Expected output:

```text
test_calculates_amount (test_transformations.TestCalculateLineAmount) ... ok
test_rejects_negative_price (test_transformations.TestCalculateLineAmount) ... ok
test_rejects_zero_quantity (test_transformations.TestCalculateLineAmount) ... ok
test_removes_spaces_and_changes_case (test_transformations.TestNormalizeStatus) ... ok
test_keeps_first_order_id (test_transformations.TestRemoveDuplicateOrderIds) ... ok

----------------------------------------------------------------------
Ran 5 tests in 0.001s

OK
```

Test discovery order and timing can vary by Python version. Five passing tests and `OK` are the important outcomes.

## 15. Understand a failing test

Temporarily change the expected amount from 200 to 201:

```python
self.assertEqual(calculate_line_amount(2, 100), 201)
```

Run:

```bash
python -m unittest -v
```

Expected behavior: the test is marked `FAIL`, and the failure message shows the expected and actual values.

Restore the correct expectation:

```python
self.assertEqual(calculate_line_amount(2, 100), 200)
```

A failing test is useful evidence. Do not simply delete a test because it found a defect.

## 16. Test edge cases systematically

For every transformation, ask:

- What is the normal input?
- What is the smallest allowed input?
- What happens with an empty collection?
- What happens when a field is missing?
- What happens with invalid types?
- What happens with duplicates?
- What is the expected error behavior?

A test case should have a clear input, action and expected result.

## 17. Unit, integration and end-to-end tests

| Test type | Scope | Example |
|---|---|---|
| Unit | One small function | Calculate order amount |
| Integration | Multiple components working together | Read CSV, transform rows and write SQLite |
| End-to-end | Whole workflow | Trigger pipeline and verify destination data |

A unit test should be fast and isolated. An integration test intentionally checks that components connect correctly. Do not call a unit test an end-to-end test merely because it uses several lines of code.

## 18. Mini-project: Transformation Test Suite

**Scenario:** A small retail pipeline uses reusable functions to calculate line amounts, standardize statuses and remove duplicate order IDs. We will build the transformation module and protect its rules with tests.

### Step 1 — Create the folder

```bash
mkdir transformation_test_suite
cd transformation_test_suite
```

Create the two Python files shown earlier.

### Step 2 — Run a direct smoke test

```bash
python -c "from transformations import normalize_status; print(normalize_status(' paid '))"
```

Expected output:

```text
PAID
```

### Step 3 — Run the full unit test suite

```bash
python -m unittest -v
```

Expected result: all five tests pass and the runner reports `OK`.

### Step 4 — Add one new business rule

Suppose order quantity cannot be zero. The existing calculation function already rejects zero. Add a test for negative quantity:

```python
def test_rejects_negative_quantity(self):
    with self.assertRaises(ValueError):
        calculate_line_amount(-1, 100)
```

Run the test suite again. The new test should pass because the function rejects all quantities less than or equal to zero.

### Step 5 — Check the project contents

```bash
dir
```

Expected files:

```text
transformations.py
test_transformations.py
```

Your directory may also contain a `__pycache__` folder created by Python. It is generated and should not be committed.

## 19. Common mistakes

| Mistake | Cause | Fix |
|---|---|---|
| No tests discovered | Filename or method does not match discovery pattern | Use `test_*.py` and `test_...` |
| Import error | Running from the wrong directory | Run command from project folder |
| Test passes but bug remains | Expected behavior was not tested | Add boundary and invalid cases |
| Tests depend on each other | Shared mutable state | Keep tests independent |
| External service makes test flaky | Unit test relies on network/database | Mock or isolate external dependency; use integration tests separately |
| Assertion is too weak | Only checks that code ran | Assert exact values and important conditions |

## 20. Practical exercises

1. Add a test for quantity equal to one.
2. Add a test for an empty status string and decide the expected behavior.
3. Add a test for an empty order list.
4. Test that duplicate removal keeps the intended record.
5. Introduce a deliberate bug and read the failure report.
6. Explain why a database connection test is an integration test.

## 21. Interview revision

- A unit test checks a small piece of logic in isolation.
- Assertions compare actual behavior with expected behavior.
- `unittest` is included in Python's standard library.
- Test names begin with `test_` for automatic discovery.
- Boundary and invalid inputs are as important as normal examples.
- Integration tests check components working together.
- End-to-end tests validate a complete workflow.
- A passing test suite proves the tested cases passed, not that all possible inputs are safe.

**Day 17 checkpoint:** You should be able to create a transformation module, write tests using unittest, run them from CMD, interpret failures and explain the difference between unit and integration testing.


---

# Beginner Teaching Extension — Day 17

## A. Why tests are needed

A transformation that works for one sample may fail for empty input, boundary values, invalid data or a changed business rule. A test runs a known input and compares actual behavior with an expected result.

## B. Start with a tiny function

Create transform.py:

```python
def calculate_amount(quantity, unit_price):
    return quantity * unit_price
```

This function has one responsibility and is easy to test without a file, database or API.

## C. Write the first unit test

Create test_transform.py:

```python
import unittest
from transform import calculate_amount

class TestCalculateAmount(unittest.TestCase):
    def test_normal_order(self):
        self.assertEqual(calculate_amount(2, 100), 200)

if __name__ == "__main__":
    unittest.main()
```

Run from the folder containing both files:

```bash
python -m unittest -v
```

Expected result includes one test marked ok and a summary showing Ran 1 test and OK. The exact elapsed time varies.

- unittest is Python's built-in testing framework.
- TestCase groups test methods.
- Test methods begin with test_.
- assertEqual compares actual and expected values.

## D. Add boundary and invalid-input tests

```python
import unittest
from transform import calculate_amount

class TestCalculateAmount(unittest.TestCase):
    def test_normal_order(self):
        self.assertEqual(calculate_amount(2, 100), 200)

    def test_zero_quantity(self):
        self.assertEqual(calculate_amount(0, 100), 0)

    def test_decimal_price(self):
        self.assertEqual(calculate_amount(2, 12.5), 25.0)
```

These tests define the current function behavior. If the business rule says zero quantity is invalid, the function and test should be changed to enforce that rule rather than asserting zero is acceptable.

## E. Understand test discovery

Python's unittest discovery convention finds files named test*.py. Run all tests with python -m unittest. Keep tests separate from production code and use predictable names.

## F. Unit, integration and end-to-end tests

- Unit test: checks one small function or component in isolation.
- Integration test: checks whether components work together, such as Python code writing to a temporary SQLite database.
- End-to-end test: checks the complete workflow from source input to expected destination output.

A passing unit test does not prove that a real API, database permissions or deployment configuration work.

## G. Test a data transformation systematically

For each rule, include a normal case, boundary case, empty input where meaningful, invalid input and duplicate case if relevant. Keep expected values small enough to calculate manually. A test should fail when behavior is wrong; do not write an assertion that merely repeats the implementation without checking the business expectation.

## H. Mini-project: Transformation Test Suite

Create a small transformation module and tests for amount calculation, status normalization and positive-quantity validation. Include normal and boundary cases. Run the suite from CMD and record the test count and result. Add one intentionally incorrect expected value and observe the test fail; then correct it. This demonstrates that tests provide evidence rather than just displaying a green status.

## I. Common mistakes

- Naming test functions without the test_ prefix.
- Running tests from the wrong project folder.
- Testing only the happy path.
- Depending on a live database for a unit test.
- Writing tests that merely duplicate the code's assumptions.
- Ignoring failing tests or weakening assertions to make them pass.

## J. Readiness check

The student can write a unit test, run unittest discovery, interpret pass/fail output, design boundary cases and explain the difference between unit and integration tests.
