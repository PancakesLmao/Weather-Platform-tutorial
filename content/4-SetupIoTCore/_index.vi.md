---
title: "Cấu hình AWS IoT Core"
weight: 4
chapter: false
pre: " <b> 4. </b> "
---

## Cấu hình AWS IoT Core

AWS IoT Core đóng vai trò là trung tâm kết nối thiết bị và định tuyến message trong Weather Platform. Phần này bao gồm các thiết lập cần thiết trước khi triển khai Amplify backend.

### Những gì bạn sẽ cấu hình

- **Thing Groups**: Tổ chức các thiết bị thời tiết theo nhóm
- **IoT Security Policies**: Định nghĩa quyền truy cập cho thiết bị và nền tảng

{{% notice info %}}
Các thành phần IoT Core này phải được cấu hình thủ công trước khi triển khai Amplify backend để đảm bảo quản lý thiết bị đúng cách.
{{% /notice %}}
