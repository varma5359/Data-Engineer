# Day 40 Interview Guide — Data Warehouse Architecture

This guide is separate from the Day 40 class notes. Answers are written as complete responses a fresher can practice and adapt.

## 1. What is a data warehouse?

**Model answer:**

A data warehouse is a managed analytical data environment that collects data from multiple business sources, integrates it using consistent rules, retains the history needed for analysis and makes trusted data available for reporting and decision-making. For example, an e-commerce company may combine orders, customers, products, payments and refunds so managers can analyze sales by month, category and city. A warehouse is more than a copy of operational tables: it needs data models, quality checks, security, documentation and operational monitoring.

**Follow-up:** Why is a warehouse not simply a database backup?

A backup is primarily intended to restore a system. A warehouse is designed to integrate and organize data for analysis. Their purposes, structures and access patterns differ.

## 2. Explain OLTP and OLAP.

**Model answer:**

OLTP stands for Online Transaction Processing. It supports day-to-day application operations such as inserting an order, updating a customer address or recording a payment. These operations are usually frequent and relatively small.

OLAP stands for Online Analytical Processing. It supports analysis such as calculating monthly revenue, comparing product categories or measuring customer trends. OLAP queries often scan and aggregate larger volumes of data.

For example, saving one new order is OLTP; calculating total paid sales by month across several years is OLAP. The distinction is about workload and design goals, not a claim that a database can only do one type of work.

**Common trap:** Saying OLTP is always normalized and OLAP is always denormalized. These are common design tendencies, not universal rules.

## 3. Why should analytical queries be separated from operational workloads?

**Model answer:**

Operational applications need predictable response times for customer-facing transactions. Large analytical queries can consume CPU, memory, I/O and connections. Running heavy reports on the same system can affect application performance. A separate analytical environment helps isolate workloads, integrate multiple sources and retain history. The separation also allows warehouse models to be designed around business questions rather than application write operations.

**Follow-up:** Is a separate warehouse always required?

Not for every small system. The decision depends on workload size, source count, reporting needs, governance and cost. A small application may begin with reporting views, but growing analytical workloads often justify a separate platform.

## 4. Explain the common warehouse layers.

**Model answer:**

A common architecture includes source systems, ingestion, raw, staging, core/integration, presentation and BI layers.

- Source systems create operational data.
- Ingestion extracts and moves it.
- Raw preserves source-aligned data and ingestion metadata.
- Staging standardizes names, types and basic structures.
- Core integrates entities and applies shared business rules.
- Presentation exposes facts, dimensions and subject-area models.
- BI tools use approved models for dashboards and analysis.

The exact number and names of layers vary by organization. The important part is that responsibilities are clear and data can be traced.

## 5. What is the difference between a data warehouse and a data mart?

**Model answer:**

A data warehouse is a broader analytical environment that integrates data across business areas. A data mart is a focused analytical area for a subject or team, such as sales, finance or marketing. A data mart can be implemented as a schema, a set of models or a logical area inside the warehouse; it does not always need a separate physical database.

## 6. Compare a data lake, warehouse and lakehouse.

**Model answer:**

A data lake stores data in many formats and is often built on object storage. A data warehouse provides governed, integrated analytical data and commonly exposes structured models for reporting. A lakehouse combines flexible lake-style storage with managed table and analytical capabilities. The exact capabilities depend on the platform. None of these architectures removes the need for ownership, quality, security and cost controls.

## 7. Explain ETL and ELT.

**Model answer:**

ETL means Extract, Transform, Load. Data is extracted, transformed before reaching the target, and then loaded. ELT means Extract, Load, Transform. Data is first loaded into the target platform, often into a raw area, and transformed there using the platform's compute. ELT is common in modern analytical platforms with scalable SQL engines, while ETL can be useful when data must be transformed before loading or when the target has limited transformation capability. A real architecture may use both patterns.

## 8. What is dimensional modeling?

**Model answer:**

Dimensional modeling organizes analytical data around measurable business events and descriptive context. Fact tables represent events or measurements, while dimension tables describe entities such as customers, products and dates. For a sales report, a sales fact may contain quantity and amount, while product, customer and date dimensions provide category, city and calendar descriptions. The grain must be declared before the fact table is designed because it determines what one row means.

## 9. What is grain and why is it important?

**Model answer:**

Grain is the precise meaning of one row in a fact table. For example, one row may represent one order or one order item. If an order has three items, an order-level fact has one row while an order-item-level fact has three rows. If an order-level amount is joined to three item rows and summed, the amount can be counted three times. Defining grain first prevents double-counting and guides key and measure design.

## 10. What is dbt and where does it fit?

**Model answer:**

dbt is a tool for managing SQL transformations in supported data platforms. It organizes SQL models, dependencies, tests and documentation. In a common architecture, an ingestion tool first loads source data into the warehouse or analytical platform. dbt then builds staging, intermediate and presentation models. dbt is not a universal replacement for API extraction, CDC, file ingestion or every orchestration requirement.

## 11. Explain dbt source and ref.

**Model answer:**

A dbt source declares an upstream relation such as a raw database table. A model can use the source function to refer to that declared input. The ref function refers to another dbt model and tells dbt about the dependency. This dependency information helps dbt determine build order and generate lineage.

## 12. What is dbt materialization?

**Model answer:**

Materialization determines how dbt builds a model in the target platform. A view stores a query definition, a table stores the result, an incremental model processes new or changed data according to its logic, and an ephemeral model is incorporated into downstream SQL rather than built as a standalone relation. The choice depends on volume, query performance, refresh behavior and platform capabilities. Incremental logic must be tested for updates, late-arriving rows and duplicate handling.

## 13. A dashboard shows a successful refresh, but payment data did not arrive. What would you do?

**Model answer:**

I would not treat the successful job status as proof that the dashboard is complete. I would check source freshness and completeness, compare the expected source delivery with the actual ingestion run, and inspect row counts and timestamps. The pipeline should have a required-source check that fails or marks the output incomplete when a critical feed is missing. The dashboard should expose its last successful complete refresh. I would alert the source owner and avoid publishing misleading totals.

## 14. How would you prevent inconsistent sales definitions?

**Model answer:**

I would work with the business owner to define sales precisely: whether it means orders placed, paid orders, shipped orders or recognized revenue; how cancellations and refunds are treated; and which date determines the reporting period. I would implement the approved definition in a shared warehouse model, document it and test it. Reports should reuse that governed model rather than independently recreating the calculation.

## 15. How would you design a warehouse for an e-commerce sales dashboard?

**Model answer:**

I would first clarify metrics, history requirements, freshness and users. Then I would inventory order, order-item, customer, product, payment and refund sources. I would land source data in a raw area with run metadata, standardize it in staging, integrate shared business entities in a core layer, and expose sales facts and dimensions in a sales mart. I would define the fact grain before selecting measures, validate keys and totals, reconcile payments and refunds, document lineage, secure sensitive customer fields and monitor source freshness and pipeline runs.

## 16. What are common data warehouse architecture mistakes?

**Model answer:**

Common mistakes include copying source tables directly into reports without shared definitions, treating raw data as trusted, failing to define fact grain, ignoring late-arriving data, losing historical changes, omitting reconciliation, exposing sensitive fields, and failing to document ownership. These issues can create inaccurate metrics and make failures difficult to investigate. I would prevent them with clear layer responsibilities, business-approved definitions, data contracts, tests, lineage, access control and monitoring.

## 17. What should a junior engineer clarify before designing a warehouse?

**Model answer:**

I would ask what business questions must be answered, how each metric is defined, which source systems own the data, what keys and update patterns exist, how much history is required, how fresh the data must be, what volume is expected, which fields are sensitive, who owns each dataset and what recovery expectations apply. These answers guide the architecture and prevent assumptions from becoming hidden business rules.

## 18. Scenario: the source changes a column from integer to text. What happens?

**Model answer:**

The ingestion or staging process may fail, or the transformation may produce invalid values. I would inspect the source schema change, identify affected records and downstream models, and determine whether the change is valid and approved. I would preserve the raw input, update the schema mapping and tests where appropriate, and communicate downstream impact. I would not silently cast invalid values to zero or discard them.

## 19. Scenario: monthly revenue doubles after a model change. How do you investigate?

**Model answer:**

I would compare row counts and totals before and after the change, inspect the model's grain and joins, and look for one-to-many join fan-out or duplicate source records. I would validate the business filters for payment status, cancellations and refunds. I would reconcile totals against a trusted source and identify the exact transformation that changed the result before publishing the model.

## 20. How do you explain the Day 40 architecture project?

**Model answer:**

I began with the sales dashboard requirements and listed the source systems: orders, order items, customers, products and payments/refunds. I proposed a layered architecture with raw, staging, core and presentation responsibilities. I identified candidate facts and dimensions, listed freshness and reconciliation controls, and documented risks such as missing payment feeds, duplicate order lines and overwritten customer attributes. I kept unresolved business definitions visible rather than guessing. The next step is to define the fact grain and physical dimensional model.

## Final interview self-check

A strong answer should:

- define the concept accurately;
- explain why it matters;
- give a business example;
- identify a risk or trade-off;
- describe how a Data Engineer validates the result.

Do not memorize answers word for word. Practice explaining the reasoning in your own language.
