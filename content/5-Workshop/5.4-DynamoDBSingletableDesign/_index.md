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
→ DynamoDB
→ Tables
→ smart-attendance-database
```

Review the table configuration in the **Overview** section.

##### Capacity Mode

The table uses:

```text
On-Demand
```

On-Demand capacity automatically scales read and write throughput based on application traffic.

This mode is suitable for the workshop because:

+ No capacity planning is required.
+ Traffic may vary between tenants.
+ The system only pays for actual read and write requests.
+ It is appropriate for development, testing, and unpredictable workloads.

##### Primary Key

Confirm that the table uses:

```text
Partition key: PK
Sort key: SK
```

The combination of `PK` and `SK` uniquely identifies each item in the table.

##### Encryption

The table is encrypted using:

```text
AWS KMS Customer Managed Key
```

The KMS key is defined in the AWS SAM template as:

```text
DataKMSKey
```

This configuration protects data at rest and allows administrators to control key permissions through AWS IAM and AWS KMS policies.

##### Point-in-Time Recovery

Verify that **Point-in-Time Recovery (PITR)** is enabled.

PITR allows the table to be restored to any point within the supported recovery window.

This feature protects the database from:

+ Accidental item deletion.
+ Incorrect application updates.
+ Data corruption.
+ Operational mistakes.

##### DynamoDB Streams

Verify that DynamoDB Streams is enabled with:

```text
NEW_AND_OLD_IMAGES
```

This setting records both the previous and updated versions of each modified item.

It allows downstream services to determine:

+ Which item was created.
+ Which attributes changed.
+ What the previous value was.
+ What the new value is.

---

#### 4. Example AWS SAM DynamoDB Resource

The DynamoDB table can be defined in `template.yaml` as follows:

```yaml
SaaSAttendanceTable:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: smart-attendance-database

    BillingMode: PAY_PER_REQUEST

    AttributeDefinitions:
      - AttributeName: PK
        AttributeType: S
      - AttributeName: SK
        AttributeType: S

    KeySchema:
      - AttributeName: PK
        KeyType: HASH
      - AttributeName: SK
        KeyType: RANGE

    StreamSpecification:
      StreamViewType: NEW_AND_OLD_IMAGES

    PointInTimeRecoverySpecification:
      PointInTimeRecoveryEnabled: true

    SSESpecification:
      SSEEnabled: true
      SSEType: KMS
      KMSMasterKeyId: !Ref DataKMSKey
```

This configuration provides:

+ On-Demand billing.
+ Composite key design.
+ DynamoDB Streams.
+ Point-in-Time Recovery.
+ AWS KMS encryption.

---

#### 5. Insert a Tenant Record

You can create a sample tenant record using AWS CLI:

```bash
aws dynamodb put-item \
  --table-name smart-attendance-database \
  --item '{
    "PK": {"S": "TENANT#company-001"},
    "SK": {"S": "METADATA"},
    "tenantName": {"S": "Demo Company"},
    "plan": {"S": "STANDARD"},
    "status": {"S": "ACTIVE"},
    "createdAt": {"S": "2026-07-22T00:00:00Z"}
  }'
```

Verify the item:

```bash
aws dynamodb get-item \
  --table-name smart-attendance-database \
  --key '{
    "PK": {"S": "TENANT#company-001"},
    "SK": {"S": "METADATA"}
  }'
```

---

#### 6. Insert a User Profile

Create a sample user profile:

```bash
aws dynamodb put-item \
  --table-name smart-attendance-database \
  --item '{
    "PK": {"S": "TENANT#company-001"},
    "SK": {"S": "USER#user-1001"},
    "email": {"S": "employee@example.com"},
    "fullName": {"S": "Demo Employee"},
    "role": {"S": "EMPLOYEE"},
    "department": {"S": "Engineering"}
  }'
```

This user is stored in the same tenant partition as the tenant metadata.

---

#### 7. Insert an Attendance Record

Create a sample check-in record:

```bash
aws dynamodb put-item \
  --table-name smart-attendance-database \
  --item '{
    "PK": {"S": "TENANT#company-001"},
    "SK": {"S": "ATTENDANCE#user-1001#2026-07-22T08:00:00Z"},
    "userId": {"S": "user-1001"},
    "checkInTime": {"S": "2026-07-22T08:00:00Z"},
    "status": {"S": "CHECKED_IN"},
    "location": {"S": "Ho Chi Minh City"}
  }'
```

Query all records belonging to the tenant:

```bash
aws dynamodb query \
  --table-name smart-attendance-database \
  --key-condition-expression "PK = :pk" \
  --expression-attribute-values '{
    ":pk": {"S": "TENANT#company-001"}
  }'
```

Query only attendance records:

```bash
aws dynamodb query \
  --table-name smart-attendance-database \
  --key-condition-expression "PK = :pk AND begins_with(SK, :sk)" \
  --expression-attribute-values '{
    ":pk": {"S": "TENANT#company-001"},
    ":sk": {"S": "ATTENDANCE#"}
  }'
```

---

#### 8. DynamoDB Streams Change Data Capture

DynamoDB Streams captures item-level changes whenever an item is:

+ Created.
+ Updated.
+ Deleted.

For example, when the `CheckInFunction` writes a new attendance item, DynamoDB Streams creates a stream record containing the new database image.

A simplified stream event looks like:

```json
{
  "eventName": "INSERT",
  "dynamodb": {
    "Keys": {
      "PK": {
        "S": "TENANT#company-001"
      },
      "SK": {
        "S": "ATTENDANCE#user-1001#2026-07-22T08:00:00Z"
      }
    },
    "NewImage": {
      "status": {
        "S": "CHECKED_IN"
      },
      "userId": {
        "S": "user-1001"
      }
    }
  }
}
```

This event can be forwarded to other AWS services without increasing the response time of the original Check-in API.

---

#### 9. EventBridge Pipe Integration

The architecture uses **Amazon EventBridge Pipes** to connect DynamoDB Streams with the default EventBridge Event Bus.

The AWS SAM resource can be defined as follows:

```yaml
DdbToEventBridgePipe:
  Type: AWS::Pipes::Pipe
  Properties:
    Name: smart-attendance-ddb-stream-pipe

    RoleArn: !GetAtt EventBridgePipeRole.Arn

    Source: !GetAtt SaaSAttendanceTable.StreamArn

    SourceParameters:
      DynamoDBStreamParameters:
        StartingPosition: LATEST
        BatchSize: 10

    Target: !Sub arn:aws:events:${AWS::Region}:${AWS::AccountId}:event-bus/default
```

The event flow is:

```text
CheckInFunction
      ↓
Amazon DynamoDB
      ↓
DynamoDB Streams
      ↓
Amazon EventBridge Pipe
      ↓
EventBridge Event Bus
      ↓
Notification or Report Workers
```

---

#### 10. Asynchronous Event Processing

When a new attendance item is committed:

1. The Check-in Lambda writes the item to DynamoDB.
2. DynamoDB returns a successful response to the Lambda function.
3. The API responds to the user without waiting for background tasks.
4. DynamoDB Streams captures the new record.
5. EventBridge Pipe forwards the change event.
6. EventBridge routes the event to subscribed targets.
7. Background Lambda functions process notifications, reports, or webhooks.

This design provides several benefits:

+ Lower Check-in API latency.
+ Loose coupling between services.
+ Independent scaling of background workers.
+ Improved reliability.
+ Easier integration with external systems.
+ Better support for retries and failure handling.

---

#### 11. Verify DynamoDB Streams

To verify the stream using AWS CLI:

```bash
aws dynamodb describe-table \
  --table-name smart-attendance-database \
  --query "Table.LatestStreamArn"
```

Expected output:

```text
arn:aws:dynamodb:ap-southeast-1:123456789012:table/smart-attendance-database/stream/2026-07-22T00:00:00.000
```

List the available stream:

```bash
aws dynamodbstreams list-streams \
  --table-name smart-attendance-database
```

---

#### 12. Verify the EventBridge Pipe

List EventBridge Pipes:

```bash
aws pipes list-pipes
```

Retrieve the pipe details:

```bash
aws pipes describe-pipe \
  --name smart-attendance-ddb-stream-pipe
```

Verify that the pipe status is:

```text
RUNNING
```

---

#### Module Completion

After completing this module, you have:

+ Reviewed the DynamoDB Single-Table Design.
+ Created tenant, user, and attendance records.
+ Applied tenant isolation through the partition key.
+ Verified On-Demand capacity mode.
+ Confirmed AWS KMS encryption.
+ Enabled Point-in-Time Recovery.
+ Enabled DynamoDB Streams with `NEW_AND_OLD_IMAGES`.
+ Connected DynamoDB Streams to Amazon EventBridge through EventBridge Pipes.
+ Prepared the event-driven data flow for asynchronous processing.

You are now ready to continue with the next module and build the asynchronous attendance report workflow.