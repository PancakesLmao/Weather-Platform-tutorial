---
title: "Yêu cầu & Thiết lập"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

## Thiết lập môi trường phát triển

### Phần mềm yêu cầu

Trước khi bắt đầu, hãy đảm bảo bạn đã cài đặt những phần mềm sau trên thiết bị của mình (để testing và debugging):

- **[Node.js 18 or higher](https://nodejs.org/)** - Tải từ nodejs.org
- **pnpm package manager** - Cài với lệnh `npm install -g pnpm`
- **[AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)** - Installation guide
- **[Git](https://git-scm.com/downloads)** - Để clone repo về máy

### Xác minh cài đặt

```bash
# Kiểm tra phiên bản Node.js
node --version  # Nên là 18+

# Kiểm tra pnpm
pnpm --version

# Kiểm tra AWS CLI
aws --version

# Kiểm tra Git
git --version
```

## Nội dung

- [Thiết lập thiết bị biên](2.1-setupedge/)
- [Thiết lập môi trường Cloud](2.2-setupcloud/)
