# Day 8 — File Operations and Directory Management | Interview

**Why use pathlib?** It provides readable, cross-platform path operations and directory/file methods.

**What does with open do?** Ensures file resources are closed even when an exception occurs.

**Difference between r, w, a?** Read; write/truncate or create; append or create.

**How process files safely?** Check file completeness, validate schema, preserve source, track filename and ingestion time, write output safely, and archive only after success.

**How handle encoding errors?** Identify source encoding, use explicit encoding, inspect malformed input, and avoid silently dropping characters.

Scenario: file is empty at scheduled time. Distinguish legitimate empty input from incomplete upload; verify producer completion marker/size and apply business policy (skip, alert, or publish empty batch).