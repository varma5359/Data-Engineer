# Day 8 — File Operations and Directory Management
**Phase 2: Python for Data Engineering**

## Outcomes
Read/write files safely, handle paths, inspect folders, and avoid common ingestion errors.

## 1. Paths
Use `pathlib` rather than manually joining strings. Paths differ across Windows/Linux.

```python
from pathlib import Path
data_dir = Path("data")
file_path = data_dir / "orders.txt"
print(file_path)
print(file_path.suffix)
```
Output depends on OS path separator; suffix is `.txt`.

## 2. Read a text file
Create `orders.txt` containing:
```
101
102
103
```
```python
from pathlib import Path
path = Path("orders.txt")
with path.open("r", encoding="utf-8") as file:
    lines = file.readlines()
print(lines)
```
Output: `['101\\n', '102\\n', '103\\n']`

The `with` block closes the file automatically. Encoding should be explicit.

## 3. Write and append
```python
from pathlib import Path
path = Path("processed.txt")
with path.open("w", encoding="utf-8") as file:
    file.write("orders_processed=3\n")
with path.open("a", encoding="utf-8") as file:
    file.write("status=success\n")
print(path.read_text(encoding="utf-8"))
```
Output:
`orders_processed=3`
`status=success`

Write mode replaces existing content; append mode adds content. Confirm behavior before using on production files.

## 4. Directory discovery
```python
from pathlib import Path
folder = Path("incoming")
folder.mkdir(parents=True, exist_ok=True)
for path in folder.glob("*.csv"):
    print(path.name, path.stat().st_size)
```
Lists CSV files and sizes. Use `rglob` for recursive discovery only when intended.

## 5. Safe file processing pattern
- Identify files and stable naming.
- Confirm file is complete (avoid reading partial upload).
- Read with expected encoding/schema.
- Record source filename and ingestion timestamp.
- Move/archive only after successful validation and load.
- Keep failed inputs for investigation under controlled access.

## 6. Mini-project — File Processing Manager
Scan an input folder, list supported files, count them, and write a processing summary. Do not delete source files automatically.

## Mistakes
- Using `w` when append was intended.
- Forgetting encoding.
- Hardcoding Windows paths.
- Reading huge files entirely into memory.
- Processing a file while it is still being uploaded.
- Deleting source before confirming destination success.

## Revision
`pathlib` handles paths; `with` manages file lifetime; mode `r/w/a` means read/replace/append.