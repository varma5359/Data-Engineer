# Day 52: AWS S3 and Storage Management

**Phase:** 6 — Cloud and Distributed Processing  
**Learning time:** 2–3 hours  
**Audience:** Complete beginners and junior Data Engineer candidates  
**Cloud service:** Amazon Simple Storage Service (Amazon S3)  
**Topic boundary:** Class notes only. Project workbook and interview guide are separate files.

> Day 51 taught object storage and data lake design. Today we apply those ideas to Amazon S3. You will learn buckets, objects, prefixes, storage classes, lifecycle policies, encryption, access control, upload/download concepts and safe operational habits. You can complete the design and CLI reading exercises without creating an AWS resource.

## Learning outcomes

By the end, you should be able to:
1. Explain the purpose of Amazon S3.
2. Distinguish a bucket, object, key and prefix.
3. Explain bucket names and regions.
4. Understand object upload and download.
5. Recognize common S3 storage classes.
6. Explain lifecycle policies.
7. Explain versioning and delete markers.
8. Understand encryption at rest and in transit.
9. Explain S3 Block Public Access and least-privilege access.
10. Recognize common AWS CLI commands.
11. Understand upload/download using the AWS SDK conceptually.
12. Design safe storage operations and cleanup.

---

# Part 1 — What is Amazon S3?

## 1. What is S3?

Amazon Simple Storage Service (Amazon S3) is AWS object storage. It stores objects in buckets and provides APIs and tools to create, retrieve, list and manage those objects.

Data engineering teams use S3 to store:
- raw source files;
- processed datasets;
- curated analytical files;
- application logs;
- backups;
- model training data;
- exports and reports.

S3 is not a relational database. It does not provide SQL tables and row-level transactions in the same way as a relational database. Analytics engines can read files stored in S3.

## 2. Understand the S3 vocabulary

| Term | Meaning |
|---|---|
| Bucket | Container for objects |
| Object | Stored data and its object attributes |
| Key | Unique object name within a bucket |
| Prefix | Beginning part of object keys used to group/list objects |
| Region | AWS geographic region associated with the bucket |
| Version ID | Identifier for a stored object version when versioning is enabled |
| Storage class | Storage tier that affects access, durability design and cost |
| Lifecycle rule | Rule to transition or expire objects |
| Bucket policy | Resource-based policy attached to a bucket |

## 3. Example S3 object

Bucket:

~~~text
techstore-data-prod
~~~

Key:

~~~text
raw/orders/ingest_date=2026-11-10/orders_001.csv
~~~

S3 URI:

~~~text
s3://techstore-data-prod/raw/orders/ingest_date=2026-11-10/orders_001.csv
~~~

Understand the URI:
- s3:// identifies the S3 scheme;
- techstore-data-prod is the bucket;
- the remaining path is the object key.

A bucket name must satisfy AWS naming rules and be unique in the applicable namespace. The example name may already be taken and is not guaranteed to be available.

## 4. Is an S3 folder a real folder?

S3 is object storage. Object keys form a flat namespace within a bucket. A key containing slashes can be displayed as nested folders by the console and other tools.

For example:

~~~text
raw/orders/2026/11/orders.csv
~~~

The prefixes raw/, raw/orders/ and raw/orders/2026/ can be used to group and list objects.

Do not assume directory rename behaves like a local file system. Moving an object is generally implemented by copying it to a new key and deleting the old object, subject to permissions and tool behavior.

---

# Part 2 — Create and organize buckets

## 5. Bucket planning

Before creating a bucket, decide:
- environment: development, test or production;
- data owner;
- region;
- sensitivity classification;
- encryption requirements;
- public access posture;
- retention;
- versioning;
- logging and monitoring;
- expected storage and request volume.

Avoid putting personal information or secrets in bucket names.

## 6. Example naming convention

~~~text
company-data-dev
company-data-test
company-data-prod
~~~

These are illustrative names only. S3 bucket names have naming constraints and must be available.

## 7. Separate environments

A development pipeline should not accidentally overwrite production data.

Possible separation:
- separate buckets;
- separate AWS accounts;
- separate roles and policies;
- separate deployment configuration.

The right level of separation depends on the organization's security and operational design.

## 8. Example TechStore layout

~~~text
techstore-data-prod/
    raw/
        orders/
        payments/
        deliveries/
    processed/
        orders/
        payments/
    curated/
        sales_daily/
        product_performance/
    quarantine/
        orders/
    archive/
~~~

Each prefix should have a clear meaning and owner.

---

# Part 3 — Upload and download concepts

## 9. What does upload mean?

Upload means transferring a local file or generated data into an S3 object.

Example:
A nightly order export is created on a trusted processing host and uploaded to the raw/orders prefix.

Before upload, the pipeline should check:
- file creation completed;
- expected file name;
- file size;
- file format;
- checksum or integrity evidence where required;
- no accidental duplicate or overwrite;
- destination bucket and key;
- identity permissions.

## 10. What does download mean?

Download retrieves an S3 object to a local or compute environment.

Before download:
- verify the bucket and key;
- confirm that the identity is authorized;
- use a safe destination path;
- check available disk space;
- avoid exposing sensitive data in shared locations;
- validate the downloaded file.

## 11. Upload is not the same as copy

A local-to-S3 upload sends data from a local filesystem to an object.

An S3-to-S3 copy copies an object between keys or buckets.

A download copies an object from S3 to a local destination.

Large transfers may use multipart upload, where an object is transferred in multiple parts. The SDK or CLI can manage this automatically for supported operations.

## 12. Why object naming matters during upload

Suppose the pipeline uploads every daily file to:

~~~text
raw/orders/orders.csv
~~~

The next day's file may replace the previous object if the same key is used.

A safer date-based key could be:

~~~text
raw/orders/ingest_date=2026-11-10/orders.csv
raw/orders/ingest_date=2026-11-11/orders.csv
~~~

This preserves separate daily objects. Whether overwrites are allowed should be a deliberate pipeline decision.

---

# Part 4 — AWS CLI basics

## 13. What is AWS CLI?

AWS Command Line Interface (CLI) is a tool that lets you interact with AWS services from a terminal.

A data engineer may use the CLI to:
- list buckets;
- list objects;
- upload files;
- download files;
- copy objects;
- inspect configuration.

The CLI does not bypass permissions. It uses credentials or an identity mechanism configured for the command environment.

## 14. Check whether AWS CLI is installed

In Windows CMD or PowerShell:

~~~bash
aws --version
~~~

Example output (the exact version differs):

~~~text
aws-cli/2.x.x Python/3.x Windows/10 exe/AMD64
~~~

If the command is not recognized, the CLI may not be installed or may not be available in PATH.

Do not copy an example version and claim it is your installed version. Record the output from your own machine.

## 15. Check the active identity

~~~bash
aws sts get-caller-identity
~~~

This asks AWS Security Token Service to identify the caller represented by the current credentials.

The response typically contains account, ARN and user ID fields.

This is a useful safety check before running commands. Confirm that you are using the intended account and role.

## 16. Configure credentials safely

Preferred approaches depend on the environment and organization. Examples include:
- AWS IAM Identity Center and an approved SSO profile;
- an attached IAM role for a workload;
- temporary credentials supplied by an approved identity system.

Avoid hardcoding long-lived access keys in scripts, notebooks or Git repositories.

If an organization uses a named CLI profile, the profile can be selected with the appropriate CLI option. Follow the organization's approved setup instructions.

## 17. List buckets

~~~bash
aws s3 ls
~~~

This lists buckets visible to the current identity, subject to permissions.

If access is denied, possible causes include:
- the wrong identity;
- missing permission;
- organization-level controls;
- profile or region configuration;
- an explicit deny.

Do not respond to access denied by giving the identity administrator access. Investigate the required action and resource.

## 18. List objects in a bucket

~~~bash
aws s3 ls s3://example-company-data/
~~~

The bucket name is a placeholder. Replace it only with an approved bucket you own or are authorized to access.

A recursive listing can show objects under prefixes:

~~~bash
aws s3 ls s3://example-company-data/raw/orders/ --recursive
~~~

Large prefixes can return many results. Avoid repeatedly listing huge production paths without a reason.

## 19. Upload a file

~~~bash
aws s3 cp orders.csv s3://example-company-data/raw/orders/ingest_date=2026-11-10/orders.csv
~~~

Explanation:
- aws s3 cp invokes the S3 copy operation;
- orders.csv is the local source;
- the S3 URI is the destination.

Expected behavior: the CLI reports an upload when the operation succeeds. The exact transfer summary depends on the CLI version and file.

This command is a template. Do not execute it with an unknown bucket.

## 20. Download a file

~~~bash
aws s3 cp s3://example-company-data/raw/orders/orders.csv downloaded_orders.csv
~~~

The S3 URI is the source and the local file is the destination.

Expected behavior: the CLI reports a download if it succeeds.

## 21. Copy an object within S3

~~~bash
aws s3 cp s3://example-company-data/raw/orders/orders.csv s3://example-company-data/processed/orders/orders.csv
~~~

This creates a copy at the destination key. It does not transform the contents.

A production pipeline should not copy raw data into a processed zone and call it cleaned. The transformation must actually happen.

## 22. Sync a directory

~~~bash
aws s3 sync ./daily_exports s3://example-company-data/raw/daily_exports/
~~~

Sync compares source and destination and transfers files as needed.

Be careful with deletion options. A sync command with delete behavior can remove destination objects that are not present in the source. Never add destructive options without understanding the exact effect and testing in a safe location.

## 23. Check command help

~~~bash
aws s3 help
aws s3 cp help
~~~

Help output can vary by CLI version. Use it to inspect supported options in your installed version.

---

# Part 5 — S3 storage classes

## 24. Why does S3 have storage classes?

Not every object is accessed at the same frequency.

Examples:
- active daily sales files may be read often;
- old audit files may be read rarely;
- completed project archives may almost never be read.

Storage classes provide different cost and retrieval characteristics.

Do not select a class using storage price alone. Consider retrieval frequency, retrieval delay, minimum storage duration, retrieval charges and availability needs.

## 25. S3 Standard

S3 Standard is designed for frequently accessed data.

Typical example:
- active raw files;
- current curated reports;
- frequently queried datasets.

## 26. S3 Intelligent-Tiering

Intelligent-Tiering can automatically move eligible objects between access tiers based on access patterns, subject to its rules and monitoring/automation charges.

It may help when access patterns are uncertain.

It is not a universal choice. Check eligibility, tier behavior, monitoring charges and retrieval characteristics.

## 27. S3 Standard-Infrequent Access

Standard-IA is designed for data accessed less frequently but requiring rapid access when needed.

It has different pricing, including retrieval charges and minimum storage duration rules.

## 28. S3 One Zone-Infrequent Access

One Zone-IA stores data in a single availability zone and is intended for infrequently accessed data that can be recreated or for which another copy exists.

It is not the right choice for the only copy of irreplaceable data.

## 29. S3 Glacier storage classes

Glacier storage classes are intended for archive use cases. Retrieval can involve waiting periods and retrieval charges depending on the class and retrieval option.

A data engineer must confirm the recovery time requirement before archiving data.

## 30. Storage class comparison

| Class family | Typical access pattern | Key consideration |
|---|---|---|
| Standard | Frequent | General active data |
| Intelligent-Tiering | Unknown or changing | Monitoring/automation and tier behavior |
| Standard-IA | Infrequent | Retrieval charges and minimum duration |
| One Zone-IA | Infrequent, recreatable | Single-AZ storage design |
| Glacier classes | Archive | Retrieval delay and cost |

AWS can change service features and pricing. Always consult the current S3 pricing and storage class documentation before making a production decision.

---

# Part 6 — Lifecycle policies

## 31. What is a lifecycle policy?

A lifecycle policy applies rules to objects over time.

Possible actions include:
- transition objects to another storage class;
- expire objects after a retention period;
- manage noncurrent versions when versioning is enabled;
- abort incomplete multipart uploads.

A lifecycle rule can be scoped to a bucket or selected prefixes/tags, depending on the supported configuration.

## 32. Example business rule

TechStore decides:
- active daily files stay in an active storage class;
- older raw files move to a lower-cost class after an approved period;
- temporary export files expire after a short retention period;
- incomplete multipart uploads are cleaned up.

The exact number of days and storage class must be selected from current business, legal and cost requirements.

## 33. Lifecycle design questions

Before creating a lifecycle rule, ask:
1. Who owns the data?
2. Is the data still needed?
3. How often is it read?
4. How quickly must it be retrieved?
5. Is there a minimum storage duration?
6. Are there retrieval charges?
7. Are object versions enabled?
8. Does a legal hold or retention rule apply?
9. Does the data have another backup?
10. Who approved the expiration?

Do not create an expiration rule based only on a guess.

---

# Part 7 — Versioning and recovery

## 34. What is S3 versioning?

Versioning keeps multiple versions of an object in a versioning-enabled bucket.

If an object with the same key is uploaded again, S3 can retain the previous version and store a new version.

This can help recover from accidental overwrites or deletes.

Versioning does not replace backup planning, access control or retention governance.

## 35. What is a delete marker?

In a versioning-enabled bucket, a normal delete operation commonly adds a delete marker as the current version rather than permanently deleting all prior versions.

Older versions may remain and can incur storage charges.

Permanent deletion of a specific version is a different operation and requires appropriate permissions.

## 36. Why versioning can increase cost

Every retained version consumes storage.

If a pipeline overwrites a large object every day and retains every version indefinitely, storage can grow.

Lifecycle rules for noncurrent versions may be needed, subject to retention and recovery requirements.

---

# Part 8 — Security and encryption

## 37. What does encryption protect?

Encryption helps protect data from being read by unauthorized parties if storage or network access is exposed.

Two important categories:
- encryption in transit;
- encryption at rest.

## 38. Encryption in transit

Encryption in transit protects data while it moves between systems.

Use secure transport such as HTTPS/TLS for supported S3 requests.

Do not send credentials or data over insecure connections.

## 39. Encryption at rest

Encryption at rest protects stored objects.

S3 supports server-side encryption options, including S3-managed keys and AWS KMS key options. Exact options and defaults can evolve.

The organization should define:
- encryption requirements;
- key ownership;
- key access;
- rotation expectations;
- audit requirements;
- recovery behavior.

## 40. Server-side encryption

With server-side encryption, S3 encrypts object data at rest as part of the storage service behavior.

The application sends and retrieves objects through the service while encryption is managed according to the selected configuration.

Encryption does not replace IAM permissions. A user who is authorized to read an object may still receive decrypted data through the service.

## 41. KMS key permissions

When customer-managed KMS keys are used, key policies and IAM permissions can affect whether a principal can use the key.

A role may have permission to read an S3 object but still fail if it lacks the required KMS key permission.

Troubleshooting should check both storage permissions and encryption-key permissions.

## 42. Block Public Access

S3 Block Public Access settings help prevent public access to buckets and objects.

For private company data, keep public access blocked unless a documented, reviewed use case requires a different configuration.

Do not make a bucket public just to solve an application permission problem.

## 43. Bucket policy and IAM policy

An IAM identity policy is attached to an identity such as a user or role.

A bucket policy is a resource-based policy attached to a bucket.

Both can participate in access evaluation. Explicit denies and organization-level controls may also affect access.

## 44. Example access design

| Identity | Access |
|---|---|
| Ingestion role | Write new objects to raw/orders |
| Transformation role | Read raw/orders and write processed/orders |
| Reporting role | Read curated/sales_daily |
| General analyst | No raw customer data access by default |

This is a design example. The actual policy must account for listing, object reads/writes, encryption, prefixes and required service actions.

---

# Part 9 — Upload/download with Python SDK concepts

## 45. What is an SDK?

An SDK is a software development kit. It provides libraries that let application code call a cloud provider's APIs.

For AWS Python development, boto3 is a common AWS SDK for Python.

The SDK uses the credential provider chain and the identity available to the process. Do not paste access keys directly into the code.

## 46. Install boto3 in a practice environment

~~~bash
python -m pip install boto3
~~~

Expected output: pip reports installation or that the package is already installed. Exact output depends on the environment and package version.

## 47. Create an S3 client

~~~python
import boto3

s3 = boto3.client("s3")

print("S3 client created")
~~~

Expected output:

~~~text
S3 client created
~~~

Important:
- this creates a client object;
- it does not prove that credentials are valid;
- it does not prove the identity has access to a bucket;
- an API request is needed to verify access.

## 48. Upload with boto3

~~~python
import boto3

s3 = boto3.client("s3")

s3.upload_file(
    "orders.csv",
    "example-company-data",
    "raw/orders/orders.csv"
)

print("Upload request completed")
~~~

This is a template. It will only work if:
- the local file exists;
- the bucket exists;
- the process has valid credentials;
- the identity has required permissions;
- the destination and encryption requirements are correct.

Do not run it with an unknown bucket or production destination.

## 49. Download with boto3

~~~python
import boto3

s3 = boto3.client("s3")

s3.download_file(
    "example-company-data",
    "raw/orders/orders.csv",
    "downloaded_orders.csv"
)

print("Download request completed")
~~~

The bucket, key and local destination are explicit.

A successful download should still be followed by validation such as file size, checksum or row count where appropriate.

## 50. Handle errors

Cloud SDK calls can fail for several reasons:
- missing credentials;
- expired credentials;
- access denied;
- bucket or key not found;
- network timeout;
- throttling;
- encryption key permission;
- incorrect region or endpoint.

A production pipeline should handle known errors, retry only where safe, log context without exposing secrets and report final failure clearly.

Do not retry every error forever. Permission errors generally require configuration correction rather than repeated retries.

---

# Part 10 — Safe operational workflow

## 51. Before upload

- Confirm the source file is complete.
- Confirm its expected name and format.
- Validate required columns and row counts.
- Calculate checksum if required.
- Confirm destination bucket and prefix.
- Confirm the active AWS identity.
- Confirm overwrite behavior.
- Confirm encryption and retention requirements.

## 52. After upload

- Verify the object exists.
- Verify expected object size.
- Compare checksum where supported and appropriate.
- Record the object key and pipeline run ID.
- Record source file and ingestion timestamp.
- Emit success/failure metrics.
- Avoid logging credentials or sensitive row values.

## 53. Before delete

- Confirm the exact bucket and key.
- Confirm data owner approval.
- Check retention and legal hold requirements.
- Check versioning and prior versions.
- Confirm backups or replay capability.
- Test the deletion process in development.
- Record the operation.

A broad recursive delete can remove many objects. Never use destructive commands against production without review and safeguards.

---

# Part 11 — Common mistakes

| Mistake | Why it is a problem | Better practice |
|---|---|---|
| Treat S3 as a relational database | It stores objects, not relational rows | Use an appropriate query engine or database |
| Assume prefixes are real folders | Key namespace is object-based | Understand keys and prefix listing |
| Reuse the same key for every daily file | New upload may overwrite prior object | Use date/run-specific keys or versioning |
| Make bucket public to fix access denied | Exposes data | Correct identity and bucket permissions |
| Store access keys in Python | Secrets can leak | Use roles or approved temporary credentials |
| Choose archive tier only by storage price | Retrieval may be slow or costly | Check access and recovery needs |
| Ignore noncurrent versions | Storage can grow | Design version retention |
| Delete without checking versioning | Prior versions may remain | Understand delete markers and version deletion |
| Assume encryption gives authorization | Encryption and access control differ | Configure IAM and key permissions |
| Use sync delete without testing | Destination data may be removed | Test and review destructive options |
| Log full credentials or sensitive data | Exposes secrets | Log safe identifiers and status |

---

# Part 12 — Practice questions

## Exercise 1

What is an S3 object key?

**Answer:** The name that identifies an object inside a bucket.

## Exercise 2

Does an S3 prefix always represent a real directory?

**Answer:** No. It is a beginning portion of object keys. Tools commonly display prefixes as folders.

## Exercise 3

Why use different keys for daily files?

**Answer:** To preserve separate deliveries and avoid accidental overwrites.

## Exercise 4

What is a lifecycle policy?

**Answer:** A rule that can transition or expire objects over time and manage other supported lifecycle actions.

## Exercise 5

What does versioning help with?

**Answer:** Recovering from accidental overwrite or deletion by retaining object versions, subject to configuration and retention.

## Exercise 6

Does encryption replace IAM?

**Answer:** No. Encryption protects data at rest or in transit; IAM and resource policies control access.

## Exercise 7

Why check the active CLI identity?

**Answer:** To avoid running a command against the wrong account or with unintended permissions.

## Exercise 8

Why should a student avoid creating a bucket or database without checking pricing?

**Answer:** Cloud resources and requests may create charges. Cost and cleanup must be planned.

---

# Part 13 — Revision sheet

| Term | Remember |
|---|---|
| S3 | AWS object storage |
| Bucket | Container for objects |
| Object key | Object identifier within bucket |
| Prefix | Grouping portion of object keys |
| AWS CLI | Terminal tool for AWS services |
| boto3 | AWS SDK for Python |
| Storage class | Storage tier with different access/cost behavior |
| Lifecycle policy | Automated transition/expiration rules |
| Versioning | Retains object versions |
| Delete marker | Current marker created by a delete in a versioned bucket |
| Encryption in transit | Protects data while moving |
| Encryption at rest | Protects stored data |
| Block Public Access | Controls that help prevent public bucket/object access |
| Bucket policy | Resource-based bucket permissions |
| IAM policy | Identity permissions |
| Multipart upload | Transfers an object in multiple parts |
| KMS | AWS key management service |

## Final self-check

- [ ] Explain S3 buckets, objects, keys and prefixes.
- [ ] Explain upload, download, copy and sync.
- [ ] Recognize common AWS CLI commands.
- [ ] Explain common S3 storage class families.
- [ ] Explain lifecycle policies and versioning.
- [ ] Explain encryption in transit and at rest.
- [ ] Explain IAM and bucket policy roles.
- [ ] Describe safe upload and delete checks.
- [ ] Explain how boto3 uses an AWS identity.
- [ ] Identify common cost and security mistakes.

**Next lesson — Day 53: Cloud Database Fundamentals.** You will learn managed databases, database connectivity, endpoints, ports, security groups, credentials and safe application connections.
