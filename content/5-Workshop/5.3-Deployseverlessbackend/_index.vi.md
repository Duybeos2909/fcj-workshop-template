---
title : "Triển khai Backend Serverless"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Triển khai hạ tầng Backend Serverless

Trong phần này, bạn sẽ xây dựng và triển khai toàn bộ hạ tầng Backend của **Smart Attendance SaaS Platform** bằng **AWS Serverless Application Model (AWS SAM)**. Sau khi triển khai hoàn tất, hệ thống sẽ bao gồm Amazon Cognito, Amazon API Gateway, AWS Lambda, Amazon DynamoDB và AWS Key Management Service (KMS), tạo thành một nền tảng Serverless hoàn chỉnh phục vụ cho hệ thống chấm công đa tenant.

---

#### 1. Xem trước hạ tầng trong tệp template.yaml

Tệp **template.yaml** mô tả toàn bộ hạ tầng AWS theo mô hình **Infrastructure as Code (IaC)**, giúp tự động hóa quá trình triển khai và quản lý tài nguyên trên đám mây.

Các tài nguyên chính được định nghĩa trong template bao gồm:

+ **Amazon Cognito User Pool** dùng để quản lý đăng ký, đăng nhập và phát hành JWT Token cho người dùng.
+ **Amazon API Gateway HTTP API** được tích hợp với **JWT Authorizer** nhằm bảo vệ các API và xác thực người dùng thông qua Cognito.
+ **AWS KMS Customer Managed Key (CMK)** dùng để mã hóa dữ liệu lưu trữ trên Amazon DynamoDB và Amazon S3.
+ **Amazon DynamoDB** áp dụng mô hình **Single-Table Design** với **Partition Key (PK)**, **Sort Key (SK)**, chế độ **On-Demand Billing** và bật **DynamoDB Streams** để theo dõi thay đổi dữ liệu.
+ **Các hàm AWS Lambda** thực hiện toàn bộ nghiệp vụ của hệ thống, bao gồm:

- **AuthFunction** – Xử lý đăng ký, đăng nhập và xác thực người dùng.
- **CheckInFunction** – Ghi nhận dữ liệu chấm công khi nhân viên Check-in.
- **CheckOutFunction** – Ghi nhận dữ liệu Check-out của nhân viên.
- **AttendanceHistoryFunction** – Truy vấn và hiển thị lịch sử chấm công.
- **AdminFunction** – Quản lý doanh nghiệp, nhân viên và các chức năng dành cho quản trị viên.
- **ReportFunction** – Tạo báo cáo chấm công theo cơ chế bất đồng bộ.
- **WebhookFunction** – Tiếp nhận và xử lý các webhook từ hệ thống bên ngoài hoặc cổng thanh toán.

---

#### 2. Build ứng dụng Serverless

Di chuyển đến thư mục **backend** và thực hiện lệnh build bằng AWS SAM:

```bash
cd backend

sam build