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


---

# Part 15 — Run a complete staging-to-warehouse upsert

The earlier sections showed individual loading strategies. Now combine them into a small workflow.

You will:
1. create a source batch;
2. stage the rows;
3. validate the batch;
4. upsert the warehouse table;
5. reconcile the target;
6. understand when a checkpoint may advance.

Keep the SQL statements in one PostgreSQL session because the training tables are temporary.

## 15.1 Understand the business situation

TechStore maintains a product dimension.

The source sends product records every day:
- new products may arrive;
- existing product names may change;
- categories may change;
- old source updates may be replayed.

The warehouse table stores the latest approved product state. It does not preserve history. This is a Type 1-style current-state table.

## 15.2 Create a fresh staging batch

~~~sql
CREATE TEMP TABLE daily_product_batch (
    product_id INTEGER,
    product_name VARCHAR(100),
    category VARCHAR(50),
    source_updated_at TIMESTAMP,
    batch_id VARCHAR(40)
);
~~~

Expected output: CREATE TABLE.

Insert the batch:

~~~sql
INSERT INTO daily_product_batch VALUES
(10, 'Laptop Pro', 'Computers', TIMESTAMP '2026-01-05 09:00:00', 'batch-001'),
(12, 'Keyboard', 'Accessories', TIMESTAMP '2026-01-05 09:00:00', 'batch-001'),
(13, 'Webcam', 'Accessories', TIMESTAMP '2026-01-05 09:05:00', 'batch-001');
~~~

Expected output: INSERT 0 3.

The batch contains:
- product 10, which may already exist;
- product 12, which may be new;
- product 13, which is new.

## 15.3 Check batch completeness

~~~sql
SELECT
    COUNT(*) AS batch_rows,
    COUNT(product_id) AS rows_with_product_id,
    COUNT(product_name) AS rows_with_product_name,
    COUNT(category) AS rows_with_category
FROM daily_product_batch;
~~~

Expected result:

| batch_rows | rows_with_product_id | rows_with_product_name | rows_with_category |
|---:|---:|---:|---:|
| 3 | 3 | 3 | 3 |

All three records have the required values in this fixture.

## 15.4 Check duplicate business keys inside the batch

~~~sql
SELECT
    product_id,
    COUNT(*) AS row_count
FROM daily_product_batch
GROUP BY product_id
HAVING COUNT(*) > 1;
~~~

Expected result: zero rows.

If this query returns a product ID, do not blindly upsert both rows. First decide which source record is authoritative. A common approach is to choose the latest source update only when the source contract guarantees that timestamp ordering is reliable. If two records have the same timestamp but conflicting values, route the conflict for investigation.

## 15.5 Check timestamp and allowed values

~~~sql
SELECT *
FROM daily_product_batch
WHERE source_updated_at IS NULL
   OR product_id IS NULL
   OR TRIM(product_name) = ''
   OR TRIM(category) = '';
~~~

Expected result: zero rows for the training fixture.

If the query returns records, inspect the original values and apply the approved reject or quarantine policy. Do not convert missing product names into invented labels.

## 15.6 Upsert only after the checks pass

The target table from Part 6 is named warehouse_product_upsert.

~~~sql
INSERT INTO warehouse_product_upsert
    (product_id, product_name, category, source_updated_at)
SELECT
    product_id,
    TRIM(product_name),
    TRIM(category),
    source_updated_at
FROM daily_product_batch
ON CONFLICT (product_id)
DO UPDATE SET
    product_name = EXCLUDED.product_name,
    category = EXCLUDED.category,
    source_updated_at = EXCLUDED.source_updated_at
WHERE EXCLUDED.source_updated_at > warehouse_product_upsert.source_updated_at;
~~~

Expected output for the original Part 6 target state and this three-row batch: INSERT 0 3.

Interpretation:
- product 10 is updated only if the incoming timestamp is newer;
- product 12 is inserted;
- product 13 is inserted;
- any older replay for an existing key is ignored by the WHERE condition.

PostgreSQL's EXCLUDED refers to the incoming row that conflicted with an existing target row. It does not mean a deleted or excluded business record.

## 15.7 Inspect the target

~~~sql
SELECT
    product_id,
    product_name,
    category,
    source_updated_at
FROM warehouse_product_upsert
ORDER BY product_id;
~~~

Expected result:

| product_id | product_name | category | source_updated_at |
|---:|---|---|---|
| 10 | Laptop Pro | Computers | 2026-01-05 09:00:00 |
| 11 | Mouse | Accessories | 2026-01-01 09:00:00 |
| 12 | Keyboard | Accessories | 2026-01-05 09:00:00 |
| 13 | Webcam | Accessories | 2026-01-05 09:05:00 |

The exact display of timestamps depends on your SQL client, but the stored values should match the inserted timestamps.

## 15.8 Replay the same batch

Run the upsert statement from 15.6 again without changing the batch.

The target should still contain four product IDs, not seven rows.

Why?
- product_id is a primary key;
- existing keys take the conflict path;
- the incoming timestamp is not newer after the first successful load;
- no duplicate product rows are created.

This is an example of idempotent current-state loading.

A production pipeline should also record batch identity and execution status. A unique key alone does not solve every replay, ordering or deletion problem.

---

# Part 16 — Understand the checkpoint and commit order

A checkpoint tells the next run where to continue. Saving it too early can permanently skip data.

## 16.1 A dangerous sequence

Imagine a pipeline does this:

1. Reads source changes through 10:00.
2. Saves checkpoint = 10:00.
3. Starts writing the target.
4. Target write fails.

The next run starts after 10:00. The failed records may never be retried.

This is a data-loss risk.

## 16.2 Safer logical sequence

Use this order:

1. Read source changes using the previous committed checkpoint.
2. Write the batch to durable staging.
3. Validate staging.
4. Apply the warehouse write.
5. Reconcile the target and record the run result.
6. Commit the new checkpoint only when the required destination work is durable.
7. Mark the batch successful.

The exact transaction boundary depends on whether staging, target and checkpoint are in the same database or different systems.

## 16.3 Why cross-system pipelines need extra care

A database transaction generally cannot atomically commit writes across unrelated systems without additional distributed transaction support.

For example:
- source is an external API;
- staging is object storage;
- target is PostgreSQL;
- checkpoint is stored in a workflow database.

A practical design may use:
- immutable batch IDs;
- durable staging files;
- idempotent destination writes;
- a run-control table;
- explicit batch states;
- safe retry rules;
- reconciliation before marking success.

Do not claim the whole multi-system workflow is atomic unless the architecture actually guarantees it.

## 16.4 Example batch states

| State | Meaning |
|---|---|
| EXTRACTED | Source data has been captured |
| STAGED | Batch is stored durably |
| VALIDATED | Required staging checks passed |
| LOADED | Target write completed |
| RECONCILED | Target comparison passed |
| COMMITTED | Checkpoint safely advanced |
| FAILED | A step failed and needs recovery |

A real pipeline may use different states, but transitions should be explicit.

---

# Part 17 — Understand deletes and tombstones

Upsert handles new and changed rows. It does not automatically remove rows that disappeared from the source.

## 17.1 Why a missing source row is ambiguous

Suppose product 11 is absent from today's source extract.

Possible explanations:
- product 11 was deleted;
- the source sent only a partial batch;
- extraction failed to include it;
- the product is inactive but retained;
- the source has a filtering defect.

Absence from an incremental batch is not proof of deletion.

## 17.2 What is a tombstone?

A tombstone is a source change record indicating that an entity was deleted or should be treated as deleted.

Example source event:

| product_id | operation | changed_at |
|---:|---|---|
| 11 | DELETE | 2026-01-06 10:00:00 |

A pipeline can process that explicit deletion event according to the warehouse policy.

## 17.3 Soft delete example

A current-state warehouse table may store an active flag.

~~~sql
ALTER TABLE warehouse_product_upsert
ADD COLUMN IF NOT EXISTS is_active BOOLEAN DEFAULT TRUE;
~~~

Expected output: ALTER TABLE.

If a trusted delete event is received for product 11:

~~~sql
UPDATE warehouse_product_upsert
SET is_active = FALSE
WHERE product_id = 11;
~~~

Expected output: UPDATE 1.

This preserves the row but marks it inactive. It is not the same as SCD Type 2 history. A historical dimension may need effective dates and a new version row.

Do not infer deletions by comparing a partial incremental batch with the full target. Only use full-snapshot absence as deletion evidence when the source contract explicitly guarantees a complete snapshot.

---

# Part 18 — Loading strategy decision workshop

Use the following situations to choose a strategy. There is no single strategy that is best for every dataset.

| Dataset situation | Likely approach | Reason |
|---|---|---|
| Small country reference list | Full load | Cheap to rebuild |
| Immutable click events | Append with event key | Events are new facts |
| Product current state | Incremental upsert | Existing products can change |
| Customer historical address | SCD Type 2 | Previous versions matter |
| Corrected daily aggregate | Rebuild one date partition | Recompute only affected date |
| Source sends database log changes | CDC-based apply | Uses source change records |
| Small derived report table | Full rebuild | Simpler and safer than complex state |

Before selecting, ask:
- Can the source update old records?
- Can it delete records?
- Does the target need history?
- Can the same batch be replayed?
- Is a stable key available?
- Is there a reliable change cursor?
- How will the result be reconciled?

---

# Part 19 — Loading failures and recovery scenarios

## 19.1 Failure before target write

Situation: extraction succeeds, but staging validation fails.

Correct response:
- keep the batch evidence;
- do not publish;
- record failed rules;
- correct the source or transformation;
- replay the batch after approval.

## 19.2 Failure during target write

Situation: the database rejects the write because of a constraint.

Correct response:
- inspect whether the transaction rolled back;
- inspect the target state;
- preserve the failed batch;
- correct the underlying issue;
- retry idempotently.

Do not assume a failed client message means zero rows were written unless transaction behavior confirms it.

## 19.3 Failure after target write but before checkpoint

Situation: target data is committed, but the workflow crashes before saving progress.

Correct response:
- the next run may read the same batch again;
- destination write must safely handle replay;
- use stable keys/upsert or a processed-batch ledger;
- reconcile before marking the batch complete.

## 19.4 Failure after checkpoint but before target write

This is the dangerous ordering described earlier. It can skip data.

Prevention:
- do not advance the checkpoint before destination durability and required validation;
- use a transaction where possible;
- use durable batch states across systems.

## 19.5 Partial batch

Situation: a file contains 10,000 expected rows, but only 6,000 arrive.

Do not overwrite a complete target partition with the partial file.

Check:
- file delivery completion marker;
- expected row count or control total;
- file size/checksum if supplied;
- source batch identifier;
- quality and reconciliation results.

---

# Part 20 — Final load validation

After any load, validate more than the SQL command status.

For a current-state product table, check:

## 20.1 Row count

~~~sql
SELECT COUNT(*) AS product_count
FROM warehouse_product_upsert;
~~~

Expected result after the training workflow: 4.

## 20.2 Duplicate business keys

~~~sql
SELECT
    product_id,
    COUNT(*) AS row_count
FROM warehouse_product_upsert
GROUP BY product_id
HAVING COUNT(*) > 1;
~~~

Expected result: zero rows. The primary key also prevents duplicate product_id values.

## 20.3 Latest source update

~~~sql
SELECT MAX(source_updated_at) AS latest_source_update
FROM warehouse_product_upsert;
~~~

Expected result: 2026-01-05 09:05:00.

This is the latest source update in the training target, not necessarily the time the pipeline ran.

## 20.4 Compare incoming and target keys

For a batch that should be fully applied, verify every staged key exists in the target:

~~~sql
SELECT b.product_id
FROM daily_product_batch AS b
LEFT JOIN warehouse_product_upsert AS w
    ON b.product_id = w.product_id
WHERE w.product_id IS NULL;
~~~

Expected result: zero rows.

This only checks key presence. Also compare important fields and source versions to prove the correct values were applied.

---

# Part 21 — Additional practice exercises with worked answers

## Exercise 7 — Append or upsert?

A product source sends product 20 today and sends a changed product 20 tomorrow. The warehouse needs only the latest product state.

**Worked answer:** Use incremental upsert, not simple append. A stable product key identifies the existing row to update.

## Exercise 8 — Preserve history

A product changes from Accessories to Electronics. Analysts must report historical sales using the category that was valid at the time of each sale.

**Worked answer:** Use a historical dimension approach such as SCD Type 2. A Type 1 upsert would overwrite the old category and lose that history.

## Exercise 9 — Duplicate replay

A batch is processed twice. What prevents the target from containing duplicate business records?

**Worked answer:** Stable keys, unique constraints, deterministic upsert or deduplication, batch tracking and reconciliation. The exact combination depends on the source and target.

## Exercise 10 — Missing product in an incremental batch

Product 25 is not present in today's incremental file. Should the warehouse delete it?

**Worked answer:** No. An incremental file contains only changes or selected rows. Absence does not prove deletion. Require an explicit delete event or a guaranteed complete snapshot with an approved comparison policy.

## Exercise 11 — Checkpoint ordering

The pipeline advances its watermark and then crashes before writing the warehouse. What can happen?

**Worked answer:** The next run may skip records because the checkpoint says they were processed. Advance the checkpoint only after destination work is durable and required validation has passed.

## Exercise 12 — Older replay

A target has product update timestamp 12:00. A replayed source row has timestamp 10:00. Should it overwrite the target?

**Worked answer:** Not under the example current-state rule. The target should retain the newer version. If timestamps are not reliable or can tie, use a stronger source sequence/version.

## Exercise 13 — Full load versus partition overwrite

One date's sales file is corrected. The warehouse has five years of daily partitions. What may be more efficient?

**Worked answer:** Rebuild and replace only the affected date partition, after validating the complete replacement dataset. Ensure the platform's partition replacement is safe and atomic enough for the use case.

## Exercise 14 — dbt run success

A dbt model builds successfully but contains duplicated order lines. Is the model ready?

**Worked answer:** Not automatically. Run uniqueness, relationship, business-rule and reconciliation tests. Successful SQL execution proves the model ran, not that the result is correct.

---

# Part 22 — Day 48 practical completion checklist

### Concepts
- [ ] Explain full load, append, overwrite, incremental and upsert.
- [ ] Choose a strategy for a small reference table.
- [ ] Choose a strategy for immutable events.
- [ ] Choose a strategy for mutable current-state records.
- [ ] Explain when SCD Type 2 is required.
- [ ] Explain why an incremental batch's missing row does not prove deletion.
- [ ] Explain tombstones and soft deletes.

### SQL practice
- [ ] Create and load a source reference table.
- [ ] Perform a full load.
- [ ] Explain why a repeated append can duplicate data.
- [ ] Build an INSERT ON CONFLICT upsert.
- [ ] Use EXCLUDED to refer to incoming values.
- [ ] Use a source timestamp condition to avoid older overwrites.
- [ ] Create and validate a staging batch.
- [ ] Re-run the same batch and inspect the target.
- [ ] Compare source batch keys with target keys.
- [ ] Record and interpret row counts and latest source update.

### Production reasoning
- [ ] Explain idempotency and replay.
- [ ] Explain safe checkpoint ordering.
- [ ] Distinguish a database transaction from a multi-system workflow.
- [ ] Describe failure recovery before and after target writes.
- [ ] Explain how deletes are captured.
- [ ] Explain how to validate a replacement partition.
- [ ] Describe where dbt fits into ELT.
- [ ] Explain why a successful load command is not proof of correct data.

## Final self-assessment

Without looking at the notes, explain:

1. What is the difference between append and upsert?
2. Why can a full load be unsafe for a large warehouse fact table?
3. Why does a mutable source require more than append?
4. What is the difference between a current-state upsert and SCD Type 2?
5. Why is a timestamp-only watermark risky?
6. Why should the checkpoint not advance before the target is durable?
7. Why does an absent row in an incremental batch not prove deletion?
8. What checks should run after a warehouse load?

If you can explain these decisions and reproduce the SQL workflow, you are ready for Day 49.

**Next lesson — Day 49: Warehouse Performance Optimization.** We will inspect query plans, understand partitioning, indexes, materialization and storage trade-offs, then perform a small before/after performance investigation without inventing benchmark results.
