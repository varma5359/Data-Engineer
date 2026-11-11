# Day 47: Metadata and Data Documentation

**Phase:** 5 — Data Warehousing and Modeling  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and junior Data Engineer candidates  
**Database:** PostgreSQL  
**Topic boundary:** Class notes only. Project workbook and interview guide are separate files.

> A warehouse table can contain millions of useful rows and still be difficult to trust if nobody knows what its columns mean, who owns it, how often it refreshes or where it came from. Metadata and documentation make data discoverable, understandable and governable. Today we build a data dictionary, business glossary and catalog-style documentation.

## Learning outcomes

By the end, you should be able to:
- explain metadata in simple terms;
- distinguish technical, business and operational metadata;
- explain a data catalog;
- create a data dictionary;
- create a business glossary;
- document a table and column;
- record ownership and data classification;
- understand lineage as metadata;
- document refresh and quality expectations;
- explain metadata management responsibilities;
- avoid common documentation mistakes.

## Today's flow

`Data asset -> Metadata capture -> Catalog entry -> Dictionary + glossary -> Ownership/classification -> Lineage -> Discovery and governance`

---

# Part 1 — What is metadata?

## 1.1 Beginner definition

Metadata is information that describes other data.

Example: A table contains a column named `order_amount`.

The value 2500.00 is data.

Information describing that column is metadata:
- column name: order_amount;
- data type: numeric;
- meaning: amount for an order line;
- currency: INR;
- nullable: no;
- source: order application;
- owner: sales data team;
- sensitivity: internal financial data;
- refresh: daily.

Metadata helps people understand and manage the data.

## 1.2 Why metadata matters

Without metadata:
- column names may be unclear;
- business meaning may be guessed;
- users may calculate metrics differently;
- source ownership may be unknown;
- sensitive fields may be exposed;
- pipeline failures are harder to investigate;
- downstream impact is difficult to assess.

Metadata is part of data engineering, not optional decoration.

---

# Part 2 — Types of metadata

## 2.1 Technical metadata

Technical metadata describes structure and implementation.

Examples:
- database name;
- schema name;
- table name;
- column name;
- data type;
- primary key;
- foreign key;
- partition key;
- file format;
- model dependency.

Example:

| Property | Value |
|---|---|
| Database | techstore_warehouse |
| Schema | mart_sales |
| Table | fact_sales |
| Column | line_sales_amount |
| Type | NUMERIC(12,2) |
| Nullable | No |

## 2.2 Business metadata

Business metadata explains meaning and usage.

Examples:
- business definition;
- metric formula;
- approved terminology;
- owner;
- domain;
- reporting use;
- inclusion/exclusion rule.

Example:

**Paid Sales:** Sum of eligible paid order-line amounts, using order date for the monthly sales report. Refunds are reported separately unless the approved metric definition says otherwise.

## 2.3 Operational metadata

Operational metadata describes how data is processed and maintained.

Examples:
- last successful run;
- source watermark;
- row count;
- load timestamp;
- run ID;
- pipeline status;
- duration;
- rejection count;
- quality test result.

Operational metadata helps answer whether a dataset is current and healthy.

## 2.4 Governance and security metadata

This describes how data must be protected and controlled.

Examples:
- public;
- internal;
- confidential;
- restricted;
- personal data indicator;
- retention period;
- access owner;
- approved usage.

Classification names vary by organization. Use the organization's approved policy.

## 2.5 Metadata type comparison

| Type | Main question | Example |
|---|---|---|
| Technical | How is it structured? | Column type and key |
| Business | What does it mean? | Paid sales definition |
| Operational | How is it running? | Last refresh and row count |
| Governance | How may it be used? | Confidential classification |

---

# Part 3 — What is a data catalog?

## 3.1 Beginner definition

A data catalog is an organized inventory of data assets and their metadata.

It helps users discover:
- which datasets exist;
- what each dataset means;
- who owns it;
- where it came from;
- whether it is trusted;
- how to request access;
- which reports depend on it.

A catalog may be provided by a dedicated platform, a cloud service, a warehouse platform or an internally maintained registry.

## 3.2 What belongs in a catalog entry?

For a warehouse table:
- asset name;
- business description;
- database/schema;
- owner;
- domain;
- columns and types;
- primary and foreign keys;
- grain;
- source;
- refresh frequency;
- quality status;
- lineage;
- sensitivity;
- access instructions.

## 3.3 Catalog versus data dictionary

A data dictionary is usually a detailed description of fields and structures.

A data catalog is a broader inventory and discovery experience that can include dictionaries, lineage, ownership, classification and quality metadata.

The exact boundary differs by organization.

---

# Part 4 — Build a data dictionary

## 4.1 What is a data dictionary?

A data dictionary describes tables and their columns in a structured way.

Example:

| Column | Type | Required? | Definition | Example |
|---|---|---|---|---|
| order_id | INTEGER | Yes | Source order identifier | 1001 |
| order_date | DATE | Yes | Date the order was placed | 2026-01-02 |
| line_sales_amount | NUMERIC(12,2) | Yes | Amount for one order line | 2500.00 |
| order_status | VARCHAR(20) | Yes | Approved order state | PAID |

A useful dictionary explains meaning, not just type.

## 4.2 Include grain and keys

For a fact table, document:
- one row per what;
- primary/surrogate key;
- business key;
- dimension foreign keys;
- measure definitions.

Example:

**Table:** fact_sales  
**Grain:** One row per product line in one order.  
**Primary key:** sales_line_key.  
**Measures:** quantity, unit_price, line_sales_amount.

Without grain, users may misinterpret row counts and totals.

## 4.3 Document nullable fields

A nullable column allows missing values.

But the dictionary should explain what NULL means:
- unknown;
- not supplied;
- not applicable;
- not yet received.

NULL is not always the same as zero, empty text or "unknown".

## 4.4 Document units and currency

For numeric fields, include:
- currency;
- measurement unit;
- precision;
- scale;
- rounding rule.

A value of 2500 could mean INR, USD, cents or another unit. A numeric type alone does not explain it.

## 4.5 Document timestamps and time zones

For timestamp fields, record:
- whether it is UTC;
- source time zone;
- whether it is event time or ingestion time;
- how daylight saving or local business date is handled.

This avoids date-boundary errors.

---

# Part 5 — Build a business glossary

## 5.1 What is a business glossary?

A business glossary defines shared business terms in language that business and technical users understand.

Examples:
- paid order;
- gross sales;
- net sales;
- active customer;
- refund;
- fulfilled order;
- late delivery.

## 5.2 Why does it matter?

Different teams may use the same word differently.

For example, "active customer" could mean:
- logged in within 30 days;
- placed an order within 90 days;
- has an active subscription.

The glossary records the approved definition.

## 5.3 Example glossary

| Term | Definition | Owner |
|---|---|---|
| Paid order | Order meeting the approved payment-success rule | Sales/Finance |
| Order line | One product line within an order | Sales Operations |
| Paid sales | Approved sum of paid order-line amounts | Finance |
| Refund | Approved return of money to a customer | Finance |
| Delivery duration | Time between approved shipment and delivery events | Logistics |

Do not invent a universal business definition. The owner approves the term.

## 5.4 Synonyms and deprecated terms

A glossary can record:
- approved term;
- synonyms;
- deprecated term;
- definition;
- owner;
- related terms.

Example:
- approved: customer_key;
- deprecated label: cust_id_wh;
- note: do not use old label in new models.

---

# Part 6 — Ownership and stewardship

## 6.1 Business owner

The business owner is accountable for the meaning, acceptable use and business rules of the data.

Example:
- Finance approves the definition of recognized revenue.

## 6.2 Technical owner

The technical owner maintains the dataset, pipelines, tests, access and documentation.

Example:
- Data Engineering team maintains the fact_sales model and its refresh workflow.

## 6.3 Data steward

A data steward helps maintain definitions, quality expectations, classifications and appropriate use.

Roles differ by organization. Small teams may assign several responsibilities to one person.

## 6.4 Why ownership matters

Without ownership:
- no one approves metric changes;
- data issues remain unresolved;
- access requests are delayed;
- definitions drift;
- users do not know whom to contact.

Every critical data asset should have a clear owner or escalation path.

---

# Part 7 — Data classification and access

## 7.1 Why classify data?

Not every user should see every column.

Examples:
- public product category;
- internal sales performance;
- confidential business pricing;
- restricted personal or payment information.

Classification helps determine:
- who can access data;
- whether masking is needed;
- where data can be copied;
- how long data can be retained;
- how logs and exports should be handled.

## 7.2 Example classification table

| Field | Example classification | Handling |
|---|---|---|
| product_category | Internal | Approved company users |
| order_amount | Confidential | Sales/Finance roles |
| customer_email | Personal/confidential | Restricted access and masking |
| payment_token | Restricted secret/sensitive | Do not expose in analytical reports |

Use the actual organization's policy and legal requirements.

## 7.3 Least privilege

Grant only the access needed for a user's job.

A report consumer may need aggregated sales but not raw customer email addresses.

A data engineer may need controlled pipeline access but should not automatically have unrestricted access to every sensitive field.

---

# Part 8 — Operational metadata and data contracts

## 8.1 Dataset operational record

A dataset record may include:
- last successful refresh;
- expected refresh;
- last source event time;
- row count;
- rejected count;
- quality status;
- pipeline owner.

This lets users judge whether the dataset is ready.

## 8.2 Data contract

A data contract is an agreed expectation between a data producer and consumer.

It may define:
- schema;
- field meaning;
- types;
- required fields;
- allowed values;
- keys;
- delivery frequency;
- change notification;
- compatibility rules;
- quality expectations.

A contract is not merely a list of columns. It includes behavioral expectations.

## 8.3 Schema change example

A source changes `customer_id` from integer to text.

A contract should explain:
- whether this is allowed;
- how the change is announced;
- whether leading zeros are meaningful;
- how downstream models adapt;
- how compatibility is tested.

Do not blindly cast a text identifier to integer if leading zeros or non-numeric characters can be meaningful.

---

# Part 9 — Create a documentation register in PostgreSQL

This is a learning example. In a real organization, catalog metadata may live in a dedicated catalog tool rather than application tables.

## 9.1 Create a simple asset register

```sql
CREATE TEMP TABLE data_asset_catalog (
    asset_name VARCHAR(100) PRIMARY KEY,
    business_description TEXT NOT NULL,
    owner_team VARCHAR(100) NOT NULL,
    refresh_frequency VARCHAR(50) NOT NULL,
    classification VARCHAR(30) NOT NULL
);
```

Expected output:

```text
CREATE TABLE
```

## 9.2 Insert two assets

```sql
INSERT INTO data_asset_catalog VALUES
(
    'mart_sales.fact_sales',
    'One row per sales order line for approved sales analysis',
    'Data Engineering',
    'Daily',
    'Confidential'
),
(
    'mart_sales.dim_product',
    'Product descriptions and approved product categories',
    'Product Analytics',
    'Daily',
    'Internal'
);
```

Expected output:

```text
INSERT 0 2
```

## 9.3 Read catalog entries

```sql
SELECT
    asset_name,
    owner_team,
    refresh_frequency,
    classification
FROM data_asset_catalog
ORDER BY asset_name;
```

Expected result:

| asset_name | owner_team | refresh_frequency | classification |
|---|---|---|---|
| mart_sales.dim_product | Product Analytics | Daily | Internal |
| mart_sales.fact_sales | Data Engineering | Daily | Confidential |

## 9.4 Add a dictionary table

```sql
CREATE TEMP TABLE data_dictionary (
    asset_name VARCHAR(100) NOT NULL,
    column_name VARCHAR(100) NOT NULL,
    data_type VARCHAR(50) NOT NULL,
    business_definition TEXT NOT NULL,
    is_required BOOLEAN NOT NULL,
    PRIMARY KEY (asset_name, column_name)
);
```

Expected output:

```text
CREATE TABLE
```

## 9.5 Insert column definitions

```sql
INSERT INTO data_dictionary VALUES
('mart_sales.fact_sales', 'order_id', 'INTEGER', 'Source order identifier', TRUE),
('mart_sales.fact_sales', 'quantity', 'INTEGER', 'Units on this order line', TRUE),
('mart_sales.fact_sales', 'line_sales_amount', 'NUMERIC(12,2)', 'Approved amount for one order line', TRUE);
```

Expected output:

```text
INSERT 0 3
```

## 9.6 Query the dictionary

```sql
SELECT *
FROM data_dictionary
WHERE asset_name = 'mart_sales.fact_sales'
ORDER BY column_name;
```

Expected result: three rows describing order_id, quantity and line_sales_amount.

---

# Part 10 — Metadata discovery using PostgreSQL system views

PostgreSQL exposes structural metadata through information_schema and system catalogs.

## 10.1 Inspect columns

For a permanent table:

```sql
SELECT
    column_name,
    data_type,
    is_nullable
FROM information_schema.columns
WHERE table_schema = 'public'
  AND table_name = 'orders'
ORDER BY ordinal_position;
```

This query assumes a permanent table named public.orders exists.

For a temporary training table, the schema may be a session-specific temporary schema. Inspect the actual schema or use the appropriate catalog query.

## 10.2 What does information_schema provide?

It can show:
- table and column names;
- data types;
- nullability;
- constraints and other standard metadata.

It does not automatically know:
- business definition;
- business owner;
- metric meaning;
- approved report use;
- business sensitivity classification.

Those need documentation and governance metadata.

---

# Part 11 — Documentation standards

For every important warehouse model, document:

1. Model name.
2. Business purpose.
3. Grain.
4. Owner.
5. Upstream sources.
6. Column definitions.
7. Key relationships.
8. Metric definitions.
9. Refresh frequency.
10. Quality tests.
11. Classification.
12. Access rules.
13. Known limitations.
14. Downstream consumers.
15. Change history.

Keep documentation near the model in version control where practical, and publish it in the organization's catalog.

## 11.1 Write for two audiences

Technical reader needs:
- type;
- key;
- model dependency;
- transformation;
- tests.

Business reader needs:
- meaning;
- metric definition;
- limitations;
- owner;
- refresh expectation.

Good documentation supports both.

## 11.2 Documentation must stay current

Outdated documentation can be more dangerous than missing documentation because users may trust incorrect definitions.

Update documentation when:
- schema changes;
- business definition changes;
- owner changes;
- refresh schedule changes;
- sensitivity classification changes;
- lineage changes.

---

# Part 12 — Common mistakes

| Mistake | Why it is harmful | Better approach |
|---|---|---|
| Documenting only column names | Meaning remains unclear | Add definitions and examples |
| No grain documentation | Users may miscount facts | State one-row meaning |
| No owner | Issues have no accountable person | Assign business and technical owners |
| Calling a numeric amount "sales" | Currency and business meaning are unclear | Document unit, currency and rule |
| Ignoring time zone | Date boundaries can be misread | Document event time and time zone |
| Putting sensitive examples in public docs | Privacy/security exposure | Use synthetic data and classification |
| Catalog not updated after schema change | Users trust stale information | Include documentation in change workflow |
| Treating information_schema as a full catalog | Business context is missing | Add glossary, ownership and governance metadata |
| No deprecation process | Old fields continue to be used | Mark deprecated fields and replacement |

---

# Part 13 — Practice exercises

## Exercise 1

A column is named `amt`. What metadata should be added?

**Answer:** Clear name, type, currency, precision, business meaning, grain, source, null rule and owner.

## Exercise 2

What is the difference between technical and business metadata?

**Answer:** Technical metadata describes structure and implementation. Business metadata explains meaning, rules and usage.

## Exercise 3

Who approves the definition of recognized revenue?

**Answer:** The designated business owner, often Finance, with technical implementation by the data team.

## Exercise 4

Why is a data catalog broader than a data dictionary?

**Answer:** A catalog can include asset discovery, ownership, classification, lineage, quality, access and dictionary details.

## Exercise 5

A source changes a numeric customer identifier to text. What do you document?

**Answer:** Schema change, compatibility, leading-zero behavior, downstream impacts, migration rule, owner approval and tests.

---

# Part 14 — Revision map

| Term | Remember |
|---|---|
| Metadata | Information describing data |
| Technical metadata | Structure and implementation |
| Business metadata | Meaning and business rules |
| Operational metadata | Run, load and freshness information |
| Governance metadata | Classification, ownership and access |
| Data catalog | Inventory and discovery system for data assets |
| Data dictionary | Detailed table and column definitions |
| Business glossary | Approved business terms and definitions |
| Data steward | Helps maintain definitions, quality and governance |
| Data contract | Agreement about schema and data behavior |
| Classification | Sensitivity and permitted handling |
| Ownership | Accountability for data meaning and maintenance |

## Final competency checklist

- [ ] Explain metadata and its types.
- [ ] Distinguish a catalog, dictionary and glossary.
- [ ] Document a table's grain and columns.
- [ ] Identify business and technical owners.
- [ ] Classify sensitive data.
- [ ] Explain data contracts.
- [ ] Inspect structural metadata in PostgreSQL.
- [ ] Document lineage, refresh and quality expectations.
- [ ] Explain why documentation must be maintained.

**Next lesson — Day 48: Warehouse Loading Strategies.** We will compare full, incremental, append, overwrite and upsert loading, then learn how staging and transformation models fit into a reliable warehouse load.
