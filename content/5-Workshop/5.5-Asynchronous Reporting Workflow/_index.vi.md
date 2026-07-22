---
title : "Build Asynchronous Reporting Workflow"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

#### Build Asynchronous Reporting Workflow

Trong các hệ thống doanh nghiệp có quy mô lớn, việc tổng hợp dữ liệu chấm công hàng tháng của hàng nghìn nhân viên có thể mất từ **10–60 giây** để xử lý. Nếu toàn bộ quá trình này được thực hiện đồng bộ thông qua API Gateway, ứng dụng rất dễ gặp giới hạn **HTTP Timeout (30 giây)**, làm người dùng phải chờ đợi hoặc giao diện bị treo.

Trong phần này, bạn sẽ xây dựng quy trình tạo báo cáo bất đồng bộ bằng cách kết hợp **AWS Step Functions Express Workflow**, **Amazon SQS**, **Amazon S3** và **Amazon SES** nhằm đảm bảo hệ thống luôn phản hồi nhanh, đồng thời vẫn xử lý được các tác vụ nặng phía sau.

---

#### 1. Kiến trúc xử lý bất đồng bộ

Luồng xử lý của hệ thống được thiết kế như sau:

```text
[Admin Request]
        │
        ▼
[Amazon API Gateway]
        │
        ▼
[Amazon SQS Report Queue]
        │
        ▼
[AWS Step Functions]
        │
        ▼
[Amazon S3]
        │
        ▼
[Lambda Email Worker]
        │
        ▼
[Amazon SES]
```

Khi quản trị viên nhấn nút **Export Monthly Report** trên giao diện React:

+ Frontend gửi yêu cầu HTTP POST đến Amazon API Gateway.
+ API Gateway đưa yêu cầu vào **Amazon SQS Report Queue**.
+ Hệ thống phản hồi ngay mã **HTTP 202 Accepted**, giúp giao diện người dùng không phải chờ quá trình tạo báo cáo hoàn thành.
+ AWS Step Functions sẽ đọc từng yêu cầu từ hàng đợi và điều phối toàn bộ quy trình xử lý.
+ Sau khi báo cáo được tạo thành công, file sẽ được lưu trên Amazon S3 và email chứa liên kết tải xuống sẽ được gửi qua Amazon SES.

---

#### 2. Quy trình xử lý với AWS Step Functions

State Machine chịu trách nhiệm điều phối toàn bộ quy trình tạo báo cáo.

Workflow bao gồm các bước:

**ValidateRequest**

- Kiểm tra dữ liệu đầu vào như `tenantId` và `yearMonth`.
- Đảm bảo yêu cầu hợp lệ trước khi tiếp tục xử lý.

**InitializeReportStatus**

- Ghi trạng thái **PROCESSING** vào Amazon DynamoDB.
- Cho phép quản trị viên theo dõi tiến trình tạo báo cáo.

**GenerateReportFile**

- Lambda truy vấn dữ liệu chấm công.
- Tổng hợp dữ liệu.
- Sinh báo cáo Excel hoặc PDF.

**WriteFileToS3**

- Lưu file báo cáo lên Amazon S3.
- Dữ liệu được mã hóa bằng AWS KMS.
- Có thể cấu hình Lifecycle để tối ưu chi phí lưu trữ.

**FinalizeReportStatus**

- Cập nhật trạng thái thành **COMPLETED**.
- Lưu S3 Object Key vào DynamoDB để phục vụ việc tải xuống sau này.

---

#### 3. Khai báo State Machine trong AWS SAM

Ví dụ khai báo trong `template.yaml`:

```yaml
ReportStateMachine:
  Type: AWS::Serverless::StateMachine
  Properties:
    Type: EXPRESS
    Role: !GetAtt ReportWorkflowRole.Arn
```

Workflow sử dụng **Express State Machine** nhằm:

+ Chi phí thấp.
+ Khả năng mở rộng cao.
+ Thời gian thực thi nhanh.
+ Phù hợp với các tác vụ tạo báo cáo.

---

#### 4. Cấu hình Amazon SQS

Để tăng độ tin cậy của hệ thống, các yêu cầu tạo báo cáo và gửi email sẽ được lưu trong hàng đợi Amazon SQS.

Ví dụ khai báo:

```yaml
EmailQueue:
  Type: AWS::SQS::Queue
```

EmailQueue đóng vai trò:

+ Đệm các yêu cầu gửi email.
+ Giảm tải cho Lambda.
+ Cho phép xử lý bất đồng bộ.
+ Hỗ trợ Retry khi xảy ra lỗi.

---

#### 5. Dead Letter Queue (DLQ)

Trong trường hợp Lambda xử lý thất bại nhiều lần liên tiếp, tin nhắn sẽ được chuyển sang **Dead Letter Queue (DLQ)**.

Ví dụ:

```yaml
EmailDLQ:
  Type: AWS::SQS::Queue
```

Cấu hình Redrive Policy:

```yaml
RedrivePolicy:
  deadLetterTargetArn: !GetAtt EmailDLQ.Arn
  maxReceiveCount: 3
```

Nếu Lambda xử lý thất bại **3 lần**, Amazon SQS sẽ tự động chuyển tin nhắn sang DLQ thay vì tiếp tục retry vô hạn.

Điều này giúp:

+ Không làm mất dữ liệu.
+ Dễ dàng kiểm tra nguyên nhân lỗi.
+ Hỗ trợ xử lý lại các yêu cầu thất bại sau này.

---

#### 6. Lưu báo cáo trên Amazon S3

Sau khi Lambda tạo thành công báo cáo:

+ File Excel hoặc PDF sẽ được lưu trên Amazon S3.
+ Dữ liệu được mã hóa bằng AWS KMS.
+ Có thể áp dụng Lifecycle Rule hoặc Intelligent-Tiering để tối ưu chi phí lưu trữ.

Ví dụ đường dẫn:

```text
reports/monthly-attendance-report.xlsx
```

Sau đó hệ thống lưu S3 Object Key vào DynamoDB.

---

#### 7. Gửi Email bằng Amazon SES

Sau khi báo cáo hoàn tất:

+ EventBridge phát sinh sự kiện ReportGenerated.
+ Lambda Email Worker nhận tin nhắn từ EmailQueue.
+ Amazon SES gửi email cho quản trị viên.

Email sẽ bao gồm:

+ Thông báo tạo báo cáo thành công.
+ Thời gian tạo.
+ Liên kết tải báo cáo từ Amazon S3.

---

#### 8. Luồng xử lý tổng thể

```text
Admin
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
      ├── Upload to Amazon S3
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

#### Hoàn thành Module

Sau khi hoàn thành phần này, bạn đã:

+ Xây dựng quy trình tạo báo cáo bất đồng bộ bằng AWS Step Functions.
+ Sử dụng Amazon SQS để đệm yêu cầu và giảm tải cho hệ thống.
+ Lưu báo cáo trên Amazon S3.
+ Gửi email tự động bằng Amazon SES.
+ Triển khai Dead Letter Queue nhằm tăng độ tin cậy của hệ thống.
+ Hiểu cách xây dựng kiến trúc Event-Driven giúp hệ thống mở rộng dễ dàng và tránh hiện tượng API bị timeout khi xử lý các tác vụ có thời gian thực thi dài.

Sau khi hoàn thành module này, hệ thống đã sẵn sàng để triển khai giao diện người dùng React và kiểm thử toàn bộ quy trình chấm công từ đầu đến cuối.