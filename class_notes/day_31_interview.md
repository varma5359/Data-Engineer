# Day 31 Interview Preparation: File-Based Data Ingestion

**Phase:** 4 — Data Processing and Pipelines  
**Syllabus topic:** File-Based Data Ingestion  
**Mini-project:** Multi-File Ingestion System  
**Level:** Fresher / Junior Data Engineer

---

# 1. Core concepts — complete answers

## 1. What is file-based data ingestion?

File-based data ingestion is the process of collecting data from files and bringing it into a data platform for further processing or storage. The source may deliver CSV, JSON, XML, Parquet or other file formats. A reliable process discovers files, validates them, reads records, records processing status and handles failures.

## 2. Is reading a CSV file the same as building an ingestion pipeline?

No. Reading a CSV is one operation. An ingestion pipeline also needs file discovery, naming rules, completeness checks, schema validation, row validation, duplicate handling, archival, failure handling and operational tracking.

## 3. What is a landing or incoming folder?

It is the location where source files arrive before processing. A pipeline scans this folder for files that meet the delivery contract.

## 4. Why separate incoming, archive and rejected folders?

Separation makes processing state clear. Incoming contains files awaiting processing, archive preserves successfully processed source files, and rejected isolates files that failed file-level validation. This supports investigation and controlled reprocessing.

## 5. What is directory scanning?

Directory scanning is the process of listing files and folders in a location and selecting entries that match processing rules.

## 6. What is pathlib?

pathlib is a Python standard-library module for working with filesystem paths. It provides methods such as exists(), is_file(), is_dir(), iterdir(), glob() and mkdir().

## 7. What is the difference between iterdir() and glob()?

iterdir() returns direct entries inside a directory. glob() returns entries matching a pattern, such as *.csv. Use glob when you want to filter during discovery.

## 8. What is rglob()?

rglob() searches recursively through a directory and its subdirectories for paths matching a pattern.

## 9. Why should we sort discovered files?

Filesystem directory iteration order is not guaranteed. Sorting provides predictable processing order, which helps repeatability and debugging. Sorting alone does not guarantee correct business sequencing; use explicit delivery metadata if sequence matters.

## 10. What is a file naming convention?

It is an agreed pattern for naming files. A name may contain dataset, source, delivery date, sequence and extension. Consistent names help pipelines discover, route and validate incoming data.

## 11. What is file-level validation?

File-level validation checks whether the file as a whole is acceptable. Examples include extension, naming pattern, non-empty size, parseability, required headers and delivery completeness.

## 12. What is row-level validation?

Row-level validation checks each record's values. Examples include required IDs, numeric amounts, allowed status values, valid dates and business constraints.

## 13. What is the difference between file-level and row-level rejection?

A file-level failure means the file cannot be processed reliably, such as a missing required header. A row-level failure means the file is readable but one record violates a data rule. Depending on the contract, valid rows may still be accepted when other rows are rejected.

## 14. What is a data contract?

A data contract is an agreement between a data producer and consumer about the data being delivered. It can define schema, data types, required fields, allowed values, delivery frequency, naming convention, compatibility and quality expectations.

## 15. Why is schema validation important?

Without schema validation, a pipeline may read the wrong column, fail with a KeyError or silently interpret data incorrectly. Validate expected columns and their meaning before processing records.

## 16. Why is CSV data usually read as strings?

CSV is a text-based tabular format and does not have a universal embedded type system. Python's csv reader returns field values as strings. The pipeline must validate and convert them to the required types.

## 17. What is csv.DictReader?

DictReader reads CSV rows as dictionaries, using header names as keys. It makes code easier to read when records have named columns.

## 18. What is the difference between csv.reader and csv.DictReader?

csv.reader returns each row as a list of values. DictReader returns each row as a dictionary keyed by column names.

## 19. How do you read JSON in Python?

Use json.load() to read JSON from an open file object, or json.loads() to parse a JSON string. The parsed result becomes Python dictionaries, lists and primitive values.

## 20. What is the difference between CSV and JSON?

CSV is designed for simple tabular rows and columns. JSON supports nested objects and arrays and is common in APIs and application data. The format should be chosen based on source structure and downstream needs.

## 21. How do you detect an empty file?

Use Path.stat().st_size == 0 to detect a zero-byte file. But a non-zero file may still have only a header or contain invalid data, so additional validation is required.

## 22. How do you detect missing columns?

Read the header and compare it with the required schema. Depending on the contract, enforce exact order or check that all required names are present. Also consider duplicate and unexpected columns.

## 23. Why should duplicate column names be detected?

Dictionary-based parsing can become ambiguous when two columns have the same name. One value may overwrite another or downstream code may read an unintended field.

## 24. What is a malformed file?

A malformed file does not follow the syntax or structure expected for its format. Examples include invalid CSV quoting, invalid JSON syntax, truncated content or unexpected encoding.

## 25. How should a pipeline handle malformed files?

Catch expected parsing or decoding errors, record the file name and failure reason, isolate the file, and avoid marking it as successfully processed. Do not silently skip it.

---

# 2. File operations and Python questions

## 26. What is the difference between an absolute and relative path?

An absolute path identifies a location from the filesystem root. A relative path identifies a location relative to the current working directory or another known project location.

## 27. Why is pathlib useful in projects?

It provides a consistent object-oriented interface for paths and avoids manually joining path strings with operating-system-specific separators.

## 28. What does Path.exists() do?

It checks whether the referenced filesystem path exists.

## 29. What does Path.is_file() do?

It returns True if the path points to a regular file.

## 30. What does Path.is_dir() do?

It returns True if the path points to a directory.

## 31. What does Path.mkdir(parents=True, exist_ok=True) mean?

mkdir() creates a directory. parents=True creates missing parent directories, and exist_ok=True prevents an error if the directory already exists.

## 32. What is the difference between Path.name, Path.stem and Path.suffix?

name returns the full file name, stem returns the file name without the final extension, and suffix returns the extension including the dot.

## 33. How do you select only CSV files?

Use a pattern such as Path("incoming").glob("*.csv"), or check that path.suffix.lower() == ".csv".

## 34. Why should a pipeline not trust only the file extension?

A file can be named .csv but contain invalid or unrelated content. Validate its actual structure and parseability. Security-sensitive systems should also apply appropriate file scanning and content checks.

## 35. What does shutil.move() do?

It moves a file or directory to another location. It is often used to archive or isolate processed files.

## 36. What is the difference between "w" and "a" file modes?

"w" creates or replaces a file. "a" appends to an existing file or creates it if it does not exist. Choosing the wrong mode can overwrite previous pipeline output.

## 37. Why use a with statement when opening files?

The with statement manages the file resource and closes it when the block ends, including when an exception occurs.

## 38. What is encoding?

Encoding defines how text characters are represented as bytes. UTF-8 is a common standard. A pipeline should agree on expected encoding with the source and handle decoding errors clearly.

## 39. What is a UTF-8 BOM?

A byte-order mark is an optional marker at the beginning of a text file. Some spreadsheet exports include a UTF-8 BOM. Python's utf-8-sig codec can remove it when reading such files.

## 40. What is a file checksum or hash?

A hash is a fixed-length digest calculated from file contents. It can help identify identical content, even if file names differ. It does not by itself prove that two files represent the same business delivery.

---

# 3. Validation and reliability scenarios

## 41. A file has the correct name but the wrong columns. What do you do?

I would compare its header with the agreed data contract. If required fields are missing or the structure is incompatible, I would reject or quarantine the file, record the expected and actual schema, and notify the source owner. I would not continue as if the file were valid.

## 42. A file has 10,000 rows and only 10 are invalid. Would you reject the entire file?

It depends on the business contract. If partial acceptance is allowed, I would accept valid rows and send invalid rows to a rejection report with reasons. If the delivery must be atomic, I would reject the entire file. The policy must be explicit.

## 43. How do you avoid reading a file while it is still uploading?

The strongest approach is a source delivery contract: upload to a temporary name or location and rename/move the file into the ready folder only after upload completes, or provide a completion marker. A file-size stability check can help but is not definitive proof of completion.

## 44. What is a manifest file?

A manifest records the processing status of source files. It can include file name, source, processing time, status, row counts and error message. It supports audit, recovery and operational reporting.

## 45. Why archive source files?

The archive preserves evidence of what the source delivered. It helps investigate discrepancies, supports controlled reprocessing and keeps the incoming folder focused on new deliveries.

## 46. What should happen to rejected files?

They should be isolated from normal processing, and the pipeline should record the reason. The source owner or support team should be able to identify what needs correction. Rejected files should follow an access and retention policy.

## 47. What is a duplicate file delivery?

It occurs when the same delivery is received more than once. It can happen because a source retries, an acknowledgement is lost or a file is manually re-sent.

## 48. How do you detect duplicate files?

Possible methods include a stable source delivery ID, a manifest with delivery identifiers, file name plus source metadata, or a content hash such as SHA-256. A name alone is not always sufficient because the same name can contain different content.

## 49. What is the difference between duplicate files and duplicate records?

Duplicate files are repeated deliveries. Duplicate records are repeated business rows, possibly across different files. File-level detection and record-level key checks solve different problems.

## 50. What is idempotency in file ingestion?

Idempotency means repeating the same processing operation does not create unintended additional effects. A pipeline can use stable file identity, processing state, unique business keys and controlled output writes to make retries safe.

## 51. What happens if a process crashes after moving a file but before writing its manifest?

The file may be archived without a manifest entry. A production system needs reconciliation between folder state and processing metadata, plus a recoverable state machine so the file can be identified and repaired safely.

## 52. Why is a plain CSV manifest not ideal for multiple workers?

Concurrent processes may append at the same time, causing coordination and consistency problems. A database table or concurrency-safe metadata service is more appropriate for multi-worker pipelines.

## 53. How do you handle very large files?

Read and process records incrementally, write outputs in batches, avoid storing all rows in memory, monitor memory use and consider columnar formats or distributed processing when the workload requires it.

## 54. How do you preserve data lineage?

Include source file name, delivery ID, ingestion timestamp, source system and batch identifier in metadata associated with each record or batch. This allows downstream data to be traced to its origin.

## 55. What should be logged for every file?

At minimum, record file identity, discovered time, start and end time, status, rows read, accepted and rejected counts, and error details. Avoid logging sensitive record values unnecessarily.

---

# 4. Design and architecture questions

## 56. Design a file ingestion pipeline for an e-commerce company.

I would first agree a delivery contract with each source: file naming, format, schema, delivery frequency, completion signal and retention. Files would arrive in a landing folder. The pipeline would discover only eligible files, claim each file safely, validate name and structure, parse records, apply row-level rules, write accepted data to a staging destination, record rejected records, and persist processing status. Successful source files would be archived. The pipeline would include duplicate protection, metrics, alerting and reconciliation.

## 57. How would you handle multiple source systems?

I would use a common ingestion framework with source-specific configuration. Each source can have its own naming convention, schema and validation rules, while shared processing stages handle discovery, metadata, error reporting and archival. I would preserve source identifiers in the output.

## 58. Would you use pandas or the csv module?

For small and medium tabular data, Pandas can be productive for transformations and analysis. The csv module is part of Python's standard library and is useful for simple row-by-row processing with low memory overhead. For very large files, I would use streaming or chunked processing rather than loading everything into a DataFrame at once.

## 59. When would you use Parquet instead of CSV?

Parquet is a columnar format that supports typed columns and efficient column-oriented analytical reads. It is often useful in data lakes and analytics workloads. CSV is simple and widely supported but stores values as text and can be larger or slower for analytical scans.

## 60. How do you monitor file ingestion?

Track files discovered, files processed, files rejected, processing duration, rows read, accepted and rejected, duplicate deliveries, file arrival delays and failures. Alert on missing expected deliveries, repeated failures or unusual volume changes.

## 61. How would you handle schema evolution?

Version the data contract and define compatibility rules. Additive optional columns may be accepted if downstream consumers can handle them. Renamed, removed or type-changed fields may require a controlled migration. Never silently reinterpret a field whose meaning has changed.

## 62. How do you handle sensitive data in files?

Restrict access to landing, archive and rejected folders; encrypt data at rest and in transit according to policy; avoid logging personal values; apply retention and deletion requirements; and use least-privilege access for pipeline identities.

## 63. What is the role of a quarantine area?

A quarantine area isolates files or records that require investigation. It prevents bad input from repeatedly entering the normal pipeline while preserving evidence for diagnosis and correction.

## 64. What is the difference between archive and quarantine?

Archive stores successfully processed source files for retention and audit. Quarantine stores files or records that cannot be safely processed until investigated or corrected.

## 65. How would you support reprocessing?

Preserve the original source file, record the processing version and outcome, and provide a controlled re-run mechanism. Reprocessing should not create duplicates or overwrite newer data unintentionally. Keep the original and corrected delivery identities distinguishable.

---

# 5. Project defence: Multi-File Ingestion System

## 66. Explain your Day 31 project.

The project scans an incoming directory for order CSV files, validates file structure and record values, collects valid rows, writes invalid rows to a rejection report, archives processed files and records each file's status in a manifest.

## 67. Why did you use pathlib?

pathlib provides readable path handling and methods to inspect and discover files. It makes the project easier to run with relative paths across different operating systems.

## 68. Why did you use DictReader?

The order file has named columns. DictReader returns each row as a dictionary, so the code can access values by names such as order_id and amount instead of remembering numeric column positions.

## 69. Why did you validate headers before reading records?

The pipeline depends on specific field names. If the header is wrong, row validation may access missing keys or interpret the wrong values. File structure must be checked before processing records.

## 70. Why did you separate file-level and row-level errors?

A file-level error can make the whole file unusable. A row-level error affects one record. Separating them allows a defined partial-acceptance policy and more useful error reporting.

## 71. Why did you add source_file to the output?

It preserves basic lineage. If a downstream user finds a problematic order, the source file can be identified and investigated.

## 72. Why did you create a manifest?

The manifest records which files were processed and the number of rows read, accepted and rejected. It provides an operational history.

## 73. What is one limitation of your project?

The classroom version keeps accepted and rejected records in memory and writes a consolidated output file. For very large datasets, I would stream rows to output in batches. The file move and manifest write are also separate operations, so production recovery logic is needed.

## 74. How would you make the project safe for multiple workers?

I would add a claim step that atomically moves each ready file into a processing location or uses a database-backed work queue. Workers would update durable processing state and use file identity to prevent duplicate work.

## 75. What would you improve before production?

I would add a formal source contract, completion markers, robust duplicate detection, streaming output, durable metadata storage, retry and recovery behavior, structured logs, alerts, schema versioning, security controls and reconciliation.

---

# 6. Practical interview exercise

A company receives 100 CSV files every day. Each file has 10,000 rows. Five files have missing columns, and 200 rows across the remaining files have invalid amounts.

## Questions

1. How many files are discovered?
2. How many files pass file-level schema validation?
3. How many rows are present in the 95 structurally valid files?
4. If all 200 invalid rows are rejected individually, how many rows are accepted?
5. What information should the manifest contain?
6. Where should the five invalid files go?
7. Where should the 200 invalid rows be recorded?
8. How would you avoid processing the same file twice?
9. How would you handle a file that is still uploading?
10. How would you prove that all 100 deliveries were accounted for?

## Answers

1. 100 files.
2. 95 files.
3. 95 × 10,000 = 950,000 rows, assuming 10,000 means data rows and not including headers.
4. 950,000 − 200 = 949,800 accepted rows.
5. File identity, source, timestamps, status, rows read, accepted and rejected counts, and error details.
6. A rejected or quarantine folder with a recorded schema failure.
7. A row-level rejection report or quarantine table with source file, row number, business key and reason.
8. Use stable delivery IDs or file fingerprints and persist processed-file state.
9. Require a source completion signal or atomic rename from temporary to ready name.
10. Reconcile expected deliveries with discovered, processed, rejected and pending deliveries, and compare row counts.

---

# 7. Interview traps

Avoid these answers:

- "I just use pandas read_csv and the ingestion is done."
- "If the file extension is CSV, it must be a valid CSV."
- "I delete the file after reading it."
- "I ignore invalid rows."
- "I reject the entire file whenever one row is invalid" without explaining the contract.
- "I use file name alone to identify duplicate content."
- "I check file size once and know the upload is complete."
- "I store every row in a Python list even if the file has hundreds of gigabytes."
- "I do not need a manifest because the script printed success."
- "I move the file and assume the whole operation is transactional."

A good junior Data Engineer explains the business rule, the failure case and how the pipeline proves what happened.

---

# 8. Day 31 interview readiness checklist

- [ ] I can define file-based ingestion.
- [ ] I can explain landing, archive, rejected and output folders.
- [ ] I can scan directories with pathlib.
- [ ] I understand iterdir, glob and rglob.
- [ ] I can filter by extension and naming convention.
- [ ] I can read CSV with reader and DictReader.
- [ ] I can read JSON with json.load.
- [ ] I can validate file structure and individual rows.
- [ ] I can explain malformed and incomplete files.
- [ ] I understand archive, quarantine and manifest.
- [ ] I can explain duplicate file versus duplicate record.
- [ ] I can discuss idempotency and safe reprocessing.
- [ ] I can explain why the classroom project needs production improvements.
- [ ] I can defend the Multi-File Ingestion System end to end.

**Interview goal:** Do not stop at "I can read multiple CSV files." Explain how you discover the right files, verify their structure, protect against bad data, track processing, preserve source evidence and recover safely when something fails.
