# Day 6 — Python Environment, Syntax, Types and Data Structures | Interview

## Questions
**Why Python for Data Engineering?** It has strong libraries and readable syntax for APIs, files, validation, transformations, automation, and orchestration integrations.

**What is a virtual environment?** An isolated dependency environment for a project, preventing package/version conflicts.

**List vs tuple?** List is mutable; tuple is immutable. Use a list for changing collections and tuple for fixed grouped values.

**Set vs list?** A set stores unique values and supports efficient membership checks; a list preserves order and duplicates.

**Dictionary?** Key/value mapping, useful for JSON records and configuration.

**What is None?** A Python object representing absence of a value; distinguish it from empty string, zero, and False.

## Practical prompts
- A customer ID is `"0012"`: why string? It is an identifier; arithmetic is not meaningful and leading zeros matter.
- Package works locally but not on teammate's machine: check Python version, active venv, dependency list, and install instructions.
- Dataset has mixed numeric/string amounts: inspect source values, define a conversion rule, and reject/quarantine invalid values.

## Quick revision
Explain environment isolation, core collection trade-offs, and why data types must be explicit at ingestion boundaries.