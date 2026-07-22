---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Building a Multi-Tenant Smart Attendance Platform with AWS Serverless Architecture

#### Overview

This workshop demonstrates how to build a **Smart Attendance SaaS Platform** for multiple organizations using a fully managed **AWS Serverless Architecture**.

Participants will learn how to design, deploy, and manage a cloud-native attendance management system that supports multi-tenancy while following AWS best practices for scalability, security, and cost optimization.

Throughout the workshop, you will practice:

+ Deploying serverless infrastructure using **AWS SAM (Serverless Application Model)**.
+ Configuring user authentication and authorization with **Amazon Cognito User Pool**.
+ Designing a **Single-Table** data model in **Amazon DynamoDB** with tenant isolation using **tenantId**.
+ Building asynchronous business workflows using **AWS Step Functions Express Workflows**, **Amazon SQS**, and **Amazon SES**.
+ Deploying a React Single Page Application (SPA) on **Amazon S3** and distributing content globally through **Amazon CloudFront**, secured using **Custom Origin Verification Headers**.

#### Content

1. [Workshop Overview](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequiste/)
3. [Deploy Serverless Backend with AWS SAM](5.3-Deploy-SAM/)
4. [DynamoDB Single-Table Design & DynamoDB Streams](5.4-DynamoDB/)
5. [Build Asynchronous Reporting Workflow](5.5-Reporting-Workflow/)
6. [Deploy React SPA Frontend & CloudFront CDN](5.6-Frontend/)
7. [End-to-End Testing & Resource Cleanup](5.7-Cleanup/)