# Day 13 — Database Connections from Python | Interview

**What is a database driver?** Library implementing database protocol/connectivity so application code can execute SQL and retrieve results.

**Why parameterized queries?** Safe value binding prevents SQL injection and handles quoting/types correctly.

**Commit vs rollback?** Commit persists transaction changes; rollback undoes uncommitted changes.

**How load millions of rows?** Use bulk APIs, bounded batches, staging/merge patterns, and monitor transaction size/locks; avoid row-by-row calls.

**How protect credentials?** Secret manager/environment, least privilege, rotation, no source control or logs.

Scenario: Python process crashes after insert but before checkpoint. Use transaction/unique key/idempotent merge and checkpoint only after commit to avoid duplicate load.