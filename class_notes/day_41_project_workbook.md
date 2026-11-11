# Day 41 Project Workbook — Sales Star Schema

## Goal

Build and validate a small sales Star Schema in PostgreSQL. The class notes explain the concepts; this workbook guides the deliverable.

## Business requirement

TechStore wants sales by month, product category and customer city.

Training definition: paid order-line sales, using order date. Refunds are excluded from this first model and must be modeled separately when required.

## Step 1 — Write the grain

Complete this sentence before creating tables:

**One row in fact_sales represents:** one product line within one order.

If your design says one row per order, stop and explain how you will report products from different categories inside the same order.

## Step 2 — Identify measures and dimensions

| Type | Candidate | Purpose |
|---|---|---|
| Fact measure | quantity | Units on the order line |
| Fact measure | unit_price | Selling price per unit |
| Fact measure | line_sales_amount | Amount for the line |
| Dimension | date | Calendar analysis |
| Dimension | product | Category and brand |
| Dimension | customer | City and customer description |
| Degenerate identifier | order_id | Drill-through to order |

## Step 3 — Build the schema

Use the PostgreSQL SQL cells in Day 41 class notes to create:

- dim_date;
- dim_product;
- dim_customer;
- fact_sales.

Run all statements in the same database session because the examples use temporary tables.

## Step 4 — Insert the training rows

Use the sample rows from the class notes. Confirm:
- three date rows;
- three product rows;
- three customer rows;
- four sales fact rows.

## Step 5 — Run analytical queries

Run and save the outputs for:
1. sales by product category;
2. sales by customer city;
3. monthly sales;
4. total sales.

Expected totals:
- Accessories: 5,500.00
- Electronics: 50,000.00
- Bengaluru: 1,500.00
- Hyderabad: 54,000.00
- January: 53,100.00
- February: 2,400.00
- Overall: 55,500.00

## Step 6 — Validate

Run:
- duplicate dimension key check;
- fact-to-dimension orphan check;
- line amount calculation check;
- overall total check.

Expected result for exception queries: zero rows.

## Step 7 — Draw the model

Submit a Star Schema diagram with:
- fact_sales in the center;
- dim_date;
- dim_product;
- dim_customer;
- key relationships;
- declared grain;
- measures.

## Step 8 — Compare with a Snowflake design

Create a second diagram in which product category is separated into its own table.

Write a short comparison:
- number of joins;
- repeated attributes;
- analyst usability;
- maintenance trade-off.

## Step 9 — Failure drills

Explain how you would investigate:
1. Category sales are twice the expected value.
2. A fact row refers to a missing product.
3. An order has four lines but only one appears.
4. A report sums unit price and produces a meaningless number.

## Submission checklist

- [ ] Grain statement.
- [ ] Fact and dimension inventory.
- [ ] SQL creation and insert scripts.
- [ ] Query outputs.
- [ ] Validation outputs.
- [ ] Star Schema diagram.
- [ ] Snowflake comparison.
- [ ] Failure-drill answers.
- [ ] README explaining assumptions and limitations.

Do not include real customer personal information or production credentials.
