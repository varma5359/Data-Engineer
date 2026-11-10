# Day 17 — Unit Testing and Reusable Pipeline Functions
**Phase 2: Python for Data Engineering**

## Why test transformations?
Tests show expected behavior and catch regressions. Unit tests isolate functions; integration tests verify components together.

## Setup
`python -m pip install pytest`

Create `transformations.py`:
```python
def normalize_status(status):
    """Trim and lowercase a status."""
    return status.strip().lower()
```
Create `test_transformations.py`:
```python
from transformations import normalize_status

def test_normalize_status():
    assert normalize_status(" PAID ") == "paid"
```
Run `python -m pytest -q`; expected: `1 passed`.

## Boundary tests
```python
def test_empty_status():
    assert normalize_status(" ") == ""

def test_uppercase_status():
    assert normalize_status("REFUNDED") == "refunded"
```
Decide expected blank behavior from requirements; test normal, boundary, invalid, and unexpected inputs.

## Arrange–Act–Assert
1. Arrange input.
2. Act by calling the function.
3. Assert expected output.

## Pipeline test levels
- Unit: transformation/validation function.
- Integration: file/API/database interactions.
- Contract: schema and source expectations.
- End-to-end: source to destination.
- Regression: previously fixed defect stays fixed.

Use deterministic fixtures; avoid uncontrolled network and production data.

## Mini-project — Transformation Test Suite
Test normalization, conversion, duplicate handling, and invalid rows with expected rejection reasons.

## Mistakes
Happy-path-only tests; vague assertions; production credentials; nondeterministic inputs; mocking everything without integration tests.

## Revision
Tests document behavior, detect regressions, and make refactoring safer. Unit tests alone do not prove the whole pipeline.