# Day 40 Project Workbook — Warehouse Architecture Plan

## Project goal

Design a beginner-level warehouse architecture for TechStore's monthly sales reporting requirement.

This workbook is a guided deliverable. It is separate from the Day 40 class notes and interview guide.

## Business request

The sales manager needs a dashboard showing:

- paid sales by month;
- paid order count;
- sales by product category;
- sales by customer city;
- refunds;
- last successful refresh time.

Sources:

1. Order database.
2. Order item table.
3. Customer database.
4. Product catalog.
5. Payment and refund system.

## Step 1 — Write down the business definitions

Complete this table with the business owner. Do not guess.

| Question | Approved definition |
|---|---|
| What counts as a sale? | Pending orders excluded; confirm paid/settled rule |
| Are cancelled orders included? | Confirm with business |
| How are refunds represented? | Show separately and define net-sales calculation |
| Which date defines the sales month? | Confirm order, payment or recognized-revenue date |
| Which customer city is shown? | Current city or city at order time |
| What is the required refresh time? | Example: before 08:00 local time |

**Evidence:** Save the approved definitions in a short requirements document.

## Step 2 — Inventory source systems

Create a source inventory.

| Source | Owner | Extraction method | Primary identifier | Refresh expectation |
|---|---|---|---|---|
| Orders | Order application team | Database incremental extract | order_id | Daily or more frequent |
| Order items | Order application team | Database incremental extract | order_item_id | Daily or more frequent |
| Customers | Customer application team | Database/API | customer_id | Confirm |
| Products | Catalog team | Database/file | product_id | Confirm |
| Payments/refunds | Payment team | API/database | payment_id | Confirm |

Replace example values with actual source details.

## Step 3 — Draw the architecture

Use this flow as a starting point:

`Source systems -> Ingestion -> Raw -> Staging -> Core -> Sales mart -> BI`

Add operational controls around the flow:

- source freshness;
- row-count reconciliation;
- schema validation;
- run ID and logs;
- failure alert;
- access permissions;
- secrets management;
- retention policy.

**Evidence:** Submit one architecture diagram and a paragraph explaining each layer.

## Step 4 — Create a layer responsibility matrix

| Layer | What belongs here? | What should not happen here? |
|---|---|---|
| Raw | Source-aligned data and ingestion metadata | Unapproved business metric definitions |
| Staging | Type conversion, naming, basic standardization | Hidden business assumptions |
| Core | Integrated business entities and approved rules | Report-specific duplicated logic |
| Presentation | Facts, dimensions and subject-area models | Uncontrolled source-specific complexity |
| BI | Reports and approved measures | Reimplementing source integration logic |

Adapt this to the chosen platform and explain any additional layers.

## Step 5 — Identify candidate facts and dimensions

For the sales dashboard, propose:

- candidate fact: sales/order-line event;
- candidate dimensions: date, product, customer, channel;
- possible refund fact or refund event model.

Do not finalize the grain without analyzing the source and business definitions. Day 41 will complete the fact/dimension design.

## Step 6 — Record risks and controls

| Risk | Impact | Proposed control |
|---|---|---|
| Payment data late | Incomplete dashboard | Freshness and completeness check |
| Duplicate order lines | Overstated sales | Business-key uniqueness test |
| Refund omitted | Incorrect net sales | Separate refund model and reconciliation |
| Customer city overwritten | Lost historical context | Approved historical dimension policy |
| Source schema changes | Broken transformation | Schema validation and change process |

Add at least three risks of your own.

## Step 7 — Write an architecture decision record

Use this format:

### Decision
What architecture decision are you making?

### Context
Which business requirement or technical constraint led to it?

### Options considered
Which alternatives did you consider?

### Chosen option
What did you choose?

### Reason
Why is this appropriate?

### Consequences
What operational cost, limitation or risk does the decision introduce?

## Step 8 — Acceptance checklist

- [ ] Business questions and metric definitions are recorded.
- [ ] All five source systems are listed.
- [ ] Source owners and extraction methods are identified or marked pending.
- [ ] Raw, staging, core and presentation responsibilities are explained.
- [ ] Candidate facts and dimensions are listed.
- [ ] Freshness, quality, security and lineage controls are included.
- [ ] At least five risks and controls are documented.
- [ ] Architecture diagram is readable.
- [ ] Open questions are clearly marked instead of guessed.

## Final submission

Submit:

1. Architecture diagram.
2. Source inventory.
3. Layer responsibility matrix.
4. Candidate fact/dimension list.
5. Risk and control register.
6. Architecture decision record.

**Do not submit passwords, API tokens, personal customer data or production connection strings.**
