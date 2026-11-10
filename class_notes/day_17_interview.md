# Day 17 — Unit Testing | Interview

**Unit vs integration?** Unit tests isolate a function; integration tests verify components/services interact correctly.

**What is pytest?** Python testing framework that discovers tests and provides assertions, fixtures, and plugins.

**What should transformation tests cover?** Typical inputs, boundaries, nulls, invalid types, and regression cases.

**Why deterministic tests?** Repeatable results make failures meaningful; control external dependencies and test data.

**Scenario: unit tests pass but load fails.** Investigate connectivity, permissions, schema, transactions, environment, and integration behavior; add integration/contract tests.