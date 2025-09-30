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

### Step 1: Install Raspberry Pi OS

1. Open **Raspberry Pi Imager**, plug in the MicroSD card and **flash the OS**. You can use any OS, I'm using Ubuntu Server, but if you are new to Raspberry Pi, I recommend **Raspberry Pi OS (RaspiOS)**, it's simple and beginner friendly
2. **Enable SSH**: This one is important, make sure you tick the box for this option
3. **Configure Wifi**: Enter your SSID and password so your Pi can connect to the internet on first boot

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
  - Git
  - [Docker](https://docs.docker.com/engine/install/debian/)
  - [Nginx](https://techworldthink.github.io/Tech-Guides/pages/nginx_pi.html)
  - [Tailscale](https://tailscale.com/kb/1031/install-linux) (Or you can use WireGuard)

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

### Step 4: Configure Device Certificates
{{% notice warning %}}
Comeback to this step after you finish [setup Cloud Environment](../2.2-setupCloud/)
{{% /notice %}}

1. **Register device** in the Weather Platform dashboard
2. **Download certificates** (certificate.pem.crt, private.pem.key, AmazonRootCA1.pem)
4. **Update the code** with your certificates and IoT endpoint

You must insert all credential files inside `/data-processor/credentials/` folder in order to publish data onto weather platform

### Step 5: Test the Setup

1. **Run the edge station again**:

   ```bash
   docker compose up --build -d
   ```

2. **Verify data transmission** from the ES32 to Raspberry Pi by checking its container status and logs
   ```bash
   docker compose ps
   docker compose logs weather-edge-processor
   ```
**Verify data** appears in the dashboard

{{% notice tip %}}
Make sure your Raspberry Pi has a stable internet connection for reliable data transmission.
{{% /notice %}}
