# Day 11: Iterators and Generators

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** Large File Reader  
**Learning time:** 2–3 hours

## 1. Learning outcomes
- Explain iterable, iterator and generator.
- Use `iter()` and `next()`.
- Create generators with `yield`.
- Understand lazy evaluation and memory efficiency.
- Process large files in bounded batches.

## 2. Why memory-efficient processing matters
A file containing a few hundred rows can often be loaded fully into memory. A file containing hundreds of millions of rows may not fit. Loading everything at once can exhaust memory and stop a pipeline.

The key idea: process a manageable portion, write or aggregate the result, then continue.

## 3. Iterable
An iterable is an object that can return its members one at a time, such as a list, tuple, string or file object.

```python
order_ids = [501, 502, 503]

for order_id in order_ids:
    print(order_id)
```

The for loop obtains values from the iterable automatically.

## 4. Iterator
An iterator remembers its current position and provides the next value when requested.

```python
order_ids = [501, 502, 503]
order_iterator = iter(order_ids)

print(next(order_iterator))
print(next(order_iterator))
print(next(order_iterator))
```

Expected output: 501, 502, 503. Calling `next()` after the final value raises `StopIteration`, which signals completion.

## 5. Generator
A generator is an iterator that produces values lazily. A generator function uses `yield` to produce one value and pause its state.

```python
def order_id_generator():
    yield 501
    yield 502
    yield 503

orders = order_id_generator()

print(next(orders))
print(next(orders))
```

Expected output: 501 and 502. The generator resumes from where it paused.

## 6. Return vs yield
- `return` finishes a function and sends back its result.
- `yield` produces a value and suspends the generator so it can continue later.

A generator is useful when values can be produced incrementally rather than stored in a large list.

## 7. Generate a sequence
```python
def order_ids(start_id, count):
    current_id = start_id

    for position in range(count):
        yield current_id
        current_id += 1

for order_id in order_ids(1001, 4):
    print(order_id)
```

Expected values: 1001, 1002, 1003, 1004.

## 8. Generator expression
```python
amounts = [100, 200, 300, 400]

squared_values = (amount * amount for amount in amounts)

for value in squared_values:
    print(value)
```

Parentheses create a generator expression. A list comprehension creates a list immediately.

## 9. Memory comparison
```python
numbers_list = [number for number in range(100000)]
numbers_generator = (number for number in range(100000))

print(type(numbers_list))
print(type(numbers_generator))
```

The list stores all generated values. The generator computes each value when requested. A generator does not make every operation automatically faster; it is useful when incremental consumption reduces memory pressure.

## 10. Process a file line by line
```python
def read_order_lines(file_path):
    with open(file_path, "r", encoding="utf-8") as file:
        for line in file:
            yield line.strip()

for line in read_order_lines("orders.txt"):
    print(line)
```

The file is read incrementally. Keep the file open only while iteration is active; the generator's context manager closes it when iteration finishes or the generator is closed.

## 11. Process records in batches
```python
def read_batches(file_path, batch_size):
    with open(file_path, "r", encoding="utf-8") as file:
        batch = []

        for line in file:
            batch.append(line.strip())

            if len(batch) == batch_size:
                yield batch
                batch = []

        if batch:
            yield batch

for batch in read_batches("orders.txt", 1000):
    print("Batch size:", len(batch))
```

The final batch may contain fewer records. Batch size should be chosen based on record size, available memory and downstream capacity.

## 12. Mini-project: Large File Reader
Create a sample file:

```python
with open("large_orders.txt", "w", encoding="utf-8") as file:
    for order_id in range(1, 10001):
        file.write(f"{order_id},100,PAID\n")
```

Count records without reading all rows into a list:

```python
row_count = 0

with open("large_orders.txt", "r", encoding="utf-8") as file:
    for line in file:
        row_count += 1

print(row_count)
```

Process in batches and calculate total rows:

```python
total_rows = 0

for batch in read_batches("large_orders.txt", 500):
    total_rows += len(batch)

print(total_rows)
```

Expected output: 10000.

## 13. Common mistakes
| Mistake | Better approach |
|---|---|
| Calling next after completion | Use a for loop or handle StopIteration |
| Expecting generator values to be reusable | Create a new generator for a second pass |
| Converting generator to list unnecessarily | Consume incrementally |
| Choosing huge batches | Measure memory and throughput |
| Forgetting final partial batch | Yield remaining values after loop |
| Treating lazy as always faster | Measure end-to-end performance |

## 14. Day 11 revision
- Iterable can provide an iterator.
- Iterator returns one item at a time.
- Generator uses yield to produce values lazily.
- File iteration is naturally incremental.
- Batching balances memory use and processing overhead.
