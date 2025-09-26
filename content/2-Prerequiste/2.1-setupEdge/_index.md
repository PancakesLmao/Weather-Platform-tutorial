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

![Edge Device Architecture](/images/edge-architecture.jpg)

## Hardware Options

### ESP32 + Weather Module

**Components needed:**

- ESP32 Development Board (ESP32-WROOM-32/WROVER-E/C6/S3/etc.)
- DFRobot Weather Module
- Solar panel (Optional)
- OLED Screen 

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

## ESP32 Setup Guide

### Step 1: Hardware Assembly

**Wiring Diagram:**

```
ESP32 Pin    |  Sensor        |  Connection
-------------|----------------|------------------
3.3V         |  DHT22         |  VCC
GND          |  DHT22         |  GND
GPIO4        |  DHT22         |  Data
3.3V         |  BMP280        |  VCC
GND          |  BMP280        |  GND
GPIO21       |  BMP280        |  SDA
GPIO22       |  BMP280        |  SCL
GPIO5        |  Anemometer    |  Signal
GPIO18       |  Rain Gauge    |  Signal
```

### Step 2: Install Arduino IDE

1. Download [Arduino IDE](https://www.arduino.cc/en/software)
2. Install ESP32 board support:
   - Go to **File** → **Preferences**
   - Add URL: `https://dl.espressif.com/dl/package_esp32_index.json`
   - Go to **Tools** → **Board** → **Boards Manager**
   - Search "ESP32" and install

### Step 3: Install Required Libraries

Install these libraries via **Library Manager**:

```
- WiFi (built-in)
- PubSubClient (for MQTT)
- ArduinoJson
- DHT sensor library
- Adafruit BMP280 Library
- NTPClient (for time synchronization)
```

### Step 4: ESP32 Code Implementation

Create a new Arduino sketch with the code below (for ESP32 Wrover-E) or use your own code:

```cpp
#include <SoftwareSerial.h>
#include <WiFi.h>
#include <esp_wifi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>
// If you use OLED display
#include <U8g2lib.h>
#include <Wire.h>

// Pins for SoftwareSerial
#define MYPORT_TX 12
#define MYPORT_RX 13

// Wi-Fi credentials
const char *ssid = "SSID";
const char *password = "PASSWORD";

// MQTT broker details
const char *mqtt_server = "endpoint_ip"; // Replace with your MQTT broker IP address
const int mqtt_port = 1883;
const char *mqtt_user = "username";
const char *mqtt_password = "password";
const char *mqtt_topic = "weather/data";

// Create an instance of the PubSubClient
WiFiClient espClient;
PubSubClient client(espClient);

// OLED Display Setup (for SSD1306 128x64 using I2C) delete this if you do not use OLED Display
U8G2_SSD1306_128X64_NONAME_F_HW_I2C u8g2(U8G2_R0, /* reset=*/U8X8_PIN_NONE);

// Create a SoftwareSerial object
EspSoftwareSerial::UART myPort;

// Data buffer for weather station data
char databuffer[35];
double temp;

void getBuffer()
{
    int index = 0;
    while (index < 35)
    {
        if (myPort.available())
        {
            databuffer[index] = myPort.read();
            if (index == 0 && databuffer[0] != 'c')
            {
                index = -1; // Restart if first character is not 'c'
            }
            index++;
        }
    }
}

int transCharToInt(char *_buffer, int _start, int _stop)
{
    int result = 0;
    for (int i = _start; i <= _stop; i++)
    {
        result = 10 * result + (_buffer[i] - '0');
    }
    return result;
}

int transCharToInt_T(char *_buffer)
{
    int result = (_buffer[13] == '-') ? -(((_buffer[14] - '0') * 10) + (_buffer[15] - '0')) : ((_buffer[13] - '0') * 100) + ((_buffer[14] - '0') * 10) + (_buffer[15] - '0');
    return result;
}

int WindDirection()
{
    return transCharToInt(databuffer, 1, 3);
}

float WindSpeedAverage()
{
    return 0.44704 * transCharToInt(databuffer, 5, 7);
}

float WindSpeedMax()
{
    return 0.44704 * transCharToInt(databuffer, 9, 11);
}

float Temperature()
{
    return (transCharToInt_T(databuffer) - 32.00) * 5.00 / 9.00;
}

float RainfallOneHour()
{
    return transCharToInt(databuffer, 17, 19) * 25.40 * 0.01;
}

float RainfallOneDay()
{
    return transCharToInt(databuffer, 21, 23) * 25.40 * 0.01;
}

int Humidity()
{
    return transCharToInt(databuffer, 25, 26);
}

float BarPressure()
{
    return transCharToInt(databuffer, 28, 32) / 10.00;
}

void connectWifi()
{
    u8g2.clearBuffer();
    u8g2.setFont(u8g2_font_6x10_tf);
    u8g2.drawStr(10, 20, "Connecting to WiFi...");
    u8g2.sendBuffer();

    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED)
    {
        delay(500);
        Serial.print(".");
    }

    // Get WiFi details
    String ipAddress = WiFi.localIP().toString();

    // Delete this if you do not use OLED Display
    u8g2.clearBuffer();
    u8g2.drawStr(10, 30, "Connected");
    u8g2.drawStr(10, 50, "IP:");
    u8g2.drawStr(35, 50, ipAddress.c_str()); // Show IP Address
    u8g2.sendBuffer();
}

void displayWiFiStats()
{
    u8g2.clearBuffer();
    u8g2.setFont(u8g2_font_6x10_tf);

    String ip = WiFi.localIP().toString();
    int rssi = WiFi.RSSI();
    int channel = WiFi.channel();
    int txPower = 19.5; // Default max power for ESP32

    u8g2.setCursor(10, 10);
    u8g2.print("IP: ");
    u8g2.print(ip);

    u8g2.setCursor(10, 20);
    u8g2.print("Signal: ");
    u8g2.print(rssi);
    u8g2.print(" dBm");

    u8g2.setCursor(10, 30);
    u8g2.print("Channel: ");
    u8g2.print(channel);

    u8g2.setCursor(10, 40);
    u8g2.print("TX Power: ");
    u8g2.print(txPower);
    u8g2.print(" dBm");

    u8g2.sendBuffer();
}

void connectToMQTT()
{
    client.setServer(mqtt_server, mqtt_port);
    Serial.println("Connecting to MQTT...");

    while (!client.connected())
    {
        if (client.connect("ESP32", mqtt_user, mqtt_password))
        {
            Serial.println("Connected to MQTT broker");
        }
        else
        {
            Serial.print("Failed to connect, rc=");
            Serial.print(client.state());
            Serial.println(" Trying again in 5 seconds...");
            delay(5000);
        }
    }
}

void publishDataToMQTT(const char *data)
{
    if (!client.connected())
    {
        connectToMQTT();
    }
    client.loop();

    Serial.print("Publishing message: ");
    Serial.println(data);

    client.publish(mqtt_topic, data);
}

void setup()
{
    Serial.begin(9600);
    Wire.begin(21, 22);
    u8g2.begin();
    u8g2.enableUTF8Print(); // Enable UTF-8 support

    // Initialize the software serial port
    myPort.begin(9600, SWSERIAL_8N1, MYPORT_RX, MYPORT_TX, false);

    if (!myPort)
    {
        Serial.println("Invalid EspSoftwareSerial pin configuration, check config");
        while (1)
        {
            delay(1000);
        }
    }

    connectWifi();
    connectToMQTT();
}

void loop()
{
    getBuffer();
    static unsigned long lastCheck = 0;
    if (millis() - lastCheck >= 5000) // Update OLED every 5 seconds
    {
        lastCheck = millis();
        displayWiFiStats();
    }
    // Serial.print("Wind Direction: ");
    // Serial.print(WindDirection());
    // Serial.println("  ");
    // Serial.print("Average Wind Speed (One Minute): ");
    // Serial.print(WindSpeedAverage());
    // Serial.println(" m/s  ");
    // Serial.print("Max Wind Speed (Five Minutes): ");
    // Serial.print(WindSpeedMax());
    // Serial.println(" m/s");
    // Serial.print("Rain Fall (One Hour): ");
    // Serial.print(RainfallOneHour());
    // Serial.println(" mm  ");
    // Serial.print("Rain Fall (24 Hour): ");
    // Serial.print(RainfallOneDay());
    // Serial.println(" mm");
    // Serial.print("Temperature: ");
    // Serial.print(Temperature());
    // Serial.println(" C  ");
    // Serial.print("Humidity: ");
    // Serial.print(Humidity());
    // Serial.println("%  ");
    // Serial.print("Barometric Pressure: ");
    // Serial.print(BarPressure());
    // Serial.println(" hPa");
    // Serial.println("");
    // Serial.println("");
    StaticJsonDocument<256> doc;

    // Populate the JSON document with weather data
    doc["Wind Direction"] = WindDirection();
    doc["Avg Wind Speed"] = WindSpeedAverage();
    doc["Max Wind Speed"] = WindSpeedMax();
    doc["Rainfall (1hr)"] = RainfallOneHour();
    doc["Rainfall (24hr)"] = RainfallOneDay();
    doc["Temperature"] = Temperature();
    doc["Humidity"] = Humidity();
    doc["Barometric Pressure"] = BarPressure();

    // JSON document to string
    char jsonString[256];
    serializeJson(doc, jsonString);
    Serial.println(jsonString);

    publishDataToMQTT(jsonString);

    delay(10000); // Set delay time
}
```

Whatever the code, just make sure it match the format:
```
{
  "Wind Direction": 45,
  "Avg Wind Speed": 3.2,
  "Max Wind Speed": 5.8,
  "Rainfall (1hr)": 0.0,
  "Rainfall (24hr)": 0.0,
  "Temperature": 24.7,
  "Humidity": 68,
  "Barometric Pressure": 1013.2
}
```

{{% notice warning %}}
Comeback after you finish [setup Cloud Environment](../2.2-setupCloud/)
{{% /notice %}}

### Step 5: Configure Device Certificates

1. **Register device** in the Weather Platform dashboard
2. **Download certificates** (certificate.pem.crt, private.pem.key, AmazonRootCA1.pem)
3. **Convert certificates** to C string format
4. **Update the code** with your certificates and IoT endpoint

### Step 6: Deploy and Test

1. **Upload code** to ESP32
2. **Monitor Serial output** for connection status
3. **Verify data** appears in the dashboard
4. **Check device status** in the Devices page

## Raspberry Pi Setup Guide

### Step 1: Prepare Raspberry Pi

1. **Flash Raspberry Pi OS** to microSD card
2. **Enable SSH and WiFi** in boot configuration
3. **Boot and update** the system:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3-pip git -y
```

### Step 2: Install Python Dependencies

```bash
# Install required Python packages
pip3 install paho-mqtt boto3 adafruit-circuitpython-dht adafruit-circuitpython-bmp280

# Install GPIO libraries
sudo apt install python3-gpiozero python3-w1thermsensor -y
```

### Step 3: Python Weather Station Code

Create `/home/pi/weather_station.py`:

```python
#!/usr/bin/env python3
import json
import time
import ssl
import paho.mqtt.client as mqtt
import board
import adafruit_dht
import adafruit_bmp280
from datetime import datetime
import RPi.GPIO as GPIO

# Configuration
DEVICE_ID = "weather-station-01"
IOT_ENDPOINT = "YOUR_IOT_ENDPOINT.iot.us-east-1.amazonaws.com"
CA_PATH = "/home/pi/certs/AmazonRootCA1.pem"
CERT_PATH = "/home/pi/certs/certificate.pem.crt"
KEY_PATH = "/home/pi/certs/private.pem.key"

# GPIO Pins
ANEMOMETER_PIN = 5
RAIN_GAUGE_PIN = 6

# Initialize sensors
dht = adafruit_dht.DHT22(board.D4)
i2c = board.I2C()
bmp = adafruit_bmp280.Adafruit_BMP280_I2C(i2c)

# Wind and rain counters
wind_count = 0
rain_count = 0

def wind_callback(channel):
    global wind_count
    wind_count += 1

def rain_callback(channel):
    global rain_count
    rain_count += 1

# Setup GPIO interrupts
GPIO.setmode(GPIO.BCM)
GPIO.setup(ANEMOMETER_PIN, GPIO.IN, pull_up_down=GPIO.PUD_UP)
GPIO.setup(RAIN_GAUGE_PIN, GPIO.IN, pull_up_down=GPIO.PUD_UP)
GPIO.add_event_detect(ANEMOMETER_PIN, GPIO.FALLING, callback=wind_callback, bouncetime=10)
GPIO.add_event_detect(RAIN_GAUGE_PIN, GPIO.FALLING, callback=rain_callback, bouncetime=10)

def read_sensors():
    global wind_count, rain_count

    try:
        # Read DHT22
        temperature = dht.temperature
        humidity = dht.humidity

        # Read BMP280
        pressure = bmp.pressure

        # Calculate wind speed (reset counter)
        wind_speed = (wind_count * 2.4) / 60.0  # Convert to m/s
        wind_count = 0

        # Calculate rainfall (reset counter)
        rainfall = rain_count * 0.2794  # Convert to mm
        rain_count = 0

        return {
            "temperature": temperature,
            "humidity": humidity,
            "pressure": pressure,
            "wind_speed": wind_speed,
            "rainfall": rainfall
        }
    except Exception as e:
        print(f"Sensor reading error: {e}")
        return None

def create_payload(sensor_data):
    if not sensor_data:
        return None

    payload = {
        "deviceId": DEVICE_ID,
        "timestamp": datetime.utcnow().isoformat() + "Z",
        "location": {
            "latitude": 10.8231,
            "longitude": 106.6297,
            "name": "ITea Lab - Room A"
        },
        "sensors": {
            "temperature": {
                "value": sensor_data["temperature"],
                "unit": "celsius"
            },
            "humidity": {
                "value": sensor_data["humidity"],
                "unit": "percent"
            },
            "pressure": {
                "value": sensor_data["pressure"],
                "unit": "hPa"
            },
            "wind": {
                "speed": sensor_data["wind_speed"],
                "direction": 0,
                "unit": "m/s"
            },
            "rainfall": {
                "value": sensor_data["rainfall"],
                "unit": "mm"
            }
        },
        "metadata": {
            "firmware_version": "1.0.0",
            "battery_level": 100,
            "signal_strength": -50
        }
    }
    return json.dumps(payload)

def on_connect(client, userdata, flags, rc):
    if rc == 0:
        print("Connected to AWS IoT Core")
        # Subscribe to configuration topic
        config_topic = f"weather/{DEVICE_ID}/config"
        client.subscribe(config_topic)
    else:
        print(f"Connection failed with code {rc}")

def on_message(client, userdata, msg):
    print(f"Received message: {msg.topic} - {msg.payload.decode()}")

def main():
    # Setup MQTT client
    client = mqtt.Client()
    client.on_connect = on_connect
    client.on_message = on_message

    # Configure SSL
    context = ssl.create_default_context(ssl.Purpose.SERVER_AUTH)
    context.load_verify_locations(CA_PATH)
    context.load_cert_chain(CERT_PATH, KEY_PATH)
    client.tls_set_context(context)

    # Connect to AWS IoT
    client.connect(IOT_ENDPOINT, 8883, 60)
    client.loop_start()

    print("Weather Station started...")

    try:
        while True:
            # Read sensors
            sensor_data = read_sensors()

            if sensor_data:
                # Create and publish payload
                payload = create_payload(sensor_data)
                topic = f"weather/{DEVICE_ID}/data"

                result = client.publish(topic, payload)
                if result.rc == 0:
                    print(f"Data published successfully: {payload}")
                else:
                    print(f"Failed to publish data: {result.rc}")

            # Wait 60 seconds
            time.sleep(60)

    except KeyboardInterrupt:
        print("Shutting down...")
    finally:
        GPIO.cleanup()
        client.loop_stop()
        client.disconnect()

if __name__ == "__main__":
    main()
```

### Step 4: Setup as System Service

Create `/etc/systemd/system/weather-station.service`:

```ini
[Unit]
Description=Weather Station Service
After=network.target

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi
ExecStart=/usr/bin/python3 /home/pi/weather_station.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```bash
sudo systemctl enable weather-station.service
sudo systemctl start weather-station.service
sudo systemctl status weather-station.service
```

## Data Simulation (No Hardware Required)

For development without physical hardware, use the built-in simulator:

### Step 1: Create Mock Data Script

Create `mock_weather_data.py`:

```python
#!/usr/bin/env python3
import json
import time
import random
import ssl
import paho.mqtt.client as mqtt
from datetime import datetime

# Configuration
DEVICE_ID = "weather-station-simulator"
IOT_ENDPOINT = "YOUR_IOT_ENDPOINT.iot.us-east-1.amazonaws.com"

def generate_mock_data():
    # Generate realistic weather data
    base_temp = 25 + random.uniform(-5, 10)  # 20-35°C
    base_humidity = 60 + random.uniform(-20, 30)  # 40-90%
    base_pressure = 1013 + random.uniform(-20, 20)  # 993-1033 hPa

    return {
        "deviceId": DEVICE_ID,
        "timestamp": datetime.utcnow().isoformat() + "Z",
        "location": {
            "latitude": 10.8231,
            "longitude": 106.6297,
            "name": "ITea Lab - Simulator"
        },
        "sensors": {
            "temperature": {
                "value": round(base_temp, 1),
                "unit": "celsius"
            },
            "humidity": {
                "value": round(base_humidity, 1),
                "unit": "percent"
            },
            "pressure": {
                "value": round(base_pressure, 2),
                "unit": "hPa"
            },
            "wind": {
                "speed": round(random.uniform(0, 15), 1),
                "direction": random.randint(0, 359),
                "unit": "m/s"
            },
            "rainfall": {
                "value": round(random.uniform(0, 5), 1),
                "unit": "mm"
            }
        },
        "metadata": {
            "firmware_version": "1.0.0-simulator",
            "battery_level": random.randint(80, 100),
            "signal_strength": random.randint(-60, -30)
        }
    }

# Run simulator
def main():
    client = mqtt.Client()

    # Configure SSL (use your certificates)
    context = ssl.create_default_context(ssl.Purpose.SERVER_AUTH)
    # Add your certificate configuration here

    client.tls_set_context(context)
    client.connect(IOT_ENDPOINT, 8883, 60)

    while True:
        data = generate_mock_data()
        payload = json.dumps(data)
        topic = f"weather/{DEVICE_ID}/data"

        client.publish(topic, payload)
        print(f"Published: {payload}")

        time.sleep(30)  # Send data every 30 seconds

if __name__ == "__main__":
    main()
```

### Step 2: Run Simulator

```bash
python3 mock_weather_data.py
```

## Troubleshooting

### Common Issues

**WiFi Connection Problems:**

- Check SSID and password
- Verify WiFi signal strength
- Try different WiFi networks

**MQTT Connection Issues:**

- Verify IoT endpoint URL
- Check certificate files
- Ensure device is registered in AWS IoT

**Sensor Reading Errors:**

- Check wiring connections
- Verify sensor power supply
- Test sensors individually

**Data Not Appearing in Dashboard:**

- Check MQTT topic format
- Verify JSON payload structure
- Monitor CloudWatch logs

### Debugging Commands

```bash
# Check WiFi status (Raspberry Pi)
iwconfig

# Monitor MQTT messages
mosquitto_sub -h YOUR_IOT_ENDPOINT -p 8883 --cafile AmazonRootCA1.pem --cert certificate.pem.crt --key private.pem.key -t "weather/+/data"

# Test sensor readings
python3 -c "import adafruit_dht; import board; dht = adafruit_dht.DHT22(board.D4); print(dht.temperature, dht.humidity)"
```

{{% notice success %}}
Edge devices are now configured to collect and transmit weather data to AWS IoT Core. The data will automatically flow through the processing pipeline to your dashboard.
{{% /notice %}}

## Next Steps

With edge devices configured, you can:

1. **Deploy multiple weather stations** across different locations
2. **Monitor device health** through the dashboard
3. **Configure data collection intervals** based on your needs
4. **Set up alerts** for device offline conditions
5. **Analyze historical data** for weather patterns

The edge devices provide the foundation for real-time weather monitoring and data collection.
