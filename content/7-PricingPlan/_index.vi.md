---
title: "Kế hoạch Chi phí"
weight: 7
chapter: false
pre: " <b> 7. </b> "
---

## Phân tích Chi phí Weather Platform

Hiểu rõ cấu trúc chi phí của Weather Platform giúp lập kế hoạch ngân sách và quyết định tối ưu hóa.

### Tổng quan Chi phí

| Thời kỳ        | Chi phí (USD) |
| -------------- | ------------- |
| **Hàng tháng** | **$0.74**     |
| **Hàng năm**   | **$8.88**     |
| **Trả trước**  | **$0.00**     |

{{% notice info %}}
Ước tính chi phí dựa trên AWS Pricing Calculator tính đến ngày 1 tháng 10, 2025. Chi phí thực tế có thể khác nhau tùy thuộc vào mẫu sử dụng và thay đổi giá AWS.
{{% /notice %}}

### Phân tích Chi phí theo Dịch vụ

#### Dịch vụ Nền tảng Chính

**AWS Amplify - $0.19/month ($2.28/year)**

- **Sử dụng**: Web app hosting và CI/CD pipeline
- **Build time**: 5 phút mỗi tháng
- **Instance**: Standard (8 GB Memory, 4 vCPUs)
- **Storage**: 0.03 GB data stored
- **SSR requests**: 20 mỗi ngày
- **Data served**: 0.003 GB mỗi tháng

**AWS Glue ETL - $0.36/month ($4.32/year)**

- **Sử dụng**: Xử lý và chuyển đổi dữ liệu
- **Apache Spark DPUs**: 2 units
- **Python Shell DPUs**: 0.0625 units
- **Processing**: Daily ETL jobs cho weather data

#### Lưu trữ & Xử lý Dữ liệu

**AWS Glue Crawlers - $0.07/month ($0.84/year)**

- **Sử dụng**: Schema discovery tự động
- **Crawlers**: 1 crawler cho data lake

**AWS Glue Data Catalog - $0.03/month ($0.36/year)**

- **Sử dụng**: Lưu trữ metadata và truy cập
- **Requests**: 3,000 access requests mỗi tháng
- **Objects**: 3,000 objects stored mỗi tháng

**S3 Storage - $0.05/month ($0.60/year tổng)**

- **Data Lake (US East N. Virginia)**: $0.04/month
  - Storage: 1 GB (Standard class)
  - Requests: 1,000 PUT/COPY/POST, 20,000 GET requests
- **Dataset (US East Ohio)**: $0.01/month
  - Storage: 0.5 GB (Standard class)
  - Requests: 1,000 PUT/COPY/POST, 10,000 GET requests

#### Phân phối Nội dung & IoT

**Amazon CloudFront - $0.02/month ($0.24/year)**

- **Sử dụng**: Phân phối nội dung toàn cầu cho datasets
- **HTTPS requests**: 20,000 mỗi tháng
- **Data transfer**: 0.01 GB mỗi tháng

**AWS IoT Core (MQTT) - $0.01/month ($0.12/year)**

- **Sử dụng**: Kết nối weather sensor
- **Devices**: 3 IoT devices
- **Messages**: 1 message mỗi device
- **Message size**: 430 bytes trung bình
- **Rules**: 1 rule triggered mỗi message

#### Compute & Xác thực

**AWS Lambda - $0.01/month ($0.12/year)**

- **Sử dụng**: Backend API functions
- **Architecture**: x86
- **Requests**: 1,000 mỗi tháng
- **Ephemeral storage**: 512 MB

**Amazon Cognito - $0.00/month (Free Tier)**

- **Sử dụng**: Xác thực và ủy quyền người dùng
- **Monthly Active Users**: 4 users
- **Advanced security**: Disabled (basic security included)

**AWS Step Functions - $0.00/month (Free Tier)**

- **Sử dụng**: Orchestration update dataset
- **Workflow requests**: 4 mỗi tháng
- **State transitions**: 70 mỗi workflow

### Chiến lược Tối ưu hóa Chi phí

#### Development vs Production

**Development (Sandbox)**:

- Sử dụng `npx ampx sandbox` cho development
- Tự động shutdown khi không sử dụng
- Chi phí xử lý dữ liệu tối thiểu

**Production**:

- Môi trường luôn sẵn sàng cho users
- Chi phí có thể dự đoán và ổn định
- Tối ưu cho độ tin cậy

#### Chi phí có thể Mở rộng vs Cố định

**Chi phí mở rộng theo:**

- Số lượng IoT devices và tần suất message
- Khối lượng xử lý dữ liệu (Glue ETL runtime)
- CloudFront requests và data transfer
- Tăng trưởng storage theo thời gian

**Chi phí cố định:**

- Amplify hosting (trừ khi build time tăng)
- Các thành phần infrastructure cơ bản

### Sự khác biệt Chi phí theo Vùng

**US East (N. Virginia)** - Vùng chính:

- Hầu hết các dịch vụ được host tại đây
- Thường có giá AWS thấp nhất
- Tổng chi phí vùng: ~$0.73/month

**US East (Ohio)** - Vùng phụ:

- Chỉ lưu trữ dataset
- Giá S3 khác biệt nhẹ
- Tổng chi phí vùng: ~$0.01/month

#### Ngân sách Hàng năm

**Quy mô hiện tại**: $10/year

- Môi trường lab nhỏ
- 3-4 trạm thời tiết
- Hoạt động phát triển thường xuyên

**Triển khai mở rộng**: $30-50/year

- 10-20 trạm thời tiết
- Tăng xử lý dữ liệu
- Hoạt động phát triển bổ sung
- Workload production

### Giám sát Chi phí

#### Cost Tags

**Tag tất cả resources để tracking:**

- Project: WeatherPlatform
- Environment: Development/Production
- Owner: ITea-Lab
- Purpose: Research

### Tuyên bố Pháp lý

{{% notice warning %}}
AWS Pricing Calculator chỉ cung cấp ước tính chi phí AWS và không bao gồm bất kỳ thuế nào có thể áp dụng. Chi phí thực tế của bạn phụ thuộc vào nhiều yếu tố khác nhau, bao gồm mức sử dụng thực tế các dịch vụ AWS.
{{% /notice %}}

**Chi tiết ước tính chi phí:**

- **Currency**: USD
- **Locale**: en_US
- **Created**: October 1, 2025
- **Calculator URL**: [https://calculator.aws/#/estimate?id=76f9b4fdb1e176538adcbca35db218885f9e8a6c](https://calculator.aws/#/estimate?id=76f9b4fdb1e176538adcbca35db218885f9e8a6c)

#### Cảnh báo Billing

**Ngưỡng billing alarm được đề xuất:**

- **Conservative**: $1.00/month (cảnh báo trước chi phí bất ngờ)
- **Quy mô hiện tại**: $5.00/month (buffer cho hoạt động phát triển)
- **Quy mô production**: $10.00/month (phù hợp với scaling)

### Giá trị Đề xuất

Với chi phí dưới $0.74/month (~$8.88/year), bạn nhận được:

- **Nền tảng giám sát thời tiết IoT hoàn chỉnh**
- **Xử lý và hiển thị dữ liệu real-time**
- **Kiến trúc serverless có khả năng mở rộng**
- **Bảo mật cấp doanh nghiệp (Cognito)**
- **Phân phối nội dung toàn cầu (CloudFront)**
- **Automated ETL pipelines**
- **Môi trường phát triển chuyên nghiệp**

**So sánh Chi phí:**

- Traditional hosting: $50-100/month
- Weather Platform: **$0.74/month**
- **Tiết kiệm**: 98%+ giảm chi phí

**Có đáng không?** Chắc chắn rồi! Với giá của một thanh kẹo, bạn có được một nền tảng IoT cấp chuyên nghiệp mà thông thường sẽ tốn hàng trăm đô la mỗi tháng.
