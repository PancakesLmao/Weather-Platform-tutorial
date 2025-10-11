---
title: "Setup Edge Device"
weight: 1
chapter: false
pre: " <b> 2.1 </b> "
---

In this step, we'll set up the edge devices that collect weather data and transmit it to the AWS cloud. The Weather Platform supports multiple hardware configurations, from simple ESP32 setups to full Raspberry Pi installations.

{{% notice info %}}
Edge devices are optional for learning the platform. You can use a script as data simulator for development and testing without physical hardware.
{{% /notice %}}

![Edge Device Architecture](/images/edge-arch.jpg)

## Hardware Options

### ESP32 + Weather Module

**Components needed:**

- ESP32 Development Board (ESP32-WROOM/WROVER/C6/S3/etc.)
- DFRobot Weather Module
- Solar panel (Optional)
- OLED Screen 0.96inch I2C
| ![ESP32](/images/2.prerequisite/setupEdge/esp32_wroom_32e.webp) | ![Weather Module](/images/2.prerequisite/setupEdge/SEN0186.jpg) |
|:---------------------------------------------------------------:|:---------------------------------------------------------------:|
| ESP32 Microcontroller                                                     | DFRobot Weather Module                                          |

**Alternative options for weather module:**
- DHT22 Temperature/Humidity Sensor
- BMP280 Pressure Sensor
- Anemometer (Wind Speed + Direction)
- Rain Gauge
- Breadboard and jumper wires

### Raspberry Pi

**Components needed:**

- Raspberry Pi 3 Model B+ / Model 4/5 (4/8/16GB RAM)
- MicroSD Card (32GB Class 10)
- Power Supply (5V 3A)
- USB card reader

| ![Raspberry Pi](/images/2.prerequisite/setupEdge/raspberry.png) | ![Micro SD Card](/images/2.prerequisite/setupEdge/MicroSD.jpg) | ![Card Reader](/images/2.prerequisite/setupEdge/card_reader.jpg) |
|:---------------------------------------------------------------:|:---------------------------------------------------------------:|:---------------------------------------------------------------:|
| Raspberry Pi                                                     |Micro SD Card                                          | Card Reader                                           |
![Card Reader](/images/2.prerequisite/setupEdge/card_reader.jpg)
## Software
### Arduino IDE

1. Download [Arduino IDE](https://www.arduino.cc/en/software) and install latest version
2. Install drivers  
In order to upload code into ESP32, you would need to download CH340 / CP210x driver, depends on your ESP model  
Download >> [CH340 Driver](https://sparks.gogo.co.nz/ch340.html?srsltid=AfmBOoqw56vghnvmBvcnyxgdozmHKAL6zkRtUcDfAHQ9vE3_kJ55k_Gj)  
Download >> [CP210x Windows Drivers v6.7.6](https://www.silabs.com/software-and-tools/usb-to-uart-bridge-vcp-drivers?tab=downloads)
3. Install ESP32 board support:
   - Go to **Tools** → **Board** → **Boards Manager**
   - Search "ESP32" and install
   ![Install Board Support](/images/2.prerequisite/2.1-setupEdge/1.png)
   
{{% notice info %}}
Depends on the ESP32 model you use, you only need to install the board support specially for your model to reduce unnecessary packages.
{{% /notice %}}

4. Install these libraries via **Library Manager**:

```
- WiFi (built-in)
- PubSubClient (for MQTT)
- ArduinoJson
- U8g2 (for OLED display, optional. You can you different library if your OLED screen is not SSD1306)
- SoftwareSerial (In case your ESP32 model does not have enough UART ports or broken pins)
```
![Install Library](/images/2.prerequisite/2.1-setupEdge/2.png)
![Install Library](/images/2.prerequisite/2.1-setupEdge/3.png)
![Install Library](/images/2.prerequisite/2.1-setupEdge/4.png)
![Install Library](/images/2.prerequisite/2.1-setupEdge/5.png)

### Raspeberry Pi Imager
Download [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
![Install Raspberry Pi Imager](/images/2.prerequisite/2.1-setupEdge/6.png)

## Next Steps
>> [Setup Weather Module](/2-prerequiste/2.1-setupedge/2.1.1-weatherModule)  
>> [Setup Raspberry Pi](/2-prerequiste/2.1-setupedge/2.1.2-raspi)