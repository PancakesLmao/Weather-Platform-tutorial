---
title: "Thiết lập Raspberry Pi"
weight: 2
chapter: false
pre: " <b> 2.1.2 </b> "
---

Trong phần này, chúng ta sẽ cấu hình Raspberry Pi để hoạt động như một device gateway thu thập dữ liệu từ ESP32 và chuyển tiếp tới AWS IoT Core.

### Yêu cầu

{{% notice info %}}
Raspberry Pi phải là model 3B trở lên để chạy source code một cách hiệu quả.
{{% /notice %}}

### Bước 1: Cài đặt Raspberry Pi OS

1. Mở **Raspberry Pi Imager**, cắm thẻ MicroSD và **flash OS**. Bạn có thể dùng bất kỳ OS nào, mình đang dùng Ubuntu Server, nhưng nếu bạn là người mới, chưa biết gì về Linux hay Raspberry Pi, mình khuyên dùng **Raspberry Pi OS (RaspiOS)**, nó đơn giản và dễ thao tác, kèm 1 lượng lớn tài liệu/blog hỗ trợ.
   {{% notice warning %}}
   Khi bạn chọn storage device, hãy đảm bảo chọn đúng vì tất cả dữ liệu trên thiết bị được chọn sẽ bị xóa.
   {{% /notice %}}

2. **Enable SSH**: Nhớ tíck vào ô cho tùy chọn này
3. **Configure Wifi**: Nhập SSID và password để Pi có thể kết nối internet ngay lần đầu khởi động
4. **Set locale settings**: Thiết lập quốc gia, ngôn ngữ và timezone
5. Nhấp **Save** rồi chọn **Yes** để flash OS lên thẻ MicroSD
6. Hoàn tất xong, tháo thẻ MicroSD và cắm vào Raspberry Pi. Kết nối nguồn điện để khởi động

![Imager Config](/images/2.prerequisite/2.1-setupEdge/imager1.png)
![Imager Config](/images/2.prerequisite/2.1-setupEdge/imager2.png)
![Imager Config](/images/2.prerequisite/2.1-setupEdge/imager3.png)
![Imager Config](/images/2.prerequisite/2.1-setupEdge/imager4.png)
![Imager Config](/images/2.prerequisite/2.1-setupEdge/imager5.png)

### Bước 2: Thiết lập ban đầu

1. **Khởi động Raspberry Pi** và kết nối qua SSH:

   ```bash
   ping -4 raspberrypi.local
   ```

   Nếu nó trả về IPv4 address thì ok. Nếu không thì cấu hình wifi ở bước trước chắc sai ở đâu đó rồi.  
   Có thể do SSID hoặc password không hợp lệ. Bạn có thể:

   1. Flash OS lại
   2. Dùng cáp LAN kết nối với laptop/PC và đặt IP

   ```bash
   ssh raspberrypi@<raspberry-pi-ip>
   ```

2. **Cập nhật hệ thống**:

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

3. **Cài đặt phần mềm/công cụ yêu cầu**:

- Git
- [Docker](https://docs.docker.com/engine/install/debian/)
- [Nginx](https://techworldthink.github.io/Tech-Guides/pages/nginx_pi.html)
- [Tailscale](https://tailscale.com/kb/1031/install-linux) (Hoặc bạn có thể dùng WireGuard)

### Bước 3: Thiết lập VPN với Tailscale trên Laptop và Raspberry Pi

VPN cho phép bạn truy cập Raspberry Pi như thể bạn đang ở cùng mạng local—ngay cả từ bất kỳ đâu trên thế giới.
Có một vài tùy chọn VPN tốt, nhưng mình xài Tailscale vì nó nhanh và cực kỳ dễ thiết lập. Tailscale là VPN được xây dựng trên WireGuard tự động thiết lập mạng mesh bảo mật giữa các thiết bị.  

1. Vào https://tailscale.com/ và tạo tài khoản > Đăng ký bằng Google / Github (chọn tùy chọn của bạn)
2. Cài đặt Tailscale client trên thiết bị từ https://tailscale.com/download  
   Hoàn tất xong, biểu tượng Tailscale sẽ xuất hiện trong system tray  
   Nhấp chuột phải vào biểu tượng > Đăng nhập bằng thông tin đăng nhập bạn dùng để tạo tài khoản  
   Tailscale sẽ yêu cầu bạn authorize thiết bị để join tailnet VPN  
   Nhấp **Connect** > Sau đó bạn sẽ có thể xem thiết bị trên Machines dashboard.
3. Nếu bạn đã enable SSH trên Raspberry Pi, chuyển sang bước 4. Nếu chưa, làm theo các bước này.  
   Chạy `sudo raspi-config` để mở Raspberry Pi Configuration Tool Interface  
   Chọn **Interface Options** > SSH > Yes > Ok
4. Để cài đặt Tailscale client trên Raspberry Pi, dùng:

```
sudo apt update
sudo apt upgrade -y
curl -fsSL https://tailscale.com/install.sh | sh
```

Sau đó khởi động Tailscale bằng:

```
sudo tailscale up
```

Bạn có thể tìm Tailscale IPv4 address bằng cách chạy:

```
tailscale ip -4
```

5. Test kết nối

```
ssh <pi_username>@<tailscale_ip>
```

hoặc
Enable SSH qua Tailscale bằng:

```
sudo tailscale up --ssh
```

Điều này cho phép bạn SSH Pi trực tiếp từ browser  
Vào admin console > Nhấp **Menu option** > SSH to machine
Nó sẽ yêu cầu thông tin đăng nhập > SSH > Tailscale sẽ mở 1 tab để chạy SSH

### Bước 4: Cấu hình Edge station

```bash
git clone https://github.com/Itea-Lab/Weather-Edge.git
cd Weather-Edge
sudo nano .env
```

Dán file .env mẫu với thông tin đăng nhập rồi chạy

```bash
docker compose up --build -d
```

Bạn sẽ cần mở giao diện của InfluxDB trên browser laptop với `<raspi_tailscale_ip>:8086`, đăng nhập và lấy token
Khi token được tạo, copy và paste vào file `.env`

```
INFLUXDB_TOKEN=<your_token_here>
```

File `.env` hoàn chỉnh sẽ như thế này:

```
# InfluxDB Configuration
INFLUXDB_USERNAME=<username>
INFLUXDB_PASSWORD=your_password
INFLUXDB_ORG=weather_org
INFLUXDB_BUCKET=weather_data
# For Docker containers, use service name
INFLUXDB_ROUTE=http://database:8086
# Token will be obtained after InfluxDB initialization (see step 6)
INFLUXDB_TOKEN=<your_token_here>

# Data Configuration
DATA_LOCATION=<your_location_name> (district1, district2, etc)
MEASUREMENT_NAME=weather_sensor

# MQTT Configuration
MQTT_PUB=espclient
SUB_USERNAME=raspiclient
MQTT_PASSWORD=your_mqtt_password
MQTT_TOPIC=weather/data
MQTT_PORT=1883
BROKER_ENDPOINT=mosquitto

# Dashboard Configuration
JWT_SECRET=your-super-secret-jwt-key-here-make-it-long-and-random
JWT_EXPIRES_IN=1d
ADMIN_USERNAME=admin
ADMIN_NAME=Administrator
ADMIN_PASSWORD_HASH=your_password_hash_here
ADMIN_EMAIL=admin@example.com
ADMIN_ROLE=admin
TEST_USERNAME=testuser
TEST_NAME=Test User
TEST_PASSWORD_HASH=your_password_hash_here
TEST_EMAIL=test@example.com
TEST_ROLE=user
```

### Bước 5: Cấu hình Device Certificates

{{% notice warning %}}
Quay lại bước này sau khi bạn hoàn thành [thiết lập môi trường Sandbox](/5-amplifyconfiguration/5.2-backendconfiguration)
{{% /notice %}}

1. **Đăng ký thiết bị** trong Weather Platform dashboard
2. **Tải certificates** (certificate.pem.crt, private.pem.key, AmazonRootCA1.pem)
3. **Cập nhật code** với certificates và IoT endpoint của bạn

Bạn phải chèn tất cả credential files vào folder `/data-processor/credentials/` để publish dữ liệu lên weather platform

### Bước 5: Test thiết lập

1. **Chạy edge station lại**:

   ```bash
   docker compose up --build -d
   ```

2. **Xác minh truyền dữ liệu** từ ES32 tới Raspberry Pi bằng cách kiểm tra container status và logs
   ```bash
   docker compose ps
   docker compose logs weather-edge-processor
   ```
   **Xác minh dữ liệu** xuất hiện trong dashboard

{{% notice tip %}}
Đảm bảo Raspberry Pi có kết nối internet ổn định để truyền dữ liệu đáng tin cậy.
{{% /notice %}}
