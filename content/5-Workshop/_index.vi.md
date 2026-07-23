---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Xây dựng Nền tảng Chấm công Đa Tenant với Kiến trúc AWS Serverless

#### Tổng quan

Workshop này hướng dẫn xây dựng một **nền tảng Smart Attendance SaaS** dành cho nhiều doanh nghiệp trên nền tảng **AWS Serverless Architecture**.

Trong suốt workshop, người học sẽ từng bước thiết kế, triển khai và vận hành một hệ thống chấm công hiện đại theo mô hình SaaS (Software as a Service), đáp ứng các yêu cầu về khả năng mở rộng, bảo mật, tính sẵn sàng cao và tối ưu chi phí theo các khuyến nghị của AWS.

Thông qua workshop, bạn sẽ thực hành:

+ Triển khai hạ tầng Serverless bằng **AWS SAM (Serverless Application Model)** theo mô hình Infrastructure as Code (IaC).
+ Cấu hình xác thực và phân quyền người dùng bằng **Amazon Cognito User Pool**.
+ Thiết kế cơ sở dữ liệu **Amazon DynamoDB** theo mô hình **Single-Table Design** với cơ chế cô lập dữ liệu giữa các tenant thông qua **tenantId**.
+ Xây dựng các quy trình xử lý bất đồng bộ bằng **AWS Step Functions Express Workflow**, **Amazon SQS** và **Amazon SES**.
+ Triển khai ứng dụng React Single Page Application (SPA) trên **Amazon S3** và phân phối toàn cầu thông qua **Amazon CloudFront**, kết hợp **Custom Origin Verification Header** để tăng cường bảo mật.

#### Nội dung Workshop

1. [Tổng quan Workshop](5.1-Workshop-overview/)
2. [Điều kiện tiên quyết](5.2-Prerequiste/)
3. [Triển khai Backend Serverless với AWS SAM](5.3-Deploy-SAM/)
4. [Thiết kế Single-Table trên DynamoDB & DynamoDB Streams](5.4-DynamoDB/)
5. [Xây dựng quy trình tạo báo cáo bất đồng bộ](5.5-Reporting-Workflow/)
6. [Triển khai Frontend React SPA & CloudFront CDN](5.6-Frontend/)
