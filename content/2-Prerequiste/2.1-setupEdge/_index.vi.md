---
title: "Thiết lập thiết bị biên"
weight: 1
chapter: false
pre: " <b> 2.1 </b> "
---

Trong bước này, chúng ta sẽ thiết lập các thiết bị biên thu thập dữ liệu thời tiết và truyền lên AWS cloud. Weather Platform hỗ trợ nhiều cấu hình phần cứng, từ ESP32 đơn giản đến cài đặt Raspberry Pi đầy đủ.

{{% notice info %}}
Thiết bị biên là tùy chọn để học platform. Bạn có thể sử dụng script làm data simulator cho development và testing mà không cần phần cứng vật lý.
{{% /notice %}}

![Edge Device Architecture](/images/edge-arch.jpg)

## Tùy chọn phần cứng

### ESP32 + Weather Module

**Linh kiện cần thiết:**

- ESP32 Development Board (ESP32-WROOM/WROVER/C6/S3/etc.)
- DFRobot Weather Module
- Solar panel (Tùy chọn)
- OLED Screen 0.96inch I2C

| ![ESP32](/images/2.prerequisite/setupEdge/esp32_wroom_32e.webp) | ![Weather Module](/images/2.prerequisite/setupEdge/SEN0186.jpg) |
| :-------------------------------------------------------------: | :-------------------------------------------------------------: |
|                      ESP32 Microcontroller                      |                     DFRobot Weather Module                      |

**Tùy chọn thay thế cho weather module:**

- Cảm biến nhiệt độ và độ ẩm DHT22
- Cảm biến BMP280
- Anemometer (Wind Speed + Direction)
- Rain Gauge
- Breadboard và jumper wires

### Raspberry Pi

**Linh kiện cần thiết:**

- Raspberry Pi 3 Model B+ / Model 4/5 (4/8/16GB RAM)
- MicroSD Card (32GB Class 10)
- Power Supply (5V 3A)
- USB card reader

| ![Raspberry Pi](/images/2.prerequisite/setupEdge/raspberry.png) | ![Micro SD Card](/images/2.prerequisite/setupEdge/MicroSD.jpg) | ![Card Reader](/images/2.prerequisite/setupEdge/card_reader.jpg) |
| :-------------------------------------------------------------: | :------------------------------------------------------------: | :--------------------------------------------------------------: |
|                          Raspberry Pi                           |                         Micro SD Card                          |                           Card Reader                            |

## Phần mềm

### Arduino IDE

1. Tải [Arduino IDE](https://www.arduino.cc/en/software) và cài đặt phiên bản mới nhất

2. Cài đặt drivers  
   Để upload code vào ESP32, bạn cần tải CH340 / CP210x driver, tùy thuộc vào model ESP của bạn đang xài  
   Tải >> [CH340 Driver](https://sparks.gogo.co.nz/ch340.html?srsltid=AfmBOoqw56vghnvmBvcnyxgdozmHKAL6zkRtUcDfAHQ9vE3_kJ55k_Gj)  
   Tải >> [CP210x Windows Drivers v6.7.6](https://www.silabs.com/software-and-tools/usb-to-uart-bridge-vcp-drivers?tab=downloads)

3. Cài đặt ESP32 board support:

   - Vào **Tools** **Board** **Boards Manager**
   - Tìm "ESP32" và cài đặt

   ![Install Board Support](/images/2.prerequisite/2.1-setupEdge/1.png)

{{% notice info %}}
Tùy thuộc vào model ESP32 bạn sử dụng, bạn chỉ cần cài đặt board support dành riêng cho model của mình để giảm các packages không cần thiết.
{{% /notice %}}

4. Cài đặt các libraries này qua **Library Manager**:

```
- WiFi (built-in)
- PubSubClient (cho MQTT)
- ArduinoJson
- U8g2 (cho OLED display, tùy chọn. Bạn có thể dùng library khác nếu OLED screen của bạn không phải SSD1306)
- SoftwareSerial (Trong trường hợp model ESP32 của bạn không có đủ UART ports hoặc pins bị hỏng)
```

![Install Library](/images/2.prerequisite/2.1-setupEdge/2.png)
![Install Library](/images/2.prerequisite/2.1-setupEdge/3.png)
![Install Library](/images/2.prerequisite/2.1-setupEdge/4.png)
![Install Library](/images/2.prerequisite/2.1-setupEdge/5.png)

### Raspeberry Pi Imager

Tải [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
![Install Raspberry Pi Imager](/images/2.prerequisite/2.1-setupEdge/6.png)

## Bước tiếp theo

> > [Thiết lập Weather Module](/2-prerequiste/2.1-setupedge/2.1.1-weatherModule)  
> > [Thiết lập Raspberry Pi](/2-prerequiste/2.1-setupedge/2.1.2-raspi)
