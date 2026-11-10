# Day 11 Interview Preparation: Iterators and Generators

**Mini-project:** Large File Reader

## 1. Core questions

1. **What is an iterable?**  
   An object that can provide its elements one at a time, such as a list or file.

2. **What is an iterator?**  
   An object that tracks its current position and returns the next value.

3. **What does iter() do?**  
   It obtains an iterator from an iterable.

4. **What does next() do?**  
   It requests the next value from an iterator.

5. **What is StopIteration?**  
   The signal that an iterator has no more values.

6. **What is a generator?**  
   An iterator produced by a generator function or expression.

7. **What does yield do?**  
   It produces a value and pauses the generator while preserving its state.

8. **What is lazy evaluation?**  
   Values are calculated or produced only when requested.

9. **Why use generators in data engineering?**  
   They can process large input incrementally and avoid storing every generated value at once.

10. **Does a generator always guarantee low memory usage?**  
    No. Downstream code may still collect all generated values into a list or dataframe.

11. **What is batching?**  
    Processing a bounded group of records at a time.

12. **What happens when a generator is exhausted?**  
    It cannot produce more values; create a new generator to iterate again.

## 2. Difference questions

| Concept | Difference |
|---|---|
| Iterable / iterator | Can provide an iterator / provides next values and tracks position |
| Iterator / generator | General next-value object / a convenient iterator created with yield |
| return / yield | Ends function and returns / produces a value and pauses |
| read() / file iteration | Loads content / reads incrementally by line |
| list / generator | Stores all items / produces items on demand |
| Row / batch processing | One record at a time / bounded group of records |

## 3. Practical coding questions

**Q1. Create an iterator.**

```python
values = [10, 20]
items = iter(values)
print(next(items))
print(next(items))
```

Output: `10`, then `20`.

**Q2. Create a generator.**

```python
def ids():
    yield 101
    yield 102

print(list(ids()))
```

Output: `[101, 102]`.

**Q3. Sum values without storing a new list.**

```python
total = 0
for value in (x for x in [10, 20, 30]):
    total += value
print(total)
```

Output: `60`.

**Q4. Why process in batches?**  
   A bounded batch controls memory and often matches bulk database or file-writing operations.

## 4. Scenario-based questions

1. **A 20 GB text file cannot fit into memory. What would you do?**  
   Iterate over the file line by line or use a streaming/chunked reader. Transform and write each record or batch rather than calling `read()`.

2. **A generator produces no output the second time. Why?**  
   Generators are single-pass. The first loop exhausted it; create a new generator.

3. **The final batch contains fewer rows than the configured size. Is it an error?**  
   No. Emit the remaining partial batch at end-of-file unless the destination requires a different policy.

4. **A generator is used but memory still grows. What do you inspect?**  
   Look for lists or dataframes accumulating all results downstream, cached objects and unbounded queues.

5. **A file contains a header row. What must the reader do?**  
   Parse or skip the header before converting data rows.

## 5. Follow-up questions

- How does a for loop use an iterator?
- Can a generator be restarted?
- What is a generator expression?
- Why does a file object support iteration?
- How do you choose a batch size?
- What trade-off exists between small and large batches?
- What should happen if one row in a batch is malformed?

## 6. Strong revision points

- Generators are useful for incremental processing.
- Lazy evaluation is beneficial only when the downstream pipeline also avoids full materialization.
- Batch size affects memory, throughput and destination load.
- Validate malformed rows and preserve rejection details.


---

## 7. Follow-up interview questions

16. What is an iterable? An object that can provide its elements one at a time to an iteration process.
17. What is an iterator? An object that remembers iteration state and returns the next value.
18. What does iter() do? Obtains an iterator from an iterable.
19. What does next() do? Requests the next item from an iterator.
20. What is StopIteration? The signal that an iterator has no more values.
21. What is a generator? An iterator commonly created by a function using yield.
22. yield versus return? yield produces a value and pauses generator execution; return ends a function or generator.
23. What is lazy evaluation? Producing a value only when it is requested.
24. Why use a generator for a large file? It can process records gradually without storing all records in memory at once.
25. Does a generator guarantee low memory? No. Downstream code may still accumulate all generated values.
26. What is batching? Processing a bounded group of records at a time.
27. Why is line-by-line reading useful? It limits the amount of file content held in memory at one time.

## 8. Scenario questions

A 20 GB text file cannot fit in memory. Iterate through the file or use a streaming parser and write results incrementally.

A generator is converted to list() before processing. This materializes all values and can remove the memory advantage.

A batch has fewer records than the configured batch size at the end. This is normal; the final partial batch must still be processed.

## 9. Mini-project defense

- What state does an iterator maintain?
- Why does a generator pause at yield?
- How did you control batch size?
- Where could memory still grow in your implementation?

## 10. Readiness rubric

| Skill | Ready when the student can… |
|---|---|
| Iteration | Explain iterable, iterator and next |
| Generator | Write and consume a yield-based generator |
| Lazy behavior | Explain when generator code executes |
| Memory | Explain streaming versus full materialization |
| Batching | Produce and process bounded groups |
