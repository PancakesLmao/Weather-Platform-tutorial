---
title: "Data Processing Pipeline"
weight: 5
chapter: false
pre: " <b> 5.5 </b> "
---

## AWS Glue Data Processing

Weather Platform sử dụng AWS Glue để chuyển đổi raw IoT messages thành structured datasets.

### Processing Flow

![Processing Flow](/images/5-amplifyConfiguration/5.5-data-processing/glue-flow.png)

**Orchestration Flow**:

1. **EventBridge** kích hoạt hàng tuần vào lúc giữa đêm UTC+7 (Chủ nhật)
2. **Step Functions** điều phối workflow:
   - Khởi động Glue Crawler để cập nhật schema
   - Chờ crawler hoàn thành
   - Khởi động Glue ETL Job để chuyển hóa dữ liệu
   - Trả lỗi trạng thái (Nếu có)

![Step Functions Flow](/images/5-amplifyConfiguration/5.5-data-processing/stepfunctions_graph.png)

### Glue Components

**Database**: `weather_data_catalog_{unique-suffix}`

- Cataloging tất cả weather data schemas
- Tự động phát hiện table structures

**Crawler**: `WeatherPlatformCrawler-{unique-suffix}`

- Quét data lake cho dữ liệu mới
- Cập nhật table schemas tự động

**ETL Job**: `WeatherDataTransformJob-{unique-suffix}`

- Chuyển đổi JSON sang CSV format
- Aggregates data theo time periods
- Tối ưu hóa cho analytical queries

### ETL Script

Glue job sử dụng Python script (`weather-transform.py`) để:

1. **Đọc raw JSON** từ data lake
2. **Normalize data structure** trên các devices khác nhau
3. **Aggregate data** theo time windows
4. **Ghi CSV files** vào processed datasets bucket

### Step Functions Orchestration

Data processing workflow sử dụng AWS Step Functions cho orchestration đáng tin cậy:

**State Machine Workflow**:

```typescript
// 1. Start Crawler
startCrawler
  .next(waitForCrawler) // 2. Chờ 2 phút
  .next(checkCrawlerStatus) // 3. Kiểm tra crawler state
  .next(
    new stepfunctions.Choice(this, "IsCrawlerComplete")
      .when(crawlerComplete, startGlueJob) // 4a. Start ETL job
      .when(crawlerRunning, waitForCrawler) // 4b. Tiếp tục chờ
      .otherwise(crawlerFailed) // 4c. Xử lý failure
  );
```

### Scheduling

Data processing được kích hoạt qua EventBridge:

- **Frequency**: Hàng tuần vào lúc nửa đêm UTC+7 (Chủ nhật)
- **Trigger**: EventBridge cron rule
- **Target**: Step Functions state machine
- **Orchestration**: Automated Crawler → ETL Job sequence

### Monitoring

Glue jobs cung cấp:

- Execution logs trong CloudWatch
- Job success/failure notifications
- Data quality metrics
- Cost tracking

{{% notice tip %}}
Glue cung cấp serverless data processing - bạn chỉ trả tiền cho processing time được sử dụng.
{{% /notice %}}
