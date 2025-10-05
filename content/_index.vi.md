---
title: "Xây dựng nền tảng IoT tập trung cho trạm thời tiết"
weight: 1
chapter: false
---

# Xây dựng nền tảng IoT tập trung cho trạm thời tiết

### Tổng quan

Workshop này giới thiệu IoT Weather Platform, một giải pháp serverless dựa trên AWS được phát triển cho ITea Lab tại Thành phố Hồ Chí Minh nhằm nâng cao việc thu thập và phân tích dữ liệu thời tiết cho mục đích nghiên cứu. Tutorial được thiết kế cho các thành viên lab và người tham gia cộng đồng để hiểu và triển khai nền tảng một cách có hệ thống.

Nền tảng được hỗ trợ bởi các công cụ hiện đại như Next.js 15, AWS Amplify Gen 2, IoT Core, và một data pipeline toàn diện hỗ trợ cả giám sát thời tiết real-time và phân tích dữ liệu lịch sử.

![Overview](/images/arc-log.png)

### Nội dung

1. [Giới thiệu](1-introduce/)
2. [Yêu cầu trước & Cài đặt](2-prerequiste/)
3. [Tạo Data Lake](3-createdatalake/)
4. [Cài đặt AWS IoT Core](4-setupiotcore/)
5. [Cấu hình Amplify](5-amplifyconfiguration/)
6. [Dọn dẹp Resources](6-resourcecleanup/)
7. [Kế hoạch Chi phí](7-pricingplan/)

### Tổng quan Kiến trúc

Weather Platform cung cấp:

- **IoT Device Management**: Đăng ký và quản lý weather sensors
- **Thu thập Dữ liệu Real-time**: Stream weather data qua MQTT
- **Data Lake Storage**: Lưu trữ raw sensor data trong S3
- **ETL Processing**: Chuyển đổi raw data với AWS Glue
- **Global Distribution**: Phục vụ datasets qua CloudFront
- **Web Dashboard**: Next.js 15 application để hiển thị dữ liệu

### Công nghệ Chính

- **Frontend**: Next.js 15 với App Router, TypeScript, Shadcn
- **Backend**: AWS Amplify Gen 2 với custom CDK constructs
- **IoT**: AWS IoT Core với MQTT protocol
- **Data Processing**: AWS Glue cho ETL operations
- **Storage**: Amazon S3 cho data lake và processed datasets
- **CDN**: CloudFront cho global content delivery
- **Authentication**: AWS Cognito User Pools
