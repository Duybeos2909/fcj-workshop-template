---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

In this section, I present the proposed architecture and implementation plan for the **Smart Attendance SaaS Platform**, which was designed during my internship as a **Cloud Serverless Architect**.

# Smart Attendance SaaS Platform
## Local and Cloud-Ready Hybrid Solution for Smart Attendance and User Management

---

## 1. Executive Summary

The Smart Attendance SaaS Platform is a comprehensive digital solution designed for attendance tracking, user registration, authentication, user management, and report generation.

The project was researched and developed to streamline attendance workflows, support role-based user management, and provide a user-friendly interface for modern organizations, academic institutions, and enterprise teams.

The system separates major pages and functions such as Login, Register, Check-in, Check-out, Dashboard, Attendance Reports, and Report Export. These components work together to provide a complete attendance management workflow.

The current project uses a modern web-based architecture that combines an HTML/CSS frontend with a Node.js and Express backend through `server.js`. The source code is organized into modular directories such as `src/handler/` and `src/shared/` to improve maintainability, local testing, debugging, and future cloud deployment.

The proposed cloud architecture uses AWS managed services to provide scalability, high availability, security, event-driven processing, monitoring, and cost optimization without requiring permanent server administration.

---

## 2. Problem Statement

### What's the Problem?

Many small and medium-sized organizations still rely on manual attendance tracking or fragmented tools to manage employee and member attendance.

Common challenges include:

- Difficulty tracking employee attendance status in real time.
- Inaccurate or duplicated attendance records.
- Complex user authentication and credential management.
- Lack of centralized role-based access control.
- Manual and time-consuming attendance report preparation.
- Limited support for multiple organizations or tenants.
- Poor scalability when the number of users increases.
- Complicated local testing, debugging, and deployment processes.
- High infrastructure maintenance requirements in traditional server-based systems.

### The Solution

The Smart Attendance SaaS Platform provides a unified application that automates user authentication, attendance tracking, reporting, and administrative management.

Users can interact through dedicated interfaces such as:

- `index.html`
- `login.html`
- `register.html`
- `checkin.html`
- `checkout.html`
- Dashboard and attendance report pages

The backend is powered by Node.js and Express, with CORS support and modular request handlers such as:

- `src/handler/attendance/checkin.js`
- `src/handler/attendance/checkout.js`
- `src/handler/reports/export.js`
- `src/shared/`
- `server.js`

This structure provides reliable request handling and allows developers to test the application locally before integrating it with AWS services.

The proposed production architecture follows the AWS Well-Architected Framework by emphasizing Security, Reliability, Performance Efficiency, Operational Excellence, and Cost Optimization.

The solution integrates:

- Amazon Route 53
- Amazon CloudFront
- AWS Shield Standard
- AWS WAF v2
- Amazon S3
- Amazon Cognito
- AWS Secrets Manager
- Amazon API Gateway
- AWS Lambda
- AWS Step Functions
- Amazon DynamoDB
- DynamoDB Streams
- AWS KMS
- Amazon EventBridge
- Amazon SQS
- Amazon SNS
- Amazon SES
- Amazon CloudWatch
- AWS X-Ray
- AWS CloudFormation
- AWS CodePipeline
- AWS CodeBuild
- AWS Security Hub
- Amazon GuardDuty

### Benefits and Return on Investment

The proposed solution provides several advantages:

- Digitizes the complete attendance tracking process.
- Reduces administrative workload and manual data entry.
- Provides real-time check-in and check-out tracking.
- Supports secure user authentication and role-based authorization.
- Enables multi-tenant user and organization management.
- Supports automated attendance report generation and export.
- Allows flexible local development and cloud deployment.
- Automatically scales according to application workload.
- Reduces infrastructure maintenance through serverless computing.
- Improves reliability through event-driven and asynchronous processing.
- Provides centralized logging, monitoring, and distributed tracing.
- Supports automated deployment using Infrastructure as Code and CI/CD pipelines.

The modular architecture also makes the platform easier to maintain, test, extend, and integrate with future features such as subscription management, payment gateways, mobile applications, and external webhooks.

---

## 3. Solution Architecture

The Smart Attendance SaaS Platform uses a local and cloud-ready hybrid architecture.

During local development, the frontend communicates with a Node.js and Express backend. The source code is divided into frontend views, business handlers, shared configurations, and server routes.

For production deployment, the MVP uses a fully serverless and highly scalable AWS architecture designed for multi-tenant SaaS applications.

Frontend content is hosted on Amazon S3 and distributed globally through Amazon CloudFront. Amazon Route 53 provides DNS resolution, while AWS Shield Standard and AWS WAF protect the application from DDoS attacks, malicious traffic, bots, and excessive requests.

Users authenticate through Amazon Cognito before accessing backend APIs exposed by Amazon API Gateway. JWT tokens include role and tenant information for secure access control.

Business logic is processed by AWS Lambda functions. Long-running and multi-step processes, such as report generation, are coordinated through AWS Step Functions.

Attendance, user, tenant, and subscription data are stored in Amazon DynamoDB using a Single-Table Design. DynamoDB Streams captures data changes and supports event-driven processing.

Amazon EventBridge routes system events, while Amazon SQS provides asynchronous message processing and Dead Letter Queue handling. Amazon SNS and Amazon SES support notification and email delivery.

Amazon CloudWatch, AWS X-Ray, AWS Security Hub, and Amazon GuardDuty provide logging, monitoring, tracing, security posture management, and threat detection.

The proposed architecture is illustrated below.

![Smart Attendance SaaS Architecture](/images/2-Proposal/SaaS_Serverless.jpg)

### AWS Services Used

- Amazon Route 53
- Amazon CloudFront
- AWS Shield Standard
- AWS WAF v2
- Amazon S3
- Amazon Cognito
- AWS Secrets Manager
- Amazon API Gateway
- AWS Lambda
- AWS Step Functions
- Amazon DynamoDB
- DynamoDB Streams
- AWS KMS
- Amazon EventBridge
- Amazon SQS
- Amazon SNS
- Amazon SES
- Amazon CloudWatch
- AWS X-Ray
- AWS IAM
- AWS CloudFormation
- AWS CodePipeline
- AWS CodeBuild
- AWS Security Hub
- Amazon GuardDuty

### Component Design

**Frontend**

- HTML and CSS interfaces for local development.
- React Single Page Application for the proposed cloud deployment.
- Amazon S3 for frontend hosting.
- Amazon CloudFront for global content delivery.
- Amazon Route 53 for DNS resolution.
- AWS WAF and AWS Shield Standard for edge protection.

**Authentication**

- Amazon Cognito User Pool.
- User registration and login.
- JWT authentication.
- User role management.
- Tenant-based access control.
- AWS Secrets Manager for sensitive credentials and API secrets.

**Backend**

- Node.js and Express for local backend processing.
- Amazon API Gateway HTTP API v2.
- AWS Lambda Functions.
- API caching and throttling.
- Modular handlers for attendance, reporting, administration, webhooks, and subscriptions.

**Compute and Workflow**

- Webhook Lambda.
- Check-in Lambda.
- Check-out Lambda.
- Attendance Lambda.
- Report Lambda.
- Admin Lambda.
- Subscription Lambda.
- Email Worker Lambda.
- AWS Step Functions for report and workflow orchestration.
- Amazon SQS for asynchronous processing.
- Dead Letter Queue for failed messages.

**Database**

- Amazon DynamoDB using Single-Table Design.
- Attendance records.
- User profiles.
- Tenant information.
- Subscription information.
- DynamoDB Streams for change data capture.
- AWS KMS Customer Managed Key for encryption.

**Storage**

- Amazon S3 for frontend hosting.
- Amazon S3 for generated report storage.
- PDF report files.
- Excel report files.
- CSV export files.

**Messaging**

- Amazon EventBridge.
- Amazon SQS.
- Amazon SNS.
- Amazon SES.
- Email Queue and Dead Letter Queue buffers.

**Monitoring**

- Amazon CloudWatch Logs.
- CloudWatch Metrics.
- CloudWatch Alarms.
- AWS X-Ray distributed tracing.
- AWS Security Hub.
- Amazon GuardDuty.

**Infrastructure and Deployment**

- AWS CloudFormation.
- AWS CodePipeline.
- AWS CodeBuild.
- Infrastructure as Code deployment.
- Automated build and deployment pipelines.

---

## 4. Technical Implementation

### Implementation Phases

The project consists of two main implementation phases.

**Phase 1 – MVP Deployment**

- Study business and SaaS requirements.
- Analyze multi-tenant isolation requirements.
- Implement Amazon Route 53 for DNS resolution.
- Configure Amazon CloudFront for global content delivery.
- Configure AWS WAF v2 for rate limiting, bot protection, and geographic filtering.
- Host the React SPA on Amazon S3.
- Configure authentication using Amazon Cognito.
- Configure JWT authorization and tenant-based access control.
- Store external credentials in AWS Secrets Manager.
- Configure Amazon API Gateway HTTP API v2.
- Deploy AWS Lambda functions for check-in, check-out, attendance, reports, administration, subscriptions, and webhooks.
- Design AWS Step Functions workflows for report processing.
- Create Amazon DynamoDB Single-Table Design.
- Enable DynamoDB Streams.
- Configure Amazon S3 report storage.
- Configure AWS KMS encryption.
- Enable event routing through Amazon EventBridge.
- Configure Amazon SQS queues and Dead Letter Queues.
- Configure Amazon SNS and Amazon SES notification workflows.
- Configure CloudWatch Logs, Metrics, and Alarms.
- Enable AWS X-Ray tracing.
- Monitor security posture through AWS Security Hub.

**Phase 2 – Future Design Expansion**

- Optimize asynchronous processing through Amazon SQS.
- Improve SNS and SES notification delivery.
- Expand Dead Letter Queue and retry mechanisms.
- Enhance CI/CD deployment using AWS CodePipeline and AWS CodeBuild.
- Automate infrastructure deployment using AWS CloudFormation.
- Expand automated security policies.
- Integrate Amazon GuardDuty threat detection.
- Improve Security Hub compliance monitoring.
- Add subscription and payment management.
- Support mobile application integration.
- Improve multi-tenant scalability.
- Prepare the system for future enterprise deployment.

### Technical Requirements

The implementation requires knowledge of:

- HTML
- CSS
- JavaScript
- Node.js
- Express
- RESTful API Design
- CORS
- Git and version control
- AWS Serverless Architecture
- Amazon Route 53
- Amazon CloudFront
- AWS Shield Standard
- AWS WAF v2
- Amazon S3
- Amazon Cognito
- AWS Secrets Manager
- Amazon API Gateway
- AWS Lambda
- AWS Step Functions
- Amazon DynamoDB
- DynamoDB Streams
- AWS KMS
- Amazon EventBridge
- Amazon SQS
- Amazon SNS
- Amazon SES
- Amazon CloudWatch
- AWS X-Ray
- AWS IAM
- AWS CloudFormation
- AWS CodePipeline
- AWS CodeBuild
- AWS Security Hub
- Amazon GuardDuty

---

## 5. Timeline & Milestones

### Project Timeline

**Week 7**

Complete the SaaS requirements specification, evaluate the feasibility of multi-tenant isolation, and define functional requirements for the Smart Attendance workflow.

**Week 8**

Design the highly available AWS Serverless architecture, model the check-in and check-out sequence flows, and prepare secure tenant authentication flowcharts.

**Week 9**

Prepare Infrastructure as Code templates such as `template.yaml` for Amazon DynamoDB, AWS KMS, Amazon S3, AWS Lambda, and supporting cloud resources.

**Week 10**

Configure Amazon Cognito User Pools, JWT tokens, API Gateway endpoints, AWS WAF v2 rate-limiting rules, bot protection, and security controls.

**Week 11**

Develop Lambda functions for attendance processing, report generation, subscription handling, webhook processing, SQS and SNS message buffering, Step Functions workflows, and Amazon SES email templates.

**Week 12**

Perform integration and performance testing, verify CloudFront content delivery, validate the CI/CD pipeline using CodePipeline and CodeBuild, and complete deployment and operational documentation.

---

## 6. Budget Estimation

The infrastructure cost was estimated using the **AWS Pricing Calculator** for a small-scale deployment of the **Smart Attendance SaaS Platform** in the **Asia Pacific (Singapore)** Region. The estimation assumes approximately **500 monthly active users (MAU)**, **500,000 API requests per month**, and a demonstration-scale workload suitable for educational and prototype environments.

### Infrastructure Costs

**AWS Services:**

Amazon Route 53: **$15.50/month** (1 Hosted Zone for DNS management and domain routing).

Amazon CloudFront: **$4.32/month** (Global CDN for React SPA distribution, approximately 30 GB outbound traffic and 500,000 HTTPS requests).

AWS WAF v2: **$11.00/month** (1 Web ACL with three security rules for rate limiting and web application protection).

Amazon S3: **$0.70/month** (Frontend hosting and attendance report storage with approximately 15 GB Standard storage).

Amazon Cognito: **$0.00/month** (500 Monthly Active Users within the AWS Free Tier).

AWS Secrets Manager: **$0.45/month** (1 secret used for application credentials and secure configuration management).

Amazon API Gateway (HTTP API): **$0.63/month** (Approximately 500,000 HTTP API requests with JWT authorization).

AWS Lambda: **$1.35/month** (Serverless compute for authentication, attendance processing, reporting, administration, and webhook functions).

AWS Step Functions: **$0.02/month** (Express Workflow orchestration for asynchronous report generation).

Amazon SQS: **$0.20/month** (Standard Queue and Dead Letter Queue for asynchronous message processing).

Amazon DynamoDB On-Demand: **$0.85/month** (Single-Table Design with approximately 2 GB storage and DynamoDB Streams enabled).

AWS Key Management Service (KMS): **$1.30/month** (1 Customer Managed Key with approximately 100,000 cryptographic requests).

Amazon EventBridge: **$0.02/month** (Event routing for asynchronous serverless workflows).

Amazon Simple Email Service (SES): **$0.56/month** (Approximately 5,000 transactional emails for attendance reports and notifications).

Amazon CloudWatch: **$1.21/month** (Application logs, monitoring dashboards, metrics, and CloudWatch alarms).

AWS X-Ray: **$0.13/month** (Distributed tracing for Lambda and API Gateway requests with 5% sampling rate).

AWS CodeBuild: **$9.00/month** (Build environment for automated application packaging and deployment).

AWS CodePipeline: **$0.00/month** (One CI/CD pipeline operating within the estimated usage level).

AWS Security Hub: **$10.00/month** (Security posture management and compliance monitoring for the AWS environment).

---

**Estimated Total Infrastructure Cost:** **Approximately USD 57.24/month**

**Estimated Annual Cost:** **Approximately USD 686.88/year**

The estimated infrastructure cost is appropriate for a prototype deployment that includes serverless application hosting, security monitoring, CI/CD automation, centralized logging, and global content delivery.

The actual monthly cost may vary depending on:

- Number of organizations and tenants.
- Number of registered employees.
- API request volume.
- AWS Lambda execution frequency and duration.
- DynamoDB read and write throughput.
- CloudFront data transfer volume.
- Number of generated attendance reports.
- Amazon SES email volume.
- CloudWatch log ingestion and monitoring usage.
- CI/CD build frequency.
- Security monitoring and compliance workload.

## 7. Risk Assessment

### Risk Matrix

- API request throttling or high API latency.
- Increased costs caused by DynamoDB traffic spikes.
- CI/CD pipeline deployment failures.
- Security vulnerabilities or unauthorized access.
- Failures from third-party APIs.
- Lambda timeout during long-running operations.
- Amazon SQS message processing failures.
- Dead Letter Queue accumulation.
- DynamoDB hot partitions.
- Amazon SES email delivery failures.

### Mitigation Strategies

- Use API Gateway throttling and caching.
- Optimize Lambda cold starts and execution duration.
- Use DynamoDB On-Demand capacity mode.
- Design fine-grained and tenant-aware partition keys.
- Apply IAM least-privilege policies.
- Configure AWS WAF rules.
- Encrypt sensitive information using AWS KMS.
- Store external credentials in AWS Secrets Manager.
- Monitor security events using AWS Security Hub.
- Enable Amazon GuardDuty threat detection.
- Implement timeout and retry handling for external APIs.
- Use Amazon SQS and Dead Letter Queues.
- Move long-running operations to AWS Step Functions.
- Monitor logs and metrics using Amazon CloudWatch.
- Use AWS X-Ray to trace distributed requests.

### Contingency Plans

- Enable CloudWatch Alarms for critical system metrics.
- Retry failed messages through Amazon SQS.
- Redirect failed events to a Dead Letter Queue.
- Review and replay failed messages when appropriate.
- Restore DynamoDB data using backup and point-in-time recovery.
- Use CloudFormation rollback policies for failed deployments.
- Separate CodePipeline stages for validation and production deployment.
- Review AWS X-Ray traces during system incidents.
- Rotate exposed credentials through AWS Secrets Manager.
- Update WAF rules based on detected attack patterns.
- Improve the architecture based on mentor and team feedback.

---

## 8. Expected Outcomes

### Technical Improvements

The project is expected to deliver:

- A complete AWS Serverless attendance tracking workflow.
- A local and cloud-ready hybrid application architecture.
- Secure user authentication using Amazon Cognito.
- JWT-based role and tenant authorization.
- A highly scalable multi-tenant SaaS platform.
- Modular Lambda functions for core business processes.
- Amazon DynamoDB Single-Table Design.
- DynamoDB Streams integration.
- Event-driven backend processing.
- Reliable Amazon SQS and Dead Letter Queue workflows.
- AWS Step Functions report orchestration.
- Secure report storage using Amazon S3.
- Data encryption using AWS KMS.
- Automated notification delivery using Amazon SES.
- Centralized monitoring using Amazon CloudWatch.
- Distributed tracing using AWS X-Ray.
- Security monitoring through AWS Security Hub and Amazon GuardDuty.
- Infrastructure as Code using AWS CloudFormation.
- Automated CI/CD deployment using AWS CodePipeline and AWS CodeBuild.

### Long-term Value

The proposed solution provides a production-ready foundation for future enterprise attendance and workforce management systems.

The architecture can be expanded to support:

- Multiple organizations and tenants.
- Mobile attendance applications.
- GPS and geofencing validation.
- Subscription plans.
- Payment gateway integration.
- External API and webhook integration.
- Automated invoicing.
- Advanced attendance analytics.
- Enterprise security controls.
- Multi-Region disaster recovery.
- Additional workforce management features.

The project also provides a reusable reference architecture for organizations adopting AWS Serverless technologies and Infrastructure as Code for future cloud-native SaaS applications.