# Day 67 Interview Guide — CI/CD and Automated Pipeline Testing

## 1. What is Continuous Integration?
CI is the practice of frequently integrating code changes into a shared repository and running automated checks so defects are detected early.

## 2. What is Continuous Delivery?
Continuous Delivery keeps changes in a releasable state after automated checks, but production release may require human approval.

## 3. What is Continuous Deployment?
Continuous Deployment automatically releases changes that pass required controls to production. It requires strong testing and release safeguards.

## 4. Delivery versus deployment?
Delivery prepares a release and may wait for approval. Deployment automatically releases qualifying changes to production.

## 5. What is a unit test?
A unit test checks one small unit of code, such as a function that calculates an order total. It should be fast and isolated.

## 6. What is an integration test?
It verifies that components work together, such as Python connecting to a test database and reading records.

## 7. What is an end-to-end test?
It exercises the complete flow from input to output, for example ETL producing a validated curated dataset.

## 8. What is regression testing?
It checks that a code change has not broken behavior that previously worked. For pipelines, compare expected aggregates, schema, key counts and transformation outputs.

## 9. What is GitHub Actions?
GitHub's automation platform. A workflow file defines trigger events, jobs, runners and steps that execute commands or reusable actions.

## 10. What is a runner?
The machine/environment that executes a workflow job. It may be GitHub-hosted or self-hosted.

## 11. What is a workflow?
An automated process defined in YAML under the .github/workflows folder. It can run on pushes, pull requests or other supported events.

## 12. What happens when a test fails?
The test command returns a non-zero exit code. The step and job are marked failed, and logs are available for investigation before release.

## 13. Why run tests on pull requests?
They provide feedback before changes are merged. Required status checks can prevent merging until important checks pass.

## 14. Why are unit tests useful for data engineers?
They test business transformation logic quickly, catch edge cases and make refactoring safer. They do not replace integration tests against databases or cloud systems.

## 15. What should a data pipeline test?
Schema, required values, uniqueness at correct grain, row counts, source/target totals, null behavior, incremental load behavior, idempotency, DAG dependencies and output correctness.

## 16. What is a smoke test?
A small post-deployment test of essential functions, such as DAG discovery, connection availability, a small query and a test output write.

## 17. Why should CI not use production credentials by default?
Tests run frequently and may be triggered by different contributors. Production credentials increase the impact of a workflow compromise. Use isolated test resources and restricted secrets.

## 18. How should secrets be used in GitHub Actions?
Use repository/environment secrets or approved identity federation. Restrict which workflows can access production credentials and never print secret values.

## 19. Does green CI prove production is healthy?
No. It proves the configured checks passed in the runner environment. Production may still differ in network, permissions, data volume, source availability or runtime.

## 20. What is rollback?
Rollback returns application code/configuration to a previously known version. It does not automatically undo data already written. Data recovery may need a restore, replay or corrective load.

## 21. Scenario: a test passes locally but fails in Actions.
Compare Python versions, dependencies, working directory, paths, environment variables, OS and test data. CI's clean environment can reveal assumptions hidden on a developer machine.

## 22. Scenario: a pull request changes a revenue total.
Inspect the transformation diff and test fixture. Determine whether the new total is an intended business-rule change and require an approved expected result. Do not simply update the assertion to make it green.

## 23. Explain your Day 67 project.
I wrote unit tests for transformation functions, added a data-quality test, created a GitHub Actions workflow for pushes and pull requests, introduced a controlled defect to demonstrate failure and restored correct behavior. I documented staging checks, secrets handling and the limits of CI validation.

## Common interview traps
- Treating CI and deployment as the same.
- Testing only syntax.
- Using production databases in unit tests.
- Putting secrets directly in YAML.
- Ignoring failed status checks.
- Updating expected results without business approval.
- Assuming code rollback reverses data writes.
- Claiming CI proves production readiness.

## Readiness checklist
- [ ] Explain CI, Continuous Delivery and Continuous Deployment.
- [ ] Distinguish unit, integration, end-to-end and regression tests.
- [ ] Explain GitHub Actions workflow, job, runner and step.
- [ ] Describe pull request checks.
- [ ] Protect CI secrets.
- [ ] Explain smoke tests and release gates.
- [ ] Explain rollback versus data recovery.
