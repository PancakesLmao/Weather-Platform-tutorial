---
title: "Giới thiệu"
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

## Bài toán thực tế

Các trạm thời tiết hiện tại tại ITea Lab phụ thuộc vào việc thu thập dữ liệu thủ công, điều này không hiệu quả khi quản lý nhiều thiết bị. Việc thiếu một hệ thống tập trung để quan sát dữ liệu theo thời gian thực và phân tích, kết hợp với chi phí cao và độ phức tạp của các nền tảng bên thứ ba, đã hạn chế khả năng nghiên cứu và hiệu quả hoạt động.

## Kiến trúc giải pháp

Nền tảng IoT Weather Platform giải quyết những thách thức này thông qua kiến trúc serverless của AWS. Các thiết bị biên Raspberry Pi, được trang bị module SEN0186, thu thập dữ liệu thời tiết và truyền qua MQTT tới AWS IoT Core. Dữ liệu được lưu trữ trong data lake Amazon S3, xử lý bằng AWS Glue cho cataloging và ETL jobs, và phân tích để tạo insights.

Ứng dụng web Next.js, được hosting trên AWS Amplify, cung cấp giao diện người dùng cho dashboard theo thời gian thực và phân tích xu hướng, với quyền truy cập được bảo mật bởi Amazon Cognito. Hệ thống hỗ trợ nhiều trạm thời tiết, có khả năng mở rộng lên 15 trạm, và duy trì chi phí vận hành thấp khoảng dưới $5 mỗi tháng.

![CloudArchitecture](/images/full-architecture.jpg)

## Tính năng chính

### Dashboard theo thời gian thực

- **Dữ liệu thời tiết trực tiếp**: Nhiệt độ, độ ẩm, áp suất, gió, lượng mưa
- **Nhiều hình thức hiển thị**: Cards, biểu đồ, và chỉ báo theo thời gian thực
- **Trạng thái kết nối**: Phản hồi trực quan cho tình trạng kết nối IoT
- **Giám sát thiết bị thông minh**: Giao diện phản hồi trong 3 giây, thông báo được gửi sau 30 giây offline

### Quản lý thiết bị IoT

- **Đăng ký thiết bị**: Đăng ký và quản lý các trạm thời tiết
- **Giám sát trạng thái**: Phát hiện offline tự động với thông báo
- **Hỗ trợ đa địa điểm**: Chuyển đổi giữa các trạm thời tiết một cách mượt mà

### Phân tích dữ liệu

- **Dữ liệu lịch sử**: Dữ liệu thời tiết với khả năng lọc và xuất
- **Xử lý tự động**: Pipeline ETL hàng ngày cho việc chuyển đổi dữ liệu
- **Phân phối toàn cầu**: CloudFront CDN cho việc truy cập dataset

### Công nghệ sử dụng

- **Frontend**: Next.js 15, React 19, TypeScript, Shadcn
- **Backend**: AWS Amplify Gen 2, Lambda functions
- **IoT**: AWS IoT Core với PubSub messaging
- **Storage**: Amazon S3 với CloudFront CDN
- **Database**: AWS Glue cho chuyển đổi dữ liệu
- **Authentication**: Amazon Cognito
- **Charts**: Recharts cho visualization dữ liệu
