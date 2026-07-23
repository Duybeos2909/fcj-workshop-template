---
title : "DynamoDB Single-Table Design"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

#### DynamoDB Single-Table Design and DynamoDB Streams CDC

In a **multi-tenant SaaS platform**, database design directly affects infrastructure cost, query performance, scalability, and tenant data isolation.

In this module, you will explore how the **Smart Attendance SaaS Platform** stores multiple entity types in a single Amazon DynamoDB table by using the **Single-Table Design** pattern.

You will also examine how **DynamoDB Streams** and **Amazon EventBridge Pipes** implement Change Data Capture (CDC) for asynchronous event processing.

---

#### 1. Single-Table Schema Design

Instead of creating separate DynamoDB tables for users, attendance records, tenants, and subscriptions, the system stores all entities in one table named:

```text
smart-attendance-database
```

The table uses a composite primary key consisting of:

+ **Partition Key:** `PK`
+ **Sort Key:** `SK`

The `PK` groups related records by tenant, while the `SK` identifies the entity type and individual record.

The proposed data model is shown below:

| Entity Type | Partition Key (PK) | Sort Key (SK) | Main Attributes |
| --- | --- | --- | --- |
| Tenant Record | `TENANT#<tenantId>` | `METADATA` | tenantName, plan, status, createdAt |
| User Profile | `TENANT#<tenantId>` | `USER#<userId>` | email, fullName, role, department |
| Attendance Log | `TENANT#<tenantId>` | `ATTENDANCE#<userId>#<timestamp>` | checkInTime, checkOutTime, status, location |
| Subscription Billing | `TENANT#<tenantId>` | `SUB#<billingId>` | planTier, amount, paymentStatus, expiryDate |

Example records:

```text
PK: TENANT#company-001
SK: METADATA
```

```text
PK: TENANT#company-001
SK: USER#user-1001
```

```text
PK: TENANT#company-001
SK: ATTENDANCE#user-1001#2026-07-22T08:00:00Z
```

```text
PK: TENANT#company-001
SK: SUB#invoice-2026-07
```

This structure allows the application to retrieve all data belonging to one organization through a single partition key.

---

#### 2. Tenant Data Isolation

Tenant isolation is one of the most important requirements in a multi-tenant SaaS application.

Every database request must use the tenant identifier extracted from the authenticated user's JWT token.

The tenant partition key follows this format:

```text
TENANT#<tenantId>
```

For example:

```text
TENANT#company-001
```

A Lambda function must never accept the tenant identifier directly from an untrusted request body without validation.

Instead, the tenant ID should be obtained from the authenticated JWT claims:

```javascript
const claims = event.requestContext.authorizer.jwt.claims;
const tenantId = claims["custom:tenantId"];
const partitionKey = `TENANT#${tenantId}`;
```

A DynamoDB query can then be scoped to the authenticated tenant:

```javascript
const command = new QueryCommand({
  TableName: process.env.TABLE_NAME,
  KeyConditionExpression: "PK = :pk",
  ExpressionAttributeValues: {
    ":pk": `TENANT#${tenantId}`
  }
});
```

This approach ensures that:

+ Users can only access records belonging to their own organization.
+ Cross-tenant queries are prevented at the application layer.
+ Authorization rules remain consistent across all Lambda functions.
+ The risk of tenant data leakage is significantly reduced.

---

#### 3. Review the DynamoDB Table in AWS Console

Open the Amazon DynamoDB Console:

```text
AWS Management Console
→ Amazon DynamoDB
→ Tables
```

The deployed infrastructure creates a DynamoDB table named:

```text
smart-attendance-database
```

Verify that the table appears in the table list and has the following configuration:

+ **Table status:** Active
+ **Partition key:** `PK`
+ **Sort key:** `SK`
+ **Read capacity mode:** On-Demand
+ **Write capacity mode:** On-Demand

![Smart Attendance DynamoDB Table](../../images/A4.png)

Select **smart-attendance-database** to open its configuration page.

---

##### General Table Configuration

In the **Settings** tab, review the general information of the DynamoDB table.

Confirm the following values:

```text
Partition key: PK
Sort key: SK
Capacity mode: On-Demand
Table status: Active
```

The combination of `PK` and `SK` uniquely identifies each item stored in the table.

The **On-Demand** capacity mode automatically adjusts read and write throughput according to application traffic without requiring manual capacity provisioning.

This capacity mode is suitable for the Smart Attendance SaaS Platform because:

+ No initial capacity planning is required.
+ Workload volume may vary across different tenants.
+ The system only pays for actual read and write requests.
+ It supports unpredictable check-in and check-out traffic.
+ It simplifies database administration during development and demonstration.

![DynamoDB General Settings](../../images/A5.png)

---

##### Point-in-Time Recovery

Open the **Backups** tab and verify that **Point-in-Time Recovery (PITR)** is enabled.

Expected status:

```text
Point-in-Time Recovery: On
```

PITR continuously protects the DynamoDB table and allows administrators to restore it to a selected point within the configured recovery period.

This feature helps protect the system from:

+ Accidental deletion of attendance records.
+ Incorrect updates from application code.
+ Data corruption.
+ Operational mistakes.
+ Unexpected deployment errors.

![DynamoDB Point-in-Time Recovery](../../images/A6.png)

The console also displays information such as:

+ Backup recovery period.
+ Earliest available restore point.
+ Latest available restore point.
+ Existing system backups.
+ Backup status and creation time.

---

##### DynamoDB Backup Status

Review the backup list displayed below the PITR configuration.

A backup with the status shown below confirms that DynamoDB data protection is active:

```text
Status: Available
```

The backup can be used to restore the table when data is accidentally modified or deleted.

![DynamoDB Backup Status](../../images/A7.png)

> **Note:** The third and fourth screenshots currently show nearly identical DynamoDB Backup pages. You may use one image for the PITR configuration and the other for the backup list, or replace the fourth screenshot with a DynamoDB Streams screenshot later.

---

##### Encryption

The DynamoDB table is encrypted using an AWS KMS key.

The AWS SAM template defines the encryption key as:

```text
DataKMSKey
```

This configuration protects data at rest and allows administrators to manage permissions through AWS IAM and AWS KMS key policies.

Expected encryption configuration:

```text
Encryption type: AWS KMS
Key type: Customer Managed Key
```

---

##### DynamoDB Streams

Open the **Exports and streams** tab and verify that DynamoDB Streams is enabled with:

```text
NEW_AND_OLD_IMAGES
```

This configuration records both the previous version and the updated version of every modified item.

It allows downstream services to determine:

+ Which item was created.
+ Which item was updated or deleted.
+ Which attributes changed.
+ What the previous values were.
+ What the new values are.

DynamoDB Streams provides the Change Data Capture source used by Amazon EventBridge Pipes and other asynchronous processing services.