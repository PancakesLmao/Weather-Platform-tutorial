---
title: "IoT Rule Engine"
weight: 3
chapter: false
pre: " <b> 4.3 </b> "
---

## IoT Rule Engine cho S3 Data Lake

IoT Rule Engine định tuyến các thông điệp telemetry thời tiết từ các thiết bị IoT trực tiếp đến S3 data lake để lưu trữ và xử lý.

### Mục đích của Rule

Rule này tự động:

- Bắt tất cả thông điệp telemetry thời tiết từ các topic `weatherPlatform/telemetry/*`
- Định tuyến thông điệp đến S3 data lake bucket (`itea-weather-data-lake-storage`)
- Tổ chức dữ liệu theo vị trí trong cấu trúc thư mục chính xác

### Tạo IoT Rule

1. **Truy cập AWS IoT Core Console**
2. **Vào Message routing** → **Rules**
3. **Nhấn "Create rule"**
4. **Cấu hình rule settings**:

   - **Rule name**: `WeatherTelemetryToS3Rule`
   - **Description**: `Route weather telemetry data to S3 data lake`
   - **Tag**: `fcj_workshop1` - `FCJ Workshop 1`

5. **SQL statement**:

   ```sql
   SELECT * FROM 'weatherPlatform/telemetry/+'
   ```

6. **Rule actions** → **Add action**:

   - **Action type**: **S3**
   - **S3 bucket**: `itea-weather-data-lake-storage-yourname` (sử dụng tên bucket duy nhất của bạn)
   - **Key**: `raw-data/weatherPlatform/telemetry/${location}/${timestamp()}.json`

7. **Create or select IAM role**:

   - **Role name**: `IoTRuleToS3Role`
   - **Attach policy**: Cho phép quyền S3 PutObject
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::itea-weather-data-lake-storage-yourname/raw-data/*"
        }
    ]
}
```
![Rule Engine](/images/4-iotcore/9.png)
![Rule Config](/images/4-iotcore/10.png)
![Rule Config](/images/4-iotcore/11.png)
![Rule Config](/images/4-iotcore/12.png)
![Rule Config](/images/4-iotcore/13.png)

8. **Nhấn "Create rule"**

### Giải thích SQL Statement

```sql
SELECT * FROM 'weatherPlatform/telemetry/+'
```

- `SELECT *`: Bắt toàn bộ message payload
- `FROM 'weatherPlatform/telemetry/+'`: Match tất cả các topic dưới telemetry (+ là wildcard)

### S3 Key Pattern

```
raw-data/weatherPlatform/telemetry/${location}/${timestamp()}.json
```

**Cấu trúc kết quả**:

```
itea-weather-data-lake-storage-yourname/
└── raw-data/
    └── weatherPlatform/
        └── telemetry/
            ├── itea-lab-room-a/
            │   ├── 1696118400000.json
            │   └── 1696118460000.json
            └── outdoor-station-1/
                ├── 1696118400000.json
                └── 1696118460000.json
```

### Kiểm tra Rule

1. **Sử dụng IoT Core Message Client**:

   - **Topic**: `weatherPlatform/telemetry/test-location`
   - **Message**:
     ```json
     {
       "deviceId": "test-device-01",
       "timestamp": "2024-01-15T10:30:00Z",
       "temperature": 25.5,
       "humidity": 60.2,
       "location": "test-location"
     }
     ```
![Test routing](/images/4-iotcore/14.png)
![Test routing](/images/4-iotcore/15.png)

2. **Kiểm tra S3 bucket** cho thông điệp đã lưu tại:
   ```
   raw-data/weatherPlatform/telemetry/test-location/{timestamp}.json
   ```
![Test routing](/images/4-iotcore/16.png)

{{% notice warning %}}
**Quan trọng**: Thay thế `itea-weather-data-lake-storage-yourname` bằng tên bucket duy nhất của bạn trong cả S3 action và IAM policy.
{{% /notice %}}

{{% notice info %}}
Rule này đảm bảo tất cả thông điệp từ thiết bị thời tiết được tự động lưu trữ trong data lake, sẵn sàng cho AWS Glue processing.
{{% /notice %}}

### Xác minh

Sau khi tạo rule, xác minh:

- Rule đã **Enabled**
- SQL statement syntax chính xác
- S3 action trỏ đến tên bucket duy nhất
- IAM role có quyền S3 phù hợp

IoT Rule Engine hiện tự động định tuyến tất cả weather telemetry đến S3 data lake của bạn để lưu trữ và xử lý tiếp theo.
