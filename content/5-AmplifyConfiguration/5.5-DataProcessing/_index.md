---
title: "Data Processing Pipeline"
weight: 5
chapter: false
pre: " <b> 5.5 </b> "
---

## AWS Glue Data Processing

The Weather Platform uses AWS Glue to transform raw IoT messages into structured datasets.

### Processing Flow

![Processing Flow](/images/5-amplifyConfiguration/5.5-data-processing/glue-flow.jpg)

**Orchestration Flow**:

1. **EventBridge** triggers weekly at midnight UTC+7 (every Sunday)
2. **Step Functions** orchestrates the workflow:
   - Starts Glue Crawler to update schema
   - Waits for crawler completion
   - Starts Glue ETL Job for data transformation
   - Export error states (If any)

### Glue Components

**Database**: `weather_data_catalog_{unique-suffix}`

- Catalogs all weather data schemas
- Automatically discovers table structures

**Crawler**: `WeatherPlatformCrawler-{unique-suffix}`

- Scans data lake for new data
- Updates table schemas automatically

**ETL Job**: `WeatherDataTransformJob-{unique-suffix}`

- Transforms JSON to CSV format
- Aggregates data by time periods
- Optimizes for analytical queries

### ETL Script

The Glue job uses a Python script (`weather-transform.py`) that:

1. **Reads raw JSON** from data lake
2. **Normalizes data structure** across different devices
3. **Aggregates data** by time windows
4. **Writes CSV files** to processed datasets bucket

### Step Functions Orchestration

The data processing workflow uses AWS Step Functions for reliable orchestration:

**State Machine Workflow**:

```typescript
// 1. Start Crawler
startCrawler
  .next(waitForCrawler) // 2. Wait 2 minutes
  .next(checkCrawlerStatus) // 3. Check crawler state
  .next(
    new stepfunctions.Choice(this, "IsCrawlerComplete")
      .when(crawlerComplete, startGlueJob) // 4a. Start ETL job
      .when(crawlerRunning, waitForCrawler) // 4b. Continue waiting
      .otherwise(crawlerFailed) // 4c. Handle failure
  );
```

### Scheduling

Data processing is triggered via EventBridge:

- **Frequency**: Weekly at midnight UTC+7 (Sundays)
- **Trigger**: EventBridge cron rule
- **Target**: Step Functions state machine
- **Orchestration**: Automated Crawler → ETL Job sequence

### Monitoring

Glue jobs provide:

- Execution logs in CloudWatch
- Job success/failure notifications
- Data quality metrics
- Cost tracking

{{% notice tip %}}
Glue provides serverless data processing - you only pay for processing time used.
{{% /notice %}}
