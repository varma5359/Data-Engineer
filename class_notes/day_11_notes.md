# Day 11 — Python Iterators, Generators and Memory Efficiency
**Phase 2: Python for Data Engineering**

## 1. Memory problem
Reading a 10 GB file into a list can exceed RAM. Processing records incrementally keeps memory closer to the current batch rather than the entire file.

## 2. Iterator
An iterator yields values one at a time. Files are iterable:
```python
with open("large.txt", encoding="utf-8") as file:
    for line in file:
        print(line.strip())
```
This avoids `readlines()` storing every line.

## 3. Generator expression
```python
numbers = (number * 2 for number in range(1, 5))
for number in numbers:
    print(number)
```
Output: 2, 4, 6, 8 (one value per iteration).

Parentheses create a generator expression; it computes values as consumed.

## 4. Generator function
```python
def valid_amounts(rows):
    """Yield amounts only for records with valid numeric values."""
    for row in rows:
        if row["amount"] is not None:
            yield row["amount"]

rows = [{"amount": 10}, {"amount": None}, {"amount": 30}]
print(list(valid_amounts(rows)))
```
Output: `[10, 30]`

`yield` returns a value and pauses execution; next iteration resumes it.

## 5. Chunk processing
```python
import pandas as pd
for chunk in pd.read_csv("orders.csv", chunksize=10000):
    print(len(chunk))
    # validate/transform/load this chunk
```
Each chunk can be processed and released instead of holding the complete file.

## 6. Trade-offs
- Streaming iteration reduces memory but may be slower for repeated passes.
- A generator is consumed once; recreate it for another pass.
- Global aggregation may need state or a separate aggregation strategy.
- Chunk boundaries can affect deduplication/joins; design keys/state carefully.

## Mini-project — Large File Reader
Read a large text/CSV incrementally, count records, reject malformed lines, and report counts without retaining every record.

## Revision
Iterator supplies values sequentially; generator creates values lazily; chunking bounds memory. Memory efficiency does not remove the need for correctness across chunks.