# Day 52 Interview Guide — AWS S3 and Storage Management

## 1. What is Amazon S3?

Amazon S3 is AWS object storage. It stores objects in buckets and provides APIs and tools to upload, retrieve, list and manage them. Data engineering teams use it for raw files, data lakes, logs, backups and curated datasets.

## 2. What is the difference between a bucket and an object?

A bucket is a container for objects. An object is an individual stored item containing data and object attributes. The object is identified by its key within the bucket.

## 3. What is an S3 object key?

An object key is the name that identifies an object within a bucket. It can contain slash characters that tools display as folder-like prefixes. The key is not necessarily a traditional filesystem path.

## 4. How is S3 different from a relational database?

S3 stores objects such as files. It does not provide relational tables, joins and row-level transactions in the same way as a relational database. Query engines and data platforms can read files stored in S3 for analytics.

## 5. How do you upload a file to S3?

A user or pipeline uses the AWS console, CLI or SDK to send a local file to a bucket and key. Before upload, the pipeline should verify file completeness, destination, identity, overwrite behavior and encryption requirements. After upload it should verify object existence and expected integrity.

## 6. What is the AWS CLI?

AWS CLI is a command-line tool for interacting with AWS services. It can list buckets, list objects, upload, download, copy and sync files. It uses the credentials and identity configured for the process and does not bypass IAM permissions.

## 7. Why run get-caller-identity before an operation?

It confirms which AWS identity and account the CLI is using. This reduces the risk of operating against the wrong account or role.

## 8. What is the difference between cp and sync?

The cp command copies a specified file or object. The sync command compares a source and destination and transfers files as needed. Destructive sync options can delete destination objects, so they must be reviewed and tested carefully.

## 9. What are S3 storage classes?

Storage classes provide different storage and access-cost characteristics for different access patterns. Common families include Standard, Intelligent-Tiering, Standard-IA, One Zone-IA and Glacier archive classes. Selection must consider retrieval time, retrieval charges, minimum storage duration and resilience needs.

## 10. What is an S3 lifecycle policy?

A lifecycle policy applies automated actions to objects, such as transitioning them to another storage class, expiring them or managing noncurrent versions. Rules should follow approved retention and recovery requirements.

## 11. What is S3 versioning?

Versioning allows multiple versions of an object key to be retained. It can help recover from accidental overwrites and deletes. It can also increase storage cost, so noncurrent version retention should be designed.

## 12. What is a delete marker?

In a versioning-enabled bucket, a normal delete commonly creates a delete marker as the current version rather than permanently removing all earlier versions. Permanently deleting a specific version is a separate operation.

## 13. What is encryption at rest?

Encryption at rest protects stored object data. S3 supports server-side encryption options, including S3-managed and AWS KMS key options. Encryption configuration and key permissions should match the organization's requirements.

## 14. What is encryption in transit?

Encryption in transit protects data while it moves between a client and the storage service. Secure transport such as HTTPS/TLS should be used for supported requests.

## 15. Does encryption replace IAM permissions?

No. Encryption and authorization solve different problems. IAM and bucket policies control access. If KMS keys are used, the principal may also need the appropriate key permissions.

## 16. What is S3 Block Public Access?

It is a set of controls that helps prevent public access to buckets and objects. Private company data should normally remain non-public. Do not make a bucket public to solve an access-denied problem.

## 17. What is the difference between an IAM policy and a bucket policy?

An IAM policy is attached to an identity such as a role. A bucket policy is a resource-based policy attached to a bucket. Both can influence access evaluation, along with explicit denies and organization controls.

## 18. What is boto3?

Boto3 is the AWS SDK for Python. It allows Python code to call AWS APIs. It uses the credential provider chain and active identity. Applications should not hardcode long-lived access keys.

## 19. How do you prevent accidental overwrite of daily files?

Use unique, predictable object keys such as a dataset prefix plus ingestion date or run identifier. Versioning can provide an additional recovery mechanism. The pipeline should define whether overwrites are allowed and validate the destination.

## 20. Scenario: upload succeeds but downstream cannot read the object.

I would verify the exact bucket and key, the downstream role identity, object read permission, bucket policy, organization controls, encryption key permissions and region/configuration. I would not immediately grant broad administrator permissions.

## 21. Scenario: a bucket contains millions of old files. How would you control cost?

I would classify the data by retention and access pattern, identify approved lifecycle transitions and expiration rules, review versioning and incomplete multipart uploads, and estimate retrieval and minimum-duration costs. I would validate legal and business retention before deleting anything.

## 22. Explain your Day 52 project.

I designed a bucket and prefix structure for TechStore, selected storage class families based on access patterns, proposed lifecycle and versioning rules, documented encryption and access boundaries, and reviewed safe CLI operations. The work is a storage-management design unless a real AWS deployment has separately been performed and verified.

## Common interview traps

- Calling S3 a relational database.
- Assuming slash-separated prefixes are real directories.
- Using the same object key for every daily delivery.
- Making a private bucket public to fix access denied.
- Assuming versioning is a backup strategy by itself.
- Ignoring noncurrent version cost.
- Choosing archive storage without checking retrieval needs.
- Treating encryption as authorization.
- Running destructive sync/delete commands without review.

## Readiness checklist

- [ ] Explain S3 buckets, objects, keys and prefixes.
- [ ] Explain CLI upload/download/copy/sync.
- [ ] Compare storage class families.
- [ ] Explain lifecycle and versioning.
- [ ] Explain encryption and Block Public Access.
- [ ] Explain IAM and bucket policies.
- [ ] Describe safe upload and deletion practices.
