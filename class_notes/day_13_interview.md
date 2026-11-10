# Day 13 Interview Preparation: Python Database Connectivity

**Mini-project:** Database Record Extractor

## 1. Core questions

1. **What is a database driver?** A library that allows an application to communicate with a database.
2. **What is a cursor?** An object used to execute SQL and retrieve query results.
3. **What is a transaction?** A group of operations managed as a unit with commit or rollback behavior.
4. **Why parameterize SQL?** To safely bind values and avoid SQL injection caused by string concatenation.
5. **What is connection pooling?** Reusing a managed set of connections instead of repeatedly opening new ones.

## 2. Difference-based questions

| Compare | Answer |
|---|---|
| fetchone vs fetchall | One row vs all remaining rows |
| commit vs rollback | Persist transaction vs undo uncommitted changes |
| cursor vs connection | Executes/fetches statements vs database session |
| offset vs keyset pagination | Skip N rows vs continue from ordered key values |
| row-by-row vs bulk load | Many individual operations vs grouped transfer |

## 3. Practical questions

6. **How do you prevent SQL injection?** Use driver parameter binding; never concatenate untrusted values into SQL.
7. **How do you extract a large table?** Use bounded batches, keyset pagination or a server-side cursor and monitor source load.
8. **When should a checkpoint advance?** Only after extracted data has been safely committed to the destination.
9. **How do you protect DB credentials?** Secret management, least-privilege accounts and no credentials in source control or logs.

## 4. Scenario questions

### Scenario A: Connection works locally but fails in deployment
Compare network route, DNS, firewall, environment variables, secret permissions, SSL requirements and service-account grants.

### Scenario B: Extraction duplicates records after retry
Use a stable key, idempotent target merge and checkpoint tied to committed output.

### Scenario C: Database slows during extraction
Inspect query plan, filters, indexes, batch size and source load. Coordinate with the source owner; use replicas or change windows if approved.

## 5. Troubleshooting
- Authentication failure: validate account, secret and permissions.
- Connection timeout: check network, host and database availability.
- SQL syntax error: test query in the target database dialect.
- Too much memory: avoid fetchall on huge result sets.
- Missing changes: inspect watermark, timestamp precision and late updates.

## 6. Follow-up questions
- What is ACID?
- What is an upsert?
- Why use staging tables?
- What is a transaction isolation level?
- How do you handle deleted source rows?
- How do you avoid long-running transactions?

## 7. Mini-project defence
Explain connection configuration, parameter binding, query scope, batch strategy, transaction handling, checkpointing and reconciliation. Be prepared to explain why a successful query is not the same as a complete incremental extraction.
