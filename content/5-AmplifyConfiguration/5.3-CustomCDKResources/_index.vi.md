---
title: "Custom CDK Constructs"
weight: 3
chapter: false
pre: " <b> 5.3 </b> "
---

## Hiểu về Custom CDK Constructs

Weather Platform sử dụng custom AWS CDK constructs để cung cấp chức năng nâng cao vượt xa các standard Amplify resources.

### WeatherDatasetStorage Construct

**Mục đích**: Cấu hình S3 bucket nâng cao với lifecycle policies và CORS.

**Tính năng chính**:

- Quản lý lifecycle tự động (temp files bị xóa sau 30 ngày)
- Cấu hình CORS cho web access
- Server-side encryption
- Custom tagging để quản lý resources

**File**: `amplify/custom/WeatherDatasetStorage/resource.ts`

```typescript
// Tạo S3 bucket với cấu hình nâng cao
this.bucket = new s3.Bucket(this, "WeatherDatasetBucket", {
  bucketName: uniqueBucketName,
  removalPolicy: cdk.RemovalPolicy.DESTROY,
  autoDeleteObjects: true,
  versioned: enableVersioning,
  encryption: s3.BucketEncryption.S3_MANAGED,
  lifecycleRules: [
    {
      id: "DeleteIncompleteMultipartUploads",
      abortIncompleteMultipartUploadAfter: cdk.Duration.days(7),
    },
    {
      id: "TempDataCleanup",
      prefix: "temp/",
      expiration: cdk.Duration.days(30),
    },
  ],
});
```

### WeatherDataGlue Construct

**Mục đích**: ETL processing pipeline để chuyển đổi raw IoT data thành structured datasets.

**Tính năng chính**:

- Glue Database để cataloging dữ liệu
- Crawler để schema discovery từ raw data bucket
- ETL Job để data transformation (JSON sang CSV)
- Tích hợp giữa source bucket (`itea-weather-data-lake-storage`) và target bucket

**File**: `amplify/custom/WeatherDataGlue/resource.ts`

**Các thành phần**:

- **Glue Database**: `weather-platform-db-{uniqueId}`
- **Glue Crawler**: Phát hiện cấu trúc dữ liệu từ S3 raw data
- **Glue Job**: Chuyển đổi telemetry data sử dụng Python script
- **Python Script**: `weather-transform.py` cho custom transformation logic
- **IAM Roles**: Quyền phù hợp cho Glue service access

### CloudFront Construct

**Mục đích**: Global content delivery network cho processed datasets.

**Tính năng chính**:

- Global edge locations để truy cập nhanh
- Origin Access Control (OAC) cho S3 integration bảo mật
- Custom cache policies được tối ưu hóa cho dataset files
- Automatic bucket policy configuration
- Tích hợp với WeatherDatasetStorage bucket

**File**: `amplify/custom/CloudFront/resource.ts`

**Technical Implementation**:

- Sử dụng CloudFormation template cho OAC setup phức tạp
- S3 bucket policy integration đúng cách
- Cache optimization cho CSV/JSON dataset files

### EventBridge Construct

**Mục đích**: Scheduled data processing automation.

**Tính năng chính**:

- Xử lý dữ liệu hàng tuần vào lúc nửa đêm UTC+7 (Chủ nhật)
- Step Functions orchestration của Glue workflow
- Automated crawler execution theo sau là ETL job
- Proper IAM roles cho cross-service integration

**File**: `amplify/custom/EventBridge/resource.ts`

**Schedule Configuration**:

```typescript
schedule: events.Schedule.cron({
  minute: "0",
  hour: "17", // 17:00 UTC = 00:00 UTC+7
  weekDay: "SUN", // Sunday
});
```

### Tích hợp với Amplify

Các constructs này tích hợp với Amplify thông qua main backend configuration:

```typescript
// Trong amplify/backend.ts

// Tạo custom CDK storage construct cho weather dataset
const weatherStorage = new WeatherDatasetStorage(
  backend.stack,
  "WeatherDatasetStorage",
  {
    bucketName: `weather-dataset-${accountId}-${branchName}`,
  }
);

// Tạo Weather Data Glue construct cho ETL processing
const weatherDataGlue = new CustomWeatherDataGlue(
  backend.stack,
  "WeatherDataGlue",
  {
    accountId,
    region,
    sourceBucketName: "itea-weather-data-lake-storage",
    targetBucketName: weatherStorage.bucket.bucketName,
  }
);

// Tạo CloudFront CDN construct cho global delivery
const cloudFrontCDN = new CustomCloudFront(backend.stack, "WeatherDatasetCDN", {
  storageBucketName: weatherStorage.bucket.bucketName,
  storageBucketDomainName: weatherStorage.bucket.bucketDomainName,
  storageBucket: weatherStorage.bucket,
});

// Tạo EventBridge construct cho scheduled processing
const eventBridge = new CustomEventBridge(
  backend.stack,
  "WeatherDataProcessing",
  {
    accountId,
    region,
    crawlerName: weatherDataGlue.crawler.name!,
    jobName: weatherDataGlue.job.name!,
  }
);
```

{{% notice info %}}
Custom CDK constructs cung cấp chức năng cấp doanh nghiệp trong khi vẫn duy trì sự dễ sử dụng của Amplify.
{{% /notice %}}
