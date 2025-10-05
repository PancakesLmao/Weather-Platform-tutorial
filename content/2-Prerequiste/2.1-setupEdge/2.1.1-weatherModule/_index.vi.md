---
title: "Thiết lập Weather module với ESP32"
weight: 1
chapter: false
pre: " <b> 2.1.1 </b> "
---

### 1. Kết nối SEN0186 với ESP32

![Wiring diagram](/images/2.prerequisite/2.1-setupEdge/wiring.png)

**Sơ đồ kết nối:**

```
ESP32 Pin    |  Sensor        |  Connection
-------------|----------------|------------------
5V           |  SEN1086       |  VCC
TX           |  SEN1086       |  RX
RX           |  SEN1086       |  TX
GND          |  SEN1086       |  GND
```

{{% notice info %}}
Tùy thuộc vào sensors bạn sử dụng, sơ đồ kết nối có thể khác nhau. Vui lòng tham khảo datasheet của sensor để có hướng dẫn đấu nối chính xác.
Cũng tham khảo sơ đồ pinout của model ESP32 để tránh sử dụng pins reserved. Chúng được gọi là Device GPIO.
{{% /notice %}}
![GPIO diagram](/images/2.prerequisite/2.1-setupEdge/wrover_e_GPIO.jpg)

### 2. Upload code lên ESP32

Tạo một Arduino sketch mới với code bên dưới (cho ESP32 Wrover-E) hoặc sử dụng code của riêng bạn:

```cpp
#include <SoftwareSerial.h>
#include <WiFi.h>
#include <esp_wifi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>
// Nếu bạn sử dụng OLED display
#include <U8g2lib.h>
#include <Wire.h>

// Pins cho SoftwareSerial
#define MYPORT_TX 12
#define MYPORT_RX 13

// Thông tin Wi-Fi
const char *ssid = "SSID";
const char *password = "PASSWORD";

// Chi tiết MQTT broker
const char *mqtt_server = "endpoint_ip"; // Thay thế bằng IP address MQTT broker của bạn
const int mqtt_port = 1883;
const char *mqtt_user = "username";
const char *mqtt_password = "password";
const char *mqtt_topic = "weather/data";

// Tạo instance của PubSubClient
WiFiClient espClient;
PubSubClient client(espClient);

// OLED Display Setup (cho SSD1306 128x64 sử dụng I2C) xóa nếu bạn không dùng OLED Display
U8G2_SSD1306_128X64_NONAME_F_HW_I2C u8g2(U8G2_R0, /* reset=*/U8X8_PIN_NONE);

// Tạo SoftwareSerial object
EspSoftwareSerial::UART myPort;

// Data buffer cho weather station data
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
                index = -1; // Khởi động lại nếu ký tự đầu tiên không phải 'c'
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

    // Lấy thông tin WiFi
    String ipAddress = WiFi.localIP().toString();

    // Xóa đoạn này nếu không dùng OLED Display
    u8g2.clearBuffer();
    u8g2.drawStr(10, 30, "Connected");
    u8g2.drawStr(10, 50, "IP:");
    u8g2.drawStr(35, 50, ipAddress.c_str()); // Hiển thị IP Address
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
    Serial.println("Đang kết nối MQTT...");

    while (!client.connected())
    {
        if (client.connect("ESP32", mqtt_user, mqtt_password))
        {
            Serial.println("Đã kết nối tới MQTT broker");
        }
        else
        {
            Serial.print("Thất bại kết nối, rc=");
            Serial.print(client.state());
            Serial.println(" Thử lại sau 5 giây...");
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

    Serial.print("Đang gửi message: ");
    Serial.println(data);

    client.publish(mqtt_topic, data);
}

void setup()
{
    Serial.begin(9600);
    Wire.begin(21, 22);
    u8g2.begin();
    u8g2.enableUTF8Print(); // Enable UTF-8 support

    // Khởi tạo software serial port
    myPort.begin(9600, SWSERIAL_8N1, MYPORT_RX, MYPORT_TX, false);

    if (!myPort)
    {
        Serial.println("Cấu hình pin EspSoftwareSerial không hợp lệ, kiểm tra config");
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
    if (millis() - lastCheck >= 5000) // Cập nhật OLED mỗi 5 giây
    {
        lastCheck = millis();
        displayWiFiStats();
    }
    // Serial.print("Hướng gió: ");
    // Serial.print(WindDirection());
    // Serial.println("  ");
    // Serial.print("Tốc độ gió trung bình (Một phút): ");
    // Serial.print(WindSpeedAverage());
    // Serial.println(" m/s  ");
    // Serial.print("Tốc độ gió tối đa (Năm phút): ");
    // Serial.print(WindSpeedMax());
    // Serial.println(" m/s");
    // Serial.print("Lượng mưa (Một giờ): ");
    // Serial.print(RainfallOneHour());
    // Serial.println(" mm  ");
    // Serial.print("Lượng mưa (24 giờ): ");
    // Serial.print(RainfallOneDay());
    // Serial.println(" mm");
    // Serial.print("Nhiệt độ: ");
    // Serial.print(Temperature());
    // Serial.println(" C  ");
    // Serial.print("Độ ẩm: ");
    // Serial.print(Humidity());
    // Serial.println("%  ");
    // Serial.print("Áp suất khí quyển: ");
    // Serial.print(BarPressure());
    // Serial.println(" hPa");
    // Serial.println("");
    // Serial.println("");
    StaticJsonDocument<256> doc;

    // Đưa dữ liệu weather vào JSON document
    doc["Wind Direction"] = WindDirection();
    doc["Avg Wind Speed"] = WindSpeedAverage();
    doc["Max Wind Speed"] = WindSpeedMax();
    doc["Rainfall (1hr)"] = RainfallOneHour();
    doc["Rainfall (24hr)"] = RainfallOneDay();
    doc["Temperature"] = Temperature();
    doc["Humidity"] = Humidity();
    doc["Barometric Pressure"] = BarPressure();

    // JSON document thành string
    char jsonString[256];
    serializeJson(doc, jsonString);
    Serial.println(jsonString);

    publishDataToMQTT(jsonString);

    delay(10000); // Thiết lập thời gian delay
}
```

Tùy vào thiết bị đang sử dụng mà bạn viết code cho phù hợp, chỉ cần đảm bảo nó khớp với định dạng:

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
