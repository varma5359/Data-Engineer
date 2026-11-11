# Day 67 Project Workbook — Data Pipeline CI Workflow

## Goal
Create automated tests for a Python transformation and run them in GitHub Actions when code is pushed or a pull request is opened.

## Step 1 — Create project structure

~~~text
ci_pipeline/
    src/
        transform.py
    tests/
        test_transform.py
    .github/
        workflows/
            python-ci.yml
    requirements.txt
~~~

## Step 2 — Implement transformation
Create calculate_total and filter_valid_orders using the class example. Document the rule for negative amounts. If refunds use negative values, revise the rule with an explicit business contract.

## Step 3 — Add unit tests
Test normal totals, empty input, valid/invalid orders, boundary values and expected counts. Run locally and record results.

## Step 4 — Add GitHub Actions
Create a workflow triggered by push and pull request. It should check out code, set up Python, install dependencies and run tests.

## Step 5 — Introduce a controlled defect
Temporarily change the expected result or transformation behavior. Confirm the test fails. Restore correct code and confirm tests pass.

## Step 6 — Add a data-quality test
Add a test for required order_id or unique keys at the declared grain.

## Step 7 — Review release safety
Document code review, test gates, staging validation, smoke tests, production approval, secrets access and rollback/data recovery.

## Step 8 — Submission
Submit source code, tests, workflow YAML, local test output, Actions run evidence, controlled failure evidence, README and validation status.

## Completion criteria
A code change triggers automated checks, a defect makes the workflow fail, the corrected change passes and the student can explain what CI proves and what still requires integration/staging validation.
