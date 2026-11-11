# Day 44 Project Workbook — Sales Data Mart

**Database:** PostgreSQL  
**Run location:** pgAdmin Query Tool or psql.  
**Goal:** Turn a reporting request into a documented, validated sales mart. Use the Day 41 warehouse schema as source; do not invent columns if your schema differs.

## Scenario and requirements

TechStore sales manager asks:

> Show paid sales by month, product category, customer city and sales channel. Show paid order count and units. Finance also needs refund totals.

Users and decisions:
- Sales manager: monitor sales trend and order volume.
- Category manager: identify categories to promote or replenish.
- Regional lead: compare city performance.
- Finance: reconcile paid sales and refunds.

## Step 1 — Write the metric contract before SQL

Fill and review this contract with a business owner:

| Metric | Definition for this lab | Important decision |
|---|---|---|
| Paid sales | Sum of eligible paid order-line amounts | Is discount already included? |
| Paid orders | COUNT(DISTINCT order_id) among eligible paid orders | How are partial payments treated? |
| Units sold | Sum of eligible line quantities | Are returns subtracted? |
| Refund amount | Sum of approved refund event amounts | Refund date or original order date? |
| Average order value | Paid sales / distinct paid orders | Currency and eligibility |

**Do not silently combine sales and refunds.** They are different business events and may have different grains and dates.

## Step 2 — State the grain

Write one sentence for each fact:

- Sales fact: one row per order line (or per documented sales event).
- Refund fact: one row per refund event (or refund line).
- Date dimension: one row per calendar date.
- Product dimension: one row per product version according to the warehouse model.

If your fact is one row per order but you join it to order lines, the order amount may repeat. Grain must be known before aggregation.

## Step 3 — Draw the mart

Create a diagram or text model:

```text
dim_date       dim_product       dim_customer       dim_channel
    \              |                  |                  /
                 fact_sales_line
                       |
                 sales metrics

dim_date ---- fact_refund_event ---- dim_customer/product (as available)
```

Explain which dimensions are conformed/reused from the warehouse and why the refund fact is separate.

## Step 4 — Inspect source/model columns first

Run this against your actual warehouse tables; replace names if your Day 41 schema differs:

```sql
SELECT table_name, column_name, data_type
FROM information_schema.columns
WHERE table_schema = 'public'
  AND table_name IN ('fact_sales', 'dim_product', 'dim_customer', 'dim_date')
ORDER BY table_name, ordinal_position;
```

Expected: one metadata row per column that exists in those tables. If no rows appear, check the schema/table names. Do not proceed by guessing fields.

## Step 5 — Build a simple sales aggregation (adapt column names)

Assumption for this query: fact_sales has one row per sales line and contains order_id, product_key, customer_key, date_key, quantity and line_amount. Confirm these columns first.

```sql
SELECT
    date_key,
    product_key,
    SUM(line_amount) AS paid_sales,
    COUNT(DISTINCT order_id) AS paid_orders,
    SUM(quantity) AS units_sold
FROM fact_sales
GROUP BY date_key, product_key
ORDER BY date_key, product_key;
```

**Expected:** one row per date_key + product_key combination. Numeric totals depend on your loaded data.

**Important:** This is only correct if fact_sales already contains eligible paid sales. If it contains all order statuses, add the approved status filter or use a curated paid-sales model. Never label all order amounts as paid sales without verifying status.

## Step 6 — Add dimensions carefully

Join the aggregate/model to product, customer and channel dimensions using their warehouse keys—not descriptive names.

Before joining, test that each dimension key is unique:

```sql
SELECT product_key, COUNT(*)
FROM dim_product
GROUP BY product_key
HAVING COUNT(*) > 1;
```

Expected: zero rows for a dimension key intended to be unique. Repeat for customer/date/channel keys.

**Join fan-out warning:** if a dimension has duplicate keys, sales rows can multiply. Check row counts and sums before and after every join.

## Step 7 — Create a reporting view (adapt after schema inspection)

A view stores a reusable SQL definition; it does not normally store the result rows itself.

Example pattern (use only after confirming names):

```sql
CREATE OR REPLACE VIEW vw_sales_mart_daily AS
SELECT
    date_key,
    product_key,
    SUM(line_amount) AS paid_sales,
    COUNT(DISTINCT order_id) AS paid_orders,
    SUM(quantity) AS units_sold
FROM fact_sales
GROUP BY date_key, product_key;
```

Expected: CREATE VIEW. Then inspect:

```sql
SELECT * FROM vw_sales_mart_daily
ORDER BY date_key, product_key;
```

Expected: aggregated rows. The values must match Step 5.

## Step 8 — Validate the mart

### 8A. Reconcile sales total

```sql
SELECT SUM(line_amount) AS source_sales_total FROM fact_sales;
SELECT SUM(paid_sales) AS mart_sales_total FROM vw_sales_mart_daily;
```

Expected: equal only if fact_sales contains exactly the same eligible sales population as the mart and no filters are applied. If statuses or exclusions exist, reconcile using the identical eligibility rule on both sides.

### 8B. Reconcile row grain

Compare:
- source eligible line count;
- mart aggregate row count;
- distinct order count;
- quantity total.

Aggregation reduces row count, so do not expect mart rows to equal source line rows. Explain the expected difference.

### 8C. Check missing dimension matches

Use a left join from facts to each dimension and count rows where the dimension key is NULL on the matched side. Expected: zero, unless a documented unknown member is used.

## Step 9 — Refunds: avoid double counting

Refunds are events, not necessarily sales-line attributes. A refund may occur days or months after an order and may be partial or repeated.

Do not join refund rows directly to sales lines and then sum both measures: one sale line with two refund events can appear twice.

Keep refund fact at its own grain, aggregate it separately, and combine only at a clearly defined reporting grain with correct date roles and join cardinality.

## Step 10 — Ownership and documentation

Write:
- business metric owner;
- technical model owner;
- refresh frequency and freshness target;
- source tables;
- quality checks and blocking rules;
- access/security classification;
- known exclusions and limitations;
- version/change process for metric definitions.

## Failure drills (answer in writing)

1. Finance and Sales report different revenue: compare metric definitions, filters, date role, currency, discounts/refunds and source reconciliation.
2. Refresh succeeds but source is missing: freshness/completeness gate should prevent misleading publication.
3. Category sales double after refund join: inspect grain and fan-out; aggregate refund separately.
4. Paid-sales definition changes: version the contract, communicate impact, update model/tests and reconcile old/new totals.
5. BI users calculate different order counts: publish one governed metric and document distinct-order logic.

## Submission checklist

- [ ] User/decision list.
- [ ] Approved metric contract and grain statements.
- [ ] Mart diagram.
- [ ] Schema inspection output.
- [ ] SQL query/view and actual result evidence.
- [ ] Sales reconciliation evidence and explanation.
- [ ] Dimension uniqueness/orphan checks.
- [ ] Refund modeling explanation.
- [ ] Ownership, freshness, security and limitations page.

**Reflection:** Explain why a mart is a business product—not merely a SELECT statement—and how you would prevent two teams from defining paid sales differently.
