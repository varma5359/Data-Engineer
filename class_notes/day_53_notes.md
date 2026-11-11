# Day 53: Cloud Database Fundamentals

**Phase:** 6 — Cloud and Distributed Processing  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and junior Data Engineer candidates  
**Primary cloud example:** AWS managed relational databases  
**Topic boundary:** Class notes only. Project workbook and interview guide are separate files.

> A data pipeline often reads from a database or writes transformed results into one. In the cloud, the database may be managed by a provider rather than installed on a company server. Today you will understand managed databases, database endpoints, ports, VPC networking, security groups, credentials, secure Python connectivity and the operational questions a Data Engineer must ask.

## Learning outcomes

- Explain a managed database.
- Compare self-managed and managed databases.
- Recognize AWS RDS and Aurora at a high level.
- Distinguish database engine, instance, endpoint and port.
- Understand VPC, subnet and security group connectivity.
- Explain public and private database access.
- Connect Python to a relational database safely.
- Use environment variables for configuration.
- Understand parameterized SQL.
- Explain TLS, secrets, backups, high availability and connection pooling.
- Troubleshoot basic connection failures.

---

# Part 1 — Why do companies use cloud databases?

## 1. A familiar scenario

TechStore has an order application. When a customer places an order, the application saves order information in a relational database.

The data engineering team needs to:
- extract order records;
- load daily changes;
- read customer and product reference data;
- store pipeline metadata;
- support analytical processing.

The database may run on a company-managed server or as a cloud-managed database service.

## 2. What is a managed database?

A managed database is a database service where the cloud provider manages parts of the infrastructure and routine operations.

Depending on the service and configuration, the provider may help manage:
- infrastructure provisioning;
- database installation;
- automated backups;
- patching options;
- monitoring integration;
- failover capabilities;
- storage management.

The customer still manages:
- schema and table design;
- SQL and indexes;
- database users and permissions;
- application connections;
- data protection choices;
- performance and capacity decisions;
- backup retention and recovery testing;
- cost.

**Important:** Managed does not mean the provider manages every database responsibility.

## 3. Self-managed versus managed database

| Area | Self-managed database on a VM | Managed database service |
|---|---|---|
| Server provisioning | Team | Provider service |
| OS maintenance | Team | More provider-managed |
| Database installation | Team | Service-managed setup |
| Schema design | Team | Team |
| SQL tuning | Team | Team |
| User permissions | Team | Team |
| Backup configuration | Team | Shared, depending on service |
| Network access | Team | Team configures approved network controls |
| Cost responsibility | Team | Team |

The exact boundary depends on the selected service.

## 4. What is AWS RDS?

Amazon Relational Database Service (Amazon RDS) is a managed relational database service.

It supports several database engines. The available engines and features depend on the service and current AWS offering.

A Data Engineer may use RDS to connect to an application database or a metadata database.

RDS is not the same thing as S3:
- RDS stores relational data in database tables;
- S3 stores objects such as files.

## 5. What is Amazon Aurora?

Amazon Aurora is a relational database engine compatible with selected MySQL and PostgreSQL workloads and managed within the AWS database ecosystem.

At a beginner level, remember:
- RDS is the managed relational database service family;
- Aurora is a particular database engine/service offering with its own architecture and compatibility behavior.

Do not assume every PostgreSQL feature, extension or operational behavior is identical between standard PostgreSQL and Aurora PostgreSQL. Check compatibility before migration.

---

# Part 2 — Understand database connection details

## 6. What information does an application need?

A database client commonly needs:

| Item | Meaning | Example |
|---|---|---|
| Host/endpoint | Network name of the database | db.example.internal |
| Port | Network port for the database listener | 5432 for PostgreSQL |
| Database name | Database to connect to | sales |
| Username | Database identity | pipeline_reader |
| Password/secret | Authentication credential | Retrieved securely |
| TLS settings | Secure transport configuration | Certificate verification |
| Network route | Path to reach database | VPC/subnet/routing |

Do not confuse the database endpoint with the database name.

## 7. What is a database endpoint?

An endpoint is the network address used by a client to reach a database service.

A cloud managed database typically provides an endpoint hostname.

Example placeholder:

~~~text
techstore-db.example-region.rds.amazonaws.com
~~~

This is not a real database address. A real endpoint is provided after a database resource is created.

The endpoint is not a username or password.

## 8. What is a port?

A port is a numbered communication endpoint used by network services.

Common database defaults include:

| Database | Common default port |
|---|---:|
| PostgreSQL | 5432 |
| MySQL | 3306 |
| Microsoft SQL Server | 1433 |

Ports can be changed by configuration. Always use the actual configured port.

## 9. What is a database name?

A database name identifies the logical database within a database server or service.

One database server can host multiple databases, depending on the engine.

The client must connect to the intended database.

## 10. What is a database user?

A database user is an identity recognized by the database engine.

It is separate from an AWS IAM identity unless an integration specifically connects those identity systems.

A cloud role that can access the AWS console does not automatically have permission to query PostgreSQL tables.

Think about two permission layers:
1. cloud/network permission to reach the database;
2. database authentication and SQL authorization.

---

# Part 3 — Network connectivity

## 11. What is a VPC?

A VPC is a logically isolated virtual network in AWS.

Cloud database resources are commonly placed within a VPC so network access can be controlled.

## 12. What is a subnet?

A subnet is an IP address range within a VPC and is associated with an availability zone.

Managed databases may use a DB subnet group to define the subnets in which database resources can be placed.

A database should generally be placed in private subnets for internal application workloads, unless a reviewed architecture requires another arrangement.

## 13. Public versus private database access

A public database endpoint may be reachable from outside its VPC if network routes and security rules permit it.

A private database endpoint is reachable through approved private network paths.

For company databases, private access is generally preferred.

**Do not enable public access just because a laptop cannot connect.** First understand VPN, bastion, approved network paths and security requirements.

## 14. What is a security group?

A security group acts as a virtual firewall for supported resources.

It controls inbound and outbound traffic.

Example:
- database security group allows inbound PostgreSQL traffic on port 5432;
- source is the security group attached to the approved application or pipeline compute resource.

A safer rule references an approved source security group rather than allowing every IP address.

## 15. Understand the connection path

A pipeline running on a cloud compute service may connect like this:

~~~text
Pipeline compute
      |
      | TCP connection to database port
      v
VPC routing and subnet path
      |
      v
Database security group
      |
      v
Managed database endpoint
      |
      v
Database authentication and SQL permissions
~~~

Every layer must permit the connection.

## 16. Why a password alone is not enough

A correct password cannot fix:
- blocked security group;
- wrong port;
- incorrect endpoint;
- missing route;
- DNS resolution failure;
- database unavailable;
- TLS configuration error.

Likewise, a network connection does not prove the database user can read a table.

---

# Part 4 — Database connectivity from Python

We will use PostgreSQL as the example engine. The same ideas apply to other drivers, but connection syntax and packages differ.

## 17. Install a PostgreSQL driver

A common Python driver is psycopg.

For this teaching example, install the binary distribution in your active Python environment:

~~~bash
python -m pip install "psycopg[binary]"
~~~

Expected behavior: pip installs psycopg and its binary support, or reports that the requirement is already satisfied.

The exact output depends on your Python version and platform.

## 18. Why use a driver?

A database driver allows Python to:
- open a database connection;
- execute SQL;
- retrieve rows;
- commit or roll back transactions;
- close resources.

The driver does not create network access or database permissions. Those must already be configured.

## 19. Understand configuration

Do not write a real password directly into a Python file.

Use configuration supplied by the environment or an approved secret manager.

Example environment variables:

~~~text
DB_HOST
DB_PORT
DB_NAME
DB_USER
DB_PASSWORD
~~~

These names are conventions for the teaching script. Your environment must actually define them.

## 20. Read environment variables in Python

~~~python
import os

db_host = os.getenv("DB_HOST")
db_port = os.getenv("DB_PORT", "5432")
db_name = os.getenv("DB_NAME")
db_user = os.getenv("DB_USER")
db_password = os.getenv("DB_PASSWORD")

print("Host configured:", bool(db_host))
print("Port configured:", db_port)
print("Database configured:", bool(db_name))
print("User configured:", bool(db_user))
print("Password configured:", bool(db_password))
~~~

Example output when variables are present:

~~~text
Host configured: True
Port configured: 5432
Database configured: True
User configured: True
Password configured: True
~~~

The script prints whether values exist. It does not print the secret itself.

If a value is missing, check your environment configuration. Do not put the password into source code as a quick fix.

## 21. Build a connection

~~~python
import os
import psycopg

connection = psycopg.connect(
    host=os.environ["DB_HOST"],
    port=os.environ.get("DB_PORT", "5432"),
    dbname=os.environ["DB_NAME"],
    user=os.environ["DB_USER"],
    password=os.environ["DB_PASSWORD"],
    connect_timeout=10
)

print("Database connection successful")

connection.close()
~~~

This is a connection template. It will work only when:
- the driver is installed;
- the variables exist;
- the endpoint and port are correct;
- network access is available;
- the database is running;
- the user is valid;
- authentication and TLS requirements are satisfied.

A successful connection is not proof that the user can query every table.

## 22. Why close a connection?

A connection uses database and network resources.

If scripts open connections and never close them, the database may eventually reach its connection limit.

For simple scripts, close the connection after work. In larger applications, use context managers or a connection pool with clear ownership and cleanup.

## 23. Execute a simple query

Assume the connected database contains a table named orders.

~~~python
import os
import psycopg

connection = psycopg.connect(
    host=os.environ["DB_HOST"],
    port=os.environ.get("DB_PORT", "5432"),
    dbname=os.environ["DB_NAME"],
    user=os.environ["DB_USER"],
    password=os.environ["DB_PASSWORD"],
    connect_timeout=10
)

cursor = connection.cursor()

cursor.execute("SELECT COUNT(*) FROM orders")

result = cursor.fetchone()

print("Order count:", result[0])

cursor.close()
connection.close()
~~~

Example output for a training database:

~~~text
Order count: 1250
~~~

The value 1250 is illustrative. Your actual result depends on your database.

## 24. What is a cursor?

A cursor is an object used by a database driver to execute SQL and retrieve results.

Common operations:
- execute(): send SQL to the database;
- fetchone(): retrieve one result row;
- fetchmany(): retrieve a limited number of rows;
- fetchall(): retrieve all remaining result rows.

Do not use fetchall() for an enormous result set without considering memory. For large data extraction, use pagination, server-side cursors or an appropriate bulk-export method.

## 25. Parameterized queries

Do not build SQL by concatenating user-provided values into query text.

Unsafe pattern:

~~~python
customer_id = "C001"

sql = "SELECT * FROM customers WHERE customer_id = '" + customer_id + "'"
~~~

This can create SQL injection risks when values are untrusted.

Safer parameterized query with psycopg:

~~~python
customer_id = "C001"

cursor.execute(
    "SELECT customer_id, city FROM customers WHERE customer_id = %s",
    (customer_id,)
)

rows = cursor.fetchall()

print(rows)
~~~

The driver sends the value as a parameter rather than treating it as SQL syntax.

The placeholder syntax is driver-specific. Do not copy psycopg placeholders into every database library.

---

# Part 5 — Transactions and connection safety

## 26. What is a transaction?

A transaction groups database operations into a unit of work.

For example, a pipeline might:
1. insert a load audit record;
2. update a processing status;
3. commit both changes together.

If an error occurs before commit, the pipeline may roll back the transaction.

## 27. Commit and rollback

- commit() makes the transaction's changes durable according to the database's transaction behavior.
- rollback() cancels uncommitted changes in the current transaction.

A read-only query may not require a manual commit, depending on driver and transaction configuration.

Do not commit partial data if the pipeline has failed validation.

## 28. Example transaction

~~~python
cursor.execute(
    "UPDATE pipeline_runs SET status = %s WHERE run_id = %s",
    ("SUCCESS", 101)
)

connection.commit()

print("Transaction committed")
~~~

This is illustrative. It assumes the table and columns exist and the database user has permission.

If an error occurs before commit, handle it and roll back the transaction where appropriate.

## 29. Connection pooling

Opening a new database connection for every small operation can create overhead.

A connection pool keeps a controlled set of reusable connections.

Benefits:
- reduce connection setup overhead;
- limit simultaneous connections;
- reuse established connections.

A pool must be sized carefully. Too many connections can overload the database.

Do not confuse a connection pool with a query cache.

---

# Part 6 — Managed database operations

## 30. Backups

A backup is a recoverable copy or recovery mechanism for database data.

Managed services may offer automated backups and snapshots, depending on configuration.

A backup is useful only if:
- retention is suitable;
- it covers the required data;
- access is protected;
- restoration is tested;
- recovery time meets business needs.

A backup that has never been restored in a test may not meet the organization's recovery expectations.

## 31. High availability

High availability aims to reduce service interruption.

A managed database may offer multi-AZ or other availability configurations.

Understand the difference:
- high availability helps keep a service available;
- backup helps recover data;
- read replicas can support read scaling or other use cases;
- disaster recovery addresses larger failure scenarios.

These capabilities are related but not interchangeable.

## 32. Read replicas

A read replica is a copy of a database used to serve read workloads, depending on engine and service behavior.

Potential use:
- separate reporting reads from primary application writes;
- scale certain read-heavy workloads.

Consider replication lag. A replica may not immediately contain the latest committed data.

Do not send a report requiring immediate consistency to a lagging replica without understanding the requirement.

## 33. Monitoring

Monitor:
- CPU usage;
- memory pressure where available;
- storage capacity;
- connection count;
- read/write activity;
- query latency;
- replication lag;
- backup status;
- database errors.

A pipeline can fail because the database is unavailable, full, overloaded or refusing new connections.

## 34. Cost factors

Managed database costs may include:
- instance or compute size;
- storage;
- backup storage;
- I/O;
- data transfer;
- licensing where applicable;
- provisioned capacity;
- monitoring or optional features.

Select capacity based on workload. Do not leave training databases running after the exercise.

---

# Part 7 — Troubleshooting a failed connection

## 35. Error: could not translate host name

Possible causes:
- endpoint typo;
- DNS issue;
- incorrect environment variable;
- network resolver problem.

Check the endpoint carefully and confirm the host is resolvable from the environment running Python.

## 36. Error: connection timed out

Possible causes:
- database not reachable;
- wrong port;
- security group blocks traffic;
- route or VPN missing;
- private endpoint accessed from outside the network.

Check network path and security rules before changing passwords.

## 37. Error: connection refused

Possible causes:
- service not listening;
- wrong port;
- database unavailable;
- network path reaches a host but no service accepts the connection.

Confirm the configured port and database state.

## 38. Error: password authentication failed

Possible causes:
- incorrect username or password;
- wrong database user;
- expired or rotated secret;
- wrong database endpoint.

Do not paste passwords into logs to debug.

## 39. Error: permission denied for table

The connection succeeded, but the database user does not have the required SQL permission.

Ask the database owner for the minimum table/schema permission needed. Do not request superuser access for a read-only pipeline.

## 40. Troubleshooting sequence

1. Confirm the active environment variables without printing secrets.
2. Confirm the endpoint hostname.
3. Confirm port and database name.
4. Confirm network/VPN/VPC route.
5. Check security group source and port.
6. Check database availability.
7. Check TLS requirements.
8. Check username and secret.
9. Check database/schema/table grants.
10. Capture safe error context and escalate to the correct owner.

---

# Part 8 — Security checklist

## 41. Protect credentials

- Never commit database passwords.
- Do not paste secrets into notebooks or screenshots.
- Use approved secret storage or workload identity.
- Rotate credentials according to policy.
- Limit who can retrieve secrets.
- Avoid logging connection strings containing passwords.

## 42. Protect the network

- Prefer private database access for internal workloads.
- Restrict inbound rules to approved sources.
- Do not open the database port to the whole internet.
- Use approved VPN or private connectivity for administration.
- Require secure transport where supported.

## 43. Protect database permissions

- Use separate application identities.
- Grant only required schemas and tables.
- Use read-only roles for extraction jobs where possible.
- Separate development and production identities.
- Review access regularly.

---

# Part 9 — Common mistakes

| Mistake | Why it is wrong | Better practice |
|---|---|---|
| Assume cloud login equals DB login | Identity systems are distinct | Check network and DB authentication |
| Confuse endpoint with database name | They identify different things | Configure host and dbname separately |
| Open port to 0.0.0.0/0 | Exposes database network surface | Allow approved source only |
| Hardcode password | Secret can leak | Use environment/secret manager |
| Print full connection string | May expose credentials | Log safe connection metadata only |
| Concatenate values into SQL | SQL injection risk | Use parameterized queries |
| Never close connections | Resource exhaustion | Close or pool connections |
| Fetch millions of rows at once | Memory pressure | Stream/page/export safely |
| Assume replica is current | Replication lag can exist | Match consistency requirement |
| Assume backup is tested | Restore may fail or be slow | Test recovery |
| Give pipeline superuser | Excessive access | Minimum schema/table grants |

---

# Part 10 — Practice questions

## Exercise 1

What is a managed database?

**Answer:** A database service where the provider manages parts of infrastructure and routine operations, while the customer still manages schema, access, data and workload choices.

## Exercise 2

What is the PostgreSQL default port?

**Answer:** 5432, unless configured differently.

## Exercise 3

What is the difference between endpoint and database name?

**Answer:** Endpoint identifies the network address of the database service. Database name identifies the logical database to use.

## Exercise 4

A password is correct but connection times out. What should you check?

**Answer:** Network path, route, endpoint, port and security group before changing credentials.

## Exercise 5

Why use parameterized queries?

**Answer:** They keep data values separate from SQL syntax and reduce SQL injection risk.

## Exercise 6

Why use a connection pool?

**Answer:** It reuses a controlled number of connections and reduces repeated connection setup overhead.

## Exercise 7

What is the difference between a backup and high availability?

**Answer:** Backup supports recovery of data. High availability reduces service interruption. They solve different problems.

## Exercise 8

Why might a read replica return older data?

**Answer:** Replication can lag behind the primary database.

---

# Part 11 — Revision sheet

| Term | Remember |
|---|---|
| Managed database | Provider manages part of database infrastructure |
| RDS | AWS managed relational database service |
| Aurora | Managed relational database engine/service offering |
| Endpoint | Network address for the database |
| Port | Number used by the database listener |
| VPC | Isolated cloud network |
| Subnet | IP range within a VPC |
| Security group | Network traffic rules |
| Database user | Identity inside the database engine |
| Driver | Library used by an application to connect |
| Cursor | Executes SQL and retrieves results |
| Parameterized query | Sends values separately from SQL syntax |
| Transaction | Group of database operations |
| Commit | Confirms transaction changes |
| Rollback | Cancels uncommitted changes |
| Connection pool | Reuses a controlled set of connections |
| Read replica | Database copy used for read workloads |
| Replication lag | Delay between primary and replica changes |
| Backup | Recovery copy/mechanism |
| High availability | Design that reduces service interruption |

## Final self-check

- [ ] Explain managed versus self-managed databases.
- [ ] Recognize RDS and Aurora.
- [ ] Explain endpoint, port and database name.
- [ ] Describe private network access and security groups.
- [ ] Explain cloud identity versus database identity.
- [ ] Read connection settings from environment variables.
- [ ] Connect Python using a database driver.
- [ ] Use parameterized SQL.
- [ ] Explain transactions and connection pooling.
- [ ] Explain backups, high availability and replicas.
- [ ] Troubleshoot timeout, authentication and permission errors.

**Next lesson — Day 54: Distributed Computing.** You will learn why one machine is sometimes not enough, how parallel work is divided, what partitions and workers mean, and what trade-offs distributed systems introduce.
