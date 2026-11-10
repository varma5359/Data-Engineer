# Day 11 Interview Preparation: Iterators and Generators

**Mini-project:** Large File Reader

## 1. Core questions

1. **What is an iterable?** An object whose elements can be visited, such as a list or file.
2. **What is an iterator?** An object that provides values one at a time and tracks its position.
3. **What is a generator?** A lazy iterator commonly created by a function using `yield`.
4. **Why use generators in data engineering?** They allow large inputs to be consumed incrementally, reducing memory requirements.
5. **What is StopIteration?** The signal that an iterator has no further values.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| Iterable vs iterator | Can provide iterator vs produces next values |
| List vs generator | Materialized collection vs lazy value production |
| return vs yield | Finish function vs pause and resume generator |
| List comprehension vs generator expression | Builds list now vs yields values lazily |

## 3. Practical questions

6. **How do you read a large file?** Iterate over the file handle or yield bounded batches.
7. **Why use batches?** They limit memory while allowing efficient bulk operations.
8. **Can you iterate over a generator twice?** Not after it is exhausted; create a new generator to start again.

## 4. Scenario questions

### Scenario A: 15 GB file causes memory error
Avoid `read()` and `list(file)`. Stream rows, validate incrementally and write outputs in batches.

### Scenario B: Final batch is missing
Check that the generator yields the remaining non-empty batch after the main loop.

### Scenario C: Second pass returns no rows
The iterator was already exhausted. Recreate it or reopen the source.

## 5. Troubleshooting
- StopIteration: iterator is exhausted.
- Generator unexpectedly empty: check whether it was consumed earlier.
- Memory remains high: inspect whether batches or results are accumulated elsewhere.
- Slow pipeline: measure parsing, I/O and destination writes separately.

## 6. Follow-up questions
- What is lazy evaluation?
- How do generators help backpressure?
- What happens if a generator is closed?
- How do you choose a batch size?
- Why can a generator still use substantial memory?

## 7. Mini-project defence
Explain how the Large File Reader reads incrementally, handles the final partial batch, and avoids retaining all records. Discuss how you would add parsing, rejection handling, checkpointing and safe output writes.
