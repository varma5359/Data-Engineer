# Day 64 Interview Guide — Pipeline Monitoring and Alerting

## 1. What is pipeline monitoring?
It is the ongoing collection and review of workflow and data health information, such as run status, duration, freshness, volume, quality results and failure counts.

## 2. What is observability?
Observability is the ability to understand a system's internal behavior from logs, metrics and traces. It helps investigate unexpected behavior beyond known dashboard thresholds.

## 3. Difference between logs and metrics?
Logs are timestamped event records, such as "loaded 10,000 rows." Metrics are numeric measurements over time, such as task duration or rejected-row count. Logs explain events; metrics show trends and thresholds.

## 4. What are traces?
Traces connect operations across components and show where time is spent, for example an API call, Spark job and warehouse write within one processing flow.

## 5. What is data freshness?
Freshness measures how recent a dataset is compared with its expected update schedule. A DAG can be green but stale if it processed an older data interval.

## 6. Which metrics would you monitor?
DAG/task status, last successful interval, freshness, duration, input/output rows, rejected records, retries, source arrival delay and quality-gate status. For streaming, also consumer lag.

## 7. Why monitor task duration?
A sudden increase may indicate larger input, source slowness, skew, resource contention or a performance regression. Trends can reveal problems before deadlines are missed.

## 8. What is alert fatigue?
Too many noisy or low-value alerts cause responders to ignore notifications. Reduce it with actionable conditions, thresholds, severity, grouping and clear ownership.

## 9. What should a failure alert contain?
Pipeline/task IDs, run ID, data interval, failure time, severity, safe error summary, link to logs and responsible owner/team.

## 10. What is a runbook?
A documented procedure for handling an operational event. It explains investigation, mitigation, recovery and verification.

## 11. Why use structured, contextual logs?
Fields such as pipeline name, run ID, task ID and source help connect events to one execution and make searching/aggregation easier.

## 12. Why should logs avoid secrets?
Logs may be broadly accessible and retained for a long time. Exposing passwords, tokens or personal data creates security risk. Log safe identifiers and redact sensitive values.

## 13. Monitoring versus alerting?
Monitoring collects and displays health information. Alerting notifies people when a defined condition requires action.

## 14. How can Airflow help?
Airflow provides DAG run history, task states, duration, retries and logs. Deployments can export metrics to external monitoring platforms.

## 15. What is a failure callback?
A function configured for certain failure events. It can send notifications through an approved provider. It must be tested and must not expose secrets.

## 16. Scenario: DAG is green but the report is yesterday's data.
Inspect the latest successful data interval and source freshness, not just DAG status. Validate output business date and publication status. The workflow may have processed an old interval or skipped the latest source.

## 17. Scenario: alerts fire for every transient retry.
Distinguish recovered retries from terminal failures. Alert on actionable conditions, group repeated events and retain retry metrics for trends.

## 18. Scenario: daily row count drops by 80%.
Compare source arrival, historical volume, business calendar, extraction filters and rejected counts. Treat the drop as an anomaly signal, not proof of an error, and block publication if the approved threshold requires it.

## 19. Explain your Day 64 project.
I defined operational expectations, selected metrics for status/freshness/duration/volume/quality, designed a dashboard, wrote actionable alert rules and created a runbook. I simulated a failure and recorded detection, investigation, recovery and prevention evidence.

## Common interview traps
- Monitoring only whether the DAG is green.
- Treating logs and metrics as interchangeable.
- Alerting on every event.
- Using thresholds without business context.
- Logging secrets or sensitive records.
- Not assigning alert ownership.
- Having no recovery runbook.

## Readiness checklist
- [ ] Explain logs, metrics and traces.
- [ ] Explain freshness, duration and volume.
- [ ] Design actionable alerts.
- [ ] Explain alert fatigue.
- [ ] Describe Airflow monitoring.
- [ ] Write a runbook and incident timeline.
- [ ] Explain safe logging.
