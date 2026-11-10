# Day 9 Interview Preparation: Data Formats and File Processing

**Mini-project:** Multi-Format Data Reader

## 1. Core questions

1. **What is a data format?**  
   A defined representation of data that allows systems to store, exchange and parse information.

2. **What is serialization?**  
   Converting an in-memory object into a representation suitable for storage or transmission.

3. **What is deserialization?**  
   Parsing stored or transmitted data back into an in-memory representation.

4. **Why are CSV values commonly read as strings?**  
   CSV is plain text and does not reliably store column data types. The application must convert and validate values.

5. **Where is JSON commonly used?**  
   REST APIs, configuration files and nested application data.

6. **What is the difference between JSON object and array?**  
   An object is a key-value structure; an array is an ordered collection of values.

7. **What is XML?**  
   A markup format that represents hierarchical data using elements and attributes.

8. **What is Parquet?**  
   A columnar storage format designed for efficient analytical workloads and typed data.

9. **Why is Parquet useful in a data lake?**  
   It supports typed columns, compression and efficient column selection for many analytical queries.

10. **Does successful parsing guarantee valid business data?**  
    No. Parsing checks representation syntax; schema and business validation are separate steps.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| CSV vs JSON | Tabular text vs object/array representation |
| JSON vs XML | JSON uses objects/arrays; XML uses nested elements and attributes |
| CSV vs Parquet | Text-oriented row exchange vs typed columnar analytics format |
| `dump()` vs `dumps()` | Write to file object vs return JSON string |
| `load()` vs `loads()` | Parse from file object vs parse JSON string |
| Element vs attribute in XML | Nested node vs metadata attached to an element |
| Parsing vs validation | Decode structure vs verify expected schema/rules |

## 3. Practical implementation questions

11. **How do you write a CSV with dictionaries?**  
    Use `csv.DictWriter`, define `fieldnames`, call `writeheader()` and then `writerows()`.

12. **How do you read a CSV as dictionaries?**  
    Use `csv.DictReader`; each row is returned as a dictionary keyed by column name.

13. **How do you convert a CSV amount to a number?**  
    Check for missing/invalid text, then convert with `float()` or `Decimal` depending on precision requirements.

14. **How do you read JSON from a file?**  
    Open the file and pass its file object to `json.load()`.

15. **How do you catch invalid JSON?**  
    Catch `json.JSONDecodeError`, record the source and error, and route the file for investigation.

16. **How do you read an XML attribute?**  
    Use `element.get("attribute_name")`.

17. **How do you read an XML child element's text?**  
    Use `element.findtext("child_name")`, then check missing values and convert types.

18. **What library can Pandas use for Parquet?**  
    PyArrow or Fastparquet. The chosen engine must be installed in the active Python environment.

## 4. Scenario-based questions

### Scenario 1: An amount column is read as text

**Problem:** A CSV amount column cannot be added because its values are strings.

**Strong answer:**
- Explain that CSV stores text values.
- Inspect missing and malformed values.
- Convert valid values to a numeric type.
- Reject or quarantine invalid records instead of silently changing them.

### Scenario 2: An API sends nested customer details

**Problem:** A response contains customer objects inside each order.

**Strong answer:** JSON naturally represents nested objects and arrays. Parse with `json.load()` or `json.loads()`, then validate the expected keys and types.

### Scenario 3: An XML field is missing

**Problem:** Some orders do not contain an optional `discount` element.

**Strong answer:** Do not assume every element exists. Use `find()` or `findtext()`, handle missing values according to the data contract, and distinguish optional from required fields.

### Scenario 4: A large analytics table is stored as CSV

**Problem:** Queries repeatedly scan only three columns from a wide dataset.

**Strong answer:** Consider converting the validated dataset to Parquet. Columnar storage can reduce unnecessary column reads and often improves analytical I/O. Measure actual workload performance rather than assuming it is always faster.

### Scenario 5: A file is named `orders.json` but contains invalid JSON

**Problem:** The extension looks correct, but parsing fails.

**Strong answer:** Treat the content as untrusted until parsed. Catch the parsing error, record the file and error location, quarantine the source and notify the owner. The extension does not prove the contents are valid.

## 5. Troubleshooting questions

19. **Why should you avoid `split(",")` for CSV?**  
    Quoted fields can contain commas and newlines. Use the CSV parser.

20. **Why might Parquet reading fail after installation?**  
    The package may have been installed into a different Python environment from the active notebook kernel.

21. **What happens if a JSON key is missing?**  
    Direct indexing may raise `KeyError`. Check required keys or use `.get()` for optional keys.

22. **Why might an XML lookup return `None`?**  
    The tag may be absent, misspelled or located at a different nesting level.

## 6. Mini-project explanation

**Question:** Explain the Multi-Format Data Reader.

**Strong answer:**

"I created the same retail order records in CSV, JSON, XML and Parquet. I wrote and read each format using the appropriate parser, converted CSV text fields to numeric types, extracted XML attributes and child elements, and used Pandas with PyArrow for Parquet. Finally, I compared record counts and checked expected columns. I learned that parsing is not the same as validation and that each format has different trade-offs."

## 7. Follow-up questions

- Which format would you choose for an API response and why?
- Which format would you choose for analytical storage and why?
- How do you handle schema changes?
- How do you preserve leading zeros in identifiers?
- How do you detect missing records between two formats?
- Why should a pipeline retain the original source?

## 8. Quick revision

- CSV is simple tabular text; explicit type conversion is usually required.
- JSON is common for APIs and nested structures.
- XML uses elements and attributes.
- Parquet is a typed columnar analytics format.
- A parser can succeed even when business values are wrong.
- Validate schema, types, required fields and business rules after parsing.


---

## 7. Follow-up interview questions

16. What is serialization? Converting an in-memory object into a representation that can be stored or transmitted.
17. What is deserialization? Reconstructing an in-memory representation from stored or transmitted data.
18. Why do CSV values often need conversion? CSV is text and parsers commonly return field values as strings.
19. What does DictReader do? It maps each row to a dictionary using header names as keys.
20. load() versus loads()? load reads JSON from a file-like object; loads parses a JSON string.
21. How does a JSON array map into Python? Usually to a list; JSON objects map to dictionaries.
22. How is XML different from JSON? XML uses elements and attributes; JSON uses objects, arrays and primitive values.
23. What is columnar storage? Values are organized by column, useful when analytical queries read selected columns.
24. Why is Parquet useful in data lakes? It supports typed columnar storage, compression and efficient column projection.
25. Is Parquet always faster than CSV? No. It depends on data size, query pattern, engine and conversion overhead.
26. What is schema drift? An unexpected change in fields, types or structure from a source.
27. How do you validate a parsed file? Check required fields, types, allowed values, row counts and business rules.
28. Why retain raw source files? They support replay, auditing and debugging, subject to retention and privacy rules.
29. Can an extension guarantee file format? No. Validate content and parser behavior.
30. How compare CSV and Parquet records? Normalize types and schema, compare counts and keys, then reconcile business totals.

## 8. Scenario questions

An API sends nested JSON but the warehouse expects flat columns. Parse it, inspect nested fields, flatten only required attributes, convert types, validate, then load.

A CSV amount contains the text unknown. Preserve the raw value, attempt controlled conversion, and route invalid rows to a rejected-record path with a reason. Do not silently replace it with zero.

An analytics job reads only three columns from a wide dataset. Parquet may reduce unnecessary I/O because an engine can project selected columns.

## 9. Mini-project defense

- Which format was easiest for a human to inspect?
- Which format supports nested records naturally?
- Why did CSV require numeric conversion?
- What dependency enabled Parquet support?
- How did you verify conversion preserved records and totals?

## 10. Readiness rubric

| Skill | Ready when the student can… |
|---|---|
| CSV | Read header-based rows and explain string conversion |
| JSON | Parse objects and arrays into Python structures |
| XML | Read attributes and child element text |
| Parquet | Explain columnar storage and dependency requirements |
| Validation | Compare schema, types, counts and totals |
