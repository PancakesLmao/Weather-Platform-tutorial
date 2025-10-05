---
title: "Xác thực và Quản lý IoT Policy"
weight: 7
chapter: false
pre: " <b> 5.7 </b> "
---

## User Authentication và IoT Policy Management

Cognito authentication và IoT policy attachment được cấu hình trong quá trình backend deployment.

### Các bước chính

1. **Tạo user account** qua frontend application
2. **Thêm user vào `platform-admin` group** trong Cognito
3. **Lấy Cognito Identity ID** của user từ Identity Pool
4. **Gắn IoT Policy** vào Identity ID cụ thể

### Chi tiết Implementation

Chi tiết đầy đủ về user setup và IoT policy attachment được đề cập trong:

- [Section 5.2: Backend Configuration - Bước 7](../5.2-backendconfiguration/)

{{% notice warning %}}
**Quan trọng**: Mỗi user cần IoT policy được gắn thủ công vào Cognito Identity ID của họ để truy cập IoT Core resources và nhận real-time telemetry data.
{{% /notice %}}

{{% notice info %}}
Đây là giới hạn của AWS IoT Core - policies không thể được gắn trực tiếp vào Identity Pool IDs, chỉ có thể gắn vào từng Identity ID cá nhân.
{{% /notice %}}
