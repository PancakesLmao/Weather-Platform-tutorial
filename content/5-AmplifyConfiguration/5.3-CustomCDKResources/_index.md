---
title: "Custom CDK Constructs"
weight: 3
chapter: false
pre: " <b> 5.3 </b> "
---

## Understanding Custom CDK Constructs

The Weather Platform uses custom AWS CDK constructs to provide advanced functionality beyond standard Amplify resources.

### WeatherDatasetStorage Construct

**Purpose**: Advanced S3 bucket configuration with lifecycle policies and CORS.

**Key Features**:

- Automatic lifecycle management (temp files deleted after 30 days)
- CORS configuration for web access
- Server-side encryption
- Custom tagging for resource management

**File**: `amplify/custom/WeatherDatasetStorage/resource.ts`

```typescript
// Creates S3 bucket with advanced configuration
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

**Purpose**: ETL processing pipeline for transforming raw IoT data into structured datasets.

**Key Features**:

- Glue Database for data cataloging
- Crawler for schema discovery from raw data bucket
- ETL Job for data transformation (JSON to CSV)
- Integration between source bucket (`itea-weather-data-lake-storage`) and target bucket

**File**: `amplify/custom/WeatherDataGlue/resource.ts`

**Components**:

- **Glue Database**: `weather-platform-db-{uniqueId}`
- **Glue Crawler**: Discovers data structure from S3 raw data
- **Glue Job**: Transforms telemetry data using Python script
- **Python Script**: `weather-transform.py` for custom transformation logic
- **IAM Roles**: Proper permissions for Glue service access

### CloudFront Construct

**Purpose**: Global content delivery network for processed datasets.

**Key Features**:

- Global edge locations for fast access
- Origin Access Control (OAC) for secure S3 integration
- Custom cache policies optimized for dataset files
- Automatic bucket policy configuration
- Integration with WeatherDatasetStorage bucket

**File**: `amplify/custom/CloudFront/resource.ts`

**Technical Implementation**:

- Uses CloudFormation template for complex OAC setup
- Proper S3 bucket policy integration
- Cache optimization for CSV/JSON dataset files

### EventBridge Construct

**Purpose**: Scheduled data processing automation.

**Key Features**:

- Weekly data processing at midnight UTC+7 (Sundays)
- Step Functions orchestration of Glue workflow
- Automated crawler execution followed by ETL job
- Proper IAM roles for cross-service integration

**File**: `amplify/custom/EventBridge/resource.ts`

**Schedule Configuration**:

```typescript
schedule: events.Schedule.cron({
  minute: "0",
  hour: "17", // 17:00 UTC = 00:00 UTC+7
  weekDay: "SUN", // Sunday
});
```

### Integration with Amplify

These constructs integrate with Amplify through the main backend configuration:

```typescript
// In amplify/backend.ts

// Create custom CDK storage construct for weather dataset
const weatherStorage = new WeatherDatasetStorage(
  backend.stack,
  "WeatherDatasetStorage",
  {
    bucketName: `weather-dataset-${accountId}-${branchName}`,
  }
);

// Create Weather Data Glue construct for ETL processing
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

// Create CloudFront CDN construct for global delivery
const cloudFrontCDN = new CustomCloudFront(backend.stack, "WeatherDatasetCDN", {
  storageBucketName: weatherStorage.bucket.bucketName,
  storageBucketDomainName: weatherStorage.bucket.bucketDomainName,
  storageBucket: weatherStorage.bucket,
});

// Create EventBridge construct for scheduled processing
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
Custom CDK constructs provide enterprise-level functionality while maintaining Amplify's ease of use.
{{% /notice %}}
