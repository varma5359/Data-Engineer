# Day 5 — Data Pipeline Fundamentals
**Pathway:** TechPhile.ai Data Engineer | **Phase:** Foundations | **Level:** Beginner

## Learning objectives
Describe pipeline stages, reliability/scalability/fault tolerance, governance and security, and write basic pipeline requirements.

## 1. What is a data pipeline?
A data pipeline is a repeatable sequence of steps that moves data from one or more sources to a destination, applying required processing and controls.

Example:
`Extract orders → Store raw → Validate → Transform → Load warehouse → Check output`

A pipeline includes more than the transformation code: configuration, scheduling, state, permissions, logs, metrics, alerts, and recovery behavior.

## 2. Pipeline lifecycle
1. **Requirements:** business purpose, users, metrics, latency, volume, privacy.
2. **Source discovery:** owners, keys, access, schema, update pattern.
3. **Design:** ingestion pattern, storage, transformations, destination, failure behavior.
4. **Build:** code, configuration, tests, documentation.
5. **Validate:** row counts, constraints, reconciliation, business checks.
6. **Deploy:** controlled release and credentials.
7. **Operate:** schedule, monitor, alert, support.
8. **Improve/retire:** performance, cost, changes, retention.

## 3. Reliability
Reliability means the pipeline performs correctly and consistently over time, including when dependencies fail.

Practices:
- Retries for transient failures.
- Timeouts for external calls.
- Idempotent writes where possible.
- Checkpoints/watermarks for incremental work.
- Safe restart and backfill strategy.
- Validation before publishing.
- Alerting and clear ownership.

**Retry is not a complete recovery strategy.** If a job writes half its output and retries, it may duplicate or corrupt data unless writes are designed safely.

## 4. Scalability
Scalability is the ability to handle increasing data volume, throughput, or users without unacceptable degradation.

Approaches:
- Process in chunks/batches.
- Partition data appropriately.
- Push filtering/aggregation to capable systems.
- Parallelize independent work.
- Avoid unnecessary full reloads.
- Monitor bottlenecks before scaling.

More machines do not fix poor data modeling or a slow external API.

## 5. Fault tolerance
Fault tolerance is the ability to continue operating or recover when components fail.

Examples:
- Source API temporarily unavailable.
- Worker crashes.
- Network disconnects.
- Disk/storage issue.
- Invalid records appear.

Design responses: retry with backoff, checkpoint, isolate bad records, preserve raw input, alert, and resume from a known safe point.

## 6. Data governance
Governance defines how data is owned, understood, accessed, protected, and used.

Key concepts:
- Ownership/stewardship.
- Data definitions and glossary.
- Classification (public, internal, confidential, sensitive).
- Access policies.
- Quality expectations.
- Lineage and audit.
- Retention and deletion.
- Compliance with organizational/legal policy.

## 7. Security fundamentals
- Use least privilege: grant only required permissions.
- Keep credentials in secrets/environment configuration, not source code.
- Use encrypted connections and storage where required.
- Restrict sensitive raw data.
- Avoid logging passwords, tokens, or unnecessary personal data.
- Rotate/revoke credentials and audit access.
- Separate development/test/production environments.

## 8. Pipeline observability
A pipeline should expose:
- Status and duration.
- Records read, written, rejected.
- Data freshness/last successful timestamp.
- Error counts and error categories.
- Source/destination reconciliation.
- Retry count and resource usage.

Logs explain events; metrics show trends; alerts notify owners when action is needed.

## 9. Requirements template
For a daily sales pipeline, document:
- **Business purpose:** daily regional revenue report.
- **Sources:** sales API, customer DB.
- **Destination:** analytical warehouse.
- **Schedule/SLA:** publish by 8:30 AM local business time.
- **Volume:** expected records/day and growth.
- **Incremental key:** updated_at or source change token.
- **Quality:** unique order ID, valid amount/date, accepted status list.
- **Late data:** update affected dates on next run.
- **Failure:** retry transient errors; alert after final failure.
- **Recovery:** rerun date safely; no duplicate output.
- **Security:** service identity with read-only source and limited warehouse write access.
- **Monitoring:** freshness, counts, duration, rejected rows.
- **Owner:** named team/person and escalation path.

## 10. Mini-project — Pipeline Requirements Document
Write requirements for an orders pipeline using the template. Include normal run, empty source, malformed row, duplicate record, API timeout, schema change, and partial destination failure.

Deliverable: requirement document + architecture sketch + acceptance checks.

## 11. Common mistakes
- No defined business SLA or metric.
- No unique key or incremental strategy.
- Retrying non-transient errors forever.
- Publishing before validation completes.
- No safe backfill or rollback.
- Secrets in code or logs.
- No ownership or alert destination.
- Measuring only job success, not data freshness/correctness.

## Quick revision
- Reliability: consistent correct operation.
- Scalability: handle growth.
- Fault tolerance: survive/recover from failures.
- Governance: ownership, definitions, access, quality, retention.
- Production pipeline = code + controls + operations.
