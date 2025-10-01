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
{{% notice warning %}}
When you select the storage device, make sure you select the correct one. All data on the selected device will be erased.
{{% /notice %}}

2. **Enable SSH**: This one is important, make sure you tick the box for this option
3. **Configure Wifi**: Enter your SSID and password so your Pi can connect to the internet on first 
4. **Set locale settings**: Set your country, language and timezone
5. Click **Save** and then select **Yes** to flash the OS onto the MicroSD card
6. Once done, eject the MicroSD card and plug it into your Raspberry Pi. Connect the power supply to boot it up

![Imager Config](/images/2.prerequisite/2.1-setupEdge/imager1.png)
![Imager Config](/images/2.prerequisite/2.1-setupEdge/imager2.png)
![Imager Config](/images/2.prerequisite/2.1-setupEdge/imager3.png)
![Imager Config](/images/2.prerequisite/2.1-setupEdge/imager4.png)
![Imager Config](/images/2.prerequisite/2.1-setupEdge/imager5.png)

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
   ssh raspberrypi@<raspberry-pi-ip>
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

### Step 3: Setup VPN with Tailscale on you Laptop and Raspberry Pi
A VPN is secure, flexible, and lets you access your Raspberry Pi like you're on the same local network—even from anywhere in the world.
There are a few good VPN options, but I highly recommend Tailscale if you want something fast and super easy to set up. Tailscale is a VPN built on WireGuard that automatically sets up a secure mesh network between your devices.  
Here’s how you can do it:

1. Go to https://tailscale.com/ and create an account > Sign up using Google / Github (choose your option)
2. Install Tailscale client on your device from https://tailscale.com/download  
Once complete, Tailscale icon should appear in your system tray  
Right-click the icon > Log in using credentials you used to create the account  
Tailscale will then ask you to authorize device to join you tailnet VPN  
Click **Connect** > Then you will be able to view your device on Machines dashboard.
3. If you have already enabled SSH on Raspberry Pi, move on to step 4. If you have not, then follow these steps.  
Run `sudo raspi-config` to open Raspberry Pi Configuration Tool Interface  
Select **Interface Options** > SSH > Yes > Ok
4. To install Tailscale client on your Raspberry Pi, use:
```
sudo apt update
sudo apt upgrade -y
curl -fsSL https://tailscale.com/install.sh | sh
```
Then start Tailscale using:
```
sudo tailscale up
```
You can find your Tailscale IPv4 address by running:
```
tailscale ip -4
```
5. Test the connection
```
ssh <pi_username>@<tailscale_ip>
```
or 
Enable SSH via Tailscale using:
```
sudo tailscale up --ssh
```
This allow you to directly SSH your Pi from a browser  
Go to admin console > Click on **Menu option** > SSH to machine
It will then ask for login credential > SSH > Tailscale will open SSH session browser window

### Step 4: Configure Edge station

   ```bash
   git clone https://github.com/Itea-Lab/Weather-Edge.git
   cd Weather-Edge
   sudo nano .env
   ```
  Paste the example .env file with your own credentials  
  Then run
   ```bash
   docker compose up --build -d
   ```
  You will then need to open InfluxDB UI on your laptop browser with `<raspi_tailscale_ip>:8086`, login and get the token
  Once token is generated, copy and paste it into `.env` file
  ```
  INFLUXDB_TOKEN=<your_token_here>
  ```
The complete `.env` file should look like this:
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

### Step 5: Configure Device Certificates
{{% notice warning %}}
Comeback to this step after you finish [setup Sandbox Environment](/5-amplifyconfiguration/5.2-backendconfiguration)
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
