---
title: "Create Data Lake"
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

In this section, we'll set up the data lake infrastructure for storing and processing weather data. The Weather Platform uses Amazon S3 as the primary data lake, with AWS Glue for data cataloging and ETL operations, and CloudFront for global data distribution.

{{% notice info %}}
The data lake architecture is automatically deployed as part of the Amplify backend, but understanding its components helps with customization and troubleshooting.
{{% /notice %}}

![Data Lake Architecture](/images/data-lake-architecture.png)

## Data Lake Components

### Amazon S3 Storage Structure

The platform organizes data in S3 with the following structure:

```
weather-platform-bucket/
├── raw-data/                    # Raw IoT sensor data
│   ├── year=2024/
│   │   ├── month=01/
│   │   │   ├── day=15/
│   │   │   │   └── device-01/
│   │   │   │       └── telemetry.json
├── processed-data/              # Processed datasets
│   ├── daily-summaries/
│   │   └── 2024-01-15.csv
│   ├── monthly-reports/
│   │   └── 2024-01.csv
├── datasets/                    # Public datasets via CloudFront
│   ├── latest/
│   │   └── weather-data.csv
│   └── historical/
│       └── 2024/
└── glue-scripts/               # ETL job scripts
    └── weather-data-transform.py
```

### AWS Glue Data Catalog

The Glue Data Catalog automatically discovers and catalogs your weather data:

- **Database**: `weather_platform_db`
- **Tables**: Automatically created from S3 data
- **Schemas**: Inferred from JSON sensor data
- **Partitions**: Organized by year/month/day for efficient querying

### CloudFront Distribution

Global CDN for fast dataset access:

- **Origin**: S3 datasets folder
- **Caching**: Optimized for CSV file downloads
- **Global Edge Locations**: Reduced latency worldwide
- **Custom Domain**: Optional custom domain support

## Data Processing Pipeline

### Step Functions Workflow

The platform uses AWS Step Functions to orchestrate data processing:

```json
{
  "Comment": "Weather Data Processing Pipeline",
  "StartAt": "RunGlueCrawler",
  "States": {
    "RunGlueCrawler": {
      "Type": "Task",
      "Resource": "arn:aws:states:::aws-sdk:glue:startCrawler",
      "Parameters": {
        "Name": "weather-data-crawler"
      },
      "Next": "WaitForCrawler"
    },
    "WaitForCrawler": {
      "Type": "Wait",
      "Seconds": 60,
      "Next": "CheckCrawlerStatus"
    },
    "CheckCrawlerStatus": {
      "Type": "Task",
      "Resource": "arn:aws:states:::aws-sdk:glue:getCrawler",
      "Parameters": {
        "Name": "weather-data-crawler"
      },
      "Next": "IsCrawlerComplete"
    },
    "IsCrawlerComplete": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.Crawler.State",
          "StringEquals": "READY",
          "Next": "RunETLJob"
        }
      ],
      "Default": "WaitForCrawler"
    },
    "RunETLJob": {
      "Type": "Task",
      "Resource": "arn:aws:states:::glue:startJobRun.sync",
      "Parameters": {
        "JobName": "weather-data-etl"
      },
      "End": true
    }
  }
}
```

### EventBridge Scheduling

Automated daily processing at midnight UTC+7:

```json
{
  "ScheduleExpression": "cron(0 17 * * ? *)",
  "Description": "Daily weather data processing",
  "State": "ENABLED",
  "Targets": [
    {
      "Id": "WeatherDataProcessing",
      "Arn": "arn:aws:states:region:account:stateMachine:weather-data-pipeline",
      "RoleArn": "arn:aws:iam::account:role/EventBridgeStepFunctionsRole"
    }
  ]
}
```

## Data Formats and Schemas

### Raw Sensor Data (JSON)

IoT devices send data in this format:

```json
{
  "deviceId": "weather-station-01",
  "timestamp": "2024-01-15T10:30:00Z",
  "location": {
    "latitude": 10.8231,
    "longitude": 106.6297,
    "name": "ITea Lab - Room A"
  },
  "sensors": {
    "temperature": {
      "value": 28.5,
      "unit": "celsius"
    },
    "humidity": {
      "value": 65.2,
      "unit": "percent"
    },
    "pressure": {
      "value": 1013.25,
      "unit": "hPa"
    },
    "wind": {
      "speed": 5.2,
      "direction": 180,
      "unit": "m/s"
    },
    "rainfall": {
      "value": 0.0,
      "unit": "mm"
    }
  },
  "metadata": {
    "firmware_version": "1.2.3",
    "battery_level": 85,
    "signal_strength": -45
  }
}
```

### Processed Dataset (CSV)

ETL jobs transform raw data into structured CSV format:

```csv
timestamp,device_id,location_name,temperature_c,humidity_pct,pressure_hpa,wind_speed_ms,wind_direction_deg,rainfall_mm,battery_level,signal_strength
2024-01-15T10:30:00Z,weather-station-01,ITea Lab - Room A,28.5,65.2,1013.25,5.2,180,0.0,85,-45
2024-01-15T10:35:00Z,weather-station-01,ITea Lab - Room A,28.7,64.8,1013.20,5.5,185,0.0,85,-44
```

## Manual S3 Setup (Optional)

While Amplify creates the S3 bucket automatically, you can also set it up manually for learning purposes:

### Step 1: Create S3 Bucket

1. Go to [S3 Console](https://console.aws.amazon.com/s3/)
2. Click **"Create bucket"**
3. Configure bucket settings:
   - **Bucket name**: `weather-platform-data-lake-[unique-id]`
   - **Region**: Same as your Amplify deployment
   - **Block Public Access**: Keep enabled for security
   - **Versioning**: Enable for data protection

### Step 2: Configure Bucket Structure

Create the folder structure using AWS CLI:

```bash
# Create folder structure
aws s3api put-object --bucket weather-platform-data-lake-[unique-id] --key raw-data/ --profile weather-platform
aws s3api put-object --bucket weather-platform-data-lake-[unique-id] --key processed-data/ --profile weather-platform
aws s3api put-object --bucket weather-platform-data-lake-[unique-id] --key datasets/ --profile weather-platform
aws s3api put-object --bucket weather-platform-data-lake-[unique-id] --key glue-scripts/ --profile weather-platform
```

### Step 3: Set Up Lifecycle Policies

Configure automatic data archival:

```json
{
  "Rules": [
    {
      "ID": "WeatherDataLifecycle",
      "Status": "Enabled",
      "Filter": {
        "Prefix": "raw-data/"
      },
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        },
        {
          "Days": 90,
          "StorageClass": "GLACIER"
        },
        {
          "Days": 365,
          "StorageClass": "DEEP_ARCHIVE"
        }
      ]
    }
  ]
}
```

## Monitoring and Optimization

### CloudWatch Metrics

Key metrics to monitor:

- **S3 Storage Usage**: Track data growth over time
- **Glue Job Duration**: Monitor ETL performance
- **CloudFront Cache Hit Ratio**: Optimize global access
- **Data Processing Errors**: Track failed transformations

### Cost Optimization

- **S3 Intelligent Tiering**: Automatic cost optimization
- **Glue Job Scheduling**: Process data during off-peak hours
- **CloudFront Caching**: Reduce S3 request costs
- **Data Lifecycle Policies**: Archive old data to Glacier

{{% notice tip %}}
The data lake infrastructure scales automatically with your data volume. Start with the default configuration and optimize based on your specific usage patterns.
{{% /notice %}}

## Next Steps

With the data lake configured, you can:

1. **Monitor data ingestion** from IoT devices
2. **Customize ETL transformations** for specific analytics needs
3. **Set up data quality checks** to ensure data integrity
4. **Create custom datasets** for specific research requirements
5. **Integrate with BI tools** like Amazon QuickSight for advanced analytics

The data lake provides a solid foundation for scalable weather data analytics and research applications.
