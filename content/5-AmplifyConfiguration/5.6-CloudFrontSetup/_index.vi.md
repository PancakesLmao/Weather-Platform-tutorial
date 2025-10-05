---
title: "Cài đặt CloudFront"
weight: 6
chapter: false
pre: " <b> 5.6 </b> "
---

## CloudFront Content Delivery

CloudFront distribution được tự động cấu hình qua Custom CDK Construct.

### Tự động Deploy

CloudFront distribution được tạo khi bạn deploy Amplify backend với Custom CDK constructs.

### Tính năng chính

- **Global edge locations** cho phân phối nội dung nhanh
- **Origin Access Control (OAC)** cho S3 integration bảo mật
- **Cache optimization** cho dataset files
- **Automatic SSL certificates**

{{% notice info %}}
CloudFront distribution được cấu hình tự động trong `amplify/custom/CloudFront/resource.ts`. Xem [Section 5.3: Custom CDK Constructs](../5.3-customcdkresources/) để biết chi tiết về implementation.
{{% /notice %}}
