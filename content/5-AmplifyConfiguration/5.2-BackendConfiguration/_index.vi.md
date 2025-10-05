---
title: "Cấu hình Amplify Backend"
weight: 2
chapter: false
pre: " <b> 5.2 </b> "
---

## Cấu hình Amplify Backend

Cấu hình và triển khai AWS Amplify backend sử dụng profile `ws1-amplify`.

### Bước 1: Cấu hình AWS Profile

Đảm bảo AWS CLI của bạn đã được cấu hình với profile `ws1-amplify`:

```bash
# Xác minh profile tồn tại
aws configure list --profile ws1-amplify

# Nếu chưa cấu hình, thiết lập nó
aws configure --profile ws1-amplify
```

### Bước 2: Cập nhật Backend Configuration

Trước khi deployment, cập nhật tên bucket được hardcoded trong backend:

**Files cần cập nhật** (sử dụng tên bucket không trùng lặp):

```bash
# Sử dụng Find & Replace để cập nhật tất cả instances
# Từ: itea-weather-data-lake-storage
# Thành: itea-weather-data-lake-storage-yourname
```

**Key files**:

- `amplify/backend.ts`
- `amplify/functions/getTotalReadings/handler.ts`
- `amplify/custom/WeatherDataGlue/resource.ts`

### Bước 3: Deploy Amplify Sandbox

Sử dụng Amplify sandbox cho development với profile chính xác:

```bash
# Deploy sandbox environment
npx ampx sandbox --profile ws1-amplify
```

### Bước 4: Quá trình Sandbox Deployment

Sandbox sẽ deploy:

1. **Authentication Resources**:
   - Cognito User Pool
   - Cognito Identity Pool
   - IAM roles cho authenticated/unauthenticated access

2. **Storage Resources**:
   - S3 bucket cho processed datasets
   - Cấu hình CORS cho web access

3. **Lambda Functions**:
   - IoT device management functions
   - Data retrieval functions
   - Dataset processing functions

4. **Custom CDK Constructs**:
   - AWS Glue database và crawler
   - CloudFront distribution
   - EventBridge scheduled processing
   - Step Functions cho orchestration
   - Dataset storage S3 bucket

5. **IAM Policies và Roles**:
   - Function execution roles
   - S3 access permissions

### Bước 5: Giám sát sandbox deployment

Theo dõi quá trình triển khai trong terminal:

```bash
# Sandbox sẽ hiển thị deployment progress
# ✓ Building backend...
# ✓ Deploying backend...
# ✓ Backend deployed successfully
```

### Bước 6: Xác minh

Sau triển khai thành công:

1. **Kiểm tra generated files**:

   ```bash
   # Amplify outputs file phải được tạo
   ls amplify_outputs.json
   ```

2. **Xác minh AWS resources** trong console:
   - **Amplify**: Kiểm tra app trong Amplify Console
   - **CloudFormation**: Xác minh stack deployment
   - **S3**: Xác nhận bucket creation
   - **Lambda**: Kiểm tra function deployment
   - **Cognito**: Xác minh User Pool creation

### Bước 7: Thiết lập User và gắn IoT Policy

Sau deployment, bạn cần cấu hình quyền truy cập người dùng vào platform:

#### 7.1: Tạo Platform User

1. **Sign up cho user mới** qua frontend application:

   ```bash
   # Khởi động development server trước
   pnpm dev
   ```

2. **Thêm user vào platform-admin group** trong Cognito:
   - Vào **Amazon Cognito** → **User pools**
   - Chọn User Pool được tạo bởi Amplify (thường có tên `amplify-{app-name}-{env}-userPool`)
   - Vào tab **Groups** → **Create group**
   - Group name: `platform-admin`
   - Description: `Authenticated users with platform access`
   - Vào tab **Users** → Chọn user của vừa được tạo → **Add to group** → `platform-admin`

#### 7.2: Lấy Cognito Identity ID

1. **Vào Amazon Cognito** → **Identity pools**
2. **Chọn Identity Pool** được tạo bởi Amplify (thường có tên `amplify-{app-name}-{env}-identityPool`)
3. **Chuyển đến tab "Identity browser"**
4. **Tìm và copy Identity ID** của user của bạn (format: `us-east-1:12345678-1234-1234-1234-123456789012`)

#### 7.3: Gắn IoT Policy vào User

**Bước Quan trọng**: Gắn IoT policy vào Identity ID cụ thể của user:

```bash
# Thay thế <identity_ID> bằng Identity ở bước 7.2
aws iot attach-principal-policy \
  --policy-name WeatherPlatformPubSubPolicy \
  --principal us-east-1:<identity_ID> \
  --region us-east-1 \
  --profile ws1-amplify
```

**Ví dụ:**

```bash
aws iot attach-principal-policy \
  --policy-name WeatherPlatformPubSubPolicy \
  --principal us-east-1:12345678-1234-1234-1234-123456789012 \
  --region us-east-1 \
  --profile ws1-amplify
```

#### 7.4: Xác minh IoT Policy Attachment

**Xác minh policy đã được gắn thành công:**

1. **Vào AWS IoT Core** → **Secure** → **Policies**
2. **Chọn `WeatherPlatformPubSubPolicy`**
3. **Chuyển đến tab "Targets"**
4. **Xác minh** bạn thấy Cognito Identity ID của user được liệt kê

**Xác minh CLI thay thế:**

```bash
# Liệt kê tất cả principals gắn với policy
aws iot list-policy-principals \
  --policy-name WeatherPlatformPubSubPolicy \
  --profile ws1-amplify
```

{{% notice warning %}}
**Hạn chế IoT Policy quan trọng**: AWS IoT Core policies có thể được gắn vào từng Cognito Identity ID nhưng **không thể gắn trực tiếp** vào Cognito Identity Pool IDs. Điều này có nghĩa là cho mỗi user được tạo, administrator phải thực hiện bước bổ sung này để cho phép user sử dụng đầy đủ các IoT Core resources.
{{% /notice %}}

{{% notice info %}}
**Tại sao bước này cần thiết**: Mỗi authenticated user nhận được một Cognito Identity ID duy nhất khi họ đăng nhập. IoT policies phải được gắn vào từng Identity ID cụ thể để cấp quyền truy cập các IoT Core resources như subscribe MQTT topics và nhận telemetry data.
{{% /notice %}}

### Bước 8: Frontend Configuration

Khởi động development server:

```bash
# Khởi động Next.js development server
pnpm dev
```

Frontend sẽ tự động sử dụng `amplify_outputs.json` cho configuration.

### Bước 9: Troubleshooting

**Các vấn đề phổ biến**:

1. **Profile không tìm thấy**:

   ```bash
   # Cấu hình lại profile
   aws configure --profile ws1-amplify
   ```

2. **Deployment thất bại**:

   ```bash
   # Clean và retry
   npx ampx sandbox delete --profile ws1-amplify
   npx ampx sandbox --profile ws1-amplify
   ```

3. **Permission errors**:

   - Xác minh IAM permissions cho profile `ws1-amplify`
   - Đảm bảo quyền đầy đủ cho CloudFormation, S3, Lambda, etc.

4. **IoT policy attachment thất bại**:

   ```bash
   # Xác minh policy tồn tại
   aws iot get-policy --policy-name WeatherPlatformPubSubPolicy --profile ws1-amplify

   # Kiểm tra nếu policy đã được gắn
   aws iot list-policy-principals --policy-name WeatherPlatformPubSubPolicy --profile ws1-amplify
   ```

5. **User không thể truy cập IoT resources**:

   - Xác minh user trong `platform-admin` group trong Cognito User Pool
   - Xác nhận IoT policy được gắn vào Identity ID của user
   - Kiểm tra nếu Identity ID của user xuất hiện trong IoT Core → Policies → WeatherPlatformPubSubPolicy → Targets

6. **Dashboard không load IoT data**:
   - Đảm bảo user đã authenticated và trong platform-admin group
   - Xác minh IoT policy attachment vào Identity ID của user
   - Kiểm tra browser console cho authentication errors

{{% notice tip %}}
Giữ sandbox running trong quá trình development. Nó cung cấp hot-reloading cho backend changes.
{{% /notice %}}

{{% notice warning %}}
Sandbox environments chỉ dành cho development. Sử dụng production deployment cho live applications.
{{% /notice %}}

**Không có bước 4, users không thể:**

- Subscribe IoT MQTT topics
- Nhận weather data real-time
- Truy cập các tính năng IoT của platform dashboard
