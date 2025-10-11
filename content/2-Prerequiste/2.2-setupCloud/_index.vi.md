---
title: "Thiết lập môi trường Cloud"
weight: 2
chapter: false
pre: " <b> 2.2 </b> "
---

Ở bước này, chúng ta sẽ thiết lập môi trường để tương tác với AWS resources từ terminal của developer sử dụng AWS CLI
![Cloud Architecture](/images/cloud-arch.jpg)
## Thiết lập AWS Account

### Bước 1: Yêu cầu AWS Account

- Trước tiên bạn cần một AWS Account với các quyền phù hợp
- Cấp quyền để tạo IAM users và policies
- Cấu hình cảnh báo billing (khuyến nghị)

### Bước 2: Tạo IAM User cho Development

1. **Vào AWS Console** IAM Users "Create user"

2. **Chi tiết User:**

   - Username: `ws1-amplify`
   - Chọn "Provide user access to the AWS Management Console" nếu cần

3. **Cài đặt permissions:**

   - Chọn "Attach policies directly"
   - Thêm các managed policies sau:
     - `AmplifyBackendDeployFullAccess` (để deploy Amplify backend resources)
     - `AWSCloudFormationReadOnlyAccess` (để kiểm tra resource creation và debug stack builds)
     - `IAMFullAccess` (để quản lý IAM policies và xác minh permissions)
     - `AWSIoTFullAccess` (cho IoT Core interactions qua CLI và attach IoT policies)

4. **Tạo Access Keys:**
   - Vào user Security credentials "Create access key"
   - Chọn "Command Line Interface (CLI)"
   - Lưu **Access Key ID** và **Secret Access Key**

{{% notice warning %}}
Lưu trữ AWS credentials của bạn an toàn và không bao giờ commit chúng vào version control!
{{% /notice %}}

### Bước 3: Cấu hình AWS CLI Profile

Thiết lập một named profile cho project này:

```bash
# Cấu hình AWS CLI với profile của bạn
aws configure --profile weather-platform

# Nhập khi được yêu cầu:
# AWS Access Key ID: [your-access-key-id]
# AWS Secret Access Key: [your-secret-access-key]
# Default region name: us-east-1 (hoặc region bạn muốn)
# Default output format: json
```

Xác minh profile:

```bash
aws sts get-caller-identity --profile weather-platform
```
