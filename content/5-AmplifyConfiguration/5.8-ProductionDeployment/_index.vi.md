---
title: "Production Deployment"
weight: 8
chapter: false
pre: " <b> 5.8 </b> "
---

## Deploy lên Production

Deploy Weather Platform lên production sử dụng AWS Amplify Hosting với CloudFormation-managed infrastructure.

### Yêu cầu trước

- Sandbox environment đã test và hoạt động
- Tất cả hardcoded bucket names đã được cập nhật trong backend code
- Git repository đã được push lên GitHub/GitLab
- AWS profile `ws1-amplify` được cấu hình với deployment permissions

### Bước 1: Chuẩn bị cho Production

**Build và test locally:**

```bash
# Build Next.js application
pnpm build

# Test production build locally
pnpm start
```

**Commit tất cả changes:**

```bash
# Đảm bảo tất cả backend configuration changes đã được committed
git add .
git commit -m "Update backend configuration for production deployment"
git push origin main
```

### Bước 2: Deploy qua Amplify Console

**Method A: AWS Amplify Console (Khuyến nghị)**

1. **Vào [AWS Amplify Console](https://console.aws.amazon.com/amplify/)**
2. **Nhấn "New app" → "Host web app"**
3. **Kết nối Git repository của bạn:**

   - Chọn GitHub làm source
   - Authorize AWS Amplify để truy cập repository của bạn
   - Chọn Weather Platform repository
   - Chọn branch `prod`

![Create Amplify App](/images/5-amplifyConfiguration/5.8-amplify-deployment/1.png)
![Amplify Connect Repo](/images/5-amplifyConfiguration/5.8-amplify-deployment/2.png)
![Add Repo and Branch](/images/5-amplifyConfiguration/5.8-amplify-deployment/3.png)
![App Settings](/images/5-amplifyConfiguration/5.8-amplify-deployment/4.png)
![Create Service Role](/images/5-amplifyConfiguration/5.8-amplify-deployment/5.png)
![Deploy Amplify App](/images/5-amplifyConfiguration/5.8-amplify-deployment/6.png)

4. **Cấu hình build settings** (auto-detected):

```yaml
version: 1
backend:
  phases:
    preBuild:
      commands:
        - export NODE_OPTIONS="--max-old-space-size=3072"
    build:
      commands:
        - corepack enable
        - pnpm install --frozen-lockfile
        - pnpm exec ampx pipeline-deploy --branch $AWS_BRANCH --app-id
          $AWS_APP_ID
frontend:
  phases:
    preBuild:
      commands:
        - export NODE_OPTIONS="--max-old-space-size=4096"
    build:
      commands:
        - pnpm run build
  artifacts:
    baseDirectory: .next
    files:
      - "**/*"
```

5. **Environment variables** (nếu cần):

   - `NODE_ENV`: `production`
   - Any custom environment variables

6. **Deploy:** Nhấn "Save and deploy"

### Bước 3: Production Resources

**Amplify tự động tạo:**

- **Production CloudFormation stacks** cho tất cả backend resources
- **CloudFront distribution** cho global content delivery
- **SSL certificates** với automatic renewal
- **Custom domain support** (tùy chọn)
- **Branch-based deployments** cho các environments khác nhau
- **Atomic deployments** với instant rollback capability

**Backend resources được deployed:**

- Production versions của tất cả Lambda functions
- S3 buckets với production naming (`weather-dataset-{account-id}-main`)
- Cognito User Pools và Identity Pools
- AWS Glue databases, crawlers, và ETL jobs
- CloudFront distribution cho processed datasets
- EventBridge rules cho scheduled processing

### Bước 4: Post-Deployment Configuration

#### IoT Policy Attachment

Cho mỗi user, gắn IoT policies thủ công (xem [Section 5.7](../5.7-authentication/)):

```bash
# Lấy Cognito Identity ID của user và gắn policy
aws iot attach-policy --policy-name WeatherPlatformPubSubPolicy \
  --target "us-east-1:12345678-1234-1234-1234-123456789012" \
  --profile ws1-amplify
```

#### Custom Domain (Tùy chọn)

1. **Trong Amplify Console:** Vào "Domain management"
2. **Thêm custom domain:** `weather.yourdomain.com`
3. **Cấu hình DNS:** Thêm CNAME record như được hướng dẫn
4. **SSL certificate:** Tự động được provision bởi AWS

### Bước 5: Xác minh Production Deployment

**Frontend verification:**

- Website truy cập được qua Amplify domain
- Authentication hoạt động (sign up/sign in)
- Dashboard load chính xác
- Real-time IoT data streaming

**Backend verification:**

- Lambda functions phản hồi
- S3 buckets được tạo với tên chính xác (Test run bằng cách kích hoạt Step functions)
- Glue ETL pipeline đang chạy
- CloudFront serving processed data

### Monitoring và Maintenance

**CloudWatch dashboards:**

- Application performance metrics
- Error rates và latency
- Cost tracking và optimization

**Amplify Console features:**

- Build history và logs
- Performance monitoring
- User analytics

### Troubleshooting

**Các vấn đề deployment phổ biến:**

1. **Build failures:**

   ```bash
   # Kiểm tra build logs trong Amplify Console
   # Xác minh dependencies trong package.json
   ```

2. **Backend deployment errors:**

   ```bash
   # Kiểm tra CloudFormation events
   # Xác minh IAM permissions
   ```

3. **Domain issues:**
   ```bash
   # Xác minh DNS configuration
   # Kiểm tra SSL certificate status
   ```

{{% notice success %}}
**Ưu điểm CloudFormation**: Amplify sử dụng CloudFormation cho infrastructure deployment, đảm bảo deployments nhất quán, có thể lặp lại với automatic rollback khi có failures.
{{% /notice %}}

{{% notice info %}}
Production deployment thường mất 10-15 phút cho initial setup. Các deployments tiếp theo mất 5-7 phút chỉ cho frontend changes.
{{% /notice %}}

### Các bước tiếp theo

- Giám sát application performance
- Thiết lập production monitoring và alerts
- Cấu hình backup và disaster recovery
- Lập kế hoạch [resource cleanup](../../6-resourcecleanup/) khi hoàn tất
