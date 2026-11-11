# Day 53 Interview Guide — Cloud Database Fundamentals

## 1. What is a managed database?

A managed database is a service where the provider manages parts of the underlying infrastructure and routine operations. The customer still owns schema design, data access, application connectivity, performance choices, backup configuration and cost decisions. The exact responsibility boundary depends on the service.

## 2. What is Amazon RDS?

Amazon Relational Database Service is an AWS managed relational database service supporting multiple database engines. It helps teams provision and operate relational databases without managing every infrastructure layer themselves.

## 3. What is Amazon Aurora?

Aurora is a relational database engine/service offering compatible with selected MySQL and PostgreSQL workloads. It has its own architecture and compatibility behavior, so engineers should validate features and extensions before migrating from standard database engines.

## 4. What is a database endpoint?

An endpoint is the network hostname or address used by a client to reach a database service. It is not the database name, username or password.

## 5. What is a database port?

A port is a numbered communication endpoint used by a service. PostgreSQL commonly uses 5432 and MySQL commonly uses 3306, but the configured value must be checked.

## 6. What is the difference between a database endpoint and database name?

The endpoint identifies the network service to connect to. The database name identifies the logical database within that service. A client may need both.

## 7. What is a VPC?

A VPC is a logically isolated virtual network in AWS. It provides a network boundary where subnets, routes and network security controls are configured.

## 8. What is a security group?

A security group is a virtual firewall for supported cloud resources. It controls allowed inbound and outbound traffic. A database should normally accept connections only from approved application or pipeline sources.

## 9. Why should a production database usually be private?

Private access reduces exposure to the public internet and allows access through controlled network paths. Public access should not be enabled merely to make a laptop connection easier. Administration should use approved VPN or private connectivity.

## 10. What is the difference between cloud IAM and database authentication?

Cloud IAM controls access to cloud resources and APIs. Database authentication verifies a database identity, unless a specific integration connects the systems. A cloud console login does not automatically grant SQL access.

## 11. How does Python connect to PostgreSQL?

Python uses a database driver such as psycopg. The application supplies endpoint, port, database name and credentials through secure configuration, opens a connection, creates a cursor, executes SQL, handles results and closes resources.

## 12. Why use environment variables or a secret manager?

They keep credentials out of source code and allow configuration to differ between environments. Production workloads should use an approved secret-management or workload-identity approach and restrict who can retrieve secrets.

## 13. What is a parameterized query?

A parameterized query sends SQL structure and data values separately through the database driver. This reduces SQL injection risk and handles values safely. Placeholder syntax depends on the driver.

## 14. What is a cursor?

A cursor is a driver object used to execute SQL and retrieve results. It commonly supports execute, fetchone, fetchmany and fetchall. Fetching an enormous result all at once can consume too much memory.

## 15. What is a transaction?

A transaction groups database operations into a unit of work. Commit confirms changes; rollback cancels uncommitted changes. Pipelines should avoid committing incomplete or invalid work.

## 16. What is connection pooling?

A connection pool reuses a controlled set of database connections. It reduces connection setup overhead and limits concurrent connections. Pool size must be chosen carefully to avoid overloading the database.

## 17. What is the difference between backup and high availability?

A backup supports recovery of data. High availability aims to reduce service interruption, for example through supported multi-AZ designs. A highly available system can still suffer accidental data changes, so backups remain important.

## 18. What is a read replica?

A read replica is a copy used to serve read workloads, depending on engine and service behavior. It can reduce load on the primary but may have replication lag. It should not be used for queries requiring immediate consistency unless the platform guarantees it.

## 19. Scenario: correct password but connection times out.

I would check endpoint, port, DNS, route, VPN/VPC connectivity, subnet configuration, security group source and database availability. A timeout usually suggests a reachability issue, not necessarily a wrong password.

## 20. Scenario: Python connects but SELECT fails with permission denied.

The network and authentication steps succeeded. The database user lacks the required schema or table privilege. I would request the minimum SELECT grant required rather than superuser access.

## 21. Scenario: the pipeline runs out of database connections.

I would inspect connection lifecycle and pool configuration, ensure connections are closed or returned to the pool, review concurrent task count and size the pool against database capacity. I would also check for leaked or long-running sessions.

## 22. Scenario: reporting replica shows older totals.

I would check replication lag and determine whether the report requires immediate consistency. If it does, I would route it to an appropriate consistent source or design a wait/consistency strategy rather than assuming the replica is current.

## 23. How do you secure a database connection?

Use private network paths where appropriate, restrict security group rules, protect credentials with approved secret management, use TLS with certificate validation where supported, grant minimum database permissions and avoid logging secrets.

## 24. Explain your Day 53 project.

I designed a managed database architecture, documented the pipeline's network path, separated cloud network access from database SQL permissions, prepared a secure environment-based connection example, used parameterized SQL and documented backup, availability and troubleshooting decisions. I would clearly state whether the connection was executed against a real database or only reviewed as a template.

## Common interview traps

- Assuming AWS IAM login automatically grants SQL access.
- Confusing endpoint, port and database name.
- Opening a database to all internet addresses.
- Hardcoding passwords.
- Concatenating untrusted values into SQL.
- Fetching huge result sets without memory planning.
- Assuming a read replica is always current.
- Treating high availability as a backup.
- Giving a pipeline database superuser access.

## Readiness checklist

- [ ] Explain managed database responsibilities.
- [ ] Explain endpoint, port and database name.
- [ ] Describe VPC and security group access.
- [ ] Explain cloud identity versus database identity.
- [ ] Describe safe Python connectivity.
- [ ] Explain parameterized queries and transactions.
- [ ] Explain pooling, backups, HA and replicas.
- [ ] Troubleshoot common connection errors.
