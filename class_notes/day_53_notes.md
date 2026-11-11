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


---

# Part 12 — Understand the connection before writing Python

A common beginner mistake is to debug Python before checking whether the database can be reached.

~~~text
Python program
    -> Read connection configuration
    -> Resolve endpoint hostname
    -> Reach host and port through the network
    -> Complete TLS negotiation if required
    -> Authenticate database user
    -> Check SQL permissions
    -> Execute query
~~~

## 42. Separate the connection layers

| Layer | Question | Example failure |
|---|---|---|
| Configuration | Did Python read the right host and port? | Empty DB_HOST |
| DNS | Can endpoint name be resolved? | Hostname not found |
| Network | Can traffic reach the endpoint and port? | Timeout |
| TLS | Can the client establish trusted encryption? | Certificate verification failure |
| Authentication | Is the database identity valid? | Password authentication failed |
| Authorization | Can this user run this SQL? | Permission denied for table |

**Checkpoint:** Python reports permission denied for table orders. Did DNS and network connectivity probably work?

**Answer:** Yes. The client reached the database and authenticated sufficiently to receive a table-permission error. Investigate database grants, not the VPC route first.

---

# Part 13 — Inspect the cloud database design before connecting

## 43. What should the database owner provide?

| Required detail | Who/where confirms it |
|---|---|
| Cloud account/environment | Cloud platform owner |
| Database engine and version | Database owner |
| Endpoint | Managed database configuration |
| Port | Database configuration |
| Database name | Database owner |
| Username or workload identity | Database/security owner |
| Secret retrieval method | Security/platform team |
| Network source | Pipeline/platform owner |
| TLS certificate requirements | Security/database owner |
| Required schema/table access | Data owner/database owner |

Do not guess an endpoint or copy a production hostname from an unrelated environment.

## 44. Read an example connection record

~~~text
Environment: development
Engine: PostgreSQL
Endpoint: dev-orders.example.internal
Port: 5432
Database: orders
User: pipeline_reader
Network: approved private route
TLS: certificate verification required
Access: SELECT on approved extraction schema
~~~

All values are illustrative. They are not real credentials or a real endpoint.

## 45. Endpoint versus database name

Imagine one managed PostgreSQL server hosts two databases:

~~~text
Endpoint: db-dev.example.internal
Port: 5432

Databases:
    orders
    reporting
~~~

The endpoint and port identify where the database service listens. The database name selects the logical database after the client reaches that service.

Connecting to the correct endpoint but the wrong database can produce a successful network connection and missing-table errors.

---

# Part 14 — Test DNS and network access before Python

These tests are useful when an organization has provided an approved endpoint. They do not require printing or sharing a password.

## 46. Check the hostname

In Windows CMD:

~~~bat
nslookup dev-orders.example.internal
~~~

The example hostname is a placeholder.

The command asks DNS to resolve the name. If it cannot be resolved, check:
- spelling;
- VPN or private DNS connection;
- DNS resolver configuration;
- whether the endpoint is private and only resolvable inside the approved network.

A private endpoint may not resolve from a home network without the organization's VPN or private connectivity.

## 47. Check the TCP port

Open PowerShell:

~~~powershell
Test-NetConnection dev-orders.example.internal -Port 5432
~~~

Inspect the TcpTestSucceeded field.

- True means a TCP connection to that host and port succeeded from this machine.
- False means the connection test did not succeed.

This does not prove database authentication or SQL permissions. It only tests TCP connectivity.

## 48. Interpret network test outcomes

| Result | What it suggests | Next check |
|---|---|---|
| DNS name not found | Hostname cannot be resolved | Endpoint spelling, VPN, DNS |
| TCP test succeeds | Network path accepts the connection | Continue to TLS and authentication |
| TCP timeout | Traffic may be blocked or route unavailable | VPN, route, security group, firewall |
| Connection refused | Host reachable but service may not listen | Database state and configured port |

Do not change a password to solve a DNS failure.

## 49. Why a laptop may not reach a private RDS endpoint

A database can be private inside a VPC. Your laptop may be outside that network.

Approved connection paths may include:
- corporate VPN;
- managed development environment inside the VPC;
- controlled bastion/access host;
- private connectivity approved by the organization.

Do not make the database public simply to make a classroom connection test easier.

---

# Part 15 — Create a local PostgreSQL practice environment

If you do not have an approved cloud database, practice the Python driver and SQL workflow against local PostgreSQL.

This teaches client/database interaction. It does not teach cloud networking, RDS failover or cloud security groups.

## 50. Confirm PostgreSQL is available

Open CMD:

~~~bat
psql --version
~~~

If Windows reports that psql is not recognized, PostgreSQL command-line tools may not be installed or their folder may not be in PATH.

## 51. Create a training database

In your local PostgreSQL SQL editor, run:

~~~sql
CREATE DATABASE techstore_cloud_practice;
~~~

Expected result:

~~~text
CREATE DATABASE
~~~

This is local practice, not an AWS resource.

If the database already exists, use it rather than deleting it.

## 52. Connect to the training database

General psql pattern:

~~~bat
psql -h localhost -p 5432 -U postgres -d techstore_cloud_practice
~~~

The command may prompt for your local database user's password. Use the password configured on your own machine. Do not share it.

## 53. Create a small orders table

Run inside the techstore_cloud_practice database:

~~~sql
CREATE TABLE orders (
    order_id INTEGER PRIMARY KEY,
    customer_id VARCHAR(20) NOT NULL,
    order_date DATE NOT NULL,
    amount NUMERIC(12, 2) NOT NULL,
    status VARCHAR(20) NOT NULL
);
~~~

Expected result:

~~~text
CREATE TABLE
~~~

Understand the definition:
- CREATE TABLE creates a database table.
- order_id INTEGER stores a whole-number identifier.
- PRIMARY KEY makes the key unique and not NULL.
- VARCHAR(20) stores short text.
- NOT NULL requires a value.
- DATE stores a calendar date.
- NUMERIC(12, 2) stores a decimal value with two fractional digits.

## 54. Insert sample rows

~~~sql
INSERT INTO orders VALUES
(501, 'C001', DATE '2026-11-10', 250.00, 'PAID'),
(502, 'C002', DATE '2026-11-10', 499.00, 'PAID'),
(503, 'C001', DATE '2026-11-11', 100.00, 'PENDING');
~~~

Expected result:

~~~text
INSERT 0 3
~~~

These are synthetic training records.

## 55. Check the data

~~~sql
SELECT *
FROM orders
ORDER BY order_id;
~~~

Expected result:

| order_id | customer_id | order_date | amount | status |
|---:|---|---|---:|---|
| 501 | C001 | 2026-11-10 | 250.00 | PAID |
| 502 | C002 | 2026-11-10 | 499.00 | PAID |
| 503 | C001 | 2026-11-11 | 100.00 | PENDING |

If the result differs, inspect the database name and SQL statements you executed.

---

# Part 16 — Connect Python to local PostgreSQL

The following cells use local PostgreSQL. Do not substitute a production endpoint.

## 56. Install the driver

In the terminal for the Python environment used by Jupyter:

~~~bat
python -m pip install "psycopg[binary]"
~~~

Expected behavior: pip installs the package or reports that it is already installed.

If the notebook reports ModuleNotFoundError, confirm that the terminal and notebook use the same Python environment.

## 57. Set practice configuration in CMD

For a local-only exercise, set these values in CMD:

~~~bat
set DB_HOST=localhost
set DB_PORT=5432
set DB_NAME=techstore_cloud_practice
set DB_USER=postgres
set DB_PASSWORD=your-local-postgres-password
~~~

Replace the final value with your local password. These variables apply to processes launched from that CMD session. An already-running Jupyter process may not inherit variables added afterward.

Do not save a real production password in a public script or notebook.

If you launch Jupyter from this same CMD window after setting the variables, it can inherit them.

## 58. Cell 1 — Confirm configuration without revealing secrets

~~~python
import os

print("DB_HOST:", os.getenv("DB_HOST"))
print("DB_PORT:", os.getenv("DB_PORT"))
print("DB_NAME configured:", bool(os.getenv("DB_NAME")))
print("DB_USER configured:", bool(os.getenv("DB_USER")))
print("DB_PASSWORD configured:", bool(os.getenv("DB_PASSWORD")))
~~~

Expected output for the local configuration:

~~~text
DB_HOST: localhost
DB_PORT: 5432
DB_NAME configured: True
DB_USER configured: True
DB_PASSWORD configured: True
~~~

The password is never printed.

## 59. Cell 2 — Open the connection

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

print("Connected to the training database")
~~~

Expected output if PostgreSQL is running and credentials are correct:

~~~text
Connected to the training database
~~~

If this fails, read the exact exception. Do not change several settings at once.

## 60. Cell 3 — Read the order count

~~~python
cursor = connection.cursor()

cursor.execute("SELECT COUNT(*) FROM orders")

result = cursor.fetchone()

print("Order count:", result[0])
~~~

Expected output:

~~~text
Order count: 3
~~~

Explanation:
- connection.cursor() creates a cursor;
- execute() sends SQL to PostgreSQL;
- COUNT(*) counts rows;
- fetchone() retrieves one result row;
- result[0] reads its first value.

## 61. Cell 4 — Read selected columns

~~~python
cursor.execute(
    "SELECT order_id, amount, status FROM orders ORDER BY order_id"
)

rows = cursor.fetchall()

for row in rows:
    print(row)
~~~

Expected output:

~~~text
(501, Decimal('250.00'), 'PAID')
(502, Decimal('499.00'), 'PAID')
(503, Decimal('100.00'), 'PENDING')
~~~

The amount is commonly returned as Decimal because the SQL column is NUMERIC. Decimal is useful for financial values.

## 62. Cell 5 — Use a parameterized query

~~~python
selected_status = "PAID"

cursor.execute(
    "SELECT order_id, amount FROM orders WHERE status = %s ORDER BY order_id",
    (selected_status,)
)

paid_rows = cursor.fetchall()

print(paid_rows)
~~~

Expected output:

~~~text
[(501, Decimal('250.00')), (502, Decimal('499.00'))]
~~~

The %s is a psycopg parameter placeholder. The tuple supplies the value separately.

The comma in (selected_status,) makes a one-item tuple. Without the comma, Python treats parentheses around one value as grouping.

## 63. Cell 6 — Close resources

~~~python
cursor.close()
connection.close()

print("Database resources closed")
~~~

Expected output:

~~~text
Database resources closed
~~~

Closing resources matters because database servers have finite connection capacity.

---

# Part 17 — Understand TLS configuration

## 64. Why TLS is needed

TLS protects information while the client communicates with the database.

Depending on organizational policy, a managed database may require encrypted connections and certificate validation.

Encryption without server identity verification may not prove that the client reached the intended server.

## 65. Common PostgreSQL SSL modes

| Mode | General behavior |
|---|---|
| disable | Do not use TLS |
| require | Require encrypted connection |
| verify-ca | Require encryption and verify certificate chain |
| verify-full | Also verify that certificate identity matches the host |

Exact behavior can depend on the client and driver. Follow the cloud/database owner's required mode.

For production, use the approved certificate-validation configuration and trusted CA certificate.

## 66. Example psycopg TLS configuration

~~~python
import os
import psycopg

connection = psycopg.connect(
    host=os.environ["DB_HOST"],
    port=os.environ.get("DB_PORT", "5432"),
    dbname=os.environ["DB_NAME"],
    user=os.environ["DB_USER"],
    password=os.environ["DB_PASSWORD"],
    sslmode="verify-full",
    sslrootcert=os.environ["DB_SSL_ROOT_CERT"],
    connect_timeout=10
)

print("TLS-verified database connection opened")
~~~

This requires a valid CA certificate file and a hostname matching the certificate. Obtain the current certificate bundle from the official provider/database documentation or your organization.

Do not disable certificate verification just to make a connection work.

## 67. Inspect TLS from PostgreSQL

~~~sql
SELECT
    ssl,
    version,
    cipher
FROM pg_stat_ssl
WHERE pid = pg_backend_pid();
~~~

This reports TLS details for the current PostgreSQL backend when the view is available and permissions permit access.

A true SSL value indicates that the current connection uses SSL/TLS. Exact version and cipher depend on the environment.

---

# Part 18 — Understand database privileges

## 68. Connection access is not table access

A user may be allowed to connect to a database but not read every schema or table.

~~~text
Can reach endpoint
      |
      v
Can connect to database
      |
      v
Can use schema
      |
      v
Can select required table
~~~

Each step can have separate controls.

## 69. Inspect a table permission

In PostgreSQL:

~~~sql
SELECT
    has_table_privilege(
        current_user,
        'public.orders',
        'SELECT'
    ) AS can_select_orders;
~~~

Expected result for a user with SELECT permission:

| can_select_orders |
|---|
| true |

The result depends on the connected user and grants.

## 70. Read-only pipeline identity

A pipeline that only extracts source data should normally use a read-only database role.

It should not need:
- table deletion;
- schema alteration;
- user administration;
- superuser privileges;
- access to unrelated sensitive tables.

Ask the database owner for the minimum required grants.

---

# Part 19 — Connection pooling and extraction design

## 71. Why too many connections are a problem

A database has finite memory and connection capacity.

If a pipeline starts 100 workers and each opens 10 connections, it may attempt 1,000 simultaneous connections. This can overwhelm the database.

## 72. What does a connection pool do?

A connection pool manages a bounded set of reusable connections.

A pipeline requests a connection, uses it and returns it to the pool. This reduces repeated setup and limits concurrency.

Pool size should reflect:
- database connection limit;
- other application workloads;
- query duration;
- worker count;
- expected concurrency;
- available database capacity.

Do not set pool size to an arbitrarily large number.

## 73. Batch extraction

For a small result, fetchall() may be acceptable.

For a large source table, consider:
- filtering by incremental watermark;
- selecting only required columns;
- pagination with a stable ordering key;
- server-side cursors or streaming;
- database-native export;
- bounded batches.

A query that reads the entire table every day may waste database resources and network bandwidth.

## 74. Example incremental extraction query

~~~sql
SELECT
    order_id,
    customer_id,
    amount,
    updated_at
FROM orders
WHERE updated_at > TIMESTAMP '2026-11-10 00:00:00'
ORDER BY updated_at, order_id;
~~~

This is a teaching query. A production design must define inclusive/exclusive boundaries, equal timestamps, late updates, precision, timezone, tie-breakers and checkpoint commit order.

---

# Part 20 — Database recovery and operational decisions

## 75. Backup versus high availability

A backup supports recovery of data. High availability aims to reduce service interruption. A read replica may support read scaling or another replication use case. They are not interchangeable.

## 76. What is RPO?

Recovery Point Objective describes the maximum acceptable amount of data loss measured in time.

Example: an RPO of 15 minutes means the business expects a recovery event to lose no more than approximately 15 minutes of committed data, subject to the chosen recovery mechanism.

## 77. What is RTO?

Recovery Time Objective describes the target time to restore service after disruption.

Example: an RTO of two hours means the business aims to restore service within two hours.

RPO and RTO are business requirements. A Data Engineer should not invent them.

## 78. Read replicas and lag

A read replica is a copy used for read workloads, depending on engine and service behavior.

Replication lag means changes committed on the primary have not yet appeared on the replica.

A report requiring the newest transactions may be stale if it reads from a lagging replica. Match the source to the report's freshness and consistency requirement.

## 79. Recovery test

A controlled recovery test should:
1. select an approved test environment;
2. identify a known backup or snapshot;
3. restore into an isolated target;
4. confirm the database becomes available;
5. validate schema and row counts;
6. run critical queries;
7. measure recovery time;
8. record the data-loss window;
9. document corrective actions.

Do not test recovery by deleting a production database.

---

# Part 21 — Connection troubleshooting decision tree

~~~text
Python cannot connect
       |
       v
Can hostname resolve?
   | No -> Check endpoint, DNS and VPN
   |
   Yes
       |
       v
Can TCP port be reached?
   | No -> Check route, security group and service
   |
   Yes
       |
       v
Does TLS verification pass?
   | No -> Check CA file, hostname and TLS policy
   |
   Yes
       |
       v
Does database authentication pass?
   | No -> Check username and secret rotation
   |
   Yes
       |
       v
Does SQL permission pass?
   | No -> Check database/schema/table grants
   |
   Yes
       |
       v
Investigate SQL, schema, data and capacity
~~~

## 80. Incident example

A daily extractor fails with:

~~~text
permission denied for table orders
~~~

Correct investigation:
1. Confirm endpoint and database.
2. Confirm the job's database user.
3. Confirm the connection succeeded.
4. Check whether SELECT is granted on the required table.
5. Ask the database owner for the minimum required permission.
6. Retry the same extraction batch.
7. Compare row count and watermark before marking success.

Incorrect response:
- open the database to the internet;
- grant superuser;
- disable TLS;
- rotate unrelated cloud credentials;
- mark the run successful without extraction evidence.

---

# Part 22 — Additional practice exercises with worked answers

## Exercise 9 — Endpoint versus database

The endpoint is correct, but Python reports that database sales_archive does not exist. What should you inspect?

**Worked answer:** Confirm the intended database name and environment. Endpoint reachability may be fine; the logical database selection may be wrong.

## Exercise 10 — DNS failure

The database is private and the laptop is not connected to the corporate VPN. The hostname does not resolve. What is a likely cause?

**Worked answer:** The endpoint may use private DNS available only inside the approved network. Follow the approved VPN or managed development environment path.

## Exercise 11 — TCP success

PowerShell reports TcpTestSucceeded: True. Does this prove the password is correct?

**Worked answer:** No. It proves TCP connectivity to the host and port. Database authentication happens later.

## Exercise 12 — Authentication success

A user can connect but receives permission denied for one table. What should change?

**Worked answer:** The database owner should grant the minimum required schema/table permission. Network rules do not fix SQL authorization.

## Exercise 13 — TLS error

A certificate verification error occurs. Should the developer disable SSL?

**Worked answer:** No. Verify endpoint hostname, trusted CA certificate and required TLS configuration. Do not disable certificate validation to bypass the error.

## Exercise 14 — Connection storm

A distributed job starts 300 tasks, and each task opens a new database connection. What risk exists?

**Worked answer:** The database may be overwhelmed by connections and concurrent queries. Limit concurrency, use bounded pools or extract through a suitable bulk mechanism.

## Exercise 15 — Replica lag

A report must include transactions committed seconds ago, but it reads from a replica that is behind. What is the risk?

**Worked answer:** The report can be stale. Use a source that meets the required freshness or account for lag explicitly.

## Exercise 16 — Backup confidence

A team has automated backups but has never restored one. Is recovery proven?

**Worked answer:** No. Run a controlled restore test and validate data, schema and recovery time.

## Exercise 17 — Parameterized SQL

A customer ID comes from an API request. Why pass it as a SQL parameter?

**Worked answer:** Parameterization keeps the value separate from SQL syntax and reduces SQL injection risk.

---

# Part 23 — Day 53 practical completion checklist

### Cloud database concepts
- [ ] Explain managed versus self-managed databases.
- [ ] Recognize RDS and Aurora.
- [ ] Distinguish endpoint, port and database name.
- [ ] Explain cloud identity versus database identity.
- [ ] Explain private database access.

### Networking and security
- [ ] Explain VPC, subnet and security group.
- [ ] Test DNS resolution.
- [ ] Test TCP reachability with an approved endpoint.
- [ ] Explain why network success is not authentication success.
- [ ] Explain TLS and certificate verification.
- [ ] Explain database/schema/table privileges.
- [ ] Explain why public access is not a beginner workaround.

### Python connectivity
- [ ] Install and import psycopg.
- [ ] Read configuration from environment variables.
- [ ] Connect to a local practice database.
- [ ] Execute a SELECT query.
- [ ] Retrieve one row and multiple rows.
- [ ] Use parameterized SQL.
- [ ] Close cursor and connection.
- [ ] Explain why connection pools need limits.

### Operations and reliability
- [ ] Explain backup, high availability and read replicas.
- [ ] Explain RPO and RTO.
- [ ] Describe a restore test.
- [ ] Troubleshoot DNS, timeout, TLS, authentication and permission failures.
- [ ] Explain incremental extraction and watermark boundaries.

## Final self-assessment

Without looking at the notes, explain:

1. What does a managed database service manage, and what remains the customer's responsibility?
2. What is the difference between endpoint, port and database name?
3. Why can a valid password still fail to connect?
4. What does a security group control?
5. Why are network access and database permissions separate?
6. What does TLS certificate verification protect?
7. Why should a pipeline use a read-only database identity when it only extracts data?
8. Why should a large extraction avoid fetching every row into memory?
9. What are connection pooling and connection limits?
10. How are backup, high availability and read replicas different?
11. What do RPO and RTO mean?
12. How would you investigate a table permission error?

If you can complete the local Python-to-PostgreSQL exercise and explain the cloud network path, you are ready for Day 54.

**Next lesson — Day 54: Distributed Computing.** You will learn why one machine is sometimes not enough, how parallel work is divided, what partitions and workers mean, and what trade-offs distributed systems introduce.
