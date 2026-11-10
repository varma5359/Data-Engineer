# Day 10 Interview Preparation: Exception Handling and Debugging

**Mini-project:** Resilient File Processor

## 1. Core questions

1. **What is an exception?** A runtime event that interrupts normal program flow.
2. **Why use try/except?** To handle expected failure conditions and take a controlled action.
3. **What is finally?** A block that runs whether an exception occurs or not, commonly used for cleanup.
4. **What is logging?** Recording application events with severity and context for debugging and operations.
5. **Why prefer specific exceptions?** They prevent unrelated failures from being hidden by overly broad handling.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| Syntax error vs exception | Parsing/grammar problem vs runtime failure |
| print vs logging | Simple display vs configurable operational event record |
| warning vs error | Unexpected recoverable condition vs failed operation |
| raise vs return | Signal failure vs return a normal result |
| DEBUG vs INFO | Diagnostic detail vs normal execution milestone |

## 3. Practical questions

6. **How do you log a traceback?** Use `logger.exception()` inside an exception handler.
7. **Should every invalid row stop a pipeline?** Not necessarily. Follow a defined reject policy and threshold; preserve rejected records and report counts.
8. **What should not be logged?** Passwords, tokens, payment secrets and unnecessary personal information.

## 4. Scenario questions

### Scenario A: A file is missing
Log the source path and run context, mark the stage failed, alert the owner and follow the missing-file policy. Do not create an empty success output.

### Scenario B: One row has an invalid amount
Quarantine the row with line number and reason if the contract allows record-level rejection. Track the rejection count and fail the batch if the threshold is exceeded.

### Scenario C: A pipeline catches Exception and reports success
Change the boundary behavior to log the traceback, mark the run failed and re-raise or return a failed status. Never convert an unknown failure into success.

## 5. Troubleshooting

- No useful traceback: preserve exception context and use logger.exception.
- Too many log messages: select appropriate levels and include run identifiers.
- Error cannot be reproduced: retain safe input samples and environment/version metadata.
- Repeated failure: distinguish transient infrastructure problems from permanent data/schema errors.

## 6. Follow-up questions

- What is exception chaining?
- When would you define a custom exception?
- What is log rotation?
- How do you correlate logs across pipeline stages?
- What is a dead-letter or quarantine location?
- Why should retries be bounded?

## 7. Mini-project defence

Explain how the Resilient File Processor separates valid and rejected records, records operational context, handles a missing file and decides whether a partial batch is acceptable. State the agreed policy rather than inventing a rejection threshold.
