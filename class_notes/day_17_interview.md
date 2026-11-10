# Day 17 Interview Preparation: Unit Testing and Reusable Pipeline Functions

**Mini-project:** Transformation Test Suite

## 1. Core questions

1. **What is a unit test?** A test that verifies a small unit of code in isolation.
2. **Why test transformations?** To protect business rules and detect regressions when code changes.
3. **What is an assertion?** A check comparing actual behavior with expected behavior.
4. **What is a fixture?** Controlled setup data or resources used by a test.
5. **What is regression testing?** Re-running tests to ensure a previous defect has not returned.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| Unit vs integration test | Isolated unit vs interaction between components |
| Integration vs end-to-end | Component boundary test vs complete workflow test |
| Mock vs real service | Controlled substitute vs actual dependency |
| assertEqual vs assertRaises | Compare result vs expect exception |
| Happy path vs boundary test | Normal input vs edge of allowed range |

## 3. Practical questions

6. **What cases should a transformation test cover?** Valid, empty, null, boundary, invalid and duplicate inputs as relevant.
7. **Should unit tests call a production database?** No. Use controlled dependencies; integration tests use isolated test infrastructure.
8. **How do you test an exception?** Use assertRaises and verify the expected exception type.
9. **What makes a test reliable?** Deterministic inputs, isolation, clear assertions and no hidden dependency on execution order.

## 4. Scenario questions

### Scenario A: Test passes locally but fails in CI
Compare Python/package versions, environment variables, working directory, test data, timezone and external dependencies.

### Scenario B: A test fails after a business-rule change
Confirm the approved requirement first. Update implementation and expected tests only when the rule genuinely changed; do not weaken tests to hide a defect.

### Scenario C: Unit tests pass but pipeline output is wrong
Inspect integration boundaries, source data, SQL, configuration, scheduling and deployment. Add an integration test that reproduces the missing behavior.

## 5. Troubleshooting
- Test not discovered: check file naming and test runner conventions.
- Flaky test: remove timing/network/shared-state dependency.
- False positive: strengthen assertion.
- Order-dependent test: isolate state and cleanup.
- Slow suite: separate fast unit tests from integration tests.

## 6. Follow-up questions
- What is code coverage?
- Does 100% coverage guarantee correctness?
- What is test-driven development?
- How do you test data quality rules?
- How do you test retry logic?
- What should be tested in a pipeline DAG?

## 7. Mini-project defence
Show the transformation functions, test matrix and test run. Explain what each test protects and how you would extend the suite to integration tests for database and file boundaries.
