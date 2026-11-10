# Day 17 Interview Preparation: Testing Data Processing Code

**Mini-project:** Transformation Test Suite

## 1. Core questions

1. **What is a unit test?**  
   A test that verifies a small piece of code, usually a function, in isolation.

2. **Why test transformations?**  
   To protect business rules and detect regressions when code changes.

3. **What is unittest?**  
   Python's built-in unit testing framework.

4. **What is an assertion?**  
   A check that actual behavior matches an expected condition.

5. **What does assertEqual do?**  
   It verifies that two values are equal.

6. **What does assertRaises do?**  
   It verifies that a block of code raises the expected exception.

7. **Why use test_ in method names?**  
   unittest discovery uses that naming pattern to identify test methods.

8. **What is a boundary test?**  
   A test for values at or near the allowed limits.

9. **What is test isolation?**  
   Each test should be independent and should not rely on another test's execution order or leftover state.

10. **Unit vs integration test?**  
    Unit tests verify a small component; integration tests verify that multiple components work together.

11. **What is an end-to-end test?**  
    It validates a complete workflow from entry point to expected output.

12. **Does a passing test suite prove a program is bug-free?**  
    No. It proves only that the tested cases passed.

## 2. Difference questions

| Concept | Difference |
|---|---|
| Unit / integration | One isolated unit / connected components |
| Integration / end-to-end | Component interaction / complete workflow |
| Assertion / exception | Checks a condition / signals an error |
| Normal / boundary test | Typical input / edge of allowed range |
| Test failure / error | Assertion did not match / test execution itself raised an unexpected exception |
| Mock / real dependency | Simulated collaborator / actual external service |

## 3. Practical coding questions

**Q1. Test a calculation.**

```python
import unittest

class TestAmount(unittest.TestCase):
    def test_amount(self):
        self.assertEqual(2 * 100, 200)
```

Run using `python -m unittest -v`.

**Q2. Test an invalid input.**

```python
with self.assertRaises(ValueError):
    raise ValueError("invalid quantity")
```

The test passes when the expected exception is raised.

**Q3. What command runs tests from the project directory?**  
   `python -m unittest -v`.

## 4. Scenario-based questions

1. **A transformation works on the sample but fails on empty input. What is missing?**  
   An empty-input test and an explicit expected behavior for empty collections.

2. **A unit test needs a real production database. Is the test well isolated?**  
   No. Unit tests should isolate external dependencies. Use a controlled integration test for database connectivity.

3. **A test passes on your laptop but fails in CI. What do you check?**  
   Python version, dependency versions, working directory, environment variables, file paths, test isolation and data assumptions.

4. **A duplicate-removal rule changes from keeping first to keeping latest. What should happen?**  
   Update the business logic and tests to encode the new rule. The test should make the expected winner explicit.

5. **A test suite takes 20 minutes because each test calls an API. What can improve it?**  
   Isolate network behavior in unit tests using mocks or fixtures; keep a smaller number of controlled integration tests.

## 5. Follow-up questions

- What is test discovery?
- Why should test data be deterministic?
- How do you test a function that raises ValueError?
- Why avoid tests that depend on order?
- What should be tested for a data transformation?
- What is regression testing?
- How do you decide which behavior deserves a test?

## 6. Strong revision points

- Test normal, boundary, missing and invalid inputs.
- Keep unit tests fast and deterministic.
- Test business behavior, not implementation details only.
- Keep integration tests separate from isolated unit tests.
- A test suite is executable documentation of expected behavior.
