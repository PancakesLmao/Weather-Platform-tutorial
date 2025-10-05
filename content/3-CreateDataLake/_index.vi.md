---
title: "Tạo Data Lake"
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

## S3 Data Lake cho IoT Messages

Weather Platform sử dụng Amazon S3 như một data lake đơn giản để lưu trữ IoT messages từ các cảm biến thời tiết. Đây là kho lưu trữ chính cho dữ liệu thô từ cảm biến.

### Data Lake là gì?

Data lake là một S3 bucket tên `itea-weather-data-lake-storage` lưu trữ:

- Raw IoT sensor messages (định dạng JSON)
- AWS Glue ETL scripts cho việc xử lý dữ liệu
### Tạo S3 Bucket
![Create Bucket](/images/3-datalake/1.png)
![Create Bucket](/images/3-datalake/2.png)
![Create Bucket](/images/3-datalake/3.png)
![Create Bucket](/images/3-datalake/4.png)
Bấm **Create bucket** để kết thúc.
### Cấu trúc Data Lake Bucket

Data lake được tổ chức với cấu trúc thư mục sau:

```
itea-weather-data-lake-storage/
├── raw-data/                   # Raw IoT sensor messages
│   └── weatherPlatform/        # Weather platform data
│       └── telemetry/          # Telemetry data được tổ chức theo vị trí
│           ├── {location-1}/   # ví dụ: "itea-lab-room-a"
│           │   └── sensor-data.json
│           ├── {location-2}/   # ví dụ: "outdoor-station-1"
│           │   └── sensor-data.json
│           └── {location-n}/   # Các vị trí bổ sung
│               └── sensor-data.json
└── glue-scripts/               # AWS Glue ETL scripts
    └── weather-transform.py    # Data transformation script
```

### Lưu trữ IoT Message

Khi các thiết bị IoT gửi dữ liệu thời tiết, messages được lưu trữ trong:

```
raw-data/weatherPlatform/telemetry/{location}/
```

Trong đó `{location}` là định danh vị trí cụ thể (ví dụ: `itea-lab-room-a`, `outdoor-station-1`).

### Ghi chú quan trọng

{{% notice warning %}}
**Quan trọng**: Tên S3 bucket phải là duy nhất toàn cầu trên tất cả các tài khoản AWS. Thêm tên hoặc timestamp của bạn để đảm bảo tính duy nhất: `itea-weather-data-lake-storage-yourname`

**Cần cập nhật Backend Code**: Code backend Amplify có hardcoded bucket names phải được cập nhật để khớp với tên bucket duy nhất của bạn.
{{% /notice %}}

{{% notice info %}}
Data lake bucket này tách biệt với processed dataset bucket. Raw IoT sensor data chảy vào bucket này, trong khi processed datasets được lưu trữ trong bucket khác.
{{% /notice %}}

### Cập nhật Backend Code

Sau khi tạo tên bucket duy nhất của bạn, bạn **phải** cập nhật các tham chiếu được hardcoded trong Amplify backend:

**Files cần cập nhật:**

1. **`amplify/backend.ts`** (Lines 233, 272, 273, 318):

   ```typescript
   // Thay đổi từ:
   sourceBucketName: "itea-weather-data-lake-storage";

   // Thành tên duy nhất của bạn:
   sourceBucketName: "itea-weather-data-lake-storage-yourname";
   ```

2. **`amplify/functions/getTotalReadings/handler.ts`** (Line 16):

   ```typescript
   // Thay đổi từ:
   const bucket = "itea-weather-data-lake-storage";

   // Thành tên duy nhất của bạn:
   const bucket = "itea-weather-data-lake-storage-yourname";
   ```

3. **`amplify/custom/WeatherDataGlue/resource.ts`** (Lines 65, 66, 195):

   ```typescript
   // Thay đổi tất cả S3 ARNs và paths từ:
   arn:aws:s3:::itea-weather-data-lake-storage
   s3://itea-weather-data-lake-storage/glue-scripts/

   // Thành tên duy nhất của bạn:
   arn:aws:s3:::itea-weather-data-lake-storage-yourname
   s3://itea-weather-data-lake-storage-yourname/glue-scripts/
   ```

{{% notice tip %}}
Sử dụng Find & Replace trong code editor của bạn để nhanh chóng cập nhật tất cả instances của `itea-weather-data-lake-storage` thành tên bucket duy nhất của bạn.
{{% /notice %}}

### Bước tiếp theo

Với S3 data lake đã cấu hình:

- Các thiết bị IoT gửi messages được lưu trữ trong `raw-data/weatherPlatform/telemetry/{location}/`
- AWS Glue scripts trong `glue-scripts/` xử lý raw data
- Kết quả được xử lý được lưu trữ trong dataset bucket riêng biệt
