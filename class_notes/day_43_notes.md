# Day 43: Keys and Data Relationships

**Phase:** 5 — Data Warehousing and Modeling  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and junior Data Engineer candidates  
**Database:** PostgreSQL  
**Topic boundary:** Class notes only. Project workbook and interview guide are separate files.

> A warehouse can contain correct-looking rows and still connect them incorrectly. Keys are the identifiers that let tables refer to the right business entities and historical versions. Today we learn natural keys, surrogate keys, composite keys, primary keys and foreign keys, then use them to build safe relationships.

## Learning outcomes

By the end, you should be able to:
- explain why keys are required;
- distinguish natural and surrogate keys;
- explain primary and foreign keys;
- identify composite keys;
- understand candidate and alternate keys;
- explain why source identifiers can be reused or changed;
- design warehouse dimension keys;
- understand fact-to-dimension relationships;
- identify one-to-one, one-to-many and many-to-many relationships;
- detect orphan records and duplicate keys;
- understand key mapping and unknown members;
- choose a key strategy for a warehouse table.

## Today's flow

`Business entity -> Source identifier -> Warehouse key -> Relationship -> Fact lookup -> Validation`

---

# Part 1 — Why do tables need keys?

## 1.1 Begin with two tables

Customer table:

| customer_id | customer_name |
|---:|---|
| 501 | Asha |
| 502 | Rahul |

Orders table:

| order_id | customer_id | order_amount |
|---:|---:|---:|
| 7001 | 501 | 2500 |
| 7002 | 502 | 1800 |
| 7003 | 501 | 900 |

How do we know which customer placed order 7001?

The value 501 in the order table points to the customer row with customer_id 501.

The identifier connects records.

Without a reliable identifier, the database may not know which customer an order belongs to.

## 1.2 Why names are poor identifiers

Two customers may both be named Asha.

A customer can change their name.

Spelling may differ across systems.

Names are descriptive attributes, not reliable unique keys.

A key should identify the intended record according to a defined rule.

## 1.3 What makes a good key?

A good key should be:
- unique within its intended scope;
- stable enough for its purpose;
- always present when required;
- unambiguous;
- managed consistently;
- documented.

A source identifier may be unique in one application but not across the whole organization.

That difference matters in a warehouse.

---

# Part 2 — Natural keys

## 2.1 What is a natural key?

A natural key is an identifier that already exists in the business or source system and has meaning outside the warehouse.

Examples:
- customer number;
- product SKU;
- invoice number;
- passport number where legally appropriate and permitted;
- order reference.

For TechStore, customer_id 501 is a natural/business key in the source application.

## 2.2 Natural key example

```sql
CREATE TEMP TABLE source_customer (
    customer_id INTEGER PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL
);
```

Expected output:

```text
CREATE TABLE
```

Insert:

```sql
INSERT INTO source_customer VALUES
(501, 'Asha'),
(502, 'Rahul');
```

Expected output:

```text
INSERT 0 2
```

Here, customer_id is the source's natural identifier.

## 2.3 Natural key advantages

- already exists in the source;
- may be meaningful to business users;
- can help trace a warehouse row back to the source;
- avoids creating another identifier for some simple models.

## 2.4 Natural key risks

A natural key may:
- change;
- be reused after deletion;
- be unique only inside one source;
- have inconsistent formats;
- be missing for early-arriving records;
- contain sensitive information.

Example:

CRM customer 501 and loyalty customer 501 may refer to different people.

The value alone is not globally unique. The source system must be part of the identity.

A combined source identity might be:

`(source_system, source_customer_id)`

This is a composite business identity.

---

# Part 3 — Surrogate keys

## 3.1 What is a surrogate key?

A surrogate key is an artificial identifier generated and managed by the warehouse. It has no business meaning.

Example:

| customer_key | customer_id | source_system | customer_name |
|---:|---:|---|---|
| 10001 | 501 | CRM | Asha |
| 10002 | 501 | Loyalty | Asha |

The warehouse key distinguishes the two rows even though the source identifier is the same.

In an SCD Type 2 dimension, different historical versions of one customer also receive different surrogate keys.

## 3.2 Why use surrogate keys?

They help:
- integrate multiple source systems;
- preserve history;
- keep fact foreign keys stable;
- avoid exposing source identifiers in every relationship;
- separate warehouse identity from source-system identity.

## 3.3 Create a generated surrogate key in PostgreSQL

```sql
CREATE TEMP TABLE dim_customer_key_demo (
    customer_key INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    source_system VARCHAR(30) NOT NULL,
    customer_id VARCHAR(30) NOT NULL,
    customer_name VARCHAR(100) NOT NULL,
    UNIQUE (source_system, customer_id)
);
```

Expected output:

```text
CREATE TABLE
```

Explanation:
- `GENERATED ALWAYS AS IDENTITY` asks PostgreSQL to generate the key value.
- `PRIMARY KEY` ensures the generated key is unique and not null.
- `source_system` identifies the originating application.
- `customer_id` stores the source's customer identifier.
- `UNIQUE (source_system, customer_id)` prevents duplicate source identities within the same source.

Insert two source records:

```sql
INSERT INTO dim_customer_key_demo
    (source_system, customer_id, customer_name)
VALUES
    ('CRM', '501', 'Asha'),
    ('LOYALTY', '501', 'Asha');
```

Expected output:

```text
INSERT 0 2
```

Inspect:

```sql
SELECT *
FROM dim_customer_key_demo
ORDER BY customer_key;
```

Expected logical result:

| customer_key | source_system | customer_id | customer_name |
|---:|---|---|---|
| 1 | CRM | 501 | Asha |
| 2 | LOYALTY | 501 | Asha |

The generated values may differ if the table has already received inserts in the current session.

## 3.4 Surrogate key is not a business identifier

Do not tell a business user that customer_key 1 is the customer's real customer number.

It is only the warehouse identifier.

Keep the source system and source key for traceability.

---

# Part 4 — Primary keys

## 4.1 What is a primary key?

A primary key uniquely identifies each row in a table.

A primary key:
- must be unique;
- cannot be NULL;
- identifies one row;
- is referenced by foreign keys when other tables need to point to it.

Example:

```sql
CREATE TEMP TABLE product_demo (
    product_key INTEGER PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL
);
```

Expected output:

```text
CREATE TABLE
```

Insert:

```sql
INSERT INTO product_demo VALUES
(1, 'Laptop'),
(2, 'Mouse');
```

Expected output:

```text
INSERT 0 2
```

## 4.2 Duplicate primary key

Try:

```sql
INSERT INTO product_demo VALUES (1, 'Keyboard');
```

Expected behavior: PostgreSQL rejects the row because product_key 1 already exists.

The exact error text includes a duplicate-key violation for the primary-key constraint.

The database protects the table from two rows having the same primary key.

## 4.3 NULL primary key

A primary key cannot be NULL.

The database enforces this rule even if you do not separately write NOT NULL.

## 4.4 Candidate key

A candidate key is a minimal set of columns that can uniquely identify a row.

A table may have more than one candidate key.

Example:
- employee_id;
- company_email.

If both are unique and required, either may be a candidate key.

## 4.5 Alternate key

A candidate key not selected as the primary key is an alternate key.

If employee_id is primary, company_email may be an alternate key enforced with UNIQUE.

---

# Part 5 — Foreign keys

## 5.1 What is a foreign key?

A foreign key is a column or group of columns that references a candidate key, usually the primary key, in another table.

It protects relationships.

Example:

- dim_customer.customer_key is the primary key.
- fact_sales.customer_key is a foreign key referencing it.

## 5.2 Create a parent table

```sql
CREATE TEMP TABLE parent_customer (
    customer_key INTEGER PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL
);
```

Expected output:

```text
CREATE TABLE
```

Insert a parent row:

```sql
INSERT INTO parent_customer VALUES (10, 'Asha');
```

Expected output:

```text
INSERT 0 1
```

## 5.3 Create a child table

```sql
CREATE TEMP TABLE child_order (
    order_id INTEGER PRIMARY KEY,
    customer_key INTEGER NOT NULL REFERENCES parent_customer(customer_key),
    order_amount NUMERIC(10, 2) NOT NULL
);
```

Expected output:

```text
CREATE TABLE
```

Insert a valid child:

```sql
INSERT INTO child_order VALUES (9001, 10, 2500.00);
```

Expected output:

```text
INSERT 0 1
```

The customer key 10 exists in the parent table.

## 5.4 Try an invalid foreign key

```sql
INSERT INTO child_order VALUES (9002, 99, 1500.00);
```

Expected behavior: PostgreSQL rejects the row because customer_key 99 does not exist in parent_customer.

This prevents an orphan relationship.

## 5.5 Why foreign keys matter in warehouses

Some analytical platforms do not enforce foreign keys physically or may treat them as informational constraints.

Even then, the relationship must be validated through data tests.

A warehouse should not assume that a declared relationship is correct merely because a diagram shows a line.

---

# Part 6 — Composite keys

## 6.1 What is a composite key?

A composite key uses more than one column together to identify a row.

Example:
- order_id;
- line_number.

One order can have several lines. The pair identifies one order line.

## 6.2 Create a composite primary key

```sql
CREATE TEMP TABLE order_line_demo (
    order_id INTEGER NOT NULL,
    line_number INTEGER NOT NULL,
    product_id VARCHAR(20) NOT NULL,
    quantity INTEGER NOT NULL,
    PRIMARY KEY (order_id, line_number)
);
```

Expected output:

```text
CREATE TABLE
```

Insert:

```sql
INSERT INTO order_line_demo VALUES
(5001, 1, 'P10', 1),
(5001, 2, 'P11', 2),
(5002, 1, 'P12', 1);
```

Expected output:

```text
INSERT 0 3
```

The pair (5001, 1) identifies the first line of order 5001.

The pair (5001, 2) identifies the second line.

## 6.3 Duplicate composite key

This row must fail:

```sql
INSERT INTO order_line_demo VALUES
(5001, 1, 'P99', 1);
```

The combination (5001, 1) already exists.

Even though product_id differs, the declared key is order_id plus line_number.

## 6.4 Composite business key versus warehouse surrogate key

A source order-line identity may be (source_system, order_id, line_number).

The warehouse may still assign a single surrogate key to the fact or dimension row.

Keep the source identity for traceability and deduplication.

---

# Part 7 — Relationship cardinality

## 7.1 One-to-one

One row in table A relates to at most one row in table B, and vice versa.

Example:
- employee and one-to-one employee profile, if the business rules enforce exactly one profile per employee.

One-to-one relationships require suitable uniqueness constraints. A foreign key alone does not guarantee one-to-one.

## 7.2 One-to-many

One row in a parent table can relate to many child rows.

Example:
- one customer can place many orders;
- one order can contain many order lines.

This is common in relational and warehouse models.

## 7.3 Many-to-many

Many rows in A can relate to many rows in B.

Example:
- one student can enroll in many courses;
- one course can have many students.

A bridge/junction table represents the relationship.

In dimensional modeling, many-to-many relationships may need a bridge table with carefully defined grain and allocation rules.

## 7.4 Why cardinality matters

A join can multiply rows.

Before joining, ask:
- Is the relationship one-to-one?
- One-to-many?
- Many-to-many?
- Is the key unique on the dimension side?
- Does the business require allocation across multiple matches?

---

# Part 8 — Warehouse key strategy

## 8.1 Keep both source identity and warehouse identity

A useful dimension design often includes:
- surrogate dimension key;
- source system;
- source business key;
- descriptive attributes;
- history columns where required.

This supports:
- warehouse joins;
- source traceability;
- SCD Type 2;
- multi-source integration;
- duplicate detection.

## 8.2 Unknown dimension member

A fact may arrive before its dimension record.

A common warehouse strategy creates a controlled unknown member, for example customer_key = 0.

This lets the fact load continue if the business allows it, while preserving the fact for later resolution.

The pipeline must record:
- why the dimension was unresolved;
- which source key was missing;
- how many facts use the unknown member;
- how and when the key will be corrected.

Never silently map every missing customer to the unknown member without monitoring.

## 8.3 Inferred dimension member

An inferred member is a placeholder dimension row created when a fact arrives before full descriptive details.

When the full dimension record later arrives, the placeholder is updated or completed according to the warehouse design.

This differs from simply discarding the fact.

## 8.4 Multi-source identity

Suppose:
- CRM customer 501;
- loyalty customer 501;
- support customer C-501.

They may or may not be the same person.

Do not merge based only on matching identifiers or names.

Use an approved identity resolution process, preserve source mappings and document confidence/ownership rules.

---

# Part 9 — Validate keys and relationships in SQL

## 9.1 Find duplicate business keys

```sql
SELECT
    source_system,
    customer_id,
    COUNT(*) AS row_count
FROM dim_customer_key_demo
GROUP BY source_system, customer_id
HAVING COUNT(*) > 1;
```

Expected result: zero rows because the unique constraint prevents duplicates.

## 9.2 Find orphan facts

```sql
SELECT
    o.order_id,
    o.customer_key
FROM child_order AS o
LEFT JOIN parent_customer AS c
    ON o.customer_key = c.customer_key
WHERE c.customer_key IS NULL;
```

Expected result: zero rows.

The foreign key already protects this temporary example. In platforms without enforced constraints, this query is a useful data test.

## 9.3 Check duplicate composite keys

```sql
SELECT
    order_id,
    line_number,
    COUNT(*) AS row_count
FROM order_line_demo
GROUP BY order_id, line_number
HAVING COUNT(*) > 1;
```

Expected result: zero rows.

## 9.4 Check relationship counts

```sql
SELECT
    customer_key,
    COUNT(*) AS order_count
FROM child_order
GROUP BY customer_key;
```

Expected result:

| customer_key | order_count |
|---:|---:|
| 10 | 1 |

This is a simple one-customer-to-many-orders pattern, even though the small fixture currently has one order.

---

# Part 10 — Common key mistakes

| Mistake | Why it is dangerous | Better approach |
|---|---|---|
| Using customer name as a key | Names can repeat or change | Use a stable business identifier |
| Assuming source IDs are globally unique | Different systems can reuse values | Include source identity |
| Using natural key as SCD2 row key | One business key has multiple versions | Use surrogate version key |
| Forgetting foreign-key validation | Facts can become orphaned | Enforce or test relationships |
| Using a composite key without documenting it | Developers may join on only part of it | Document all key columns |
| Generating a new surrogate on every reload | Same source row becomes a new identity | Maintain deterministic key mapping |
| Reusing deleted key values | Old references may point to a different entity | Define key lifecycle and reuse policy |
| Mapping every unknown to one key without audit | Data loss becomes invisible | Track unknown-key counts and resolution |
| Joining on non-unique dimension columns | Rows multiply | Join on the declared key |
| Treating a foreign key as a business rule | Referential integrity alone does not validate meaning | Add business validation |

---

# Part 11 — Key design decision guide

When designing a warehouse entity:

1. Identify the source system.
2. Identify the source business key.
3. Check whether the key is unique within that source.
4. Check whether the key can change or be reused.
5. Decide whether multiple sources represent one shared entity.
6. Decide whether historical versions need separate identities.
7. Choose a surrogate key strategy if required.
8. Preserve source key and source system for traceability.
9. Define foreign-key relationships.
10. Add uniqueness and not-null rules.
11. Validate orphan records.
12. Monitor unknown and inferred members.
13. Document key ownership and mapping.

---

# Part 12 — Practice exercises

## Exercise 1

CRM and Loyalty systems both contain customer_id 501.

Can customer_id alone be the enterprise key?

**Answer:** No. It is only unique within each source unless the organization guarantees otherwise. Include source identity or use an approved enterprise identity mapping.

## Exercise 2

An order has five lines. Which key can identify a line if the source has no line_id?

**Answer:** A composite key such as order_id plus line_number, if the source guarantees that combination is unique and stable.

## Exercise 3

Why does SCD Type 2 require a surrogate key?

**Answer:** One business key can have several historical dimension versions. A surrogate key identifies each version separately.

## Exercise 4

What is the difference between a primary key and a foreign key?

**Answer:** A primary key identifies a row in its own table. A foreign key references a key in another table to represent a relationship.

## Exercise 5

A dimension has duplicate product_key values in a platform where constraints are not enforced. What do you do?

**Answer:** Identify duplicate source mappings, determine the valid dimension record, repair the model and add automated uniqueness tests.

## Exercise 6

A fact arrives before the customer record. What are two possible warehouse strategies?

**Answer:** Use a governed unknown member or create an inferred dimension member, then resolve it when the full customer data arrives.

---

# Part 13 — Revision map

| Term | Remember |
|---|---|
| Natural key | Existing business/source identifier |
| Surrogate key | Warehouse-generated identifier |
| Primary key | Uniquely identifies a row |
| Foreign key | References a key in another table |
| Candidate key | Minimal unique identifier option |
| Alternate key | Candidate key not selected as primary |
| Composite key | Multiple columns identify a row together |
| Cardinality | How many rows can relate across tables |
| Bridge table | Represents a many-to-many relationship |
| Unknown member | Controlled placeholder for unresolved dimension |
| Inferred member | Placeholder dimension record awaiting full details |
| Source identity | Source system plus source key |
| Orphan record | Child/fact row without a valid parent/dimension |

## Final competency checklist

- [ ] Explain natural and surrogate keys.
- [ ] Explain primary and foreign keys.
- [ ] Identify candidate and alternate keys.
- [ ] Create a composite primary key.
- [ ] Explain one-to-one, one-to-many and many-to-many.
- [ ] Explain why source system belongs in multi-source identity.
- [ ] Explain why Type 2 needs a surrogate version key.
- [ ] Find duplicate keys and orphan rows.
- [ ] Explain unknown and inferred dimension members.
- [ ] Defend a warehouse key strategy.

**Next lesson — Day 44: Data Marts and Subject-Oriented Modeling.** We will turn business requirements into a focused sales data mart, define its subject area and reusable dimensions, and design the model for actual reporting users.
