# Day 8: File Operations

**Phase:** 2 — Python for Data Engineering  
**Mini-project:** File Processing Manager  
**Learning time:** 2–3 hours

## 1. Learning outcomes

- Read and write text files safely.
- Understand file modes and context managers.
- Work with paths and directories using pathlib.
- Process multiple files in a folder.
- Handle large files without loading everything into memory.
- Build a simple file processing workflow.

## 2. Why file handling matters

Data arrives as daily exports, supplier files, application logs and configuration documents. A pipeline must discover files, verify them, read their contents, process records and write outputs safely.

A file-processing workflow:

`Discover -> Check -> Read -> Validate -> Transform -> Write -> Archive`

## 3. File paths

A path identifies a file or folder.

- Absolute path: complete path from a drive/root.
- Relative path: path from the current working directory.

```python
from pathlib import Path

project_folder = Path.cwd()
data_folder = project_folder / "data"
input_file = data_folder / "orders.txt"

print(project_folder)
print(data_folder)
print(input_file)
```

Use `pathlib` rather than manually joining path strings. It handles platform-specific separators.

## 4. Check file and directory existence

```python
from pathlib import Path

data_folder = Path("data")
input_file = data_folder / "orders.txt"

print(data_folder.exists())
print(data_folder.is_dir())
print(input_file.is_file())
```

Create a directory:

```python
output_folder = Path("output")
output_folder.mkdir(parents=True, exist_ok=True)

print(output_folder.exists())
```

`parents=True` creates missing parent folders. `exist_ok=True` avoids an error when the directory already exists.

## 5. File modes

| Mode | Meaning |
|---|---|
| `r` | Read; file must exist |
| `w` | Write; creates or truncates file |
| `a` | Append to end |
| `x` | Create new file; fail if it exists |
| `b` | Binary mode modifier |
| `t` | Text mode modifier (default) |
| `+` | Read and write |

Be careful with `w`: it overwrites existing contents.

## 6. Write a text file

```python
from pathlib import Path

output_file = Path("output") / "pipeline_status.txt"

with output_file.open("w", encoding="utf-8") as file:
    file.write("Pipeline started\n")
    file.write("Orders validated\n")
    file.write("Pipeline completed\n")

print(output_file.read_text(encoding="utf-8"))
```

The `with` statement closes the file even if an error occurs. Explicit encoding avoids relying on machine defaults.

## 7. Read a complete text file

```python
from pathlib import Path

input_file = Path("output/pipeline_status.txt")

with input_file.open("r", encoding="utf-8") as file:
    content = file.read()

print(content)
```

`read()` loads the full file into memory. Use it only when file size is appropriate.

## 8. Read line by line

```python
with input_file.open("r", encoding="utf-8") as file:
    for line in file:
        print(line.strip())
```

This approach reads one line at a time and is more memory-efficient for large text files.

## 9. Read a limited number of characters

```python
with input_file.open("r", encoding="utf-8") as file:
    first_part = file.read(20)

print(first_part)
```

A limited read is useful for inspecting a sample or reading a fixed-size text segment.

## 10. Append without overwriting

```python
with output_file.open("a", encoding="utf-8") as file:
    file.write("Quality check passed\n")

print(output_file.read_text(encoding="utf-8"))
```

Append mode adds content. In production, consider whether a log or output should be appended, rotated or written to a unique run-specific file.

## 11. Process a CSV-like text file line by line

```python
from pathlib import Path

input_file = Path("orders_sample.txt")

input_file.write_text(
    "order_id,amount\n501,1200\n502,800\n",
    encoding="utf-8"
)

with input_file.open("r", encoding="utf-8") as file:
    header = next(file).strip()
    print("Header:", header)

    for line in file:
        print("Record:", line.strip())
```

This demonstrates line iteration only. For real CSV, use Python's `csv` module because quoted fields can contain commas and newlines.

## 12. List files in a directory

```python
from pathlib import Path

data_folder = Path("data")
data_folder.mkdir(exist_ok=True)

for path in data_folder.iterdir():
    if path.is_file():
        print(path.name)
```

Find CSV files recursively:

```python
for path in Path("data").rglob("*.csv"):
    print(path)
```

Do not assume every discovered file is ready or belongs to the current processing batch. Use naming rules and arrival checks.

## 13. File metadata

```python
from pathlib import Path

file_path = Path("orders_sample.txt")

print(file_path.name)
print(file_path.suffix)
print(file_path.parent)
print(file_path.stat().st_size)
```

Size is measured in bytes. File modification time can help with discovery, but it is not a reliable business watermark by itself.

## 14. Rename and move files

```python
from pathlib import Path

source = Path("orders_sample.txt")
archive_folder = Path("archive")
archive_folder.mkdir(exist_ok=True)

destination = archive_folder / "orders_sample_processed.txt"

source.rename(destination)
print(destination.exists())
```

Rename/move operations may fail across filesystems or when the destination already exists. Use unique names and verify the result.

## 15. Safe output strategy

A safer file write pattern is:
1. Write to a temporary path.
2. Flush and close the file.
3. Validate the completed output.
4. Rename it to the final path.
5. Record the run outcome.

This reduces the chance that downstream consumers read a partially written output. Exact atomicity depends on filesystem and storage behavior.

## 16. Binary files

Text files are decoded into characters. Binary files contain bytes.

```python
from pathlib import Path

binary_file = Path("sample.bin")

with binary_file.open("wb") as file:
    file.write(b"DATA")

with binary_file.open("rb") as file:
    content = file.read()

print(content)
```

Use binary mode for formats such as images, compressed archives and some serialized data. Do not decode arbitrary binary data as UTF-8.

## 17. Mini-project: File Processing Manager

Create:

```text
file_processing_manager/
    input/
    processed/
    rejected/
    main.py
```

Create a sample input file:

```python
from pathlib import Path

input_folder = Path("input")
input_folder.mkdir(exist_ok=True)

sample_file = input_folder / "daily_orders.txt"
sample_file.write_text(
    "order_id,amount,status\n"
    "501,1200,PAID\n"
    "502,800,PENDING\n"
    "503,invalid,PAID\n",
    encoding="utf-8"
)
```

Discover files:

```python
files = list(input_folder.glob("*.txt"))
print(files)
```

Read and inspect rows:

```python
with sample_file.open("r", encoding="utf-8") as file:
    header = next(file).strip()
    print(header)

    for line in file:
        print(line.strip())
```

For the exercise, split and validate the simple sample rows:

```python
valid_rows = []
rejected_rows = []

with sample_file.open("r", encoding="utf-8") as file:
    next(file)

    for line in file:
        order_id, amount, status = line.strip().split(",")

        if order_id.isdigit() and amount.isdigit() and status in ["PAID", "PENDING"]:
            valid_rows.append(line)
        else:
            rejected_rows.append(line)

print("Valid:", valid_rows)
print("Rejected:", rejected_rows)
```

Write outputs:

```python
processed_folder = Path("processed")
rejected_folder = Path("rejected")

processed_folder.mkdir(exist_ok=True)
rejected_folder.mkdir(exist_ok=True)

(processed_folder / "valid_orders.txt").write_text(
    "order_id,amount,status\n" + "".join(valid_rows),
    encoding="utf-8"
)

(rejected_folder / "invalid_orders.txt").write_text(
    "".join(rejected_rows),
    encoding="utf-8"
)

print("Files written")
```

Expected behavior: two valid rows and one rejected row. This deliberately simple parser is only suitable for the controlled sample; use `csv.DictReader` for general CSV files.

## 18. Common mistakes and troubleshooting

| Problem | Cause | Better practice |
|---|---|---|
| FileNotFoundError | Wrong path or missing file | Check current directory and path |
| PermissionError | No access or file locked | Check permissions and open handles |
| UnicodeDecodeError | Wrong encoding | Confirm source encoding |
| Empty output | Wrong mode or filter | Inspect input and processing counts |
| Output overwritten | Used `w` unintentionally | Use unique names or append deliberately |
| Memory spike | Read entire large file | Iterate in chunks/lines |
| Duplicate processing | Same file discovered again | Track file/batch identity |

## 19. Day 8 revision

- Use `pathlib.Path` for portable path operations.
- Use `with open(...)` to manage file handles.
- Choose read/write/append modes carefully.
- Stream large text files line by line.
- Use format-aware parsers for CSV, JSON and XML.
- Separate input, processed and rejected outputs.
- Never treat file discovery as proof that a file is complete or valid.
