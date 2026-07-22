---
title : "Deploy Frontend & CloudFront CDN"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

#### Deploy React SPA Frontend and CloudFront CDN

In this module, you will build the React Single Page Application (SPA), upload the production build artifacts to Amazon S3, and distribute the application globally through Amazon CloudFront.

The frontend architecture also uses a **Custom Origin Verification Header** named `x-origin-verify` to prevent users from bypassing CloudFront and directly accessing the Amazon API Gateway endpoint.

---

#### 1. Edge Protection Architecture

The frontend request flow is designed as follows:

```text
User Browser
      │
      ▼
Amazon Route 53
      │
      ▼
Amazon CloudFront CDN
AWS WAF + AWS Shield
      │
      ├──────────────► Amazon S3
      │                React Static Assets
      │
      └──────────────► Amazon API Gateway
                       Header: x-origin-verify
```

The main components include:

+ **Amazon S3 SPA Bucket** stores the React static assets, including `index.html`, JavaScript, CSS, fonts, and image files.
+ **Amazon CloudFront** distributes the frontend application through global edge locations to reduce latency.
+ **AWS WAF** protects the CloudFront distribution against common web attacks and suspicious requests.
+ **AWS Shield Standard** provides basic protection against distributed denial-of-service attacks.
+ **Custom Verification Header** allows CloudFront to attach a secret value to requests forwarded to Amazon API Gateway.
+ **Amazon API Gateway** rejects requests that do not contain the expected `x-origin-verify` header.

Direct public access to the S3 bucket should remain disabled. Users access the frontend through CloudFront instead of requesting objects directly from Amazon S3.

---

#### 2. Configure Frontend Environment Variables

Navigate to the frontend directory:

```bash
cd ../frontend
```

Create a production environment file:

```text
.env.production
```

Add the backend and authentication values obtained from the AWS SAM deployment outputs:

```env
VITE_API_BASE_URL=https://xxxxxxx.execute-api.ap-southeast-1.amazonaws.com/prod
VITE_COGNITO_USER_POOL_ID=ap-southeast-1_xxxxxxxxx
VITE_COGNITO_CLIENT_ID=xxxxxxxxxxxxxxxxxxxxxxxxxx
VITE_AWS_REGION=ap-southeast-1
```

The environment variables are used for:

+ Connecting the React application to Amazon API Gateway.
+ Identifying the Amazon Cognito User Pool.
+ Identifying the Cognito App Client.
+ Configuring the AWS Region used by the application.

Do not place AWS Access Keys or Secret Access Keys inside frontend environment files. Frontend code is delivered to user browsers and must never contain permanent AWS credentials.

---

#### 3. Install Frontend Dependencies

Install the required npm packages:

```bash
npm install
```

After installation, verify that the `node_modules` directory and the package lock file are available.

You can start the application locally for testing:

```bash
npm run dev
```

The development server is typically available at:

```text
http://localhost:5173
```

Verify the following functions before building:

+ Login with Amazon Cognito.
+ Access protected application pages.
+ Call backend APIs through Amazon API Gateway.
+ Display attendance information.
+ Submit check-in and check-out requests.
+ Request monthly report generation.

---

#### 4. Build the React Application

Compile the production application:

```bash
npm run build
```

After a successful build, the `dist/` directory is generated:

```text
dist/
├── assets/
│   ├── index-xxxx.js
│   └── index-xxxx.css
├── favicon.ico
└── index.html
```

The generated files are optimized for production deployment.

The build process usually performs:

+ JavaScript and CSS minification.
+ Asset optimization.
+ Module bundling.
+ Environment variable replacement.
+ Production dependency optimization.

---

#### 5. Create the Frontend S3 Bucket

Retrieve the current AWS Account ID:

```bash
aws sts get-caller-identity \
  --query Account \
  --output text
```

Create a globally unique bucket name using the Account ID:

```bash
aws s3 mb \
  s3://smart-attendance-spa-hosting-<AWS_ACCOUNT_ID> \
  --region ap-southeast-1
```

Example:

```bash
aws s3 mb \
  s3://smart-attendance-spa-hosting-123456789012 \
  --region ap-southeast-1
```

Keep S3 Block Public Access enabled when the bucket is used behind Amazon CloudFront.

---

#### 6. Upload the React Build to Amazon S3

Synchronize the local `dist/` directory with the S3 bucket:

```bash
aws s3 sync \
  dist/ \
  s3://smart-attendance-spa-hosting-<AWS_ACCOUNT_ID> \
  --delete
```

The `--delete` option removes old S3 objects that no longer exist in the local build directory.

To verify the uploaded files:

```bash
aws s3 ls \
  s3://smart-attendance-spa-hosting-<AWS_ACCOUNT_ID> \
  --recursive
```

Expected objects include:

```text
index.html
favicon.ico
assets/index-xxxx.js
assets/index-xxxx.css
```

---

#### 7. Configure Amazon CloudFront

Open the Amazon CloudFront Console and create a new distribution.

Configure the S3 bucket as the frontend origin.

Recommended settings:

+ **Origin domain:** Smart Attendance S3 Bucket.
+ **Origin access:** Origin Access Control.
+ **Viewer protocol policy:** Redirect HTTP to HTTPS.
+ **Allowed HTTP methods:** GET and HEAD.
+ **Compress objects automatically:** Enabled.
+ **Default root object:** `index.html`.

Create an additional origin for Amazon API Gateway if API requests are routed through the same CloudFront distribution.

For the API Gateway origin, configure the custom header:

```text
Header name:
x-origin-verify

Header value:
SaaS-Secure-Verification-Token-2026
```

The header value must match the secret configured in the backend deployment.

---

#### 8. Protect Amazon API Gateway with the Verification Header

The backend should validate the custom header before processing protected API requests.

Example Lambda validation:

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

This configuration helps prevent direct access to the API Gateway URL.

The expected request path becomes:

```text
User
  → CloudFront
  → x-origin-verify header
  → API Gateway
  → AWS Lambda
```

A direct request that does not pass through CloudFront should receive:

```text
HTTP 403 Forbidden
```

---

#### 9. Configure SPA Error Handling

React applications use client-side routing. When users refresh a route such as:

```text
/dashboard
```

Amazon CloudFront may request `/dashboard` as a physical object and receive a `403` or `404` response from S3.

Configure CloudFront custom error responses:

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

This allows React Router to handle application routes correctly.

---

#### 10. CloudFront Account Verification Issue

New AWS accounts may receive an error similar to:

```text
Your account must be verified before you can add new CloudFront resources
```

This restriction may prevent immediate CloudFront distribution creation.

As a temporary workaround, enable Amazon S3 Static Website Hosting:

```bash
aws s3 website \
  s3://smart-attendance-spa-hosting-<AWS_ACCOUNT_ID>/ \
  --index-document index.html \
  --error-document index.html
```

The temporary website URL follows this format:

```text
http://smart-attendance-spa-hosting-<AWS_ACCOUNT_ID>.s3-website-ap-southeast-1.amazonaws.com
```

For the production architecture, request CloudFront access through the AWS Support Center under account verification.

---

#### 11. Invalidate the CloudFront Cache

After uploading a new frontend version, create a CloudFront invalidation so users receive the latest assets immediately:

```bash
aws cloudfront create-invalidation \
  --distribution-id <DISTRIBUTION_ID> \
  --paths "/*"
```

Verify the invalidation status:

```bash
aws cloudfront get-invalidation \
  --distribution-id <DISTRIBUTION_ID> \
  --id <INVALIDATION_ID>
```

Wait until the status becomes:

```text
Completed
```

---

#### 12. Verify the Deployed Application

Open the CloudFront domain in a web browser:

```text
https://dxxxxxxxxxxxxx.cloudfront.net
```

Verify the following functions:

+ The React SPA loads successfully.
+ Static assets are delivered through CloudFront.
+ HTTP requests are redirected to HTTPS.
+ Login works with Amazon Cognito.
+ API requests reach Amazon API Gateway.
+ Direct API Gateway requests without `x-origin-verify` are rejected.
+ Check-in and check-out functions work correctly.
+ Attendance history is displayed.
+ Monthly report requests return an accepted response.
+ Browser refresh works on nested React routes.

---

#### Module Summary

After completing this module, you have:

+ Configured the production environment variables for the React application.
+ Installed frontend dependencies.
+ Built the optimized React SPA.
+ Created an Amazon S3 bucket for frontend assets.
+ Uploaded the application using AWS CLI.
+ Configured Amazon CloudFront for global content delivery.
+ Protected the API origin using `x-origin-verify`.
+ Configured client-side route fallback.
+ Invalidated the CloudFront cache after deployment.
+ Verified the frontend and backend integration.

The Smart Attendance SaaS Platform is now available through a globally distributed and secure web frontend.