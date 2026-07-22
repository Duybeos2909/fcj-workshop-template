---
title : "Triển khai Frontend và CloudFront CDN"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

#### Triển khai React SPA và CloudFront CDN

Trong phần này, bạn sẽ build ứng dụng React Single Page Application (SPA), tải các file Production lên Amazon S3 và phân phối ứng dụng trên toàn cầu thông qua Amazon CloudFront.

Kiến trúc Frontend cũng sử dụng **Custom Origin Verification Header** có tên `x-origin-verify` nhằm ngăn người dùng bỏ qua CloudFront và truy cập trực tiếp vào Amazon API Gateway.

---

#### 1. Kiến trúc bảo vệ tại tầng Edge

Luồng truy cập Frontend được thiết kế như sau:

```text
Trình duyệt người dùng
        │
        ▼
Amazon Route 53
        │
        ▼
Amazon CloudFront CDN
AWS WAF + AWS Shield
        │
        ├──────────────► Amazon S3
        │                Tài nguyên tĩnh React
        │
        └──────────────► Amazon API Gateway
                         Header: x-origin-verify
```

Các thành phần chính bao gồm:

+ **Amazon S3 SPA Bucket** lưu trữ các tài nguyên tĩnh của React như `index.html`, JavaScript, CSS, font chữ và hình ảnh.
+ **Amazon CloudFront** phân phối ứng dụng qua hệ thống Edge Location trên toàn cầu để giảm độ trễ.
+ **AWS WAF** bảo vệ CloudFront trước các request bất thường và những kiểu tấn công Web phổ biến.
+ **AWS Shield Standard** cung cấp lớp bảo vệ cơ bản trước các cuộc tấn công từ chối dịch vụ phân tán.
+ **Custom Verification Header** cho phép CloudFront tự động gắn một giá trị bí mật vào request gửi đến Amazon API Gateway.
+ **Amazon API Gateway** từ chối những request không chứa đúng Header `x-origin-verify`.

Quyền truy cập công khai trực tiếp vào S3 Bucket nên được vô hiệu hóa. Người dùng sẽ truy cập ứng dụng thông qua CloudFront thay vì lấy file trực tiếp từ Amazon S3.

---

#### 2. Cấu hình biến môi trường cho Frontend

Di chuyển đến thư mục Frontend:

```bash
cd ../frontend
```

Tạo file cấu hình Production:

```text
.env.production
```

Điền các giá trị Backend và Authentication nhận được từ Output của AWS SAM:

```env
VITE_API_BASE_URL=https://xxxxxxx.execute-api.ap-southeast-1.amazonaws.com/prod
VITE_COGNITO_USER_POOL_ID=ap-southeast-1_xxxxxxxxx
VITE_COGNITO_CLIENT_ID=xxxxxxxxxxxxxxxxxxxxxxxxxx
VITE_AWS_REGION=ap-southeast-1
```

Các biến môi trường được sử dụng để:

+ Kết nối ứng dụng React với Amazon API Gateway.
+ Xác định Amazon Cognito User Pool.
+ Xác định Cognito App Client.
+ Cấu hình AWS Region của ứng dụng.

Không đưa AWS Access Key hoặc Secret Access Key vào file cấu hình Frontend. Mã Frontend được tải xuống trình duyệt của người dùng nên tuyệt đối không được chứa thông tin xác thực AWS cố định.

---

#### 3. Cài đặt các thư viện Frontend

Cài đặt các package cần thiết:

```bash
npm install
```

Sau khi hoàn tất, kiểm tra thư mục `node_modules` và file khóa package đã được tạo.

Có thể chạy thử ứng dụng trên máy:

```bash
npm run dev
```

Development Server thường được mở tại:

```text
http://localhost:5173
```

Trước khi build, kiểm tra các chức năng:

+ Đăng nhập bằng Amazon Cognito.
+ Truy cập các trang được bảo vệ.
+ Gọi Backend API qua Amazon API Gateway.
+ Hiển thị dữ liệu chấm công.
+ Gửi yêu cầu Check-in và Check-out.
+ Gửi yêu cầu tạo báo cáo tháng.

---

#### 4. Build ứng dụng React

Build phiên bản Production:

```bash
npm run build
```

Sau khi build thành công, thư mục `dist/` sẽ được tạo:

```text
dist/
├── assets/
│   ├── index-xxxx.js
│   └── index-xxxx.css
├── favicon.ico
└── index.html
```

Các file này đã được tối ưu để triển khai lên môi trường Production.

Quá trình build thường bao gồm:

+ Thu gọn JavaScript và CSS.
+ Tối ưu tài nguyên.
+ Đóng gói module.
+ Thay thế biến môi trường.
+ Tối ưu các thư viện Production.

---

#### 5. Tạo S3 Bucket cho Frontend

Lấy AWS Account ID hiện tại:

```bash
aws sts get-caller-identity \
  --query Account \
  --output text
```

Tạo Bucket có tên duy nhất bằng Account ID:

```bash
aws s3 mb \
  s3://smart-attendance-spa-hosting-<AWS_ACCOUNT_ID> \
  --region ap-southeast-1
```

Ví dụ:

```bash
aws s3 mb \
  s3://smart-attendance-spa-hosting-123456789012 \
  --region ap-southeast-1
```

Giữ tính năng S3 Block Public Access ở trạng thái bật khi Bucket được truy cập thông qua Amazon CloudFront.

---

#### 6. Tải ứng dụng React lên Amazon S3

Đồng bộ thư mục `dist/` với S3 Bucket:

```bash
aws s3 sync \
  dist/ \
  s3://smart-attendance-spa-hosting-<AWS_ACCOUNT_ID> \
  --delete
```

Tham số `--delete` xóa những Object cũ trên S3 không còn xuất hiện trong thư mục build hiện tại.

Kiểm tra các file đã tải lên:

```bash
aws s3 ls \
  s3://smart-attendance-spa-hosting-<AWS_ACCOUNT_ID> \
  --recursive
```

Kết quả mong đợi gồm:

```text
index.html
favicon.ico
assets/index-xxxx.js
assets/index-xxxx.css
```

---

#### 7. Cấu hình Amazon CloudFront

Mở Amazon CloudFront Console và tạo Distribution mới.

Cấu hình S3 Bucket làm Frontend Origin.

Các thiết lập được đề xuất:

+ **Origin domain:** S3 Bucket của Smart Attendance.
+ **Origin access:** Origin Access Control.
+ **Viewer protocol policy:** Redirect HTTP to HTTPS.
+ **Allowed HTTP methods:** GET và HEAD.
+ **Compress objects automatically:** Bật.
+ **Default root object:** `index.html`.

Tạo thêm một Origin cho Amazon API Gateway nếu API được định tuyến qua cùng CloudFront Distribution.

Tại API Gateway Origin, cấu hình Custom Header:

```text
Tên Header:
x-origin-verify

Giá trị Header:
SaaS-Secure-Verification-Token-2026
```

Giá trị này phải trùng với Secret đã được cấu hình khi triển khai Backend.

---

#### 8. Bảo vệ Amazon API Gateway bằng Verification Header

Backend phải kiểm tra Custom Header trước khi xử lý những API được bảo vệ.

Ví dụ kiểm tra trong Lambda:

```javascript
const expectedSecret = process.env.ORIGIN_VERIFY_SECRET;
const receivedSecret =
  event.headers?.["x-origin-verify"] ||
  event.headers?.["X-Origin-Verify"];

if (receivedSecret !== expectedSecret) {
  return {
    statusCode: 403,
    body: JSON.stringify({
      message: "Forbidden"
    })
  };
}
```

Cấu hình này giúp hạn chế người dùng gọi trực tiếp API Gateway URL.

Luồng request mong muốn:

```text
Người dùng
  → CloudFront
  → Header x-origin-verify
  → API Gateway
  → AWS Lambda
```

Request trực tiếp không đi qua CloudFront sẽ nhận:

```text
HTTP 403 Forbidden
```

---

#### 9. Cấu hình xử lý lỗi cho React SPA

Ứng dụng React sử dụng Client-side Routing. Khi người dùng Refresh tại một Route như:

```text
/dashboard
```

Amazon CloudFront có thể tìm `/dashboard` như một Object vật lý và nhận phản hồi `403` hoặc `404` từ S3.

Cấu hình CloudFront Custom Error Response:

```text
HTTP Error Code: 403
Response Page Path: /index.html
HTTP Response Code: 200
```

```text
HTTP Error Code: 404
Response Page Path: /index.html
HTTP Response Code: 200
```

Cấu hình này cho phép React Router xử lý đúng các Route trong ứng dụng.

---

#### 10. Lỗi xác minh tài khoản CloudFront

Tài khoản AWS mới có thể gặp lỗi:

```text
Your account must be verified before you can add new CloudFront resources
```

Giới hạn này có thể ngăn việc tạo CloudFront Distribution.

Để xem trước ứng dụng, có thể tạm thời bật Amazon S3 Static Website Hosting:

```bash
aws s3 website \
  s3://smart-attendance-spa-hosting-<AWS_ACCOUNT_ID>/ \
  --index-document index.html \
  --error-document index.html
```

Địa chỉ Website tạm thời có dạng:

```text
http://smart-attendance-spa-hosting-<AWS_ACCOUNT_ID>.s3-website-ap-southeast-1.amazonaws.com
```

Đối với kiến trúc Production, cần gửi yêu cầu tại AWS Support Center để xác minh tài khoản và mở quyền tạo CloudFront Resource.

---

#### 11. Xóa Cache CloudFront

Sau khi tải một phiên bản Frontend mới lên S3, tạo CloudFront Invalidation để người dùng nhận ngay tài nguyên mới nhất:

```bash
aws cloudfront create-invalidation \
  --distribution-id <DISTRIBUTION_ID> \
  --paths "/*"
```

Kiểm tra trạng thái:

```bash
aws cloudfront get-invalidation \
  --distribution-id <DISTRIBUTION_ID> \
  --id <INVALIDATION_ID>
```

Chờ đến khi trạng thái trở thành:

```text
Completed
```

---

#### 12. Kiểm tra ứng dụng đã triển khai

Mở CloudFront Domain bằng trình duyệt:

```text
https://dxxxxxxxxxxxxx.cloudfront.net
```

Kiểm tra các chức năng:

+ React SPA tải thành công.
+ Tài nguyên tĩnh được phân phối qua CloudFront.
+ HTTP được chuyển hướng sang HTTPS.
+ Đăng nhập hoạt động với Amazon Cognito.
+ Request API được chuyển đến Amazon API Gateway.
+ Request trực tiếp không có `x-origin-verify` bị từ chối.
+ Chức năng Check-in và Check-out hoạt động.
+ Lịch sử chấm công được hiển thị.
+ Yêu cầu tạo báo cáo tháng nhận phản hồi Accepted.
+ Refresh trình duyệt tại các React Route không gây lỗi.

---

#### Hoàn thành Module

Sau khi hoàn thành phần này, bạn đã:

+ Cấu hình biến môi trường Production cho ứng dụng React.
+ Cài đặt các thư viện Frontend.
+ Build React SPA đã được tối ưu.
+ Tạo Amazon S3 Bucket để lưu tài nguyên Frontend.
+ Tải ứng dụng lên bằng AWS CLI.
+ Cấu hình Amazon CloudFront để phân phối nội dung toàn cầu.
+ Bảo vệ API Origin bằng `x-origin-verify`.
+ Cấu hình Fallback cho Client-side Routing.
+ Xóa Cache CloudFront sau khi triển khai.
+ Kiểm tra kết nối giữa Frontend và Backend.

Smart Attendance SaaS Platform hiện đã có một giao diện Web được phân phối toàn cầu và được bảo vệ thông qua kiến trúc AWS Edge.