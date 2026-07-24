---
title: "Đề xuất giải pháp"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

Trong phần này, em trình bày đề xuất kiến trúc hệ thống, kế hoạch triển khai và giải pháp kỹ thuật cho dự án **Smart Attendance SaaS Platform**, được nghiên cứu trong quá trình thực tập với vai trò **Cloud Serverless Architect**.

# Smart Attendance SaaS Platform
## Giải pháp chấm công thông minh Hybrid (Local & Cloud-Ready)

---

## 1. Tổng quan dự án

Smart Attendance SaaS Platform là nền tảng quản lý chấm công được xây dựng nhằm hỗ trợ doanh nghiệp và tổ chức quản lý nhân viên, điểm danh, báo cáo và xác thực người dùng trên một hệ thống tập trung.

Hệ thống được thiết kế theo mô hình Hybrid, cho phép vừa phát triển và kiểm thử trên môi trường Local, vừa sẵn sàng triển khai lên AWS Cloud trong tương lai.

Các chức năng chính bao gồm:

- Đăng nhập
- Đăng ký
- Check-in
- Check-out
- Dashboard
- Báo cáo chấm công
- Xuất báo cáo

Frontend được xây dựng bằng HTML/CSS, backend sử dụng Node.js và Express với cấu trúc module rõ ràng (`server.js`, `src/handler`, `src/shared`) giúp dễ dàng bảo trì, mở rộng và triển khai lên môi trường Cloud.

Đối với phiên bản Production, hệ thống được đề xuất triển khai trên AWS Serverless nhằm đảm bảo khả năng mở rộng, tính sẵn sàng cao, tối ưu chi phí và tăng cường bảo mật.

---

## 2. Đặt vấn đề

### Thực trạng

Hiện nay nhiều doanh nghiệp nhỏ và vừa vẫn sử dụng phương pháp chấm công thủ công hoặc nhiều hệ thống rời rạc, gây ra nhiều hạn chế trong quá trình quản lý.

Các vấn đề thường gặp gồm:

- Không theo dõi được trạng thái chấm công theo thời gian thực.
- Dữ liệu điểm danh dễ sai lệch hoặc trùng lặp.
- Quản lý tài khoản và phân quyền còn thủ công.
- Khó tổng hợp báo cáo.
- Khó mở rộng khi số lượng người dùng tăng.
- Mất nhiều thời gian bảo trì hạ tầng máy chủ.

### Giải pháp

Smart Attendance SaaS Platform được xây dựng nhằm số hóa toàn bộ quy trình chấm công.

Người dùng thao tác thông qua các giao diện:

- Login
- Register
- Check-in
- Check-out
- Dashboard

Backend được phát triển bằng Node.js và Express với cấu trúc module độc lập giúp dễ dàng phát triển và tích hợp lên AWS.

Kiến trúc Cloud sử dụng các dịch vụ AWS Managed Services nhằm đảm bảo:

- Khả năng mở rộng tự động.
- Bảo mật.
- Hiệu năng.
- Tính sẵn sàng cao.
- Tối ưu chi phí.

Các dịch vụ AWS được đề xuất bao gồm:

- Amazon Route 53
- Amazon CloudFront
- AWS WAF
- Amazon S3
- Amazon Cognito
- AWS Secrets Manager
- Amazon API Gateway
- AWS Lambda
- Amazon DynamoDB
- DynamoDB Streams
- Amazon EventBridge
- Amazon SQS
- Amazon SNS
- Amazon SES
- AWS Step Functions
- AWS KMS
- Amazon CloudWatch
- AWS X-Ray
- AWS CloudFormation
- AWS CodePipeline
- AWS CodeBuild
- AWS Security Hub
- Amazon GuardDuty

### Lợi ích mang lại

Giải pháp giúp:

- Tự động hóa quy trình chấm công.
- Giảm chi phí vận hành.
- Hỗ trợ đa doanh nghiệp (Multi-Tenant).
- Theo dõi dữ liệu theo thời gian thực.
- Dễ dàng mở rộng hệ thống.
- Triển khai nhanh nhờ mô hình Serverless.
- Đảm bảo bảo mật và phân quyền người dùng.

---

## 3. Kiến trúc giải pháp

Hệ thống được thiết kế theo mô hình Hybrid.

Trong giai đoạn phát triển, frontend giao tiếp với backend Node.js thông qua Express.

Khi triển khai lên AWS, hệ thống sử dụng kiến trúc Serverless.

Frontend được lưu trên Amazon S3 và phân phối thông qua Amazon CloudFront.

Amazon Route 53 chịu trách nhiệm phân giải tên miền.

AWS WAF và AWS Shield Standard bảo vệ hệ thống trước các cuộc tấn công từ Internet.

Người dùng xác thực thông qua Amazon Cognito.

Sau khi đăng nhập thành công, JWT Token sẽ được gửi tới API Gateway để truy cập các Lambda Function.

Toàn bộ nghiệp vụ được xử lý bằng AWS Lambda.

Các quy trình bất đồng bộ sử dụng:

- Amazon EventBridge
- Amazon SQS
- AWS Step Functions

Dữ liệu được lưu trên Amazon DynamoDB theo mô hình Single Table Design.

Amazon SES chịu trách nhiệm gửi Email.

Amazon CloudWatch và AWS X-Ray phục vụ giám sát và theo dõi hệ thống.

Kiến trúc tổng thể được minh họa bên dưới.

![Smart Attendance SaaS Architecture](/images/2-Proposal/SaaS_Serverless.jpg)

### Các dịch vụ AWS sử dụng

- Amazon Route 53
- Amazon CloudFront
- AWS Shield Standard
- AWS WAF
- Amazon S3
- Amazon Cognito
- AWS Secrets Manager
- Amazon API Gateway
- AWS Lambda
- Amazon DynamoDB
- DynamoDB Streams
- Amazon EventBridge
- Amazon SQS
- Amazon SNS
- Amazon SES
- AWS Step Functions
- AWS KMS
- Amazon CloudWatch
- AWS X-Ray
- AWS CloudFormation
- AWS CodePipeline
- AWS CodeBuild
- AWS Security Hub
- Amazon GuardDuty

---

## 4. Kế hoạch triển khai

### Giai đoạn 1

- Phân tích yêu cầu hệ thống.
- Thiết kế kiến trúc AWS.
- Xây dựng Frontend.
- Xây dựng Backend.
- Thiết kế DynamoDB.
- Triển khai API Gateway.
- Triển khai Lambda.
- Cấu hình Cognito.
- Thiết lập EventBridge, SQS và SES.
- Cấu hình CloudWatch.

### Giai đoạn 2

- Tối ưu hiệu năng.
- Thiết lập CI/CD.
- Tích hợp CodePipeline.
- Hoàn thiện CloudFormation.
- Tăng cường bảo mật với GuardDuty và Security Hub.
- Mở rộng Multi-Tenant.
- Chuẩn bị triển khai Production.

### Yêu cầu kỹ thuật

- HTML
- CSS
- JavaScript
- Node.js
- Express
- REST API
- AWS Serverless
- Amazon Cognito
- API Gateway
- AWS Lambda
- DynamoDB
- EventBridge
- Amazon SQS
- Amazon SNS
- Amazon SES
- AWS Step Functions
- AWS KMS
- CloudWatch
- AWS CloudFormation

---

## 5. Tiến độ thực hiện

**Tuần 7**

Khảo sát yêu cầu và phân tích nghiệp vụ.

**Tuần 8**

Thiết kế kiến trúc AWS Serverless.

**Tuần 9**

Triển khai hạ tầng và cơ sở dữ liệu.

**Tuần 10**

Xây dựng Authentication và API Gateway.

**Tuần 11**

Hoàn thiện Lambda, Event-Driven Architecture và Notification.

**Tuần 12**

Kiểm thử, tối ưu và hoàn thiện tài liệu.

---

## 6. Ước tính chi phí hạ tầng

Chi phí hạ tầng được ước tính bằng **AWS Pricing Calculator** cho mô hình triển khai thử nghiệm của **Smart Attendance SaaS Platform** tại khu vực **Asia Pacific (Singapore)**.

### Chi phí hạ tầng

**Các dịch vụ AWS:**

Amazon Route 53: **15.50 USD/tháng** (01 Hosted Zone dùng để quản lý tên miền và định tuyến DNS).

Amazon CloudFront: **4.32 USD/tháng** (Mạng phân phối nội dung (CDN) toàn cầu cho React SPA với khoảng 30 GB dữ liệu truyền ra Internet và 500.000 HTTPS requests mỗi tháng).

AWS WAF v2: **11.00 USD/tháng** (01 Web ACL với 03 luật bảo vệ nhằm chống tấn công web, giới hạn tốc độ truy cập và lọc các yêu cầu độc hại).

Amazon S3: **0.70 USD/tháng** (Lưu trữ giao diện React SPA và các tệp báo cáo với khoảng 15 GB dữ liệu chuẩn).

Amazon Cognito: **0.00 USD/tháng** (500 người dùng hoạt động hàng tháng (MAU), vẫn nằm trong Free Tier).

AWS Secrets Manager: **0.45 USD/tháng** (01 Secret dùng để lưu trữ thông tin nhạy cảm và khóa cấu hình của hệ thống).

Amazon API Gateway (HTTP API): **0.63 USD/tháng** (Khoảng 500.000 yêu cầu HTTP API mỗi tháng sử dụng JWT Authorizer).

AWS Lambda: **1.35 USD/tháng** (Xử lý các chức năng đăng nhập, chấm công, quản trị, tạo báo cáo và webhook theo mô hình Serverless).

AWS Step Functions: **0.02 USD/tháng** (Điều phối quy trình tạo báo cáo bất đồng bộ bằng Express Workflow).

Amazon SQS: **0.20 USD/tháng** (Hàng đợi xử lý bất đồng bộ và Dead Letter Queue).

Amazon DynamoDB (On-Demand): **0.85 USD/tháng** (Cơ sở dữ liệu Single-Table Design với khoảng 2 GB dữ liệu và bật DynamoDB Streams).

AWS Key Management Service (KMS): **1.30 USD/tháng** (01 Customer Managed Key với khoảng 100.000 yêu cầu mã hóa/giải mã mỗi tháng).

Amazon EventBridge: **0.02 USD/tháng** (Định tuyến sự kiện giữa các dịch vụ Serverless).

Amazon Simple Email Service (SES): **0.56 USD/tháng** (Khoảng 5.000 email giao dịch gửi báo cáo và thông báo hệ thống).

Amazon CloudWatch: **1.21 USD/tháng** (Thu thập log, giám sát hệ thống, Dashboard và CloudWatch Alarms).

AWS X-Ray: **0.13 USD/tháng** (Theo dõi Distributed Tracing cho Lambda và API Gateway với tỷ lệ lấy mẫu 5%).

AWS CodeBuild: **9.00 USD/tháng** (Môi trường Build phục vụ quá trình CI/CD và triển khai ứng dụng).

AWS CodePipeline: **0.00 USD/tháng** (01 Pipeline triển khai tự động, nằm trong mức sử dụng ước tính).

AWS Security Hub: **10.00 USD/tháng** (Giám sát tình trạng bảo mật và tuân thủ của môi trường AWS).

---

**Tổng chi phí hạ tầng ước tính:** **Khoảng 57.24 USD/tháng**

**Chi phí ước tính mỗi năm:** **Khoảng 686.88 USD/năm**

Chi phí trên phù hợp với môi trường triển khai thử nghiệm (Prototype) và nghiên cứu, bao gồm đầy đủ các thành phần của hệ thống Serverless, bảo mật, giám sát, CI/CD và phân phối nội dung toàn cầu.

Chi phí thực tế có thể thay đổi tùy thuộc vào:

- Số lượng tổ chức (tenant) sử dụng hệ thống.
- Số lượng người dùng đăng ký và hoạt động.
- Khối lượng API Requests.
- Tần suất và thời gian thực thi của AWS Lambda.
- Lưu lượng đọc/ghi của Amazon DynamoDB.
- Dung lượng truyền tải qua Amazon CloudFront.
- Số lượng báo cáo được tạo mỗi tháng.
- Số lượng email gửi qua Amazon SES.
- Dung lượng log và mức độ giám sát trên Amazon CloudWatch.
- Tần suất Build và Deploy của CI/CD Pipeline.
- Khối lượng kiểm tra và giám sát bảo mật của AWS Security Hub.

## 7. Đánh giá rủi ro

### Các rủi ro

- API quá tải.
- Lambda Timeout.
- Chi phí DynamoDB tăng.
- Lỗi Queue.
- Hot Partition.
- Lỗi triển khai CI/CD.
- Nguy cơ mất an toàn thông tin.

### Biện pháp giảm thiểu

- Throttling API Gateway.
- Sử dụng SQS + DLQ.
- Thiết kế Partition Key hợp lý.
- Least Privilege IAM.
- Mã hóa bằng AWS KMS.
- Giám sát CloudWatch.
- Security Hub và GuardDuty.

### Kế hoạch dự phòng

- CloudFormation Rollback.
- Backup DynamoDB.
- Retry Queue.
- CloudWatch Alarm.
- Theo dõi X-Ray.
- Cải tiến kiến trúc theo phản hồi của mentor.

---

## 8. Kết quả mong đợi

### Kết quả kỹ thuật

- Hệ thống chấm công Serverless hoàn chỉnh.
- Kiến trúc Multi-Tenant.
- Authentication bằng Cognito.
- Xử lý bất đồng bộ với EventBridge và SQS.
- Báo cáo tự động.
- Gửi Email bằng SES.
- Giám sát bằng CloudWatch.
- Triển khai bằng Infrastructure as Code.

### Giá trị lâu dài

Giải pháp tạo nền tảng để mở rộng thành hệ thống SaaS hoàn chỉnh, hỗ trợ nhiều doanh nghiệp, tích hợp GPS, Mobile App, Subscription, Payment Gateway và các dịch vụ AWS khác trong tương lai.