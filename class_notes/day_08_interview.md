# Day 8 Interview Preparation: File Operations

**Mini-project:** File Processing Manager

## 1. Core questions

1. **Why is file handling important in data engineering?**  
   Data often arrives as files. Pipelines must discover, read, validate, transform, write and archive these files reliably.

2. **What is the difference between an absolute and relative path?**  
   An absolute path identifies a location from the drive or root. A relative path is resolved from the current working directory.

3. **Why is `pathlib` preferred for path handling?**  
   It provides readable path operations and handles platform-specific separators.

4. **What does file mode `w` do?**  
   It creates a file or truncates an existing file. Existing content is lost.

5. **What is the difference between `w` and `a`?**  
   `w` replaces existing content; `a` adds content at the end.

6. **What is the purpose of a context manager?**  
   The `with` statement ensures the file is closed after the block, including when an exception occurs.

7. **What is the difference between `read()` and iterating over a file?**  
   `read()` loads the complete content into memory. Iteration processes one line at a time.

8. **What does `Path.exists()` check?**  
   Whether a filesystem path exists. It does not prove the file contains valid data.

9. **What is the purpose of `shutil.copy2()`?**  
   It copies a file and attempts to preserve metadata such as timestamps.

10. **Why should a source file be archived only after output verification?**  
    If processing fails, the original should remain available for retry and investigation.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| File vs directory | A file stores content; a directory organizes filesystem entries |
| Relative vs absolute path | Relative depends on working directory; absolute identifies the full location |
| `read()` vs `readlines()` | String containing all content vs list of lines |
| `readlines()` vs file iteration | Both can load all lines vs iterative reading |
| `w` vs `a` | Overwrite vs append |
| `rename()` vs `copy2()` | Changes path/name vs creates a copy |
| Raw vs processed folder | Original source vs transformed output |

## 3. Practical implementation questions

11. **How do you create a folder only when it does not exist?**  
    Use `Path.mkdir(parents=True, exist_ok=True)`. The flag prevents an error if the directory already exists.

12. **How do you check that a file exists and is not empty?**  
    Check `path.is_file()` and `path.stat().st_size > 0`. Then validate the file's contents.

13. **How would you process every CSV file in a folder?**  
    Use `Path.glob("*.csv")` to discover files and process each path. Use the CSV parser to read records.

14. **How do you avoid loading a large text file into memory?**  
    Iterate over the file object and process each line or a bounded batch.

15. **How would you copy a source file before transforming it?**  
    Use `shutil.copy2(source, backup)`, then verify the backup.

16. **How do you move a processed file to an archive?**  
    Verify the output first, then use `shutil.move()` to move the original to the archive directory.

17. **Why should file encoding be specified?**  
    Explicit encoding reduces differences between machines and avoids relying on environment defaults.

## 4. Scenario-based questions

### Scenario 1: The pipeline says the file is missing

**Problem:** The file is visible in the project folder, but Python raises `FileNotFoundError`.

**Strong answer:**
- Print `Path.cwd()` to inspect the current working directory.
- Build the path from a known project directory.
- Check `path.exists()` and `path.is_file()`.
- Avoid assuming that the notebook starts in the same directory as the script.

**Reasoning:** Relative paths are interpreted from the process's working directory, not necessarily the source-code file's location.

### Scenario 2: Yesterday's records disappeared

**Problem:** A job opens the daily output using mode `w`, and yesterday's records are lost.

**Strong answer:**
- Explain that `w` truncates the file.
- Use a date-partitioned output path or append mode only if the data contract allows it.
- Prefer writing to a temporary output and validating it before replacing a published file.
- Preserve raw source data.

### Scenario 3: A 10 GB log file crashes the script

**Problem:** The script calls `read()` and runs out of memory.

**Strong answer:**
- Iterate over the file line by line or in bounded chunks.
- Avoid retaining all processed records in a list.
- Write output incrementally.
- Monitor memory and throughput for production-sized data.

### Scenario 4: The source was archived but the output is incomplete

**Problem:** The input file was moved before the output write completed.

**Strong answer:**
- Change the order: write output, close it, verify existence and expected counts, then archive.
- If verification fails, preserve the source and mark the batch as failed.
- Use a temporary output path to avoid exposing partial output.

## 5. Troubleshooting questions

18. **What causes `PermissionError`?**  
    The process may not have permission, or another application may hold a conflicting lock. Check access and close the other application.

19. **Why can a file exist but still be unusable?**  
    It may be empty, malformed, encoded unexpectedly or contain an invalid schema.

20. **Why is manual comma splitting unsafe for CSV?**  
    Quoted fields can contain commas, quotes and newlines. Use the `csv` module.

21. **What should happen if the same source file arrives twice?**  
    Define duplicate-delivery behavior using file identity, batch IDs, checksums or source metadata. Avoid accidental double processing.

## 6. Mini-project explanation

**Question:** Explain the File Processing Manager project.

**Strong answer:**

"I created input, output and archive folders using pathlib. I checked that the incoming file existed and was not empty. I read the file, removed blank lines, wrote a cleaned output to a separate folder, verified the output row count and only then moved the source into the archive. This taught me why source retention, output verification and safe file modes matter in a data pipeline."

## 7. Follow-up questions an interviewer may ask

- How would you handle a file that is still being uploaded?
- How would you detect a duplicate file?
- How would you process files in date order?
- How would you retain rejected files for investigation?
- How would you make the process safe to rerun?
- What metadata would you record for each processed file?

A good junior answer should mention validation, source retention, clear failure handling and repeatable processing.

## 8. Quick revision

- Always inspect the working directory when a relative path fails.
- Use `with open(...)` for safe file access.
- Understand the destructive behavior of `w`.
- Iterate over large files instead of reading everything at once.
- Verify outputs before archiving source data.
- Use structured parsers for CSV/JSON/XML rather than manual string splitting.


---

## 7. Follow-up interview questions

16. **What is a file path?** A location that identifies a file or directory.
17. **Absolute versus relative path?** An absolute path starts from a filesystem root; a relative path is interpreted from a current base directory.
18. **What is the current working directory?** The directory from which the process resolves relative paths.
19. **What does file mode `r` mean?** Open for reading.
20. **What does `w` mean?** Open for writing and replace existing content or create a new file.
21. **What does `a` mean?** Open for appending at the end of the file.
22. **Why use `with` when opening a file?** It ensures the file is closed when the block finishes, even if an exception occurs.
23. **What is encoding?** A rule for converting text characters to bytes and bytes back to text.
24. **Why specify UTF-8?** It makes the intended text encoding explicit and supports a broad range of characters.
25. **What is the difference between `read()` and line-by-line iteration?** `read()` can load the entire content; iteration processes one line at a time.
26. **Why use pathlib?** It provides a consistent object-oriented interface for paths and path operations across operating systems.
27. **What does `mkdir(parents=True, exist_ok=True)` do?** Creates missing parent folders and does not fail if the target directory already exists.
28. **Why verify a copied file?** A successful-looking workflow should confirm that the destination exists and contains expected data.
29. **When should a source file be archived?** After successful processing and validation, with a recovery plan for failures.
30. **What is a safe file-ingestion practice?** Use deterministic naming, validate the input, write output to a separate destination, verify it, and only then mark the source as processed.

## 8. Scenario questions

**The file exists in Explorer but Python reports FileNotFoundError.** The script may be running from a different working directory. Print `Path.cwd()`, inspect the resolved path and use a deliberate project-relative or absolute path.

**The output file contains only the latest record.** The code may be opening the file with `w` on every iteration. Open once or use append mode where appropriate, and understand the overwrite behavior.

**A file contains accented or non-English names incorrectly.** Confirm the source encoding and decode it consistently. Do not guess encodings silently in a production ingestion process.

**The input file is very large.** Avoid reading the whole file into memory. Iterate over lines or use a format-aware chunking strategy, while accounting for headers, quoted delimiters and multi-line records.

## 9. Mini-project defense

- How does the script identify the input file?
- How does it behave if the file is missing?
- How do you know the output is complete?
- Why keep input and processed folders separate?
- When is it safe to archive the source?

## 10. Readiness rubric

| Skill | Ready when the student can… |
|---|---|
| Paths | Explain relative paths and inspect current directory |
| File modes | Choose read, write or append correctly |
| Safety | Use context managers and preserve source data |
| Folders | Create and inspect directories |
| Verification | Confirm output exists and contains expected content |
| Operations | Explain validation-before-archive |
