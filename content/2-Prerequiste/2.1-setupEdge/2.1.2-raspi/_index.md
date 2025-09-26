---
title: "Setup Raspberry Pi"
weight: 2
chapter: false
pre: " <b> 2.1.2 </b> "
---

In this section, we'll configure the Raspberry Pi to act as a gateway device that collects data from ESP32 sensors and forwards it to AWS IoT Core.

### Prerequisites

{{% notice info %}}
Your Raspberry Pi must be model 3B or higher to run the provided source code effectively.
{{% /notice %}}

### Required Components

- Raspberry Pi (Model 3B or higher)
- MicroSD card (32GB or higher)
- Power supply
- Network connection (WiFi or Ethernet)

### Step 1: Install Raspberry Pi OS

1. **Download Raspberry Pi Imager** from the >> [Official website](https://www.raspberrypi.com/software/)
2. Open Raspberry Pi Imager, plug in the MicroSD card and **flash the OS** to your card. You can use any OS, I'm using Ubuntu Server, but if you are new to Raspberry Pi, I suggest use RaspiOS, it's simple and beginner friendly
3. **Enable SSH**: This one is important, would need to tick the box for this option
4. **Configure Wifi**: The software will ask you to type in your current SSID and password in order to connect to the internet

### Step 2: Initial Setup

1. **Boot your Raspberry Pi** and connect via SSH:

   ```bash
   ping -4 raspberrypi.local
   ```
   If it return IPv4 address then you are good to go. If not then there your wifi configuration in the previous step must had wrong.  
   It could be due to invalid SSID or password. You can either:
   1. Flash OS again
   2. Use a LAN cable and connect it to your laptop/PC and set it an IP
   
   ```bash
   ssh pi@<raspberry-pi-ip>
   ```

2. **Update the system**:

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

3. **Install required software/tools**:
   ```bash
   
   ```
   >>  
   >>

### Step 3: Configure Edge station

   ```bash
   cd Weather-Edge
   sudo nano .env
   ```
   Paste the example .env file with your own credentials
  Then run
   ```bash
   docker compose up --build -d
   ```
  You will then need to open InfluxDB UI on `localhost:8086`, login and get the token
  Once token is generated, copy and paste it into `.env` file
  ```
  INFLUXDB_TOKEN=<your_token_here>
  ```

### Step 4: Test the Setup

1. **Run the edge station again**:

   ```bash
   docker compose up --build -d
   ```

2. **Verify data transmission** from the ES32 to Raspberry Pi by checking its container status and logs
   ```bash
   docker compose ps
   docker compose logs weather-edge-processor
   ```

{{% notice tip %}}
Make sure your Raspberry Pi has a stable internet connection for reliable data transmission.
{{% /notice %}}
