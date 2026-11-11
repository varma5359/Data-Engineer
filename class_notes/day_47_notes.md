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


---

# Part 15 — Document one warehouse table from beginning to end

The previous sections introduced catalog entries, dictionaries and glossaries separately. Now combine them into one documentation exercise.

Imagine a business analyst asks:

"Can I use the sales fact table to calculate monthly paid sales by product category?"

Before answering, the Data Engineer must establish what the table contains and what the metric means.

## 15.1 Start by identifying the asset

Example asset:

| Property | Documentation |
|---|---|
| Asset name | mart_sales.fact_sales |
| Domain | Sales |
| Purpose | Analyze eligible sales order lines |
| Grain | One row per order line |
| Business owner | Sales/Finance data owner |
| Technical owner | Data Engineering |
| Refresh | Daily, after source ingestion |
| Classification | Confidential |
| Main consumers | Sales dashboard and finance reporting |
| Status | Certified only after approved checks |

The names and owners are illustrative. In a real company, confirm them with the relevant teams.

## 15.2 Why grain must be written first

Grain explains what one row represents.

Suppose an order contains three products:

| order_id | line_number | product | amount |
|---:|---:|---|---:|
| 5001 | 1 | Keyboard | 2000 |
| 5001 | 2 | Mouse | 500 |
| 5001 | 3 | Monitor | 12000 |

This is three rows because the table grain is one row per order line.

If a student assumes one row per order, they may count three orders instead of one.

A good documentation page says:
- one row represents one order line;
- order_id can repeat;
- order_id plus line_number identifies a line;
- sales amount is stored at line grain.

## 15.3 Document the columns

| Column | Type | Required | Business definition | Example |
|---|---|---|---|---|
| sales_line_key | BIGINT | Yes | Warehouse-generated row identifier | 900001 |
| order_id | INTEGER | Yes | Source order identifier | 5001 |
| line_number | INTEGER | Yes | Position of the product line within the order | 2 |
| product_key | INTEGER | Yes | Warehouse product dimension key | 301 |
| order_date_key | INTEGER | Yes | Warehouse date key for order date | 20260102 |
| quantity | INTEGER | Yes | Number of units on the line | 2 |
| unit_price | NUMERIC(12,2) | Yes | Price per unit in INR | 250.00 |
| line_sales_amount | NUMERIC(14,2) | Yes | Quantity multiplied by unit price under the approved sales rule | 500.00 |

The example values are illustrative. Use actual model types and business rules in a real dictionary.

## 15.4 Document the measure definition

Do not write only "sales amount".

Write a definition that explains:
- what is included;
- what is excluded;
- the calculation;
- the grain;
- the currency;
- the date role;
- whether refunds are included.

Example:

**Paid line sales:** Sum of approved paid order-line amounts, stored in INR, grouped by order date. Pending and cancelled orders are excluded. Refunds are reported separately in the standard gross paid sales metric.

A Finance-approved metric may use a different rule. The business owner must approve it.

---

# Part 16 — Create a business glossary that prevents conflicting reports

## 16.1 The problem with undefined terms

Imagine two analysts produce a metric called "monthly sales".

Analyst A includes paid orders only.

Analyst B includes paid and pending orders.

Both dashboards display "Monthly Sales", but the values differ.

The problem is not SQL syntax. The problem is that the business term has no shared definition.

## 16.2 Build a glossary entry

| Property | Example |
|---|---|
| Term | Paid Sales |
| Definition | Sum of approved paid order-line amounts |
| Calculation | SUM(line_sales_amount) |
| Grain | Order line before aggregation |
| Date role | Order date |
| Included statuses | PAID |
| Excluded statuses | PENDING, CANCELLED |
| Refund treatment | Reported separately |
| Currency | INR |
| Owner | Finance |
| Related terms | Gross Sales, Net Sales, Refund Amount |
| Status | Approved |

## 16.3 Compare similar business terms

| Term | Example meaning | Why separate definitions matter |
|---|---|---|
| Order value | Value recorded for an order | May include pending orders |
| Paid sales | Value of successfully paid orders | Payment success rule matters |
| Gross sales | Sales before approved deductions | Refund treatment must be explicit |
| Net sales | Sales after defined deductions | Discounts/refunds must be defined |
| Cash received | Actual cash or settlement received | May differ from order date sales |

These are examples, not universal definitions. Never assume two organizations use these terms identically.

## 16.4 Add a change approval process

If Finance changes the definition of Paid Sales:
1. Record the requested change.
2. Identify the business reason.
3. Get approval from the term owner.
4. Identify affected models and reports.
5. Update the glossary and metric documentation.
6. Update SQL transformations if needed.
7. Reconcile historical and current results.
8. Communicate the effective date.
9. Preserve the previous definition and change history.

Do not overwrite an old definition without recording when it changed.

---

# Part 17 — Inspect real table metadata in PostgreSQL

The earlier example used a permanent table called public.orders. Here is a practical way to inspect tables and columns in a database.

## 17.1 Find tables in a schema

Run:

~~~sql
SELECT
    table_schema,
    table_name
FROM information_schema.tables
WHERE table_schema NOT IN ('information_schema', 'pg_catalog')
  AND table_type = 'BASE TABLE'
ORDER BY table_schema, table_name;
~~~

This lists visible base tables in schemas other than PostgreSQL's standard metadata schemas.

Expected output depends on your database. If no rows appear, check that you connected to the intended database and have permission to see the tables.

## 17.2 Inspect columns for a known table

Replace the schema and table names with ones that exist in your database.

~~~sql
SELECT
    ordinal_position,
    column_name,
    data_type,
    is_nullable,
    character_maximum_length,
    numeric_precision,
    numeric_scale
FROM information_schema.columns
WHERE table_schema = 'public'
  AND table_name = 'orders'
ORDER BY ordinal_position;
~~~

Understand the output:
- ordinal_position gives the column's position;
- column_name gives its name;
- data_type gives the SQL type;
- is_nullable indicates whether NULL is allowed;
- character_maximum_length applies to character types;
- numeric_precision and numeric_scale describe numeric capacity.

If the query returns zero rows, the table may not exist in public, may have a different name or may not be visible to your database role.

## 17.3 Inspect constraints

~~~sql
SELECT
    tc.constraint_name,
    tc.constraint_type,
    kcu.column_name
FROM information_schema.table_constraints AS tc
LEFT JOIN information_schema.key_column_usage AS kcu
    ON tc.constraint_name = kcu.constraint_name
   AND tc.table_schema = kcu.table_schema
   AND tc.table_name = kcu.table_name
WHERE tc.table_schema = 'public'
  AND tc.table_name = 'orders'
ORDER BY tc.constraint_type, tc.constraint_name, kcu.ordinal_position;
~~~

This can help discover primary keys, foreign keys, unique constraints and other table constraints.

A constraint name alone may not communicate the business meaning. Add that meaning to your dictionary.

## 17.4 Technical metadata is not business documentation

PostgreSQL can tell you:
- column name;
- type;
- nullability;
- constraints.

It cannot reliably infer:
- whether amount means gross or net sales;
- whether a date is order date or settlement date;
- which team approves the metric;
- whether a customer field is allowed in a report;
- why a particular status is excluded.

Engineers must capture that context from approved requirements and owners.

---

# Part 18 — Document data contracts and schema changes

A data contract prevents producers and consumers from making incompatible assumptions.

## 18.1 Example contract for an order feed

| Contract item | Agreement |
|---|---|
| Dataset | orders feed |
| Delivery | Daily by 07:00 local business time |
| Format | CSV |
| Required fields | order_id, order_date, status |
| order_id | Stable identifier; not reused |
| order_date | ISO date format |
| status | Approved status list |
| Duplicate policy | One record per order in header feed |
| Change notice | Notify consumers before breaking changes |
| Owner | Order application team |
| Consumer | Warehouse ingestion team |

A contract should also define late-arriving data, deletions, null handling, versioning and compatibility.

## 18.2 Breaking and non-breaking changes

Examples of possible non-breaking changes:
- adding an optional field, if consumers tolerate additional fields;
- adding a new value that consumers are explicitly designed to handle.

Examples of possible breaking changes:
- renaming a required column;
- changing a numeric identifier to a format consumers cannot parse;
- changing a field's business meaning;
- removing a field used by downstream reports.

Whether a change is breaking depends on the contract and consumer behavior.

## 18.3 A safe schema-change workflow

1. Producer proposes the change.
2. Data owner explains the business reason.
3. Consumers and downstream dependencies are identified.
4. Contract version is updated.
5. Compatibility is assessed.
6. Test data is shared.
7. Staging and warehouse transformations are updated.
8. Quality and reconciliation tests run.
9. Release timing is agreed.
10. Old-field deprecation and removal are communicated.

Avoid changing production schemas without an impact review.

## 18.4 Preserve identifiers correctly

Suppose customer codes are:

~~~text
00125
00126
00127
~~~

If these are converted to integers, they become 125, 126 and 127. The leading zeros disappear.

If the identifier is a code rather than a quantity, text may be the correct type.

A data dictionary should explain whether an identifier is numeric for arithmetic or merely contains digits.

---

# Part 19 — Classify data and document safe handling

## 19.1 Classification is a decision, not a guess

A column should be classified according to the organization's security and privacy policy.

Examples of questions:
- Is the field publicly available?
- Does it identify a person?
- Could it expose confidential business performance?
- Does it contain credentials, tokens or payment details?
- Is it permitted in a development dataset?
- Does it need masking or restricted access?

Do not place real personal information in classroom examples or public documentation.

## 19.2 Example data handling register

| Field | Classification | Example handling |
|---|---|---|
| product_category | Internal | Company users with approved access |
| order_amount | Confidential | Restrict detailed access to approved roles |
| customer_email | Personal/confidential | Mask or restrict |
| password_hash | Restricted security data | Never expose in analytics output |
| payment_token | Restricted sensitive data | Keep out of general reporting |

Use your organization's actual classification categories and retention policy.

## 19.3 Documentation itself can leak information

Do not include:
- real customer names and emails;
- API keys;
- passwords;
- access tokens;
- database connection strings;
- private file paths;
- screenshots containing sensitive records.

Use synthetic examples and redact secrets.

---

# Part 20 — Documentation in a team workflow

Documentation is useful only when people can find it and trust that it is current.

## 20.1 Where can documentation live?

| Location | Suitable content |
|---|---|
| Git repository | Model descriptions, schema files, change history |
| Data catalog | Asset discovery, owners, classification, lineage |
| Business glossary | Approved business terminology |
| dbt documentation | Model and column descriptions, tests and dependencies |
| Run metadata store | Refresh time, run status, row counts |
| Access management system | Roles and approved access |

The same information may appear in more than one place, but assign one authoritative owner/source for each definition to avoid conflicting copies.

## 20.2 Documentation review checklist

Before publishing a new model, ask:
- Does the model have a clear description?
- Is grain documented?
- Are keys and relationships listed?
- Does every important column have a definition?
- Are units and date roles clear?
- Is the business owner identified?
- Is the technical owner identified?
- Are quality checks documented?
- Is refresh frequency clear?
- Is classification assigned?
- Are upstream and downstream dependencies known?
- Are limitations and known issues recorded?

## 20.3 Keep change history

A simple change record can look like this:

| Version | Date | Change | Approved by |
|---|---|---|---|
| 1.0 | 2026-01-01 | Initial Paid Sales definition | Finance owner |
| 1.1 | 2026-02-01 | Clarified refund treatment | Finance owner |
| 1.2 | 2026-03-01 | Added order channel dimension | Sales owner |

The dates and approvals are examples. Record real approval evidence in a real project.

---

# Part 21 — Worked documentation scenario

A new student joins the Data Engineering team. The student sees a table called sales_final and asks whether it can be used for monthly revenue reporting.

Do not answer yes based on the table name.

Investigate in this order:

1. Find the schema and table definition.
2. Inspect column names, types and constraints.
3. Identify the table grain.
4. Find the source-to-target mapping.
5. Read the business glossary definition for revenue.
6. Identify the owner.
7. Check the refresh schedule and last successful run.
8. Review data quality results.
9. Trace upstream and downstream lineage.
10. Confirm the report's date, currency and inclusion rules.
11. Check classification and access permissions.
12. Ask the business owner to confirm the intended use if documentation is incomplete.

Possible conclusion:

"The table is technically available, but its revenue definition and refund treatment are not documented. We should not certify it for finance reporting until the Finance owner approves the metric definition and the data team records the mapping and validation evidence."

This is good engineering judgment. A table existing does not mean it is ready for every business use.

---

# Part 22 — Additional practice exercises with worked answers

## Exercise 6 — Grain

A fact table has five rows for order 7001 because the customer bought five different products. What should the table documentation say?

**Worked answer:** One row per order line. order_id can repeat. A line-level key such as order_id plus line_number identifies each order line.

## Exercise 7 — Currency

A numeric column contains 1500. Is its meaning fully documented?

**Worked answer:** No. Document currency, precision, scale, rounding rule and whether the amount is gross, net, paid or refunded.

## Exercise 8 — Business owner

A data engineer knows how a revenue column is calculated in SQL but does not know which deductions Finance expects. Who approves the business definition?

**Worked answer:** The designated Finance/business owner. The engineer documents and implements the approved rule.

## Exercise 9 — Metadata discovery

PostgreSQL shows a column named event_time with type timestamp. Does this tell you the time zone or business meaning?

**Worked answer:** No. Document whether it is UTC or local time, whether it is event time or ingestion time, and which business process it represents.

## Exercise 10 — Contract change

A producer wants to rename a required field tomorrow. What should happen first?

**Worked answer:** Identify consumers and dependencies, assess compatibility, notify owners, update the contract and plan a tested migration. Do not make an unannounced breaking change.

## Exercise 11 — Catalog and dictionary

A user wants to discover all datasets owned by the Sales team and understand the columns of one table. Which resources help?

**Worked answer:** The catalog helps discover assets by owner/domain. The data dictionary provides detailed table and column definitions.

## Exercise 12 — Sensitive examples

A documentation page includes real customer email addresses to demonstrate the format. Is this a good practice?

**Worked answer:** No. Use synthetic examples or redacted values. Documentation must follow privacy, security and access rules.

## Exercise 13 — Outdated definition

A glossary says Paid Sales excludes refunds, but the model was changed to subtract refunds last month. What is the risk?

**Worked answer:** Users may interpret the metric incorrectly and make conflicting decisions. Update the glossary, model documentation, change history and affected reports after business approval.

---

# Part 23 — Day 47 practical completion checklist

### Metadata and discovery
- [ ] Explain metadata in simple words.
- [ ] Distinguish technical, business, operational and governance metadata.
- [ ] Explain what a data catalog is.
- [ ] Distinguish a catalog, data dictionary and business glossary.
- [ ] Explain why table grain belongs in documentation.

### Documentation practice
- [ ] Write a catalog entry for a warehouse table.
- [ ] Document each important column's type and meaning.
- [ ] Document currency, units, precision and time zone.
- [ ] Create a business glossary definition.
- [ ] Record business and technical owners.
- [ ] Record refresh frequency and quality expectations.
- [ ] Record upstream and downstream dependencies.
- [ ] Document known limitations and change history.

### PostgreSQL and governance
- [ ] List tables using information_schema.tables.
- [ ] Inspect columns using information_schema.columns.
- [ ] Inspect table constraints.
- [ ] Explain what database metadata cannot tell you.
- [ ] Explain data contracts and breaking schema changes.
- [ ] Classify sensitive fields and describe safe handling.
- [ ] Explain why documentation examples should use synthetic data.

## Final self-assessment

Without looking at the notes, explain:

1. What is metadata, and why does a Data Engineer need it?
2. How are a data catalog, dictionary and glossary different?
3. Why must a fact table's grain be documented?
4. Why is a numeric amount incomplete without currency and business meaning?
5. Who should approve a business metric definition?
6. What is a data contract?
7. How do you investigate a schema change safely?
8. Why can a technically available table still be unfit for a business report?

If you can answer these questions and document a table independently, you are ready for Day 48.

**Next lesson — Day 48: Warehouse Loading Strategies.** We will compare full, incremental, append, overwrite and upsert loading, then learn how staging and transformation models fit into a reliable warehouse load.
