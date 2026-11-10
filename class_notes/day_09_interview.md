# Day 9 Interview Preparation: Data Formats and File Processing

**Mini-project:** Multi-Format Data Reader

## 1. Core questions

1. **What is serialization?** Converting an in-memory object into a representation that can be stored or transmitted.
2. **What is deserialization?** Parsing a stored or transmitted representation back into an in-memory object.
3. **Why use CSV?** It is simple and widely supported for flat tabular exchange, but it does not preserve rich types reliably.
4. **Why use JSON?** It represents objects, arrays and nested structures and is common in APIs.
5. **Where is XML used?** Enterprise integrations and structured document exchange, especially where tag hierarchies and schemas are established.
6. **Why use Parquet?** Columnar storage supports efficient analytical reads, compression and typed data.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| `json.load` vs `json.loads` | Read from file object vs parse string |
| `json.dump` vs `json.dumps` | Write to file object vs return JSON string |
| CSV vs Parquet | Text row exchange vs typed columnar analytics |
| XML element vs attribute | Child content node vs property on an element |
| Row vs column storage | Records stored together vs values organized by column |

## 3. Practical questions

7. **How do you preserve leading zeros in an ID?** Treat the identifier as text and prevent numeric type inference.
8. **How do you handle nested JSON arrays?** Preserve parent identifiers and decide whether each array element becomes a child row or remains nested.
9. **What is predicate pushdown?** Applying filters as close to the data scan as possible so unnecessary data can be skipped when supported.
10. **Why can small Parquet files be a problem?** File-open and metadata overhead can dominate distributed processing; compaction may help.

## 4. Scenario questions

### Scenario A: CSV amount contains "N/A"
Do not blindly convert it to zero. Define whether it is missing, invalid or a business-specific code; quarantine or map according to the data contract.

### Scenario B: JSON field is absent in some events
Determine whether it is optional. Validate required fields, handle optional values explicitly and document schema evolution.

### Scenario C: Parquet query scans every column
Inspect selected columns, filters, file layout, partitioning and query-engine support for column pruning/predicate pushdown.

## 5. Troubleshooting

- JSONDecodeError: malformed/truncated JSON or wrong input.
- CSV columns shift: delimiter/quoting issue.
- XML lookup returns None: wrong tag, namespace or optional element.
- Parquet engine error: install compatible engine in active environment.
- Schema mismatch: compare actual file schema with the source contract.

## 6. Follow-up questions

- What is schema evolution?
- What is a nullable field?
- What is a columnar format?
- What is a partition key?
- How do you validate file completeness?
- How do you handle unknown fields?

## 7. Mini-project defence

Explain how the same logical order records are represented in CSV, JSON and Parquet, what information each format preserves, how types are normalized and how you would detect incomplete or incompatible files.
