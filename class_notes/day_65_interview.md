# Day 65 Interview Guide — Data Pipeline Security

## 1. What is authentication?
Authentication verifies the identity of a user or workload, using mechanisms such as passwords, tokens, certificates or workload identity.

## 2. What is authorization?
Authorization determines which actions an authenticated identity can perform on specific resources.

## 3. What is a secret?
Sensitive information used to authenticate or protect access, such as a password, API token, private key or client secret.

## 4. Why should credentials not be hardcoded?
They can be committed to source control, copied into notebooks, exposed in reviews or retained in repository history. They are difficult to rotate safely.

## 5. What are environment variables?
Values supplied to a process through its runtime environment. They help inject configuration without source-code hardcoding, but are not automatically a complete production secrets solution.

## 6. How should an application read a required secret?
Read it from an approved runtime secret source, verify it is present and fail clearly if missing. Never print it or include it in exception messages.

## 7. What is an Airflow Connection?
A structured record for external system details such as type, host, schema, login and password. DAG code can refer to a connection ID instead of embedding credentials.

## 8. Connection versus Variable?
A Connection models access to an external system. A Variable stores general configuration. Secrets should use approved secure storage rather than being placed casually in generic variables.

## 9. What is a secrets backend?
An integration that lets Airflow retrieve secrets from an external managed store or vault, centralizing access control, audit and rotation.

## 10. What is IAM?
Identity and Access Management controls identities and permissions in a cloud environment. Policies specify actions an identity can perform on resources.

## 11. What is least privilege?
Grant only permissions required for a task. An ETL workload that reads one source and writes one curated prefix should not receive administrator access to the entire account.

## 12. IAM user versus role?
A user is a long-lived identity in some patterns. A role can be assumed by an authorized principal and commonly provides temporary credentials. Cloud workloads generally benefit from role/workload identity instead of embedded static keys.

## 13. Why are temporary credentials useful?
They expire and can be refreshed through an approved identity mechanism, reducing reliance on long-lived keys.

## 14. How do you secure a database connection?
Use private network access, a least-privilege account, TLS with certificate validation, secure credential retrieval, timeouts, rotation and audit logging.

## 15. What if a secret is committed to Git?
Treat it as exposed. Revoke or rotate it immediately, inspect audit logs, remove it from current code, review repository history/access, migrate to managed secrets and add scanning. Deleting the line alone does not invalidate the credential.

## 16. Why is DAG write access sensitive?
DAG code can execute with permissions available to the task environment. Restrict who can modify DAGs and deploy workflow code.

## 17. Scenario: a developer requests administrator access so ETL can write to S3.
Identify the exact bucket/prefix and grant only required actions there. Avoid account-wide administrator access.

## 18. Scenario: the password is missing from the environment.
Fail safely with a clear configuration error. Do not use a default password or print secrets. Verify the injection path and permissions.

## 19. Scenario: production credentials are reused in development.
Separate identities and secrets. Development should use non-production resources so a test environment cannot modify production data.

## 20. Explain your Day 65 project.
I classified configuration and secrets, created a safe environment-variable example, designed an Airflow Connection and secrets-backend approach, created a least-privilege permission matrix and documented network, encryption, rotation and leaked-secret response. I checked that no real credentials appear in submitted code or logs.

## Common interview traps
- Confusing authentication with authorization.
- Saying environment variables are perfectly secure.
- Giving administrator permissions to a pipeline.
- Reusing production credentials in development.
- Deleting a leaked key without rotating it.
- Printing secrets in logs.
- Ignoring DAG write access.

## Readiness checklist
- [ ] Explain authentication and authorization.
- [ ] Explain environment variables and secrets backends.
- [ ] Explain Airflow Connections.
- [ ] Explain IAM roles and temporary credentials.
- [ ] Apply least privilege.
- [ ] Describe secure database access.
- [ ] Explain secret rotation and incident response.
