---
title: "Setup Weather module with ESP32"
weight: 1
chapter: false
pre: " <b> 2.1.1 </b> "
---

In this step, we will connect our ESP32 to SEN0186 DFRobot weather module, configure it to send MQTT message over the Wifi to edge device which is our Raspberry Pi

### 1. Connect SEN0186 TO ESP32

### 2. Upload code to ESP32

#### Download Arduino IDE (if you don't have one)

Go to >> [Arduino Official Download Page](https://www.arduino.cc/en/software/) and install latest version

#### Install drivers

In order to upload code into ESP32, you would need to download CH340 / CP210x driver, depends on your ESP model  
Download >> [CH340 Driver](https://sparks.gogo.co.nz/ch340.html?srsltid=AfmBOoqw56vghnvmBvcnyxgdozmHKAL6zkRtUcDfAHQ9vE3_kJ55k_Gj)  
Download >> [CP210x Windows Drivers v6.7.6](https://www.silabs.com/software-and-tools/usb-to-uart-bridge-vcp-drivers?tab=downloads)

#### Install board and libraries

Once complete, open the IDE and

![VPC](/images/2.prerequisite/001-createvpc.png)

2. At the **Create VPC** page.
   - In the **Name tag** field, enter **Lab VPC**.
   - In the **IPv4 CIDR** field, enter: **10.10.0.0/16**.
   - Click **Create VPC**.

![VPC](/images/2.prerequisite/002-createvpc.png)
