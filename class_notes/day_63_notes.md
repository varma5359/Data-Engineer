# Day 63: Automated Data Quality

**Phase:** 7 — Orchestration, Quality and Monitoring  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and junior Data Engineer candidates  
**Technology:** Python validation logic orchestrated with Airflow  
**Topic boundary:** Class notes only. Project workbook and interview guide are separate.

> A pipeline can finish without an error and still deliver incorrect data. A file may be missing customer IDs, contain negative order amounts or include duplicate order IDs. Today you will learn how to turn business expectations into executable checks, classify failures, quarantine bad records and use Airflow quality tasks as a gate before publishing data.

## Learning outcomes

- Explain data quality in practical terms.
- Understand completeness, validity, uniqueness, consistency, accuracy and timeliness.
- Define a data contract.
- Build schema and business-rule checks.
- Check nulls, duplicates, ranges and allowed values.
- Distinguish warning from failure.
- Separate accepted and rejected records.
- Reconcile source and target counts/totals.
- Create a quality gate in an Airflow workflow.
- Understand why a successful task is not proof of correct data.
- Record validation results for audit and troubleshooting.

## 1. Start with a business problem

TechStore receives 10,000 order records.

The file loads successfully. Spark completes. The warehouse load reports success.

But the data contains:
- 300 rows without order_id;
- 120 rows with negative amounts;
- 45 duplicate order IDs;
- 80 rows with an invalid order status.

The pipeline technically ran, but the data is not ready for reporting.

**Data quality** means checking whether data is fit for its intended business use.

## 2. What is a data quality rule?

A data quality rule is a testable expectation about a dataset.

Examples:
- order_id must not be null;
- order_id must be unique at the order grain;
- amount must be greater than or equal to zero;
- order_status must be one of the approved values;
- order_date must be a valid date;
- customer_id must exist in the customer reference data.

A rule must be specific enough to evaluate.

Weak rule: "Orders should look correct."

Better rule: "Every accepted order row must have a non-null order_id."

## 3. Main dimensions of data quality

| Dimension | Beginner meaning | Example check |
|---|---|---|
| Completeness | Required data is present | order_id is not null |
| Validity | Values follow allowed rules | amount >= 0 |
| Uniqueness | Key is not duplicated at its defined grain | one row per order_id |
| Consistency | Related values agree | order total matches line totals |
| Accuracy | Data reflects the real-world value | compare to trusted source |
| Timeliness | Data arrives when expected | daily file arrives before deadline |

Accuracy is often difficult to prove from one dataset alone. A value can be valid in format but still be factually wrong.

## 4. Understand the grain before checking uniqueness

Grain means what one row represents.

If the table contains one row per order, order_id should be unique.

If the table contains one row per order item, order_id can appear multiple times. A suitable unique key might be order_id plus product_id or order_line_id.

Never apply a uniqueness rule without understanding the dataset grain.

## 5. What is a data contract?

A data contract describes the expectations between a data producer and consumer.

It may include:
- dataset name and owner;
- schema and data types;
- required/optional columns;
- nullability;
- allowed values;
- unique key and grain;
- freshness expectation;
- volume expectations;
- change-management rules.

A contract reduces surprises between source teams and data platform teams.

## 6. Sample TechStore data

Create this sample in Python:

~~~python
orders = [
    {"order_id": 101, "customer_id": "C001", "amount": 250.0, "status": "PAID"},
    {"order_id": 102, "customer_id": "C002", "amount": 499.0, "status": "PAID"},
    {"order_id": 103, "customer_id": None, "amount": 100.0, "status": "PENDING"},
    {"order_id": 104, "customer_id": "C003", "amount": -50.0, "status": "PAID"},
    {"order_id": 102, "customer_id": "C002", "amount": 499.0, "status": "PAID"},
    {"order_id": 105, "customer_id": "C004", "amount": 300.0, "status": "UNKNOWN"}
]

print("Input rows:", len(orders))
~~~

Expected output:

~~~text
Input rows: 6
~~~

We intentionally inserted problems so that the checks have something to detect.

## 7. Check required fields

~~~python
required_fields = ["order_id", "customer_id", "amount", "status"]

missing_field_rows = []

for row_number, row in enumerate(orders, start=1):
    missing_fields = []

    for field in required_fields:
        if row.get(field) is None:
            missing_fields.append(field)

    if missing_fields:
        missing_field_rows.append({
            "row_number": row_number,
            "missing_fields": missing_fields
        })

print("Rows with missing required values:", len(missing_field_rows))
print(missing_field_rows)
~~~

Expected output:

~~~text
Rows with missing required values: 1
[{'row_number': 3, 'missing_fields': ['customer_id']}]
~~~

Explanation:
- enumerate gives each input row a position beginning at 1;
- row.get(field) reads the field safely;
- None indicates a missing value in this sample;
- missing_field_rows stores the row number and missing field names.

In real CSV ingestion, empty strings may also represent missing values. Normalize them before checking.

## 8. Check negative amounts

~~~python
negative_amount_rows = []

for row_number, row in enumerate(orders, start=1):
    if row["amount"] < 0:
        negative_amount_rows.append(row_number)

print("Negative amount rows:", negative_amount_rows)
~~~

Expected output:

~~~text
Negative amount rows: [4]
~~~

A negative amount may be invalid for a simple order table, but some financial datasets use negative values for refunds or reversals. Business rules must reflect the real meaning.

## 9. Check duplicate order IDs

~~~python
seen_order_ids = set()
duplicate_order_ids = set()

for row in orders:
    order_id = row["order_id"]

    if order_id in seen_order_ids:
        duplicate_order_ids.add(order_id)
    else:
        seen_order_ids.add(order_id)

print("Duplicate order IDs:", sorted(duplicate_order_ids))
~~~

Expected output:

~~~text
Duplicate order IDs: [102]
~~~

A set stores unique values. When an ID is already present, we know it appeared earlier.

This check assumes one row per order. For an order-line table, use the correct composite key.

## 10. Check allowed status values

~~~python
allowed_statuses = {"PAID", "PENDING", "CANCELLED"}

invalid_status_rows = []

for row_number, row in enumerate(orders, start=1):
    if row["status"] not in allowed_statuses:
        invalid_status_rows.append(row_number)

print("Invalid status rows:", invalid_status_rows)
~~~

Expected output:

~~~text
Invalid status rows: [6]
~~~

The allowed values should come from a documented business contract, not a guess in the code.

## 11. Create accepted and rejected records

A quality process should define what happens to invalid rows.

Possible policies:
- fail the entire batch;
- quarantine invalid records and continue with accepted rows;
- warn but continue for non-critical rules;
- route the data to a manual review process.

For this classroom example, we will quarantine rows that violate any critical rule.

~~~python
accepted_orders = []
rejected_orders = []

for row_number, row in enumerate(orders, start=1):
    errors = []

    if row["order_id"] is None:
        errors.append("order_id is missing")

    if row["customer_id"] is None:
        errors.append("customer_id is missing")

    if row["amount"] < 0:
        errors.append("amount is negative")

    if row["status"] not in allowed_statuses:
        errors.append("status is not allowed")

    if errors:
        rejected_orders.append({
            "row_number": row_number,
            "record": row,
            "errors": errors
        })
    else:
        accepted_orders.append(row)

print("Accepted:", len(accepted_orders))
print("Rejected:", len(rejected_orders))
~~~

Expected output for these field, range and status checks:

~~~text
Accepted: 3
Rejected: 3
~~~

At this stage, the rejected rows are:
- row 3: customer_id is missing;
- row 4: amount is negative;
- row 6: status is not allowed.

Row 5 is still accepted here because this first combined example has not yet checked duplicate order IDs. The next section adds duplicate detection and changes the final accounting to 2 accepted and 4 rejected.

## 12. Add duplicate detection to the combined validation

~~~python
seen_ids = set()
accepted_orders = []
rejected_orders = []

for row_number, row in enumerate(orders, start=1):
    errors = []

    if row["order_id"] is None:
        errors.append("order_id is missing")
    elif row["order_id"] in seen_ids:
        errors.append("duplicate order_id")
    else:
        seen_ids.add(row["order_id"])

    if row["customer_id"] is None:
        errors.append("customer_id is missing")

    if row["amount"] < 0:
        errors.append("amount is negative")

    if row["status"] not in allowed_statuses:
        errors.append("status is not allowed")

    if errors:
        rejected_orders.append({
            "row_number": row_number,
            "record": row,
            "errors": errors
        })
    else:
        accepted_orders.append(row)

print("Accepted:", len(accepted_orders))
print("Rejected:", len(rejected_orders))
~~~

Expected output:

~~~text
Accepted: 2
Rejected: 4
~~~

Now the duplicate row is explicitly rejected. The first occurrence of order_id 102 remains accepted; the second is rejected.

For production, the duplicate policy must be explicit. Some sources require keeping the latest record rather than the first one.

## 13. Completeness ratio

A completeness metric can describe the percentage of records with required fields populated.

For this sample, required-field completeness is not the same as overall record acceptance.

~~~python
complete_rows = 0

for row in orders:
    if all(row.get(field) is not None for field in required_fields):
        complete_rows += 1

completeness_percent = (complete_rows / len(orders)) * 100

print("Complete rows:", complete_rows)
print("Completeness percent:", round(completeness_percent, 2))
~~~

Expected output:

~~~text
Complete rows: 5
Completeness percent: 83.33
~~~

The duplicate, negative amount and invalid status rows can still be complete with respect to null checks.

## 14. Reconcile source, accepted and rejected counts

If each input row is either accepted or rejected exactly once:

~~~python
input_count = len(orders)
accepted_count = len(accepted_orders)
rejected_count = len(rejected_orders)

print("Input:", input_count)
print("Accepted:", accepted_count)
print("Rejected:", rejected_count)
print("Counts reconcile:", input_count == accepted_count + rejected_count)
~~~

Expected output:

~~~text
Input: 6
Accepted: 2
Rejected: 4
Counts reconcile: True
~~~

This check confirms row accounting. It does not prove that accepted records are correct.

## 15. Schema validation

Schema validation checks:
- required column names;
- unexpected columns;
- data types;
- nullable behavior;
- column order, if order is part of the contract.

Example using a Python dictionary as a simplified schema description:

~~~python
expected_types = {
    "order_id": int,
    "customer_id": str,
    "amount": float,
    "status": str
}

sample_row = orders[0]

for column_name, expected_type in expected_types.items():
    actual_value = sample_row[column_name]
    print(
        column_name,
        isinstance(actual_value, expected_type)
    )
~~~

Expected output:

~~~text
order_id True
customer_id True
amount True
status True
~~~

Be careful: Python's bool is a subclass of int, and real CSV readers may infer different types. Production schema validation should use explicit parsing and clear type rules.

## 16. Cross-dataset consistency

Some rules need more than one dataset.

Example:
- every order.customer_id should exist in customer reference data;
- every order.product_id should exist in product reference data.

This is a referential integrity check.

A missing customer may mean:
- source data is wrong;
- customer file arrived late;
- customer was deleted;
- identifiers are inconsistent.

Do not automatically delete unmatched records without a business decision.

## 17. Reconciliation checks

Reconciliation compares source and target results.

Examples:
- input count versus accepted plus rejected count;
- source sales amount versus transformed sales amount;
- number of distinct order IDs;
- target partition date versus expected date.

Example:

~~~python
source_total = sum(
    row["amount"]
    for row in accepted_orders
)

print("Accepted amount total:", source_total)
~~~

Use decimal arithmetic for financial values in production rather than binary floating-point calculations.

## 18. Warning versus failure

Not every rule needs to stop the pipeline.

| Severity | Example | Action |
|---|---|---|
| Critical | Required key missing | Fail or quarantine according to contract |
| High | Duplicate business key | Fail or quarantine |
| Medium | Unusual daily volume drop | Warn and investigate |
| Low | Optional description missing | Record metric; continue |

Define thresholds with business owners. A 20% volume drop may be normal on a holiday and serious on an ordinary weekday.

## 19. What is a quality gate?

A quality gate is a workflow step that decides whether data is allowed to move to the next stage.

~~~text
Ingest -> Transform -> Quality checks
                         |
                  +------+------+
                  |             |
                PASS           FAIL
                  |             |
                  v             v
              Publish       Quarantine
                              + Alert
~~~

A quality gate should run before publishing data to consumers.

## 20. Add a quality task to an Airflow DAG

Example using TaskFlow API:

~~~python
import pendulum

from airflow.decorators import dag, task


@dag(
    dag_id="day63_quality_gate",
    start_date=pendulum.datetime(2026, 1, 1, tz="UTC"),
    schedule=None,
    catchup=False
)
def quality_gate_workflow():

    @task
    def transform_data():
        print("Transformation completed")
        return {
            "input_count": 100,
            "accepted_count": 98,
            "rejected_count": 2
        }

    @task
    def validate_counts(result):
        expected_count = result["accepted_count"] + result["rejected_count"]

        if result["input_count"] != expected_count:
            raise ValueError("Input/accepted/rejected counts do not reconcile")

        print("Count reconciliation passed")

    @task
    def publish_data():
        print("Curated data published")

    result = transform_data()
    validate_counts(result) >> publish_data()


quality_gate_workflow()
~~~

The validation task raises an exception when the check fails. Under the normal success dependency rule, publish_data will not execute.

The example passes only small counts through task metadata. Actual datasets remain in storage.

## 21. What happens when a quality check fails?

The operator should:
1. inspect the validation task log;
2. identify the failed rule;
3. determine whether the source or transformation is responsible;
4. preserve rejected records for investigation;
5. notify the correct owner;
6. correct the issue;
7. rerun safely;
8. verify the quality gate passes before publication.

Do not simply disable a failing check to make the DAG green.

## 22. Quality results should be recorded

A useful validation result includes:
- dataset name;
- run ID;
- rule ID;
- rule description;
- severity;
- input count;
- failed count;
- pass/fail status;
- execution time;
- sample error reference;
- owner.

Avoid storing sensitive record values in logs. Store controlled references to quarantined data.

## 23. Common mistakes

| Mistake | Why it is wrong | Better approach |
|---|---|---|
| Check only row count | Wrong values can still have same count | Validate schema and business rules |
| Apply uniqueness without grain | Valid line-item rows may repeat order ID | Define row meaning first |
| Treat every negative amount as invalid | Refunds may be negative | Confirm business semantics |
| Drop invalid rows silently | Data loss becomes invisible | Quarantine and report |
| Use warnings for critical failures | Bad data may be published | Define severity and gates |
| Disable a failing check | Hides the defect | Fix the cause or approve rule change |
| Assume task success means quality | Technical execution is not correctness | Add explicit quality tasks |
| Log sensitive records | Creates privacy/security risk | Log rule IDs and safe references |

## 24. Knowledge check

**Q1. What is data quality?**  
How well data meets defined requirements for its intended use.

**Q2. What is completeness?**  
Whether required values are present.

**Q3. What is uniqueness?**  
Whether a defined key is unique at the table's grain.

**Q4. Why define grain first?**  
Because repeated values may be valid when one row represents an order line rather than an order.

**Q5. What is a data contract?**  
An agreement describing schema, keys, rules, ownership and expectations for a dataset.

**Q6. What is quarantine?**  
Separating rejected records so they can be investigated without silently publishing them.

**Q7. What is a quality gate?**  
A validation step that controls whether data can proceed to publication.

**Q8. Why reconcile counts?**  
To ensure records are accounted for between input, accepted and rejected outputs.

## Final self-check

- [ ] Explain data quality dimensions.
- [ ] Define a data contract.
- [ ] Identify the correct grain and unique key.
- [ ] Check required values, ranges and allowed values.
- [ ] Detect duplicate IDs.
- [ ] Separate accepted and rejected records.
- [ ] Reconcile input, accepted and rejected counts.
- [ ] Explain warning versus failure.
- [ ] Add a quality task to a DAG.
- [ ] Prevent publication after a failed quality gate.
- [ ] Record validation results safely.

**Next lesson — Day 64: Pipeline Monitoring and Alerting.** You will learn how to observe workflow health, define useful metrics, configure failure notifications and build an operational dashboard.
