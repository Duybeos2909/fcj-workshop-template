---
title : "Introduction"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Workshop Overview

This workshop provides a hands-on experience in building a **multi-tenant Smart Attendance SaaS Platform** using a fully **AWS Serverless Architecture**. Throughout the lab, participants will deploy and integrate core AWS services to create a scalable, secure, and cost-effective attendance management system for multiple organizations.

The solution follows modern cloud-native design principles, allowing automatic scaling, high availability, simplified infrastructure management, and optimized operational costs through AWS managed services.

#### System Architecture

During this workshop, you will build and deploy a complete **Smart Attendance SaaS** solution consisting of the following AWS services:

+ **AWS SAM (Serverless Application Model)** for Infrastructure as Code (IaC) deployment.
+ **Amazon Cognito User Pool** for user authentication and multi-tenant authorization.
+ **Amazon API Gateway** to expose secure REST APIs.
+ **AWS Lambda** to implement serverless business logic.
+ **Amazon DynamoDB** using **Single-Table Design** with **tenantId** for tenant isolation.
+ **Amazon EventBridge**, **AWS Step Functions**, **Amazon SQS**, and **Amazon SES** to automate asynchronous report generation and email delivery.
+ **Amazon S3** and **Amazon CloudFront** to host and globally distribute the React Single Page Application (SPA).

#### Architecture Diagram

The following diagram illustrates the overall architecture that will be implemented throughout this workshop.

![Smart Attendance SaaS Architecture](/static/images/SaasArchitecture.png)

#### Learning Objectives

After completing this workshop, you will be able to:

+ Deploy serverless infrastructure using **AWS SAM** following Infrastructure as Code (IaC) best practices.
+ Configure authentication and authorization with **Amazon Cognito** and **JWT Authorizer** integrated with Amazon API Gateway.
+ Design a **Single-Table DynamoDB** data model for a multi-tenant SaaS application.
+ Build asynchronous workflows using **Amazon EventBridge**, **AWS Step Functions**, **Amazon SQS**, and **Amazon SES**.
+ Deploy a React Single Page Application on **Amazon S3** and distribute content globally through **Amazon CloudFront**.
+ Understand the end-to-end development and deployment process of a production-style AWS Serverless SaaS application.

#### Estimated Duration

+ **Estimated Time:** 60–90 minutes.
+ **Difficulty Level:** Intermediate to Advanced.