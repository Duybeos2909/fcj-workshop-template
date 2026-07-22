---
title : "Build Asynchronous Reporting Workflow"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

#### Build Asynchronous Reporting Workflow

In large-scale enterprise systems, generating monthly attendance reports for thousands of employees can take anywhere from **10 to 60 seconds**. Processing these tasks synchronously through Amazon API Gateway may exceed the **30-second HTTP timeout**, resulting in poor user experience and blocked requests.

In this module, you will build an **asynchronous reporting workflow** using **AWS Step Functions**, **Amazon SQS**, **Amazon S3**, and **Amazon SES**. This architecture allows report generation to run in the background while users receive an immediate response from the application.

---

#### 1. Asynchronous Workflow Architecture

The overall workflow is illustrated below:

```text
Administrator
        │
        ▼
Amazon API Gateway
        │
        ▼
Amazon SQS Report Queue
        │
        ▼
AWS Step Functions
        │
        ▼
Amazon S3
        │
        ▼
Lambda Email Worker
        │
        ▼
Amazon SES
```

When an administrator clicks **Export Monthly Report** from the React application:

+ The frontend sends an HTTP POST request to Amazon API Gateway.
+ API Gateway places the request into **Amazon SQS Report Queue**.
+ The system immediately returns an **HTTP 202 Accepted** response, allowing the user interface to remain responsive.
+ AWS Step Functions retrieves the queued request and coordinates the entire report generation workflow.
+ After the report is successfully created, it is uploaded to Amazon S3 and a download link is emailed to the administrator through Amazon SES.

---

#### 2. Report Generation Workflow

AWS Step Functions orchestrates the complete report generation process.

The workflow consists of the following stages:

**ValidateRequest**

- Validate input parameters such as `tenantId` and `yearMonth`.
- Ensure the request is valid before processing.

**InitializeReportStatus**

- Create or update a report record in Amazon DynamoDB.
- Set the report status to **PROCESSING**.

**GenerateReportFile**

- Invoke AWS Lambda.
- Retrieve attendance records from Amazon DynamoDB.
- Generate Excel or PDF attendance reports.

**WriteFileToS3**

- Upload the generated report to Amazon S3.
- Encrypt report files using AWS KMS.
- Store reports securely with lifecycle management if required.

**FinalizeReportStatus**

- Update the report status to **COMPLETED**.
- Save the generated S3 Object Key into DynamoDB.

---

#### 3. AWS Step Functions Definition

The Express State Machine is defined inside **template.yaml**.

Example configuration:

```yaml
ReportStateMachine:
  Type: AWS::Serverless::StateMachine
  Properties:
    Type: EXPRESS
    Role: !GetAtt ReportWorkflowRole.Arn
```

Using an **Express Workflow** provides several advantages:

+ Fast execution.
+ Lower operational cost.
+ Automatic scaling.
+ Suitable for high-volume reporting workloads.

---

#### 4. Configure Amazon SQS

Amazon SQS acts as a buffering layer between the API and the backend processing services.

Example resource:

```yaml
EmailQueue:
  Type: AWS::SQS::Queue
```

Amazon SQS provides several benefits:

+ Decouples frontend requests from backend processing.
+ Improves application responsiveness.
+ Buffers workload spikes.
+ Supports automatic retry for failed processing.

---

#### 5. Configure Dead Letter Queue (DLQ)

To improve system reliability, failed messages are automatically redirected to a **Dead Letter Queue (DLQ)** after multiple unsuccessful processing attempts.

Example resource:

```yaml
EmailDLQ:
  Type: AWS::SQS::Queue
```

Redrive Policy:

```yaml
RedrivePolicy:
  deadLetterTargetArn: !GetAtt EmailDLQ.Arn
  maxReceiveCount: 3
```

If the Lambda Email Worker fails to process the same message **three consecutive times**, Amazon SQS automatically moves the message to the Dead Letter Queue.

This approach helps to:

+ Prevent message loss.
+ Simplify troubleshooting.
+ Support message replay after issues are resolved.

---

#### 6. Store Reports in Amazon S3

Once the report has been successfully generated:

+ Upload the report to Amazon S3.
+ Encrypt the object using AWS KMS.
+ Apply Lifecycle Rules or Intelligent-Tiering to optimize storage costs.

Example report location:

```text
reports/monthly-attendance-report.xlsx
```

The generated S3 Object Key is then stored in Amazon DynamoDB for future retrieval.

---

#### 7. Send Notification Emails with Amazon SES

After report generation is completed:

+ Amazon EventBridge publishes a **ReportGenerated** event.
+ The Lambda Email Worker retrieves messages from Amazon SQS.
+ Amazon SES sends an email notification to the administrator.

The email includes:

+ Report generation status.
+ Report creation timestamp.
+ Secure download link stored in Amazon S3.

---

#### 8. Complete Processing Flow

```text
Administrator
        │
        ▼
React Web Application
        │
        ▼
Amazon API Gateway
        │
        ▼
Amazon SQS Report Queue
        │
        ▼
AWS Step Functions
        │
        ├── Validate Request
        ├── Update Report Status
        ├── Generate Report
        ├── Upload Report to Amazon S3
        └── Update DynamoDB
        │
        ▼
Amazon EventBridge
        │
        ▼
Amazon SQS Email Queue
        │
        ▼
Lambda Email Worker
        │
        ▼
Amazon SES
        │
        ▼
Administrator
```

---

#### Module Summary

After completing this module, you have successfully:

+ Built an asynchronous reporting workflow using AWS Step Functions.
+ Buffered report generation requests with Amazon SQS.
+ Stored generated reports securely in Amazon S3.
+ Automated email notifications using Amazon SES.
+ Implemented a Dead Letter Queue (DLQ) to improve fault tolerance.
+ Learned how an Event-Driven Architecture enables scalable and reliable processing without blocking user requests.

The reporting workflow is now ready to integrate with the React frontend, providing administrators with a responsive and reliable monthly attendance reporting system.