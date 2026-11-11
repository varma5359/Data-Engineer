# Day 50 Interview Guide — Cloud Data Engineering Fundamentals

## 1. What is cloud computing?

Cloud computing is the delivery of computing resources such as servers, storage, databases and networking as services over a network. Instead of owning and operating all physical infrastructure, an organization provisions resources from a cloud provider. The organization still configures its workloads, identities, data and security controls.

**Example:** A data engineering team stores daily order files in cloud object storage and runs transformations using cloud compute.

## 2. Why do data engineering teams use cloud platforms?

They use cloud platforms for scalable storage and compute, managed services, automation, global infrastructure and flexible provisioning. These capabilities can reduce hardware procurement and infrastructure maintenance. However, cloud use introduces cost, security, network and operational responsibilities, so it is not automatically cheaper or simpler.

## 3. What is the difference between on-premises and cloud infrastructure?

In an on-premises model, the organization owns or directly manages its physical infrastructure and data-center operations. In a cloud model, the provider operates the underlying cloud infrastructure and the customer provisions services. The customer remains responsible for workload configuration, access, data and other responsibilities defined by the service model.

## 4. Explain IaaS, PaaS and SaaS.

- **IaaS:** infrastructure such as virtual machines, storage and networking. The customer manages more of the operating system and software stack.
- **PaaS:** a managed platform for deploying applications. The provider manages more of the runtime infrastructure.
- **SaaS:** a finished software application delivered as a service.

For example, installing Python on a rented virtual machine is closest to IaaS; deploying code to a managed application runtime is closest to PaaS; using hosted email is SaaS.

## 5. What is the difference between public, private and hybrid cloud?

Public cloud services are operated by a provider and offered to multiple customers with logical isolation. A private cloud environment is dedicated to one organization. Hybrid cloud combines private infrastructure with public cloud services. The correct choice depends on requirements such as control, regulation, integration, latency and cost.

## 6. What is a cloud region?

A region is a geographic area in which a cloud provider operates infrastructure. Organizations select regions based on latency, data residency, service availability, resilience and cost. Not every service is available in every region.

## 7. What is an availability zone?

An availability zone is an isolated location within a region. A design that uses multiple AZs can reduce dependence on one location. It does not automatically protect against application defects, accidental deletion or every regional failure.

## 8. What is AWS?

Amazon Web Services is a cloud platform that provides infrastructure, storage, databases, networking, security, analytics and application services. Data engineers commonly encounter services such as S3, IAM, EC2, RDS, Glue, Redshift, Lambda, CloudWatch and VPC.

## 9. What is IAM?

Identity and Access Management controls which identities can access resources and which actions they can perform. Identities may represent people, applications or cloud workloads. IAM policies define permissions, and roles are commonly assumed by workloads to obtain temporary credentials.

## 10. Explain authentication and authorization.

Authentication verifies identity: who is making the request. Authorization determines whether that identity is allowed to perform the requested action on a resource. A user may successfully sign in but still lack permission to read a protected object.

## 11. What is the difference between an IAM user and an IAM role?

An IAM user is an identity in an AWS account. An IAM role is an identity with permissions that an authorized principal can assume. Roles are commonly used by applications and cloud workloads because they can obtain temporary credentials rather than storing long-lived access keys.

## 12. What is an IAM policy?

An IAM policy is a permissions document that specifies allowed or denied actions, resources and sometimes conditions. For example, a policy may allow an application to read objects from one approved storage prefix. The exact action-resource combination must be valid for the service.

## 13. What is least privilege?

Least privilege means granting only the permissions required for a task. A pipeline that reads one input prefix should not receive administrator access or permission to delete every object. Narrow permissions reduce the impact of credential exposure and mistakes.

## 14. What is the shared responsibility model?

The cloud provider is responsible for security of the cloud, including underlying infrastructure. The customer is responsible for security in the cloud, such as identity permissions, data protection and workload configuration. The exact division depends on the service. A virtual machine leaves more operating-system responsibility with the customer than a fully managed application service.

## 15. What is a VPC?

A Virtual Private Cloud is a logically isolated network environment in AWS. It provides a network boundary in which an organization configures IP ranges, subnets, routes and network controls. It helps control how workloads communicate.

## 16. What is a security group?

A security group is a virtual firewall associated with supported resources or network interfaces. It controls allowed inbound and outbound traffic. A database security group should normally allow traffic only from approved application sources rather than from every internet address.

## 17. Explain a basic cloud data engineering architecture.

A source application produces data. An ingestion process moves it to raw object storage. A processing engine validates and transforms it. Curated data is stored for analytical use, and a warehouse or reporting service makes it available to analysts. Monitoring tracks pipeline health, while IAM and network controls restrict access.

## 18. Why keep a raw data layer?

The raw layer preserves the source data as received, subject to retention and security rules. It supports reprocessing, debugging, source-to-target comparison and audit needs. Raw data may contain sensitive information, so access and retention must be controlled.

## 19. How do you avoid unexpected cloud costs?

I identify the services and pricing dimensions before deployment, choose appropriate resource sizes, configure budgets or alerts where available, track resource ownership and remove temporary resources after use. I also review storage, data transfer, logs and backups. A budget alert is not necessarily a hard spending cap.

## 20. Scenario: a developer commits an AWS access key to a public repository. What do you do?

I would immediately deactivate or revoke the exposed key, investigate whether it was used, rotate credentials as required and review affected resources. I would remove the secret from the repository history using the organization's process, move the workload to an approved role or secrets-management solution, reduce permissions and enable secret scanning. Simply deleting the key from the latest file is not sufficient.

## 21. Scenario: a reporting analyst needs sales totals but not raw customer details. How would you design access?

I would provide access to the approved curated reporting dataset and avoid granting access to the raw customer data. The identity would receive only the read permissions needed for the report. I would verify access with positive and negative tests to confirm both permitted and prohibited actions.

## 22. Scenario: an application has the correct database password but cannot connect.

I would check the network path first: VPC placement, routing, subnet configuration, security group rules and DNS/connectivity. Then I would verify the database endpoint, port, TLS settings and credentials. Network reachability and database authorization are separate checks.

## 23. Scenario: a student creates a cloud database and closes the browser. Is the database stopped?

No. Closing the browser does not stop a cloud resource. The database may continue running and incur charges until it is stopped or deleted according to the service's behavior. The student must verify resource cleanup and billing.

## 24. How would you explain your Day 50 project?

I designed a cloud architecture for TechStore's daily sales reporting. I identified source systems, raw and curated storage, processing, analytics, monitoring and security responsibilities. I created a permission matrix for the engineer, pipeline role and analyst, applied least privilege, documented risks and listed cost controls. It was an architecture design exercise rather than a claim that production infrastructure had been deployed.

## Common interview traps

- Saying cloud means there are no servers.
- Saying cloud is always cheaper.
- Confusing authentication with authorization.
- Treating public cloud as publicly accessible data.
- Giving a workload administrator permissions.
- Storing access keys in source code.
- Assuming multi-AZ solves every disaster scenario.
- Assuming a budget alert prevents all charges.
- Claiming a design was deployed when only a diagram was created.

## Readiness checklist

- [ ] Explain IaaS, PaaS and SaaS with examples.
- [ ] Compare public, private and hybrid cloud.
- [ ] Explain regions and availability zones.
- [ ] Describe IAM users, roles and policies.
- [ ] Explain least privilege.
- [ ] Describe shared responsibility.
- [ ] Explain VPCs and security groups.
- [ ] Draw a raw-to-curated cloud data flow.
- [ ] Explain cloud cost and cleanup responsibilities.
