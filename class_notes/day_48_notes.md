# Day 48: Warehouse Loading Strategies

**Phase:** 5 — Data Warehousing and Modeling  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and junior Data Engineer candidates  
**Database:** PostgreSQL  
**Topic boundary:** Class notes only. Project workbook and interview guide are separate files.

> A warehouse model is useful only when the data gets into it correctly. Today we compare full load, incremental load, append, overwrite and upsert strategies. We will build a small staging-to-warehouse load, understand when each strategy fits, and learn why checkpoints, idempotency and reconciliation must work together.

## Learning outcomes

By the end, you should be able to:
- explain full and incremental loading;
- distinguish append, overwrite and upsert;
- choose a loading strategy for a dataset;
- understand staging-to-warehouse loading;
- build a simple PostgreSQL full load;
- build a simple incremental/upsert pattern;
- understand source watermarks;
- handle updates and late-arriving rows;
- explain idempotent loading;
- understand dbt transformation workflow fundamentals;
- validate a warehouse load;
- identify common loading failures.

## Today's flow

`Source -> Raw/staging -> Validate -> Select load strategy -> Load warehouse -> Reconcile -> Save checkpoint`

---

# Part 1 — What is a warehouse load?

A warehouse load moves prepared data into a target analytical table or model.

Examples:
- load all product reference rows;
- append yesterday's click events;
- update changed customer attributes;
- insert new sales facts;
- rebuild a monthly aggregate.

A load strategy determines how the target is changed.

The strategy must match:
- source update behavior;
- target grain;
- data volume;
- history requirements;
- recovery needs;
- platform capability;
- reporting freshness.

---

# Part 2 — Full load

## 2.1 What is a full load?

A full load reads the complete selected source dataset and loads it into the target.

Typical flow:

`Read all eligible source rows -> Validate -> Replace or rebuild target`

It is suitable when:
- the table is small;
- the source is a reference dataset;
- the complete reload is inexpensive;
- a full rebuild is needed after a logic change.

## 2.2 Full load advantages

- simple to understand;
- does not require a change watermark;
- can correct target drift if the source is authoritative;
- useful for small lookup tables.

## 2.3 Full load disadvantages

- reads and writes more data;
- can be expensive for large tables;
- may create a period where target is unavailable if replacement is not atomic;
- can overwrite history if target design is wrong;
- can cause source load if extraction is expensive.

## 2.4 Full load SQL example

Create source:

```sql
CREATE TEMP TABLE source_products (
    product_id INTEGER PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50)
);
```

Expected output:

```text
CREATE TABLE
```

Insert source rows:

```sql
INSERT INTO source_products VALUES
(10, 'Laptop', 'Electronics'),
(11, 'Mouse', 'Accessories'),
(12, 'Keyboard', 'Accessories');
```

Expected output:

```text
INSERT 0 3
```

Create target:

```sql
CREATE TEMP TABLE warehouse_products (
    product_id INTEGER PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50)
);
```

Expected output:

```text
CREATE TABLE
```

Load all rows:

```sql
INSERT INTO warehouse_products
SELECT product_id, product_name, category
FROM source_products;
```

Expected output:

```text
INSERT 0 3
```

Inspect:

```sql
SELECT * FROM warehouse_products ORDER BY product_id;
```

Expected result: three products matching the source.

## 2.5 Re-running a full load

If you run the INSERT again, the primary key rejects duplicate product IDs.

A full reload must define how the target is replaced.

A simple training example:

```sql
TRUNCATE TABLE warehouse_products;

INSERT INTO warehouse_products
SELECT product_id, product_name, category
FROM source_products;
```

Expected outputs:

```text
TRUNCATE TABLE
INSERT 0 3
```

This demonstrates a replace-style reload.

**Production warning:** TRUNCATE followed by INSERT can expose an empty target if the operations are not safely managed. Use a transaction or build-and-swap pattern supported by the platform. Consider readers, locks, rollback, permissions and table dependencies.

---

# Part 3 — Append load

## 3.1 What is append?

Append adds new rows to the target without replacing existing rows.

Typical uses:
- immutable event records;
- clickstream events;
- audit events;
- append-only transaction feeds.

## 3.2 Main risk

If the same batch is appended twice, duplicate rows may appear.

Append is safe only when:
- source events are truly immutable;
- a stable event identity exists;
- duplicates are prevented or detected;
- replay behavior is defined.

## 3.3 Append example

Create target:

```sql
CREATE TEMP TABLE warehouse_events (
    event_id INTEGER PRIMARY KEY,
    event_type VARCHAR(30),
    event_date DATE
);
```

Expected output:

```text
CREATE TABLE
```

Insert first batch:

```sql
INSERT INTO warehouse_events VALUES
(1, 'ORDER_CREATED', DATE '2026-01-01'),
(2, 'PAYMENT_COMPLETED', DATE '2026-01-01');
```

Expected output:

```text
INSERT 0 2
```

Insert a new event:

```sql
INSERT INTO warehouse_events VALUES
(3, 'ORDER_SHIPPED', DATE '2026-01-02');
```

Expected output:

```text
INSERT 0 1
```

Target now contains three rows.

Trying to append event_id 3 again fails because the primary key already exists.

That is useful protection, but production loading must also define how duplicate source events are handled and reported.

---

# Part 4 — Overwrite load

## 4.1 What is overwrite?

Overwrite replaces the existing contents of a target table or partition with a new result.

It is common when rebuilding:
- a small reference table;
- a daily snapshot partition;
- a monthly aggregate partition.

## 4.2 Table overwrite versus partition overwrite

Table overwrite replaces the entire target.

Partition overwrite replaces only a selected partition, such as one business date or month, when the platform supports it.

Do not overwrite the entire fact table just to correct one day's data unless the design requires a full rebuild.

## 4.3 When is partition overwrite useful?

Suppose one day's source file is corrected.

Instead of rebuilding five years of sales data, a pipeline may:
1. identify the affected date partition;
2. rebuild that date's complete result;
3. validate it;
4. replace only that partition.

The exact implementation differs by warehouse platform and file format.

## 4.4 Overwrite risks

- wrong partition predicate can delete valid data;
- incomplete source extract can replace a complete partition with partial data;
- readers may see inconsistent state if replacement is not atomic;
- late-arriving data may be missed.

Always validate the replacement dataset before publishing it.

---

# Part 5 — Incremental load

## 5.1 What is incremental loading?

Incremental loading processes only new or changed source records since a known point.

Examples:
- rows created after the last watermark;
- rows updated after the last successful extraction;
- CDC events after a log position;
- files not yet processed.

It reduces repeated processing, but it introduces state management.

## 5.2 What is a watermark?

A watermark records progress through a source.

Example:
- last successfully processed updated_at = 2026-01-05 10:00:00.

The next load requests rows after that point, with a safe overlap or tie-breaking strategy.

A timestamp alone may not uniquely order records.

## 5.3 Timestamp tie problem

Two rows may share the same updated_at timestamp.

A query using only:

`WHERE updated_at > last_watermark`

can skip a second row with the same timestamp if the watermark advances after the first.

Possible solutions:
- composite watermark (timestamp, unique key);
- CDC sequence/log position;
- overlap window plus deduplication;
- source change version.

The design must match the source guarantees.

## 5.4 Update problem

A new-row-only filter misses changes to existing records.

If product 10 changes category, an append-only incremental query may never update the target product row.

Use an upsert or a history-preserving strategy depending on the business requirement.

---

# Part 6 — Upsert

## 6.1 What is an upsert?

Upsert means:
- insert the row if the key does not exist;
- update the existing row if the key already exists.

PostgreSQL supports `INSERT ... ON CONFLICT`.

## 6.2 Create target

```sql
CREATE TEMP TABLE warehouse_product_upsert (
    product_id INTEGER PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    source_updated_at TIMESTAMP NOT NULL
);
```

Expected output:

```text
CREATE TABLE
```

## 6.3 Insert initial products

```sql
INSERT INTO warehouse_product_upsert VALUES
(10, 'Laptop', 'Electronics', TIMESTAMP '2026-01-01 09:00:00'),
(11, 'Mouse', 'Accessories', TIMESTAMP '2026-01-01 09:00:00');
```

Expected output:

```text
INSERT 0 2
```

## 6.4 Upsert a new and changed product

```sql
INSERT INTO warehouse_product_upsert
    (product_id, product_name, category, source_updated_at)
VALUES
    (10, 'Laptop Pro', 'Computers', TIMESTAMP '2026-01-05 09:00:00'),
    (12, 'Keyboard', 'Accessories', TIMESTAMP '2026-01-05 09:00:00')
ON CONFLICT (product_id)
DO UPDATE SET
    product_name = EXCLUDED.product_name,
    category = EXCLUDED.category,
    source_updated_at = EXCLUDED.source_updated_at
WHERE EXCLUDED.source_updated_at > warehouse_product_upsert.source_updated_at;
```

Expected output:

```text
INSERT 0 2
```

Interpretation:
- product 10 already exists, so it is updated;
- product 12 does not exist, so it is inserted;
- product 11 remains unchanged.

The WHERE condition prevents an older incoming record from overwriting a newer target value.

## 6.5 Inspect the result

```sql
SELECT *
FROM warehouse_product_upsert
ORDER BY product_id;
```

Expected result:

| product_id | product_name | category | source_updated_at |
|---:|---|---|---|
| 10 | Laptop Pro | Computers | 2026-01-05 09:00:00 |
| 11 | Mouse | Accessories | 2026-01-01 09:00:00 |
| 12 | Keyboard | Accessories | 2026-01-05 09:00:00 |

This is a Type 1-style current-state update. It does not preserve the old product category history.

For historical product category reporting, use an SCD Type 2 strategy instead.

---

# Part 7 — Compare loading strategies

| Strategy | Behavior | Common use | Main risk |
|---|---|---|---|
| Full load | Read all selected source data | Small reference table | Cost and replacement risk |
| Append | Add new rows | Immutable events | Duplicate replay |
| Overwrite | Replace target or partition | Snapshot/aggregate rebuild | Incomplete replacement |
| Incremental | Read only new/changed source data | Large changing datasets | Missed updates or watermark gaps |
| Upsert | Insert new, update existing | Current-state tables | Incorrect key or update logic |

These strategies can be combined. For example, a daily pipeline can incrementally extract source changes, stage them, then upsert a current-state dimension.

---

# Part 8 — Staging-to-warehouse loading

## 8.1 Why use staging?

Staging separates source ingestion from warehouse publication.

It gives the engineer a place to:
- validate types;
- standardize fields;
- identify duplicates;
- map keys;
- apply approved business rules;
- compare counts before writing the mart.

## 8.2 Common load flow

`Raw source -> Staging -> Validate -> Warehouse target -> Reconcile -> Checkpoint`

Do not advance the checkpoint before the target write and required validation are durable.

## 8.3 Example staging table

```sql
CREATE TEMP TABLE stg_product_load (
    product_id INTEGER,
    product_name VARCHAR(100),
    category VARCHAR(50),
    source_updated_at TIMESTAMP
);
```

Expected output:

```text
CREATE TABLE
```

Insert a batch:

```sql
INSERT INTO stg_product_load VALUES
(10, 'Laptop Pro', 'Computers', TIMESTAMP '2026-01-05 09:00:00'),
(12, 'Keyboard', 'Accessories', TIMESTAMP '2026-01-05 09:00:00');
```

Expected output:

```text
INSERT 0 2
```

## 8.4 Validate the staging batch

```sql
SELECT *
FROM stg_product_load
WHERE product_id IS NULL
   OR product_name IS NULL
   OR category IS NULL;
```

Expected result: zero rows for the training fixture.

Check duplicate source keys:

```sql
SELECT product_id, COUNT(*)
FROM stg_product_load
GROUP BY product_id
HAVING COUNT(*) > 1;
```

Expected result: zero rows.

## 8.5 Load only after validation

Use the upsert pattern from Part 6.

After the write, compare:
- staged accepted rows;
- inserted/updated target rows;
- target keys;
- target values;
- run control totals.

---

# Part 9 — Idempotency and replay

## 9.1 What is idempotent loading?

A load is idempotent when repeating the same logical input does not create unintended additional effects.

Example:
- process batch 2026-01-05;
- process the same batch again;
- target still represents the correct final state.

## 9.2 Why is idempotency important?

Jobs fail:
- after destination write;
- before checkpoint;
- during acknowledgement;
- after a timeout where the destination may have committed.

A retry must not duplicate the data.

## 9.3 Idempotency techniques

- stable business/event keys;
- unique constraints;
- deterministic upserts;
- batch IDs;
- processed-file registries;
- transactional writes;
- compare-and-set checkpoint updates;
- reconciliation after replay.

No single technique is sufficient for every source and target.

---

# Part 10 — dbt transformation workflow fundamentals

## 10.1 Where dbt fits

A common ELT flow:

`Ingestion -> Warehouse raw tables -> dbt staging -> dbt intermediate -> dbt marts -> BI`

dbt manages SQL transformations and model dependencies in supported data platforms. It does not automatically replace every source extraction or CDC tool.

## 10.2 Model layers

A simple dbt project may organize models into:
- staging;
- intermediate;
- marts.

Staging standardizes source fields.

Intermediate models combine or prepare business logic.

Marts expose reporting-ready facts, dimensions and metrics.

This is a convention, not a mandatory directory structure.

## 10.3 Example staging model

```sql
SELECT
    product_id,
    TRIM(product_name) AS product_name,
    INITCAP(TRIM(category)) AS category,
    source_updated_at
FROM {{ source('raw', 'raw_products') }}
```

This is dbt-templated SQL. PostgreSQL cannot execute the Jinja expression directly; dbt compiles it.

## 10.4 Example downstream model

```sql
SELECT
    product_id,
    product_name,
    category
FROM {{ ref('stg_products') }}
```

The ref function declares a dependency on the staging model.

## 10.5 A safe development workflow

1. Confirm target platform and dbt adapter.
2. Configure a development target and secure credentials.
3. Run `dbt debug`.
4. Run `dbt parse`.
5. Run `dbt compile`.
6. Inspect compiled SQL.
7. Run a selected model in development.
8. Run its tests.
9. Inspect output and lineage.
10. Promote through the approved deployment process.

Do not experiment directly against a production schema.

---

# Part 11 — Loading strategy decision guide

Ask:

1. Is the source small enough for full reload?
2. Are records immutable?
3. Can existing records change?
4. Are deletes represented?
5. Is there a reliable change timestamp or sequence?
6. Can timestamps tie?
7. Can events arrive late?
8. Does the target preserve history?
9. What is the safe replay boundary?
10. How is the load reconciled?
11. When is the checkpoint advanced?
12. What is the recovery plan?

Examples:
- small country lookup: full load may be sufficient;
- append-only click events: append with event-key deduplication;
- product current state: incremental upsert;
- customer history: SCD Type 2;
- corrected monthly aggregate: rebuild the affected month/partition.

---

# Part 12 — Common mistakes

| Mistake | Why it is dangerous | Better approach |
|---|---|---|
| Append the same batch twice | Duplicate rows | Stable key and idempotent write |
| Use full load for huge table without need | Expensive and slow | Incremental strategy |
| Use append for mutable source rows | Updates are missed | Upsert or CDC |
| Use upsert for history requirement | Previous values are overwritten | SCD Type 2 |
| Advance watermark before write | Data can be skipped after failure | Commit target and validation before checkpoint |
| Use timestamp alone with ties | Records can be missed | Composite cursor or CDC sequence |
| Overwrite partition with incomplete data | Valid target rows disappear | Validate replacement before swap |
| Ignore deletes | Target keeps removed source records | Define delete capture/tombstone policy |
| Trust dbt run success alone | Model may still be wrong | Tests and reconciliation |
| Run development SQL in production | Operational risk | Isolated development target |

---

# Part 13 — Practice exercises

## Exercise 1

A country lookup contains 200 rows and changes monthly. Which strategy may be suitable?

**Answer:** Full load may be appropriate if the complete reload is inexpensive and safely replaceable.

## Exercise 2

A source sends immutable event IDs. What strategy?

**Answer:** Append with a stable unique event ID and duplicate/replay protection.

## Exercise 3

A product row can change category. The target needs only the latest category. What strategy?

**Answer:** Incremental upsert, with a reliable update timestamp or change feed.

## Exercise 4

A customer category must be historically accurate for every past order. What strategy?

**Answer:** SCD Type 2 with facts referencing the correct historical dimension version.

## Exercise 5

Why is a timestamp-only watermark risky?

**Answer:** Multiple rows can share the same timestamp, and late-arriving changes can fall behind the watermark. Use a reliable sequence/composite cursor or an overlap and deduplication strategy.

## Exercise 6

Why should checkpoint be saved after target validation?

**Answer:** If the checkpoint advances before the destination is durable, a failure can cause source records to be skipped on the next run.

---

# Part 14 — Revision map

| Term | Remember |
|---|---|
| Full load | Reload all selected source rows |
| Append | Add rows without replacing existing rows |
| Overwrite | Replace a target or selected partition |
| Incremental load | Process new or changed data |
| Upsert | Insert if missing, update if present |
| Watermark | Progress marker through source data |
| CDC | Capture source database changes |
| Idempotency | Repeating same logical input has no unintended duplicate effect |
| Staging | Validated preparation area before warehouse target |
| Checkpoint | Durable record of processed progress |
| dbt model | SQL transformation managed by dbt |
| Materialization | How dbt builds the model in the platform |

## Final competency checklist

- [ ] Compare full, append, overwrite, incremental and upsert.
- [ ] Choose a strategy for immutable and mutable sources.
- [ ] Explain watermark risks.
- [ ] Explain why upsert is not the same as SCD Type 2.
- [ ] Build a PostgreSQL upsert.
- [ ] Explain idempotency and replay.
- [ ] Validate staging before warehouse load.
- [ ] Explain when to advance a checkpoint.
- [ ] Describe the dbt transformation workflow.
- [ ] Explain loading failure scenarios and recovery.

**Next lesson — Day 49: Warehouse Performance Optimization.** We will inspect query plans, understand partitioning, indexes, materialization and storage trade-offs, then perform a small before/after performance investigation without inventing benchmark results.
