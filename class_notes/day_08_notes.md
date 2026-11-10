# Day 8: File Operations

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** File Processing Manager  
**Learning time:** 2–3 hours

## 1. Learning outcomes

By the end of this lesson, you can:

- Explain why file handling is part of a data pipeline.
- Build file and folder paths safely with `pathlib`.
- Create, read, overwrite and append text files.
- Copy, rename and move files.
- Inspect folders and find files by pattern.
- Process a file line by line when it is too large to load at once.
- Build a small file-processing workflow and verify its output.

## 2. A real data-engineering situation

A retail company receives a daily order export from its stores. A simple file pipeline must:

1. Find the incoming file.
2. Check that it exists and is not empty.
3. Read its records.
4. Save a cleaned copy to an output folder.
5. Move the original file to an archive folder.

This is the workflow we will build gradually. The examples use local files, so students can run them without a database or internet connection.

**Important:** Run the cells in order in one Jupyter notebook, or save the code into a single `.py` file and run it from the project folder. Later examples reuse folders and variables created earlier.

## 3. Prepare the project folder

### 3.1 Understand the folder layout

We will create this structure:

```text
file_processing_manager/
    input/
    output/
    archive/
```

- `input`: files arriving from a source system.
- `output`: processed files.
- `archive`: original files retained after successful processing.

### 3.2 Create the folders

**Task:** Create the project directories if they do not already exist.

```python
from pathlib import Path

project_folder = Path("file_processing_manager")
input_folder = project_folder / "input"
output_folder = project_folder / "output"
archive_folder = project_folder / "archive"

input_folder.mkdir(parents=True, exist_ok=True)
output_folder.mkdir(parents=True, exist_ok=True)
archive_folder.mkdir(parents=True, exist_ok=True)

print("Project folders are ready")
print(input_folder.exists())
print(output_folder.exists())
print(archive_folder.exists())
```

**Expected output**

```text
Project folders are ready
True
True
True
```

**Line-by-line explanation**

- `Path("file_processing_manager")` represents a path. It does not create a folder by itself.
- The `/` operator joins path parts. It works on Windows, Linux and macOS.
- `mkdir()` creates the directory.
- `parents=True` creates missing parent directories.
- `exist_ok=True` prevents an error if the folder already exists.
- `exists()` checks whether the path currently exists.

**Interpretation:** All three directories are available. Running the cell again is safe because `exist_ok=True` is used.

## 4. Understand file paths

### 4.1 Current working directory

```python
print(Path.cwd())
```

**Expected output**

The absolute path of the folder from which Python is currently running. It differs between machines.

**Why it matters:** A relative path such as `input/orders.txt` is resolved from the current working directory. If Python is launched from a different folder, the same relative path may point somewhere else.

### 4.2 Absolute and relative paths

| Type | Example | Meaning |
|---|---|---|
| Relative | `input/orders.txt` | Location relative to the current working directory |
| Absolute | `D:/training/project/input/orders.txt` | Full location from a drive/root |

Use `Path.cwd()` to understand where a notebook or script is running. Avoid hard-coding a machine-specific absolute path into a reusable project.

### 4.3 Inspect a path

```python
sample_path = input_folder / "orders.txt"

print(sample_path)
print(sample_path.parent)
print(sample_path.name)
print(sample_path.suffix)
```

**Expected output**

```text
file_processing_manager/input/orders.txt
file_processing_manager/input
orders.txt
.txt
```

The displayed path separator may differ by operating system.

## 5. Create a sample source file

Before reading a file, we need one. We will create a small order file.

```python
source_file = input_folder / "orders.txt"

with source_file.open("w", encoding="utf-8") as file:
    file.write("order_id,customer,amount,status\n")
    file.write("1001,Ravi,1200,PAID\n")
    file.write("1002,Anu,850,PENDING\n")
    file.write("1003,Kiran,640,PAID\n")

print("Source file created:", source_file.exists())
```

**Expected output**

```text
Source file created: True
```

**Explanation**

- `open("w")` opens the file in write mode.
- If the file does not exist, Python creates it.
- If the file already exists, write mode replaces its previous content.
- `encoding="utf-8"` makes text encoding explicit.
- `with` automatically closes the file after the indented block finishes.
- `\n` moves the next record to a new line.

**Common mistake:** Re-running this cell overwrites the file with the three sample rows. This is expected because the mode is `w`.

## 6. Read a complete text file

```python
with source_file.open("r", encoding="utf-8") as file:
    content = file.read()

print(content)
```

**Expected output**

```text
order_id,customer,amount,status
1001,Ravi,1200,PAID
1002,Anu,850,PENDING
1003,Kiran,640,PAID
```

**Explanation**

- `r` means read mode. The file must already exist.
- `read()` reads the entire file into one string.
- `content` contains the text after the file is closed.

**When to use:** Small configuration files or small text files.

**When not to use:** A multi-gigabyte log file. Reading the entire file at once can consume too much memory.

## 7. Read one line at a time

```python
with source_file.open("r", encoding="utf-8") as file:
    for line in file:
        print(line.strip())
```

**Expected output**

```text
order_id,customer,amount,status
1001,Ravi,1200,PAID
1002,Anu,850,PENDING
1003,Kiran,640,PAID
```

**Explanation**

- The file object is iterable.
- The loop reads one line at a time.
- `strip()` removes surrounding whitespace, including the newline character.
- The loop does not need to store the whole file in a separate list.

**Data-engineering connection:** Line-by-line reading is a useful pattern for large text files. It reduces the amount of file content held in memory at one time.

## 8. Understand file modes

| Mode | Behavior | Important caution |
|---|---|---|
| `r` | Read existing file | Fails if file is missing |
| `w` | Create or overwrite | Existing content is erased |
| `a` | Add content at the end | Creates file if missing |
| `x` | Create a new file only | Fails if file already exists |
| `rb` | Read binary bytes | Used for non-text files |
| `wb` | Write binary bytes | Replaces existing bytes |

### 8.1 Append without deleting existing content

```python
with source_file.open("a", encoding="utf-8") as file:
    file.write("1004,Meena,1500,PAID\n")

print(source_file.read_text(encoding="utf-8"))
```

**Expected output**

```text
order_id,customer,amount,status
1001,Ravi,1200,PAID
1002,Anu,850,PENDING
1003,Kiran,640,PAID
1004,Meena,1500,PAID
```

**Interpretation:** Append mode preserves the previous records and adds the new line.

### 8.2 Create a file only if it does not exist

```python
control_file = output_folder / "run_status.txt"

with control_file.open("x", encoding="utf-8") as file:
    file.write("STARTED")

print(control_file.read_text(encoding="utf-8"))
```

**Expected output on the first run**

```text
STARTED
```

If you run this cell again, Python raises `FileExistsError`. That is the purpose of exclusive-create mode. For repeatable notebook execution, delete the file first or use a different filename.

## 9. Check files before processing

```python
print("Exists:", source_file.exists())
print("Is file:", source_file.is_file())
print("Size in bytes:", source_file.stat().st_size)
```

**Expected output**

```text
Exists: True
Is file: True
Size in bytes: <machine-independent file size for the sample text>
```

The byte count is determined by the exact text encoding and line endings on the machine. Do not write business logic that assumes a particular byte count for this example.

A useful empty-file check:

```python
if source_file.exists() and source_file.stat().st_size > 0:
    print("File is ready for processing")
else:
    print("File is missing or empty")
```

**Expected output**

```text
File is ready for processing
```

**Interpretation:** A successful existence check is not enough. A file can exist and still contain no data.

## 10. Copy a file

```python
import shutil

backup_file = output_folder / "orders_backup.txt"
shutil.copy2(source_file, backup_file)

print("Backup created:", backup_file.exists())
print("Backup content matches:", backup_file.read_text(encoding="utf-8") == source_file.read_text(encoding="utf-8"))
```

**Expected output**

```text
Backup created: True
Backup content matches: True
```

**Explanation**

- `shutil` provides high-level file operations.
- `copy2()` copies the file and attempts to preserve metadata such as modification time.
- The equality check verifies the copied text is identical.

**When useful:** Keep a source copy before a risky transformation. A backup is not a substitute for a proper retention and recovery policy.

## 11. Rename and move a file

### 11.1 Rename in the same folder

```python
renamed_file = output_folder / "orders_backup_before_load.txt"
backup_file.rename(renamed_file)

print("Old name exists:", backup_file.exists())
print("New name exists:", renamed_file.exists())
```

**Expected output**

```text
Old name exists: False
New name exists: True
```

### 11.2 Move the source into the archive

```python
archived_file = archive_folder / "orders_source.txt"
shutil.move(str(source_file), str(archived_file))

print("Source still in input:", source_file.exists())
print("Archived copy exists:", archived_file.exists())
```

**Expected output**

```text
Source still in input: False
Archived copy exists: True
```

**Important operational rule:** Archive a source file only after the output has been written and verified. If processing fails, retain the original in the input or move it to a clearly named rejected folder.

## 12. List files in a directory

```python
for item in project_folder.rglob("*"):
    if item.is_file():
        print(item)
```

**Expected output**

The file paths created so far, including the output and archive files. The exact order is not guaranteed.

- `glob("*.txt")` finds matching files in one directory.
- `rglob("*.txt")` searches recursively through subdirectories.
- `is_file()` excludes folders.

## 13. Process multiple text files

Create two incoming files:

```python
(input_folder / "store_a.txt").write_text(
    "order_id,amount\n2001,500\n2002,700\n",
    encoding="utf-8"
)

(input_folder / "store_b.txt").write_text(
    "order_id,amount\n2003,900\n",
    encoding="utf-8"
)

print(sorted(path.name for path in input_folder.glob("*.txt")))
```

**Expected output**

```text
['store_a.txt', 'store_b.txt']
```

Read each file and count its data rows:

```python
for file_path in sorted(input_folder.glob("*.txt")):
    lines = file_path.read_text(encoding="utf-8").splitlines()
    data_rows = lines[1:]
    print(file_path.name, "->", len(data_rows), "records")
```

**Expected output**

```text
store_a.txt -> 2 records
store_b.txt -> 1 records
```

**Explanation:** The first line is a header, so `lines[1:]` selects the data rows. This simple example assumes each file has one header row and no blank lines. Real CSV parsing should use the `csv` module, covered in Day 9.

## 14. Large-file processing pattern

A file can be too large to fit comfortably in memory. Compare these approaches:

| Approach | Memory behavior |
|---|---|
| `file.read()` | Loads all text into memory |
| `file.readlines()` | Loads all lines into a list |
| `for line in file` | Reads iteratively, one line at a time |

Create a repeatable demonstration file:

```python
large_sample = output_folder / "large_sample.txt"

with large_sample.open("w", encoding="utf-8") as file:
    for number in range(1, 6):
        file.write(f"record_{number}\n")

print("Sample rows written:", 5)
```

**Expected output**

```text
Sample rows written: 5
```

Iterate through it:

```python
processed_count = 0

with large_sample.open("r", encoding="utf-8") as file:
    for line in file:
        record = line.strip()
        if record:
            processed_count += 1
            print("Processing:", record)

print("Total processed:", processed_count)
```

**Expected output**

```text
Processing: record_1
Processing: record_2
Processing: record_3
Processing: record_4
Processing: record_5
Total processed: 5
```

This pattern avoids building a list containing every line. It does not make processing infinitely scalable: each record still needs to be processed, and any data retained in other lists or dictionaries still consumes memory.

## 15. Mini-project: File Processing Manager

### 15.1 Business requirement

A store sends a daily text export. The manager must:

1. Verify the incoming file exists.
2. Read the file.
3. Remove blank lines.
4. Write the cleaned file into the output folder.
5. Verify the output exists and contains the expected number of lines.
6. Archive the source only after verification.

### 15.2 Create a fresh incoming file

```python
incoming_file = input_folder / "daily_orders.txt"

incoming_file.write_text(
    "order_id,amount,status\n"
    "3001,1200,PAID\n"
    "\n"
    "3002,800,PENDING\n"
    "3003,450,PAID\n",
    encoding="utf-8"
)

print("Incoming file ready")
```

**Expected output**

```text
Incoming file ready
```

### 15.3 Validate the source

```python
if not incoming_file.exists():
    raise FileNotFoundError("Daily order file was not found")

if incoming_file.stat().st_size == 0:
    raise ValueError("Daily order file is empty")

print("Source checks passed")
```

**Expected output**

```text
Source checks passed
```

### 15.4 Read and clean the lines

```python
with incoming_file.open("r", encoding="utf-8") as file:
    source_lines = file.readlines()

clean_lines = []

for line in source_lines:
    if line.strip():
        clean_lines.append(line)

print("Original line count:", len(source_lines))
print("Clean line count:", len(clean_lines))
```

**Expected output**

```text
Original line count: 5
Clean line count: 4
```

The blank line is removed. The header remains because it is not blank.

### 15.5 Write the processed output

```python
processed_file = output_folder / "daily_orders_cleaned.txt"

with processed_file.open("w", encoding="utf-8") as file:
    file.writelines(clean_lines)

print(processed_file.read_text(encoding="utf-8"))
```

**Expected output**

```text
order_id,amount,status
3001,1200,PAID
3002,800,PENDING
3003,450,PAID
```

### 15.6 Verify before archiving

```python
output_lines = processed_file.read_text(encoding="utf-8").splitlines()

if processed_file.exists() and len(output_lines) == len(clean_lines):
    print("Output verification passed")
else:
    raise RuntimeError("Output verification failed")
```

**Expected output**

```text
Output verification passed
```

### 15.7 Archive the original

```python
final_archive_file = archive_folder / "daily_orders_processed.txt"
shutil.move(str(incoming_file), str(final_archive_file))

print("Source archived:", final_archive_file.exists())
print("Input file remains:", incoming_file.exists())
```

**Expected output**

```text
Source archived: True
Input file remains: False
```

### 15.8 Final project summary

```python
print("PROJECT SUMMARY")
print("Processed output:", processed_file)
print("Output rows including header:", len(output_lines))
print("Archived source:", final_archive_file)
```

**Expected output**

```text
PROJECT SUMMARY
Processed output: file_processing_manager/output/daily_orders_cleaned.txt
Output rows including header: 4
Archived source: file_processing_manager/archive/daily_orders_processed.txt
```

The path display may use different separators on Windows. The important checks are that the processed file exists, the expected records are present, and the source is retained in the archive.

## 16. Common mistakes and troubleshooting

| Problem | Likely reason | What to do |
|---|---|---|
| `FileNotFoundError` | Wrong path or missing file | Print `Path.cwd()` and check the path |
| `PermissionError` | Process lacks access or file is locked | Close the other application and check permissions |
| Output unexpectedly empty | File opened in write mode before reading | Read first, then write to a separate output file |
| Old content disappeared | Used `w` instead of `a` | Choose the correct mode |
| Wrong folder is being searched | Relative path resolved from another working directory | Use `Path.cwd()` and construct paths deliberately |
| Encoding error | File encoding differs from assumed encoding | Confirm source encoding; do not blindly ignore invalid bytes |
| File moved too early | Archived before output verification | Verify output before moving the source |

## 17. Production-minded file handling

- Do not overwrite raw source files during transformation.
- Keep raw, processed and rejected files separate.
- Use deterministic file names and include a batch date or unique run ID where appropriate.
- Verify output row counts and required columns before archiving.
- Avoid loading huge files completely into memory.
- Use CSV/JSON-specific parsers instead of manually splitting structured records.
- Do not log passwords, personal data or confidential record contents.
- Decide what happens when the same input is delivered twice.
- Make archive and retry behavior explicit.

## 18. Day 8 revision

- `Path` represents file-system paths.
- `mkdir()` creates directories.
- `read()` loads all text; iterating over a file reads line by line.
- `w` overwrites, `a` appends, `x` creates only if absent.
- `shutil.copy2()` copies a file; `Path.rename()` renames it; `shutil.move()` moves it.
- Verify a processed output before archiving the source.
- Day 9 builds on this lesson by reading structured CSV, JSON, XML and Parquet data.
