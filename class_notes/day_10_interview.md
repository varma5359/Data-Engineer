# Day 10 — Exception Handling, Logging and Debugging | Interview

**Exception vs error?** Exceptions are runtime objects that represent abnormal execution and can be handled or propagated.

**Why catch specific exceptions?** It avoids hiding unrelated defects and allows appropriate recovery.

**What is finally?** A block executed whether success or exception occurs; useful for cleanup.

**Logging vs print?** Logging has levels, timestamps, configurable destinations, and structured context; print is basic console output.

**What should an ETL log contain?** Run/batch ID, source, start/end, counts, duration, status, retry/error details—without secrets or sensitive payloads.

Scenario: pipeline catches every exception and reports success. Change it to log exception context, fail the task or quarantine only approved bad records, and alert according to severity. Never convert an unknown failure into success.