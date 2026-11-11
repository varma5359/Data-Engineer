# Day 67: CI/CD and Automated Pipeline Testing

**Phase:** 7 — Orchestration, Quality and Monitoring | **Time:** 2–3 hours  
**Audience:** Complete beginners and junior Data Engineers | **Tools:** Git, GitHub Actions, Python unittest and YAML  
**Topic boundary:** Class notes only. Project workbook and interview guide are separate.

> A pipeline may work on one laptop but fail after a code change is merged. Production teams need repeatable checks before release. Today you will learn CI/CD, automated tests, GitHub Actions, release gates and safe deployment practices for data pipelines.

## Learning outcomes
- Explain CI, Continuous Delivery and Continuous Deployment.
- Distinguish unit, integration, end-to-end and regression tests.
- Create and run a Python unit test.
- Understand GitHub Actions workflow, job, runner and step.
- Trigger CI on pushes and pull requests.
- Read job results and logs.
- Protect CI secrets.
- Design release gates and rollback plans.
- Explain what a green CI run does and does not prove.

## 1. Business scenario

A developer changes order transformation logic. The code runs locally, but accidentally changes the revenue total. Without automated tests, the defect may reach production.

CI can automatically install dependencies, check syntax, run unit tests and validate configuration before merge.

## 2. What is CI?

CI means Continuous Integration. Developers frequently integrate code into a shared repository and run automated checks so defects are detected early.

Typical steps:
1. get latest code;
2. set up runtime;
3. install dependencies;
4. run lint/syntax checks;
5. run unit tests;
6. run selected integration tests;
7. publish the result.

## 3. What is CD?

CD can mean Continuous Delivery or Continuous Deployment.

- Continuous Delivery keeps changes in a releasable state, but production release may require human approval.
- Continuous Deployment automatically releases changes that pass required controls to production.

These are related but not identical. State which process the team uses.

## 4. Why CI/CD matters for data pipelines

Changes may affect extraction filters, schema mappings, business rules, deduplication, SQL, Airflow dependencies, Docker dependencies and quality thresholds. Automated checks reduce the chance that a small change silently changes business results.

## 5. Types of tests

| Test | What it checks | Example |
|---|---|---|
| Unit | One small function | Calculate order total |
| Integration | Components work together | Python connects to test DB |
| End-to-end | Whole workflow | Source file to curated output |
| Data quality | Dataset meets rules | No duplicate business key |
| Regression | Existing behavior remains correct | Revenue total unchanged |
| Configuration | Deployment files are valid | YAML/DAG import check |

Tests should be layered. Unit tests are fast; integration and end-to-end tests are more expensive.

## 6. Create a Python project

~~~text
ci_pipeline/
    src/
        transform.py
    tests/
        test_transform.py
    requirements.txt
~~~

Create src/transform.py:

~~~python
def calculate_total(amounts):
    return sum(amounts)


def filter_valid_orders(orders):
    valid_orders = []

    for order in orders:
        if order["amount"] >= 0:
            valid_orders.append(order)

    return valid_orders
~~~

These are simple teaching functions. A real business contract must define whether negative amounts represent refunds or invalid values.

## 7. Write unit tests

Create tests/test_transform.py:

~~~python
import unittest

from src.transform import calculate_total, filter_valid_orders


class TestTransform(unittest.TestCase):

    def test_calculate_total(self):
        amounts = [100, 200, 50]
        result = calculate_total(amounts)
        self.assertEqual(result, 350)

    def test_filter_valid_orders(self):
        orders = [
            {"order_id": 1, "amount": 100},
            {"order_id": 2, "amount": -20},
            {"order_id": 3, "amount": 50}
        ]

        result = filter_valid_orders(orders)

        self.assertEqual(len(result), 2)
        self.assertEqual(result[0]["order_id"], 1)


if __name__ == "__main__":
    unittest.main()
~~~

### 7.1 Explain the parts
- unittest is Python's built-in testing framework.
- TestTransform groups related tests.
- Methods beginning with test_ are discovered by unittest.
- assertEqual compares actual and expected values.
- A failed assertion makes the test fail.
- The main guard allows direct execution of the test file.

## 8. Run tests locally

From the project root:

~~~bash
python -m unittest discover -s tests -v
~~~

Expected output shape:

~~~text
test_calculate_total ... ok
test_filter_valid_orders ... ok

Ran 2 tests
OK
~~~

Exact timing varies. Read assertion messages when a test fails.

## 9. Regression testing

Regression testing checks that a change has not broken behavior that previously worked.

For data pipelines, compare expected aggregates, key counts, schema, transformation outputs and null behavior.

## 10. What is GitHub Actions?

GitHub Actions is an automation platform integrated with GitHub. A workflow file describes when automation starts, which jobs run, which runner executes a job and which steps are performed.

Workflow files are stored in the folder .github/workflows/.

## 11. Create a basic CI workflow

Create file .github/workflows/python-ci.yml:

~~~yaml
name: Python Data Pipeline CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

      - name: Run unit tests
        run: python -m unittest discover -s tests -v
~~~

This is a starter workflow. Production teams should review and pin third-party actions according to supply-chain policy.

## 12. Explain the YAML

- name labels the workflow.
- on defines events that start it.
- push runs after a matching branch receives a push.
- pull_request runs for a pull request targeting the configured branch.
- jobs defines jobs.
- runs-on selects the runner.
- steps lists actions and commands.
- checkout downloads repository source into the runner.
- setup-python configures Python.
- pip installs dependencies.
- unittest runs the test suite.

## 13. What is a runner?

A runner is the machine/environment that executes a GitHub Actions job. GitHub-hosted runners are managed by GitHub. Self-hosted runners are managed by the organization and need additional security and maintenance.

## 14. What happens when a test fails?

The test command returns a non-zero exit code. The step is marked failed, the job normally stops at that failure and the workflow run reports failure. The team inspects logs and fixes the issue.

A failed CI run is useful feedback. Do not bypass a failing test just to make status green.

## 15. Syntax check

~~~bash
python -m compileall src tests
~~~

This checks Python compilation. It does not prove business logic or database connectivity.

## 16. Integration tests

Integration tests verify components work together.

Example:
- connect to a temporary test database;
- create a small test table;
- insert test rows;
- run extraction code;
- verify returned result;
- clean up.

Use isolated test infrastructure and credentials. Never point automated tests at production databases.

## 17. End-to-end tests

End-to-end tests exercise the complete flow:
- place a small source file in a test folder;
- run ETL;
- validate curated output;
- verify counts and totals.

They are more realistic but slower and more environment-dependent.

## 18. CI versus deployment

~~~text
Developer change
      |
      v
Pull request
      |
      v
Automated tests
      |
      v
Code review and approval
      |
      v
Merge
      |
      v
Build/package artifact
      |
      v
Deploy to test/staging
      |
      v
Smoke test and approval
      |
      v
Production release
~~~

Not every organization uses exactly this sequence. Controls should match release risk.

## 19. What is a smoke test?

A small check confirming essential deployed functions are available. For a pipeline it may check DAG discovery, connection availability, a small query, test output write and quality check.

A smoke test does not replace a full test suite.

## 20. Secrets in CI

Do not put tokens or passwords directly in workflow YAML. Use repository/environment secrets or approved identity federation. Restrict which workflows and environments can access production secrets.

Never print secret values. Be careful with workflows triggered by contributions from forks; secrets are generally restricted in those contexts.

## 21. Deployment safety

Before release:
- tests pass;
- code is reviewed;
- configuration is validated;
- dependencies are known;
- secrets are available securely;
- database changes are reviewed;
- rollback or forward-fix plan exists;
- monitoring is ready.

Rolling back application code does not automatically reverse data already written. Data recovery needs its own plan.

## 22. Data pipeline-specific tests

- required schema exists;
- business keys are unique at intended grain;
- row counts are within expected ranges;
- source and target totals reconcile;
- transformations handle nulls;
- incremental loads do not duplicate records;
- reruns are idempotent;
- DAG dependencies are valid;
- container image builds successfully.

## 23. Common mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Test only on laptop | Environments differ | Run repeatable CI checks |
| No unit tests | Logic defects go unnoticed | Test pure functions |
| Test production DB | Risk to real data | Use isolated resources |
| Put tokens in YAML | Secret exposure | Use secure CI secrets/identity |
| Ignore failed CI | Defects may be released | Fix or formally review |
| Test only syntax | Business behavior may be wrong | Add assertions and data checks |
| Assume rollback restores data | Writes may remain | Plan data recovery separately |
| Use unpinned actions blindly | Supply-chain risk | Review/pin actions per policy |

## 24. Knowledge check

**Q1. What is CI?** Continuous Integration: automatically checking code changes as they are integrated.

**Q2. What can CD mean?** Continuous Delivery or Continuous Deployment; delivery may require approval, deployment automatically releases qualifying changes.

**Q3. Unit test?** Test of a small code unit such as a function.

**Q4. Integration test?** Verifies components work together.

**Q5. Runner?** The environment executing a workflow job.

**Q6. What happens when a CI test fails?** The step/job reports failure and logs are available.

**Q7. Does green CI prove production works?** No. It proves only the configured checks passed in that runner environment.

## Final self-check
- [ ] Explain CI and CD.
- [ ] Distinguish unit, integration, end-to-end and regression tests.
- [ ] Write and run Python unittest tests.
- [ ] Explain a GitHub Actions workflow.
- [ ] Understand jobs, runners and steps.
- [ ] Explain failure behavior.
- [ ] Protect CI secrets.
- [ ] Describe release gates, smoke tests and rollback limits.
- [ ] Identify data pipeline-specific tests.

**Phase 7 content ends here.** You have learned orchestration, DAGs, scheduling, reliability, data quality, monitoring, security, containers and automated testing. The next phase applies these skills in an end-to-end Data Engineering capstone.
