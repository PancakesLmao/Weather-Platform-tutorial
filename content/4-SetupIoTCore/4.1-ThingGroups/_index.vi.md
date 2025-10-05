---
title: "Cấu hình Thing Groups"
weight: 1
chapter: false
pre: " <b> 4.1 </b> "
---

AWS IoT Thing Groups cung cấp cơ chế quản lý nhiều thiết bị một cách tập trung. Tất cả các thiết bị thời tiết được tổ chức dưới một Thing Group trung tâm để đơn giản hóa việc quản lý policy và tổ chức thiết bị.

## Tạo Thing Group

1. Truy cập **AWS IoT Core** Console
2. Vào **Manage** → **Thing Groups**
3. Nhấn **Create thing group**
4. Chọn **Create static thing group**
5. Nhập tên Thing Group: `ITeaWeatherHub`
6. Tùy chọn thêm mô tả và tag fcj_workshop1 - FCJ Workshop 1 (Để quản lý tài nguyên)
7. Nhấn **Create thing group**

![Search IoT Core](/images/4-iotcore/1.png)
![Configure Thing Group](/images/4-iotcore/2.png)
![Create Thing Group](/images/4-iotcore/3.png)

{{% notice tip %}}
Thing Groups cho phép bạn áp dụng policies và quản lý nhiều thiết bị như một đơn vị duy nhất, giúp việc quản lý thiết bị dễ dàng hơn nhiều.
{{% /notice %}}

{{% notice info %}}
Thing Group hiện đã sẵn sàng để nhận các thiết bị trạm thời tiết. Khi các thiết bị được đăng ký qua nền tảng, chúng sẽ tự động được thêm vào nhóm này.
{{% /notice %}}
