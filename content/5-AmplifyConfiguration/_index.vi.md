---
title: "Cấu hình Amplify"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

## Cài đặt AWS Amplify Backend

Phần này bao gồm việc clone Weather Platform project, hiểu kiến trúc của nó, và triển khai Amplify backend với cấu hình chính xác.

### Tổng quan

Weather Platform sử dụng AWS Amplify Gen 2 với cấu hình backend tùy chỉnh bao gồm:

- Next.js 15 frontend với App Router
- Custom CDK constructs cho các dịch vụ AWS nâng cao
- Lambda functions để quản lý IoT device
- AWS Glue để xử lý dữ liệu
- CloudFront để phân phối nội dung toàn cầu

### Những gì bạn cần làm

Trong phần này, bạn sẽ:

- Clone Weather Platform repository
- Kiểm tra cấu trúc project và cấu hình Amplify backend
- Thiết lập môi trường development với pnpm
- Triển khai backend sử dụng AWS Amplify với profile `ws1-amplify`
- Hiểu cách custom CDK constructs tích hợp với Amplify

{{% notice info %}}
Phần này sử dụng AWS profile `ws1-amplify`. Đảm bảo profile của bạn đã được cấu hình trước khi tiếp tục.
{{% /notice %}}

### Yêu cầu trước

Trước khi bắt đầu, đảm bảo bạn có:

- AWS CLI được thiết lập profile `ws1-amplify`
- Node.js 18+ đã cài đặt
- pnpm package manager đã cài đặt
- Git để clone repository

### Cấu trúc phần

- **[5.1: Project Setup và Inspection](5.1-projectsetup/)** - Clone repository và khám phá cấu trúc project
- **[5.2: Cấu hình Amplify Backend](5.2-backendconfiguration/)** - Deploy sandbox với profile `ws1-amplify`
- **[5.3: Tổng quan Custom CDK Constructs](5.3-customcdkresources/)** - Hiểu về các tích hợp AWS nâng cao
- **[5.4: Cài đặt Lambda Functions](5.4-lambdafunctions/)** - Quản lý IoT device và data APIs
- **[5.5: Data Processing Pipeline](5.5-dataprocessing/)** - AWS Glue ETL automation
- **[5.6: Cài đặt Content Distribution](5.6-cloudfrontsetup/)** - CloudFront global delivery
- **[5.7: Xác thực Người dùng và Quản lý IoT Policy](5.7-authentication/)** - Cognito và IoT policies
- **[5.8: Triển khai lên production](5.8-productiondeployment/)** - Triển khai web lên Amplify

{{% notice info %}}
Amplify backend chạy CloudFormation ở phía sau, giúp triển khai sandbox infrastructure nhanh chóng và dễ dàng tích hợp với các project hiện có. Môi trường sandbox cho phép phát triển và kiểm tra nhanh trước khi triển khai production.
{{% /notice %}}

{{% notice info %}}
**Development Cleanup**: Đối với môi trường sandbox, bạn có thể dễ dàng xóa tất cả resources với một lệnh duy nhất: `npx ampx sandbox delete --profile ws1-amplify`. Xem [Phần 6: Resource Cleanup](../6-resourcecleanup/) để biết thủ tục cleanup hoàn chỉnh.
{{% /notice %}}
