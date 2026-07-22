---
title : "Thiết kế Single-Table trên DynamoDB"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

#### Thiết kế Single-Table trên DynamoDB và DynamoDB Streams CDC

Trong mô hình **SaaS đa tenant (Multi-Tenant SaaS)**, thiết kế cơ sở dữ liệu ảnh hưởng trực tiếp đến chi phí hạ tầng, hiệu năng truy vấn, khả năng mở rộng và mức độ cô lập dữ liệu giữa các tenant.

Trong phần này, bạn sẽ tìm hiểu cách **Smart Attendance SaaS Platform** lưu trữ nhiều loại dữ liệu khác nhau trong cùng một bảng Amazon DynamoDB bằng mô hình **Single-Table Design**.

Ngoài ra, bạn cũng sẽ khám phá cách **DynamoDB Streams** kết hợp với **Amazon EventBridge Pipes** để triển khai cơ chế **Change Data Capture (CDC)**, giúp xử lý các sự kiện bất đồng bộ khi dữ liệu thay đổi.

---

#### 1. Thiết kế Schema Single-Table

Thay vì tạo nhiều bảng DynamoDB riêng biệt cho người dùng, dữ liệu chấm công, doanh nghiệp và gói đăng ký, toàn bộ dữ liệu sẽ được lưu trong một bảng duy nhất có tên:

```text
smart-attendance-database
```

Bảng sử dụng khóa chính tổng hợp gồm:

+ **Partition Key:** `PK`
+ **Sort Key:** `SK`

Trong đó:

- `PK` dùng để nhóm dữ liệu theo từng doanh nghiệp (tenant).
- `SK` dùng để phân biệt từng loại dữ liệu và từng bản ghi cụ thể.

Mô hình dữ liệu như sau:

| Loại dữ liệu | Partition Key (PK) | Sort Key (SK) | Thuộc tính |
| --- | --- | --- | --- |
| Thông tin doanh nghiệp | `TENANT#<tenantId>` | `METADATA` | tenantName, plan, status, createdAt |
| Hồ sơ nhân viên | `TENANT#<tenantId>` | `USER#<userId>` | email, fullName, role, department |
| Nhật ký chấm công | `TENANT#<tenantId>` | `ATTENDANCE#<userId>#<timestamp>` | checkInTime, checkOutTime, status, location |
| Thông tin thanh toán | `TENANT#<tenantId>` | `SUB#<billingId>` | planTier, amount, paymentStatus, expiryDate |

Ví dụ dữ liệu:

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

Thiết kế này giúp hệ thống có thể truy vấn toàn bộ dữ liệu của một doanh nghiệp chỉ với một Partition Key.

---

#### 2. Cô lập dữ liệu giữa các Tenant

Trong ứng dụng SaaS đa tenant, việc cô lập dữ liệu là yêu cầu bắt buộc.

Mọi truy vấn tới DynamoDB đều phải sử dụng **tenantId** lấy từ JWT Token sau khi người dùng xác thực thành công.

Định dạng Partition Key:

```text
TENANT#<tenantId>
```

Ví dụ:

```text
TENANT#company-001
```

Không nên lấy `tenantId` trực tiếp từ Request Body vì có thể dẫn đến nguy cơ truy cập dữ liệu trái phép.

Thay vào đó, Lambda sẽ lấy tenantId từ JWT Claims:

```javascript
const claims = event.requestContext.authorizer.jwt.claims;
const tenantId = claims["custom:tenantId"];
const partitionKey = `TENANT#${tenantId}`;
```

Sau đó thực hiện truy vấn:

```javascript
const command = new QueryCommand({
  TableName: process.env.TABLE_NAME,
  KeyConditionExpression: "PK = :pk",
  ExpressionAttributeValues: {
    ":pk": `TENANT#${tenantId}`
  }
});
```

Cách triển khai này đảm bảo:

+ Người dùng chỉ truy cập dữ liệu thuộc doanh nghiệp của mình.
+ Ngăn chặn truy vấn dữ liệu giữa các tenant.
+ Chính sách phân quyền luôn thống nhất trên tất cả Lambda Function.
+ Giảm thiểu nguy cơ rò rỉ dữ liệu.

---

#### 3. Kiểm tra bảng DynamoDB trên AWS Console

Mở:

```text
AWS Management Console
→ DynamoDB
→ Tables
→ smart-attendance-database
```

Kiểm tra các thông tin trong phần **Overview**.

##### Capacity Mode

Bảng sử dụng:

```text
On-Demand
```

Ưu điểm:

+ Không cần cấu hình trước số lượng Read/Write Capacity.
+ Tự động mở rộng theo lưu lượng truy cập.
+ Chỉ tính phí theo số lượng request thực tế.
+ Phù hợp cho môi trường phát triển và hệ thống có lưu lượng không ổn định.

##### Primary Key

Kiểm tra bảng sử dụng:

```text
Partition key: PK
Sort key: SK
```

##### Mã hóa dữ liệu

Dữ liệu được mã hóa bằng:

```text
AWS KMS Customer Managed Key
```

Khóa KMS được khai báo trong AWS SAM với tên:

```text
DataKMSKey
```

##### Point-in-Time Recovery (PITR)

Đảm bảo **Point-in-Time Recovery** đã được bật.

Tính năng này giúp khôi phục bảng về bất kỳ thời điểm nào trong khoảng thời gian được AWS hỗ trợ.

PITR giúp bảo vệ dữ liệu khỏi:

+ Xóa nhầm dữ liệu.
+ Lỗi cập nhật ứng dụng.
+ Hỏng dữ liệu.
+ Sai sót trong quá trình vận hành.

##### DynamoDB Streams

Đảm bảo DynamoDB Streams được bật với chế độ:

```text
NEW_AND_OLD_IMAGES
```

Chế độ này lưu lại cả dữ liệu trước và sau khi thay đổi.

---

#### 4. Khai báo DynamoDB trong AWS SAM

Ví dụ cấu hình trong `template.yaml`:

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

---

#### 5. Thêm dữ liệu Tenant

```bash
aws dynamodb put-item \
--table-name smart-attendance-database \
...
```

Sau đó kiểm tra:

```bash
aws dynamodb get-item \
--table-name smart-attendance-database \
...
```

---

#### 6. Thêm dữ liệu người dùng

```bash
aws dynamodb put-item \
--table-name smart-attendance-database \
...
```

Người dùng sẽ được lưu cùng Partition Key của doanh nghiệp.

---

#### 7. Thêm bản ghi chấm công

```bash
aws dynamodb put-item \
--table-name smart-attendance-database \
...
```

Truy vấn toàn bộ dữ liệu của doanh nghiệp:

```bash
aws dynamodb query \
--table-name smart-attendance-database \
...
```

Chỉ lấy dữ liệu chấm công:

```bash
aws dynamodb query \
--table-name smart-attendance-database \
...
```

---

#### 8. DynamoDB Streams và Change Data Capture

Mỗi khi dữ liệu được:

+ Thêm mới
+ Cập nhật
+ Xóa

DynamoDB Streams sẽ tự động tạo một bản ghi sự kiện.

Ví dụ khi **CheckInFunction** ghi dữ liệu chấm công, DynamoDB Streams sẽ sinh ra một Stream Record chứa thông tin vừa được thêm.

---

#### 9. Tích hợp EventBridge Pipes

Hệ thống sử dụng **Amazon EventBridge Pipes** để chuyển dữ liệu từ DynamoDB Streams sang EventBridge Event Bus.

Ví dụ cấu hình:

```yaml
DdbToEventBridgePipe:
  Type: AWS::Pipes::Pipe
  Properties:
    Name: smart-attendance-ddb-stream-pipe
    ...
```

Luồng xử lý:

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
Notification / Report Workers
```

---

#### 10. Xử lý sự kiện bất đồng bộ

Khi một nhân viên thực hiện Check-in:

1. Lambda ghi dữ liệu vào DynamoDB.
2. DynamoDB phản hồi thành công.
3. API trả kết quả ngay cho người dùng.
4. DynamoDB Streams ghi nhận thay đổi.
5. EventBridge Pipe chuyển sự kiện.
6. EventBridge gửi tới các dịch vụ đăng ký.
7. Lambda nền xử lý gửi email, tạo báo cáo hoặc webhook.

Kiến trúc này giúp:

+ Giảm thời gian phản hồi API.
+ Giảm sự phụ thuộc giữa các dịch vụ.
+ Dễ mở rộng hệ thống.
+ Tăng độ tin cậy.
+ Dễ tích hợp với hệ thống bên ngoài.
+ Hỗ trợ Retry khi xảy ra lỗi.

---

#### 11. Kiểm tra DynamoDB Streams

Kiểm tra Stream ARN:

```bash
aws dynamodb describe-table \
--table-name smart-attendance-database \
--query "Table.LatestStreamArn"
```

Liệt kê các Stream:

```bash
aws dynamodbstreams list-streams \
--table-name smart-attendance-database
```

---

#### 12. Kiểm tra EventBridge Pipe

Liệt kê các Pipe:

```bash
aws pipes list-pipes
```

Kiểm tra chi tiết:

```bash
aws pipes describe-pipe \
--name smart-attendance-ddb-stream-pipe
```

Trạng thái mong đợi:

```text
RUNNING
```

---

#### Hoàn thành Module

Sau khi hoàn thành phần này, bạn đã:

+ Tìm hiểu mô hình Single-Table Design.
+ Tạo dữ liệu mẫu cho Tenant, User và Attendance.
+ Áp dụng cơ chế cô lập dữ liệu theo Tenant.
+ Kiểm tra chế độ On-Demand của DynamoDB.
+ Cấu hình mã hóa bằng AWS KMS.
+ Bật Point-in-Time Recovery.
+ Kích hoạt DynamoDB Streams với chế độ `NEW_AND_OLD_IMAGES`.
+ Tích hợp DynamoDB Streams với Amazon EventBridge thông qua EventBridge Pipes.
+ Chuẩn bị nền tảng cho luồng xử lý bất đồng bộ trong các bước tiếp theo.

Tiếp theo, bạn sẽ triển khai quy trình tạo và gửi báo cáo chấm công bất đồng bộ bằng AWS Step Functions và Amazon SQS.