---
title: "Cài đặt và Kiểm tra Project"
weight: 1
chapter: false
pre: " <b> 5.1 </b> "
---

## Clone Weather Platform Project

Bắt đầu bằng việc clone Weather Platform repository và khám phá cấu trúc của nó.

### Bước 1: Clone Repository

```bash
# Clone Weather Platform project
git clone https://github.com/Itea-Lab/Weather-Platform.git
cd Weather-Platform
```

### Bước 2: Cài đặt Dependencies

Project sử dụng **pnpm** làm package manager:

```bash
# Cài đặt dependencies với pnpm
pnpm install
```

### Bước 3: Kiểm tra Cấu trúc Project

Khám phá các thư mục chính:

```
Weather-Platform/
├── amplify/                    # Cấu hình Amplify backend
│   ├── backend.ts             # Định nghĩa backend chính
│   ├── auth/                  # Xác thực Cognito
│   ├── custom/                # Custom CDK constructs
│   │   ├── CloudFront/        # Cấu hình CDN
│   │   ├── EventBridge/       # Xử lý dữ liệu theo lịch
│   │   ├── GetDataset/        # Dataset API
│   │   ├── WeatherDataGlue/   # Xử lý ETL
│   │   └── WeatherDatasetStorage/ # S3 storage
│   └── functions/             # Lambda functions
│       ├── addThing/          # Thêm IoT devices
│       ├── deleteThing/       # Xóa IoT devices
│       ├── fetchThings/       # Liệt kê IoT devices
│       ├── getDataset/        # Truy xuất dữ liệu đã xử lý
│       ├── getIoTEndpoint/    # Lấy IoT endpoint URL
│       └── getTotalReadings/  # Lấy thống kê telemetry
├── src/                       # Next.js frontend
│   ├── app/                   # App Router pages
│   ├── components/            # React components
│   ├── contexts/              # React contexts
│   ├── hooks/                 # Custom hooks
│   ├── lib/                   # Utility libraries
│   └── types/                 # TypeScript type definitions
│   └── utils/                 # Utility functions
├── public/                    # Static assets
├── package.json               # Dependencies và scripts
└── amplify_outputs.json       # Được tạo sau deployment
```

### Bước 4: Hiểu kiến trúc Backend

File `amplify/backend.ts` định nghĩa:

**Dịch vụ chính:**

- **Authentication**: Cognito User Pool và Identity Pool
- **Storage**: Custom S3 bucket với lifecycle policies
- **Functions**: Lambda functions để quản lý IoT device
- **Data Processing**: AWS Glue cho ETL operations
- **Content Delivery**: CloudFront distribution

**Custom CDK Constructs:**

- **WeatherDatasetStorage**: Cấu hình S3 nâng cao
- **WeatherDataGlue**: ETL processing pipeline
- **CloudFront**: Phân phối nội dung toàn cầu
- **EventBridge**: Xử lý dữ liệu dịnh kỳ

### Bước 5: Environment Configuration

Tạo `.env.local` cho development:

```bash
# Copy environment template
cp .env.example .env.local

# Chỉnh sửa với giá trị cụ thể của bạn
# DEFAULT_REGION=us-east-1
```

{{% notice info %}}
File `amplify_outputs.json` được **tạo tự động** sau backend deployment và chứa tất cả cấu hình cần thiết cho frontend.
{{% /notice %}}

{{% notice warning %}}
Nhớ cập nhật tên bucket được hardcoded trong backend như đã đề cập trong phần Data Lake.
{{% /notice %}}
