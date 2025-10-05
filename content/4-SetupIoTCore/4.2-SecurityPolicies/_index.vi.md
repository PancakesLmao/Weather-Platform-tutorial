---
title: "IoT Security Policies"
weight: 2
chapter: false
pre: " <b> 4.2 </b> "
---

Hai security policies được yêu cầu để vận hành nền tảng đúng cách: Device Policy cho các trạm thời tiết và Platform Policy cho quyền truy cập người dùng.

## Device Policy (WeatherStationPolicies)

Policy này cho phép các thiết bị thời tiết kết nối và publish dữ liệu telemetry tới IoT Core.

### Bước 1: Tạo Device Policy

1. Truy cập **AWS IoT Core** Console
2. Vào **Secure** → **Policies**
3. Nhấn **Create policy**
4. Nhập tên Policy: `WeatherStationPolicies`
5. Thêm tag (tùy chọn): fcj_workshop1 - FCJ Workshop 1
6. Copy và paste policy document sau:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iot:Publish",
      "Resource": "arn:aws:iot:<region>:<account_ID>:topic/weatherPlatform/telemetry/*"
    },
    {
      "Effect": "Allow",
      "Action": "iot:Connect",
      "Resource": "arn:aws:iot:<region>:<account_ID>:client/*"
    }
  ]
}
```

![Configure Policy](/images/4-iotcore/4.png)
![Create Policy](/images/4-iotcore/5.png) 7. Nhấn **Create**

### Bước 2: Attach Policy vào Thing Group

1. Truy cập IoT Core → **All devices** → **Thing Groups**
2. Chọn **ITeaWeatherHub** Thing Group
3. Vào tab **Policies** → **Manage Policies**
4. Nhấn **Attach policy**
5. Chọn **WeatherStationPolicies**
6. Nhấn **Attach**

![Configure Policy](/images/4-iotcore/6.png)
![Update Policy](/images/4-iotcore/7.png)

{{% notice info %}}
Policy này tự động áp dụng cho tất cả các thiết bị trong Thing Group.
{{% /notice %}}

## Platform Policy (WeatherPlatformPubSubPolicy)

Policy này cho phép web platform tương tác với IoT devices, nhận dữ liệu telemetry, và gửi thông báo.

### Bước 1: Tạo Platform Policy

1. Trong **AWS IoT Core** Console, vào **Secure** → **Policies**
2. Nhấn **Create policy**
3. Nhập tên Policy: `WeatherPlatformPubSubPolicy`
4. Thêm tag (tùy chọn): fcj_workshop1 - FCJ Workshop 1
5. Copy và paste policy document sau:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iot:Connect",
      "Resource": "arn:aws:iot:<region>:<account_ID>:client/*"
    },
    {
      "Effect": "Allow",
      "Action": "iot:Receive",
      "Resource": "arn:aws:iot:<region>:<account_ID>:topic/weatherPlatform/*"
    },
    {
      "Effect": "Allow",
      "Action": "iot:Subscribe",
      "Resource": [
        "arn:aws:iot:<region>:<account_ID>:topicfilter/weatherPlatform/telemetry/*",
        "arn:aws:iot:<region>:<account_ID>:topicfilter/weatherPlatform/notifications"
      ]
    },
    {
      "Effect": "Allow",
      "Action": "iot:Publish",
      "Resource": "arn:aws:iot:<region>:<account_ID>:topic/weatherPlatform/notifications"
    }
  ]
}
```

6. Nhấn **Create**

{{% notice warning %}}
Policy này được attach vào từng Cognito Identity ID để cấp quyền truy cập IoT Core resources cho người dùng đã xác thực. Xem phần User IoT Policy Attachment để biết các bước triển khai chi tiết.
{{% /notice %}}

## Xác minh

Sau khi tạo cả hai policies, bạn sẽ thấy:

- **WeatherStationPolicies**: Đã attach vào ITeaWeatherHub Thing Group
- **WeatherPlatformPubSubPolicy**: Sẵn sàng để gán cho người dùng

## Các bước tiếp theo

- Hoàn thành triển khai Amplify backend
- Thiết lập xác thực người dùng và gán policy
