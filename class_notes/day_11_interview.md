# Day 11 — Iterators, Generators and Memory Efficiency | Interview

**What is an iterator?** An object that returns items one at a time through iteration.

**What is a generator?** A convenient iterator created with `yield` or a generator expression; values are produced lazily.

**Why use generators?** Reduce memory usage when processing large or unbounded data.

**Generator vs list?** A list stores all values; a generator computes values as consumed and is generally single-pass.

**How process a large CSV?** Read chunks, validate/transform each chunk, write results incrementally, and maintain cross-chunk state for deduplication or aggregates.

Scenario: duplicate IDs occur in different chunks. Chunk-local deduplication is insufficient; use a global key store, partition by key, or database constraint/merge strategy based on scale.