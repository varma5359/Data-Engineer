# Day 50: Cloud Data Engineering Fundamentals

**Phase:** 6 — Cloud and Distributed Processing  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners, freshers and junior Data Engineer candidates  
**Primary cloud example:** Amazon Web Services (AWS)  
**Topic boundary:** Class notes only. The project workbook and interview guide are separate files.

> Until now, most examples have run on a laptop or a local database. Real companies often run data platforms on cloud infrastructure. Today you will learn what cloud computing means, why data engineers use it, how IaaS/PaaS/SaaS differ, what a basic cloud data architecture looks like, and how IAM and least-privilege access protect cloud resources.

## Learning outcomes

By the end of this lesson, you should be able to:
1. Explain cloud computing in simple words.
2. Compare on-premises infrastructure and cloud infrastructure.
3. Distinguish IaaS, PaaS and SaaS.
4. Explain public, private and hybrid cloud.
5. Understand regions and availability zones.
6. Recognize common AWS services used by data engineers.
7. Draw a basic cloud data engineering architecture.
8. Explain identity, authentication and authorization.
9. Understand IAM users, roles and policies.
10. Apply the principle of least privilege.
11. Identify common cloud security mistakes.
12. Distinguish a learning exercise from a deployment that may create charges.

### Today's learning flow

Business requirement -> Cloud identity -> Storage and compute -> Data pipeline -> Monitoring and security

---

# Part 1 — Why are we learning cloud data engineering?

## 1. Start with a company example

Imagine TechStore is an online shopping company. Customers use its website to create accounts, search products, place orders, make payments, request refunds and track deliveries.

Every activity creates data.

| System | Example data |
|---|---|
| Website | Page visits and clicks |
| Order application | Order ID, customer ID, order date |
| Payment service | Payment ID, amount, payment status |
| Delivery application | Shipment ID, delivery status |
| Customer support | Ticket ID, issue category |

The company wants a daily report that answers:
- How many orders were placed yesterday?
- What was yesterday's sales amount?
- Which products sold the most?
- Which deliveries are delayed?
- How many customers returned products?

A data engineer collects data from different systems, stores it, transforms it and makes it available for analysis.

A small laptop may be enough for a classroom example. A company may need storage for terabytes of data, compute for large transformations, secure access for many teams and reliable systems that operate continuously.

This is where cloud services can help.

## 2. What is cloud computing?

**Beginner definition:** Cloud computing means using computing resources over a network, usually the internet, instead of owning and managing every physical server yourself.

Computing resources can include:
- virtual machines;
- storage;
- databases;
- networks;
- application platforms;
- analytics services;
- monitoring tools.

A cloud provider owns and operates the underlying data-center infrastructure. Customers select and configure the services they need. Security and operations are shared responsibilities.

### 2.1 A simple real-life comparison

Think about electricity. You usually do not build a power station beside your home. You connect to an electricity provider and pay according to the applicable plan and usage.

Cloud computing has a similar service idea:
- the provider operates large infrastructure;
- you request computing or storage capacity;
- you use the resources;
- you pay according to the service and pricing model.

This is only an analogy. Cloud resources still need configuration, security, capacity planning and cost monitoring.

### 2.2 Before cloud and after cloud

| Area | Traditional on-premises approach | Cloud approach |
|---|---|---|
| Hardware | Company buys servers | Provider supplies infrastructure |
| Data center | Company manages facility | Provider operates data centers |
| Capacity | Purchase and install hardware | Provision supported resources |
| Maintenance | Company manages physical hardware | Provider manages underlying infrastructure; customer manages configured services |
| Cost | Large upfront investment may be needed | Often usage-based, but not automatically cheaper |
| Access | Company-managed network | Network and identity controls are essential |
| Scaling | May require hardware procurement | Often faster, within service limits and quotas |

**Important:** Cloud does not mean "no servers". Servers still exist. The difference is who owns and operates the underlying infrastructure and how customers consume it.

## 3. Why do data engineering teams use cloud?

### 3.1 Scalable storage

A company may store daily CSV exports, JSON API responses, Parquet files, application logs, historical transactions and machine learning datasets.

Cloud object storage can hold large volumes of files without requiring the data engineering team to buy and maintain local storage hardware.

### 3.2 Scalable compute

A transformation that takes several hours on a small machine may be run using larger or distributed compute resources.

Scaling is not unlimited or free. Compute size, runtime, data movement and service quotas matter.

### 3.3 Managed services

A managed database service can reduce the amount of infrastructure administration the application team performs.

The provider may manage underlying hardware and parts of maintenance. The customer still has responsibilities such as:
- schema design;
- access control;
- database configuration;
- backups and recovery choices;
- data protection;
- query performance;
- cost.

### 3.4 Global availability

A business may serve customers in multiple countries. Cloud regions allow organizations to choose infrastructure locations that suit latency, resilience, legal and business requirements.

Not every service is available in every region.

### 3.5 Automation

Cloud resources can be provisioned and managed using web consoles, command-line tools, APIs and infrastructure-as-code tools.

Automation helps teams create repeatable environments. It also means mistakes can be repeated quickly, so reviews and access controls matter.

## 4. When is cloud not automatically the best answer?

Cloud is useful, but it is not magic.

Possible disadvantages:
- ongoing usage charges;
- unexpected cost from forgotten resources;
- network dependency;
- data transfer charges;
- service limits;
- provider-specific designs;
- security misconfiguration;
- operational complexity;
- migration effort.

A good data engineer chooses services based on requirements, not because a service is popular.

---

# Part 2 — Understand cloud service models

## 5. What is IaaS?

IaaS means **Infrastructure as a Service**.

The provider offers basic computing infrastructure such as virtual machines, storage and networking.

The customer manages much of the software stack:
- operating system configuration;
- installed software;
- application runtime;
- application code;
- data;
- workload access.

**Example:** A team creates a virtual machine, installs Python, installs database client libraries and runs a data extraction script.

The cloud provider supplies the virtual machine infrastructure. The team still manages the operating system and application.

**Remember:** IaaS = You receive infrastructure and manage more of the software environment.

## 6. What is PaaS?

PaaS means **Platform as a Service**.

The provider offers a managed platform where developers can deploy or run applications without managing every server detail.

The platform may manage server provisioning, runtime infrastructure, platform maintenance and scaling features.

The customer still manages application code, configuration, data and access choices.

**Example:** A team deploys an application to a managed application platform instead of creating a virtual machine and configuring its runtime manually.

**Remember:** PaaS = You focus more on the application; the provider manages more of the platform.

## 7. What is SaaS?

SaaS means **Software as a Service**.

The provider delivers a complete application that users access through a browser, desktop client or API.

Examples include:
- email applications;
- online document applications;
- customer relationship management software;
- hosted collaboration platforms.

The customer generally configures and uses the software rather than managing its servers or runtime.

**Remember:** SaaS = You use the finished software.

## 8. Compare IaaS, PaaS and SaaS

| Question | IaaS | PaaS | SaaS |
|---|---|---|---|
| What do you receive? | Infrastructure | Managed application platform | Finished software |
| Who manages physical hardware? | Provider | Provider | Provider |
| Who manages much of the OS/runtime? | Customer | Provider | Provider |
| Who manages application code? | Customer | Customer | Usually provider |
| Typical user | Infrastructure/DevOps team | Developer | Business or end user |

The exact responsibility boundary depends on the service. Always read the provider's shared-responsibility documentation.

### 8.1 A simple memory trick

- IaaS: rent the building and arrange much of the inside.
- PaaS: rent a prepared workspace.
- SaaS: use a finished service.

These are learning analogies, not contractual definitions.

## 9. Check your understanding

**Question:** Your team creates a virtual server and installs Python and all required packages. Which model is closest?

**Answer:** IaaS, because the team manages the virtual machine's operating system and software environment.

**Question:** Your team uses a hosted business application through a browser and does not deploy its application code. Which model is closest?

**Answer:** SaaS.

**Question:** Your team deploys its application to a managed runtime and does not manage the underlying virtual machines. Which model is closest?

**Answer:** PaaS.

---

# Part 3 — Cloud deployment types

## 10. Public cloud

A public cloud is operated by a cloud provider and offers services to multiple customers.

Each customer's resources are logically isolated using the provider's security and infrastructure controls.

Examples of public cloud providers include AWS, Microsoft Azure and Google Cloud.

Public cloud does not mean that your data must be publicly accessible. Private access controls and network configuration are essential.

## 11. Private cloud

A private cloud environment is dedicated to one organization.

It may be operated by the organization or a service provider.

Organizations may choose private cloud arrangements for specific control, regulatory or operational requirements.

Private cloud does not automatically guarantee security. It still requires good identity, network, patching and data controls.

## 12. Hybrid cloud

Hybrid cloud combines a private environment with public cloud services.

Example:
- an organization keeps a sensitive operational system in its own data center;
- it copies approved analytical data to cloud storage;
- cloud compute processes the approved data;
- analysts use a controlled reporting environment.

Data movement, identity, network connectivity and governance must be designed carefully.

## 13. Compare deployment types

| Type | Beginner meaning | Example |
|---|---|---|
| Public cloud | Provider-operated services used by multiple customers | A company uses AWS services |
| Private cloud | Cloud environment dedicated to one organization | A bank operates a dedicated private cloud |
| Hybrid cloud | Combination of private and public environments | On-premises source plus cloud analytics |

---

# Part 4 — Cloud regions and availability zones

## 14. What is a cloud region?

A region is a geographic area where a cloud provider operates infrastructure.

Organizations choose regions based on:
- customer location;
- application latency;
- data residency requirements;
- service availability;
- disaster recovery design;
- pricing and business requirements.

A region name is provider-specific. Do not assume all services exist in every region.

## 15. What is an availability zone?

An availability zone (AZ) is an isolated location within a cloud region, designed to provide separation from other zones.

A region generally contains multiple availability zones.

A multi-AZ design can help improve availability when a zone has a problem, provided the application and data architecture are designed to use it.

### 15.1 Region versus availability zone

| Term | Meaning |
|---|---|
| Region | Geographic cloud location |
| Availability zone | Isolated location within a region |
| Multi-region | Architecture using more than one region |
| Multi-AZ | Architecture using more than one availability zone in a region |

### 15.2 Example

TechStore selects a region close to its main customers.

Its critical database is configured for a supported multi-AZ deployment.

The purpose is to reduce the impact of a single availability-zone failure. It does not automatically protect against every failure, accidental deletion, application bug or regional outage.

### 15.3 Beginner architecture picture

Region -> Availability Zone A + Availability Zone B + Availability Zone C

Think of a region as a geographic area and each AZ as a separate infrastructure location inside it.

---

# Part 5 — Meet AWS

## 16. What is AWS?

Amazon Web Services (AWS) is a cloud platform that provides infrastructure, storage, databases, networking, security, analytics and many other services.

A Data Engineer does not need to memorize every AWS service.

Start with the services that appear frequently in data platforms.

## 17. AWS services you should recognize

| Service | Beginner purpose | Data engineering use |
|---|---|---|
| IAM | Identity and access management | Control who can access data resources |
| S3 | Object storage | Store raw, processed and curated files |
| EC2 | Virtual servers | Run scripts or self-managed tools |
| RDS | Managed relational databases | Host application or metadata databases |
| Glue | Managed data integration/catalog capabilities | Catalog and transform data |
| Redshift | Cloud data warehouse | Analytical SQL workloads |
| Lambda | Serverless functions | Event-triggered lightweight processing |
| CloudWatch | Monitoring and logs | Observe services and workloads |
| VPC | Isolated virtual network | Control network placement and traffic |
| KMS | Key management service | Manage encryption keys |

These are introductory descriptions. Each service has different features, limits and pricing.

### 17.1 Which services are most important for this course?

- Day 50: Cloud concepts and IAM fundamentals.
- Day 51: Object storage and data lake layout.
- Day 52: S3 buckets, objects and storage management.
- Day 53: Managed cloud databases and connectivity.
- Day 54: Distributed processing concepts.
- Days 55–58: Apache Spark.
- Day 59: Integrating storage and distributed processing.

Do not try to learn all AWS services in one day.

---

# Part 6 — Shared responsibility and security basics

## 18. What is the shared responsibility model?

Cloud security is shared between the cloud provider and the customer.

The provider is responsible for security **of** the cloud, such as the underlying facilities and infrastructure.

The customer is responsible for security **in** the cloud, such as:
- identity and permissions;
- data classification;
- resource configuration;
- network rules selected by the customer;
- application security;
- credentials;
- workload-specific settings.

The exact boundary changes depending on whether the service is IaaS, PaaS or SaaS.

### Example

For a virtual machine:
- provider manages physical data-center infrastructure;
- customer manages the guest operating system, installed packages, application and configured access.

For a managed database:
- provider manages more of the underlying platform;
- customer still manages database users, permissions, data, configuration choices and application access.

## 19. What is IAM?

IAM means **Identity and Access Management**.

IAM answers two important questions:
1. Who or what is making the request?
2. What is that identity allowed to do?

Examples of identities:
- a human cloud administrator;
- a data engineer;
- an application;
- a scheduled pipeline;
- a virtual machine workload.

An identity should receive only the permissions required for its job.

## 20. Authentication versus authorization

| Term | Simple meaning | Example |
|---|---|---|
| Authentication | Prove who you are | Sign in using an approved identity |
| Authorization | Decide what you can do | Permission to read a specific storage bucket |

A successful sign-in does not mean the identity should be allowed to access every resource.

## 21. IAM users, groups, roles and policies

### IAM user

An IAM user represents an identity in an AWS account.

Long-lived access keys should not be casually created or shared. Prefer temporary credentials and approved identity mechanisms where possible.

### IAM group

A group collects IAM users so permissions can be managed together.

A group is not the same as a role. Workloads generally use roles rather than sharing a human user's credentials.

### IAM role

A role is an identity with permissions that can be assumed by an authorized principal.

Roles are commonly used by applications, virtual machines, cloud services, cross-account access and human users through approved federation.

A role can provide temporary credentials rather than requiring an application to store a long-lived secret.

### IAM policy

A policy is a permissions document that defines allowed or denied actions and, where applicable, the resources and conditions.

A policy is not a password. It describes permissions.

## 22. What is least privilege?

**Least privilege** means giving an identity only the minimum permissions needed to perform its task.

### Bad example

A daily reporting job only needs to read files from one approved data prefix, but it receives administrator-level access to the entire cloud account.

This is too much permission.

### Better design

Give the job a dedicated role that can:
- list only the required bucket or prefix where necessary;
- read only the approved input objects;
- write only to its approved output location if required;
- use only the other services it needs.

The exact policy must match the workload. A read-only task should not receive write or delete permission.

### 22.1 Why least privilege matters

If credentials are leaked or a workload is compromised, unnecessary permissions increase the possible damage.

Least privilege reduces the scope of accidental or malicious actions.

## 23. Read an IAM policy structure

An AWS IAM policy is commonly written as JSON.

This is a **teaching example only**, not a ready-to-deploy policy. The resource name is a placeholder and the required actions depend on the exact task.

~~~json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::example-company-data/approved-input/*"
    }
  ]
}
~~~

Understand each field:

| Field | Meaning |
|---|---|
| Version | Policy language version |
| Statement | One or more permission statements |
| Effect | Allow or Deny |
| Action | Operation being controlled |
| Resource | Resource to which the statement applies |

This example grants an object-read action for objects matching the example path. It does not automatically grant permission to list a bucket. Listing and object reading are different actions.

### 23.1 Why does Resource matter?

A policy that grants an action on all resources may apply broadly.

A resource-specific policy narrows the scope where the service supports resource-level permissions.

Always verify the exact action/resource combination in the AWS service authorization reference before deploying a policy.

### 23.2 Explicit deny

An explicit Deny takes precedence over an Allow in IAM policy evaluation.

Do not assume that attaching one Allow policy guarantees access. Other policy types, boundaries, organization controls and explicit denies can affect the result.

---

# Part 7 — Cloud networking vocabulary

## 24. What is a VPC?

A Virtual Private Cloud (VPC) is a logically isolated network environment in AWS.

A VPC lets an organization define network boundaries and configure:
- IP address ranges;
- subnets;
- route tables;
- network gateways;
- security controls.

A VPC is not the same thing as an S3 bucket. A bucket is an object storage resource; a VPC is a network environment.

## 25. What is a subnet?

A subnet is a range of IP addresses inside a VPC.

Subnets are associated with availability zones.

A common design separates resources into different network areas, such as:
- public-facing components;
- private application components;
- private database components.

The labels "public" and "private" depend on routing and access configuration, not merely on the subnet's name.

## 26. What is a security group?

A security group is a virtual firewall associated with supported cloud resources, such as network interfaces.

It controls allowed inbound and outbound traffic using rules.

Example: A database should accept connections only from the approved application security group, not from every IP address on the internet.

Do not use an open-to-everyone inbound database rule as a beginner shortcut.

## 27. Why does networking matter to data engineers?

A pipeline may need to connect:
- from a compute service to object storage;
- from a Spark cluster to a database;
- from an orchestration service to an API;
- from an approved office or VPN network to a cloud database.

A correct username and password are not enough if the network path is blocked.

Likewise, an open network path does not grant database authorization.

---

# Part 8 — A simple cloud data engineering architecture

## 28. Business requirement

TechStore wants a daily sales report.

The source system exports order data every night.

The data engineering workflow should:
1. receive the source file;
2. store the original file;
3. validate the file;
4. transform the data;
5. store a curated result;
6. make the result available for reporting;
7. monitor failures;
8. restrict access.

## 29. Architecture components

| Layer | Example cloud capability | Responsibility |
|---|---|---|
| Source | Order application | Creates operational data |
| Ingestion | Scheduled extraction or event | Moves source data |
| Raw storage | Object storage | Keeps original files |
| Processing | Managed or distributed compute | Cleans and transforms |
| Curated storage | Object storage or warehouse | Stores trusted output |
| Analytics | Data warehouse/query service | Supports analytical queries |
| Monitoring | Logs and metrics service | Tracks health and failures |
| Security | IAM and network controls | Restricts access |

### 29.1 Flow

Order database -> Ingestion -> Raw object storage -> Processing -> Curated data -> Warehouse/BI

Monitoring and access controls apply across the workflow.

## 30. Why keep raw data?

Raw data is the original source copy received by the pipeline.

It can help with:
- reprocessing after a transformation bug;
- investigating source changes;
- comparing original and transformed values;
- audit requirements;
- recovering from downstream failures.

Raw storage still needs access control, retention rules, encryption, lifecycle management, data classification and deletion policies.

Do not keep sensitive raw data forever without a valid business and legal reason.

## 31. Why separate raw, processed and curated data?

A common data lake layout is:
- **Raw:** original source data.
- **Processed:** standardized, cleaned or conformed data.
- **Curated:** business-ready datasets with documented definitions.

The names and exact number of layers vary between organizations.

The goal is to make data state and quality clear.

## 32. Cloud architecture design questions

Before choosing a service, ask:
1. What business problem are we solving?
2. What is the source?
3. How much data arrives?
4. How often does it arrive?
5. How quickly must the result be available?
6. Is processing batch or streaming?
7. What data is sensitive?
8. Which identities need access?
9. How will failures be detected?
10. How will data be recovered?
11. What is the expected cost?
12. Who owns the pipeline?

A good architecture starts with requirements, not with a list of cloud product names.

---

# Part 9 — Cost and resource awareness

## 33. Is cloud free?

Not necessarily.

Some providers offer free usage allowances or credits subject to eligibility, service limits and time periods. Many resources can create charges even when a student is not actively using them.

Possible charge sources include:
- running virtual machines;
- managed database instances;
- storage;
- requests;
- network data transfer;
- logs and monitoring;
- public IP addresses;
- snapshots and backups;
- managed clusters.

Always check current pricing and account-specific free-tier conditions before creating resources.

## 34. Safe learning habits

Before creating a cloud resource:
1. identify the service;
2. read the pricing page;
3. choose the smallest suitable configuration;
4. understand the expected lifetime;
5. set a budget or billing alert where available;
6. avoid uploading sensitive personal or company data;
7. record the resource name and region;
8. delete resources when the exercise is finished;
9. verify that dependent resources were also removed;
10. review the billing dashboard.

A budget alert is not necessarily a hard spending cap. Do not treat an alert as a guarantee that charges cannot exceed the target.

## 35. Classroom practice without a cloud bill

You can learn many concepts without deploying a cloud resource.

For today's lesson:
- draw the architecture;
- classify IaaS/PaaS/SaaS examples;
- design an IAM permission matrix;
- review a sample policy;
- identify public and private network boundaries;
- estimate which services might create cost.

Actual AWS console practice can be done later with an approved training account and supervision.

---

# Part 10 — A guided IAM permission-design exercise

## 36. Scenario

TechStore has a daily reporting pipeline.

It reads order files from one approved input location and writes summary files to one approved output location.

There are three identities:

| Identity | Responsibility |
|---|---|
| Data engineer | Develop and troubleshoot the pipeline |
| Pipeline role | Run the scheduled data job |
| Reporting analyst | Read approved curated reports |

## 37. Design permissions before writing policies

| Identity | Needs | Does not need by default |
|---|---|---|
| Data engineer | Approved development access | Unrestricted production administrator access |
| Pipeline role | Read input, write output, required logs | Delete all source data, manage all IAM users |
| Reporting analyst | Read curated reports | Read raw sensitive files, modify pipeline resources |

This table is a permission design. The exact policy must be tested against the service and organization's access model.

## 38. Think through a security failure

A developer places a cloud access key directly inside a Python file and pushes the file to a public Git repository.

What is wrong?
- The credential can be copied by others.
- Removing it from the latest file may not remove it from Git history.
- The key may remain active.
- An attacker may use its permissions.
- The team may incur unexpected charges or expose data.

Safer response:
1. revoke or deactivate the exposed credential immediately;
2. investigate its use;
3. rotate credentials as required;
4. remove secrets from the repository and history using the organization's process;
5. move workloads to an approved role or secrets-management approach;
6. review and reduce permissions;
7. add secret scanning and review controls.

Never paste real cloud keys into notes, chat, screenshots or source code.

---

# Part 11 — Common beginner mistakes

| Mistake | Why it is a problem | Better practice |
|---|---|---|
| Cloud means no servers | Physical infrastructure still exists | Understand provider/customer responsibilities |
| Cloud is always cheaper | Usage and data transfer can be expensive | Estimate and monitor cost |
| Public cloud means public data | Public cloud resources can be private | Configure identity and network controls |
| IaaS, PaaS and SaaS are interchangeable | Responsibility differs | Identify what the provider manages |
| Authentication means full access | Sign-in and permission are different | Apply authorization policies |
| Give every job admin access | A mistake can affect the entire account | Use least privilege |
| Put keys in Python files | Secrets can leak | Use roles or approved secret management |
| Open database ports to everyone | Exposes the database | Restrict network sources |
| One AZ means resilient | A single location can fail | Design supported redundancy |
| Create resources without checking price | Charges can continue | Check cost and clean up |
| Choose services before requirements | Architecture may be unsuitable | Start with business and workload needs |

---

# Part 12 — Practice questions with explanations

## Exercise 1: Choose a service model

A company rents a virtual machine and installs its own operating system packages and Python application.

**Answer:** IaaS.

**Why:** The company is managing the operating system and application environment on top of provider infrastructure.

## Exercise 2: Choose a service model

A business uses a hosted email application through a browser.

**Answer:** SaaS.

**Why:** The provider delivers a finished software application.

## Exercise 3: Authentication or authorization?

A user signs in successfully but cannot read a protected data bucket.

**Answer:** Authentication succeeded, but authorization did not permit the requested action.

## Exercise 4: Identify least privilege

A pipeline only reads one input location but has administrator access to the account.

**Answer:** It violates least privilege. Give it only the required read and operational permissions.

## Exercise 5: Region or availability zone?

A company deploys resources in two isolated locations within one geographic cloud region.

**Answer:** It is using multiple availability zones in one region.

## Exercise 6: Shared responsibility

Who configures the permissions that determine which application identity can read a customer's data?

**Answer:** The customer is responsible for configuring the workload's identity and data access appropriately, even though the provider operates the underlying cloud infrastructure.

## Exercise 7: Cost

A student starts a managed database for practice and closes the browser without deleting the database.

**Answer:** The database may continue running and incur charges. Closing a browser tab does not stop a cloud resource.

## Exercise 8: Architecture

Where should original source files be retained before transformation in a common data lake design?

**Answer:** In a controlled raw layer, subject to retention, security and data governance requirements.

---

# Part 13 — Revision sheet

| Term | Remember this |
|---|---|
| Cloud computing | Consuming computing resources as services over a network |
| IaaS | Infrastructure service; customer manages more of the software stack |
| PaaS | Managed application platform |
| SaaS | Finished software service |
| Public cloud | Provider-operated cloud services used by multiple customers |
| Private cloud | Cloud environment dedicated to one organization |
| Hybrid cloud | Combination of private and public environments |
| Region | Geographic cloud location |
| Availability zone | Isolated location within a region |
| AWS | Amazon Web Services |
| IAM | Identity and Access Management |
| Authentication | Proving identity |
| Authorization | Checking permissions |
| IAM role | Assumable identity with permissions |
| IAM policy | Document describing permissions |
| Least privilege | Minimum access required for a task |
| VPC | Logically isolated virtual network |
| Subnet | IP address range within a VPC |
| Security group | Virtual firewall for supported resources |
| Shared responsibility | Provider and customer have different security duties |
| Raw layer | Original source data |
| Curated layer | Business-ready data |
| Cloud cost awareness | Understand, monitor and clean up resource usage |

## Final self-check

- [ ] Explain why a data engineering team may use cloud services.
- [ ] Compare on-premises infrastructure and cloud.
- [ ] Explain IaaS, PaaS and SaaS with examples.
- [ ] Compare public, private and hybrid cloud.
- [ ] Explain regions and availability zones.
- [ ] Describe IAM users, roles and policies.
- [ ] Explain authentication versus authorization.
- [ ] Explain least privilege using a pipeline example.
- [ ] Describe shared responsibility.
- [ ] Explain why VPCs, subnets and security groups matter.
- [ ] Draw a basic raw-to-curated cloud data flow.
- [ ] Explain why cloud cost and cleanup must be planned.

**Next lesson — Day 51: Object Storage and Data Lakes.** You will learn how files are stored as objects, why object storage is different from a traditional file system, and how to organize raw, processed and curated data in a data lake.
