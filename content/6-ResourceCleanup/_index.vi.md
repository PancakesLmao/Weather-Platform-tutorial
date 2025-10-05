---
title: "Dọn dẹp tài nguyên"
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

## Dọn dẹp tài nguyên AWS

Dọn dẹp đúng cách tất cả resources của Weather Platform để tránh chi phí không cần thiết. Infrastructure dựa trên CloudFormation của Amplify giúp việc cleanup đơn giản với các lệnh đơn giản.

### Lệnh Cleanup Nhanh

#### Đối với môi trường Development Sandbox

```bash
# Một lệnh duy nhất để xóa toàn bộ sandbox environment
npx ampx sandbox delete --profile ws1-amplify
```

#### Đối với môi trường Production

```bash
# Xóa production deployment
npx ampx pipeline-deploy --branch main --appId <your-app-id> --delete --profile ws1-amplify
```

### Những gì được Cleanup

**Amplify tự động xóa:**

- Tất cả CloudFormation stacks
- Lambda functions và execution roles của chúng
- S3 buckets và nội dung (nếu được cấu hình tự động xóa)
- Cognito User Pools và Identity Pools
- AWS Glue databases, crawlers, và jobs
- CloudFront distributions
- EventBridge rules
- IAM roles và policies được tạo bởi Amplify

### Cleanup Thủ công

Một số resources có thể cần xóa thủ công:

#### IoT Core Resources

```bash
# Xóa IoT policies
aws iot delete-policy --policy-name WeatherStationPolicies --profile ws1-amplify
aws iot delete-policy --policy-name WeatherPlatformPubSubPolicy --profile ws1-amplify

# Xóa IoT Rule
aws iot delete-topic-rule --rule-name WeatherTelemetryToS3Rule --profile ws1-amplify

# Xóa Thing Group
aws iot delete-thing-group --thing-group-name ITeaWeatherHub --profile ws1-amplify
```

#### Data Lake Bucket

```bash
# Làm trống và xóa data lake bucket
aws s3 rm s3://itea-weather-data-lake-storage-yourname --recursive --profile ws1-amplify
aws s3 rb s3://itea-weather-data-lake-storage-yourname --profile ws1-amplify
```

### Các bước Xác minh

Sau khi cleanup, xác minh tất cả resources đã bị xóa:

1. **AWS Console Verification:**

   - **Amplify**: Kiểm tra không còn app nào
   - **CloudFormation**: Xác minh stacks đã bị xóa
   - **S3**: Xác nhận buckets đã được gỡ bỏ
   - **Lambda**: Kiểm tra không còn functions nào
   - **Cognito**: Xác minh User Pools đã bị xóa

2. **CLI Verification:**

   ```bash
   # Kiểm tra S3 buckets còn lại
   aws s3 ls --profile ws1-amplify | grep weather

   # Kiểm tra Lambda functions còn lại
   aws lambda list-functions --profile ws1-amplify | grep weather

   # Kiểm tra CloudFormation stacks
   aws cloudformation list-stacks --profile ws1-amplify | grep amplify
   ```

### Giám sát Chi phí

**Xác minh sau cleanup:**

1. **AWS Cost Explorer**: Kiểm tra các chi phí còn lại
2. **Billing Dashboard**: Giám sát chi phí bất thường trong vài ngày tiếp theo
3. **Budget Alerts**: Thiết lập cảnh báo cho chi tiêu bất thường

{{% notice info %}}
**Ưu điểm của Amplify**: Không giống như các thiết lập AWS thủ công, cách tiếp cận dựa trên CloudFormation của Amplify đảm bảo cleanup hoàn chỉnh và đáng tin cậy của tất cả các resources liên quan.
{{% /notice %}}

### Emergency Cleanup

Nếu cleanup tiêu chuẩn thất bại:

```bash
# Force delete CloudFormation stacks
aws cloudformation delete-stack --stack-name amplify-<app-name> --profile ws1-amplify

# List và xóa thủ công các resources còn lại
aws resourcegroupstaggingapi get-resources --tag-filters Key=amplify:project-name --profile ws1-amplify
```

{{% notice success %}}
Cleanup hoàn chỉnh đảm bảo bạn tránh chi phí hằng tháng của platform. Để phân tích chi phí chi tiết, xem [phần Pricing Plan](../7-pricingplan/).
{{% /notice %}}
