# Day 8 Interview Preparation: File Operations

**Mini-project:** File Processing Manager

## 1. Core questions

1. **Why use a context manager when opening files?**  
   It reliably closes the file after the block completes, including when an exception occurs.

2. **What is the difference between read, write and append modes?**  
   Read retrieves existing content; write creates or truncates; append adds content at the end.

3. **Why use pathlib?**  
   It provides readable, cross-platform path operations.

4. **How do you process a large text file?**  
   Iterate over the file handle line by line or use bounded chunks rather than loading the complete file.

5. **What is the risk of write mode?**  
   It truncates an existing file. A wrong path can destroy an earlier output.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| Absolute vs relative path | Full path vs path resolved from working directory |
| Text vs binary mode | Decoded characters vs raw bytes |
| `read()` vs iteration | Read all/selected characters vs process lines incrementally |
| `write` vs `append` | Replace contents vs add to existing contents |
| File vs directory | Data object vs container of filesystem entries |

## 3. Practical questions

6. **How do you find all CSV files in nested folders?**  
   Use `Path("folder").rglob("*.csv")`.

7. **How do you avoid reading an incomplete output?**  
   Write to a temporary file, close and validate it, then move/rename to the final location where the storage system supports safe rename semantics.

8. **Why not parse every CSV with split(",")?**  
   Quoted values may contain commas, quotes or newlines. Use the `csv` module or a suitable data library.

## 4. Scenario questions

### Scenario A: FileNotFoundError on scheduled execution
Check the working directory, configured absolute/relative path, mount availability and file-arrival timing. Do not assume the scheduler starts in the developer's folder.

### Scenario B: A 20 GB text file causes memory failure
Use line-by-line or chunked processing, write incremental outputs and avoid keeping all records in memory.

### Scenario C: Same input file is processed twice
Track stable file identity, checksum or source batch ID; mark successful processing and make downstream writes duplicate-safe.

## 5. Troubleshooting

- UnicodeDecodeError: verify encoding with the source owner.
- PermissionError: inspect service account permissions and file locks.
- Empty file: verify path, size, producer completion and filters.
- Partial file: check arrival protocol; require a completion marker or atomic publish pattern where possible.
- Unexpected rows: use a format-aware parser and validate schema.

## 6. Follow-up questions

- What is a file descriptor?
- Why is encoding important?
- What is a checksum?
- How would you archive source files?
- How do you handle zero-byte files?
- What metadata should be recorded for each processed file?

## 7. Mini-project defence

Explain how the File Processing Manager discovers files, validates records, separates accepted and rejected data, and prevents duplicate processing. State why the demonstration parser is not a replacement for a proper CSV parser.
