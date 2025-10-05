---
title: "Cài đặt Lambda Functions"
weight: 4
chapter: false
pre: " <b> 5.4 </b> "
---

## Tổng quan Lambda Functions

Weather Platform sử dụng nhiều Lambda functions để quản lý IoT devices và cung cấp APIs để truy cập dữ liệu.

### Tổng quan Functions

**IoT Device Management:**

- `addThing`: Đăng ký weather devices mới
- `deleteThing`: Xóa weather devices
- `fetchThings`: Liệt kê tất cả devices đã đăng ký
- `getIoTEndpoint`: Lấy IoT Core endpoint URL

**Data Access:**

- `getDataset`: Truy xuất processed weather datasets
- `getTotalReadings`: Lấy thống kê telemetry

### Chi tiết Key Functions

#### addThing Function

**Mục đích**: Đăng ký IoT weather devices mới
**Trigger**: API Gateway (POST)
**Chức năng**:

- Tạo IoT Thing trong AWS IoT Core
- Thêm device vào ITeaWeatherHub Thing Group
- Gắn WeatherStationPolicies
- Trả về device credentials

#### getTotalReadings Function

**Mục đích**: Lấy thống kê từ data lake
**Tích hợp**: Kết nối với `itea-weather-data-lake-storage`
**Chức năng**:

- Quét S3 bucket cho telemetry files
- Đếm tổng readings theo location
- Trả về aggregated statistics

#### getDataset Function

**Mục đích**: Truy xuất processed weather datasets
**Tích hợp**: Kết nối với processed data bucket
**Chức năng**:

- Liệt kê available datasets
- Cung cấp download URLs
- Xử lý CloudFront integration

### Deployment và Configuration

Lambda functions được tự động deploy với Amplify backend:

```bash
# Functions deploy với sandbox
npx ampx sandbox --profile ws1-amplify
```

### Environment Variables

Functions tự động nhận:

- IoT Core endpoint URL
- S3 bucket names
- Cognito User Pool information
- AWS region configuration

{{% notice info %}}
Lambda functions được tích hợp hoàn toàn với hệ thống authentication và authorization của Amplify.
{{% /notice %}}
