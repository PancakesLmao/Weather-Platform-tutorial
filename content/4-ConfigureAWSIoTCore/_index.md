---
title: "Configure AWS IoT Core"
weight: 4
chapter: false
pre: " <b> 4. </b> "
---

In this section, we'll configure AWS IoT Core for the Weather Platform. IoT Core serves as the central hub for device communication, handling MQTT messaging, device authentication, and data routing to other AWS services.

{{% notice info %}}
AWS IoT Core configuration is automatically handled by Amplify, but understanding the components helps with device management and troubleshooting.
{{% /notice %}}

![IoT Core Architecture](/images/iot-core-architecture.png)

## IoT Core Components

### Device Registry

The IoT Core Device Registry manages all weather station devices:

- **Things**: Individual weather station devices
- **Thing Types**: Templates for device categories
- **Thing Groups**: Logical groupings for device management
- **Certificates**: X.509 certificates for device authentication
- **Policies**: Permissions for device operations

### MQTT Topics Structure

The platform uses a structured topic hierarchy:

```
weather/                          # Root topic for weather data
├── {deviceId}/                   # Individual device topics
│   ├── data                      # Sensor data publishing
│   ├── status                    # Device status updates
│   ├── config                    # Configuration updates
│   └── alerts                    # Alert notifications
├── broadcast/                    # System-wide messages
│   ├── config                    # Global configuration
│   └── alerts                    # System alerts
└── admin/                        # Administrative topics
    ├── device-management         # Device lifecycle events
    └── system-status            # Platform health
```

### Example Topic Usage

```
weather/weather-station-01/data   # Device data publishing
weather/weather-station-01/status # Device online/offline status
weather/broadcast/config          # Global configuration updates
weather/admin/device-management   # Device registration events
```

## Device Authentication

### X.509 Certificates

Each weather station uses unique X.509 certificates:

```json
{
  "certificateArn": "arn:aws:iot:us-east-1:123456789012:cert/abc123...",
  "certificateId": "abc123def456ghi789jkl012mno345pqr678stu",
  "status": "ACTIVE",
  "creationDate": "2024-01-15T10:30:00Z"
}
```

### IoT Policies

Device permissions are managed through IoT policies:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["iot:Connect"],
      "Resource": "arn:aws:iot:us-east-1:123456789012:client/${iot:Connection.Thing.ThingName}"
    },
    {
      "Effect": "Allow",
      "Action": ["iot:Publish"],
      "Resource": [
        "arn:aws:iot:us-east-1:123456789012:topic/weather/${iot:Connection.Thing.ThingName}/data",
        "arn:aws:iot:us-east-1:123456789012:topic/weather/${iot:Connection.Thing.ThingName}/status"
      ]
    },
    {
      "Effect": "Allow",
      "Action": ["iot:Subscribe", "iot:Receive"],
      "Resource": [
        "arn:aws:iot:us-east-1:123456789012:topicfilter/weather/${iot:Connection.Thing.ThingName}/config",
        "arn:aws:iot:us-east-1:123456789012:topic/weather/${iot:Connection.Thing.ThingName}/config",
        "arn:aws:iot:us-east-1:123456789012:topicfilter/weather/broadcast/*",
        "arn:aws:iot:us-east-1:123456789012:topic/weather/broadcast/*"
      ]
    }
  ]
}
```

## Device Registration Process

### Automatic Registration via Amplify

The Weather Platform provides a web interface for device registration:

1. **Navigate to Devices Page**: http://localhost:3000/devices
2. **Click "Add Device"**
3. **Fill Device Information**:
   - Device Name: `weather-station-01`
   - Location: `ITea Lab - Room A`
   - Description: `Primary weather monitoring station`
4. **Generate Certificates**: Automatically created and downloaded
5. **Device Provisioning**: Thing, certificate, and policy created automatically

### Manual Registration (Advanced)

For advanced users, devices can be registered manually:

#### Step 1: Create Thing Type

```bash
aws iot create-thing-type \
  --thing-type-name "WeatherStation" \
  --thing-type-description "Weather monitoring station" \
  --thing-type-properties "description=IoT weather station for environmental monitoring" \
  --profile weather-platform
```

#### Step 2: Create Thing

```bash
aws iot create-thing \
  --thing-name "weather-station-01" \
  --thing-type-name "WeatherStation" \
  --attribute-payload '{
    "attributes": {
      "location": "ITea Lab - Room A",
      "firmware_version": "1.2.3",
      "installation_date": "2024-01-15"
    }
  }' \
  --profile weather-platform
```

#### Step 3: Create Certificate

```bash
aws iot create-keys-and-certificate \
  --set-as-active \
  --certificate-pem-outfile "weather-station-01-certificate.pem.crt" \
  --public-key-outfile "weather-station-01-public.pem.key" \
  --private-key-outfile "weather-station-01-private.pem.key" \
  --profile weather-platform
```

#### Step 4: Attach Policy to Certificate

```bash
# First create the policy (if not exists)
aws iot create-policy \
  --policy-name "WeatherStationPolicy" \
  --policy-document file://weather-station-policy.json \
  --profile weather-platform

# Attach policy to certificate
aws iot attach-policy \
  --policy-name "WeatherStationPolicy" \
  --target "arn:aws:iot:us-east-1:123456789012:cert/abc123..." \
  --profile weather-platform
```

#### Step 5: Attach Certificate to Thing

```bash
aws iot attach-thing-principal \
  --thing-name "weather-station-01" \
  --principal "arn:aws:iot:us-east-1:123456789012:cert/abc123..." \
  --profile weather-platform
```

## IoT Rules for Data Processing

### Rule for S3 Data Storage

Automatically route sensor data to S3:

```json
{
  "ruleName": "WeatherDataToS3",
  "sql": "SELECT *, timestamp() as aws_timestamp FROM 'weather/+/data'",
  "description": "Route weather sensor data to S3 data lake",
  "ruleDisabled": false,
  "actions": [
    {
      "s3": {
        "roleArn": "arn:aws:iam::123456789012:role/IoTRuleS3Role",
        "bucketName": "weather-platform-bucket",
        "key": "raw-data/year=${year(timestamp())}/month=${month(timestamp())}/day=${day(timestamp())}/${topic(2)}/${timestamp()}.json",
        "cannedAcl": "private"
      }
    }
  ]
}
```

### Rule for Real-time Processing

Forward data to Lambda for real-time processing:

```json
{
  "ruleName": "WeatherDataProcessing",
  "sql": "SELECT * FROM 'weather/+/data' WHERE sensors.temperature.value > 35 OR sensors.humidity.value > 90",
  "description": "Process weather data for alerts and real-time updates",
  "ruleDisabled": false,
  "actions": [
    {
      "lambda": {
        "functionArn": "arn:aws:lambda:us-east-1:123456789012:function:processWeatherData"
      }
    }
  ]
}
```

### Rule for Device Status Monitoring

Monitor device connectivity:

```json
{
  "ruleName": "DeviceStatusMonitoring",
  "sql": "SELECT * FROM '$aws/events/presence/+/+'",
  "description": "Monitor device connection status",
  "ruleDisabled": false,
  "actions": [
    {
      "lambda": {
        "functionArn": "arn:aws:lambda:us-east-1:123456789012:function:deviceStatusHandler"
      }
    }
  ]
}
```

## Device Shadow Configuration

### Shadow Document Structure

IoT Device Shadows maintain device state:

```json
{
  "desired": {
    "config": {
      "sampling_interval": 300,
      "reporting_interval": 60,
      "alert_thresholds": {
        "temperature_max": 40,
        "humidity_max": 95,
        "pressure_min": 950
      }
    }
  },
  "reported": {
    "config": {
      "sampling_interval": 300,
      "reporting_interval": 60
    },
    "status": {
      "firmware_version": "1.2.3",
      "battery_level": 85,
      "last_reboot": "2024-01-15T08:00:00Z"
    }
  },
  "metadata": {
    "desired": {
      "config": {
        "sampling_interval": {
          "timestamp": 1705312200
        }
      }
    },
    "reported": {
      "status": {
        "battery_level": {
          "timestamp": 1705312800
        }
      }
    }
  },
  "version": 15,
  "timestamp": 1705312800
}
```

## Testing IoT Configuration

### Test MQTT Connection

Use AWS IoT Device Simulator or MQTT client:

```bash
# Subscribe to device data
aws iot-data subscribe-to-topic \
  --topic "weather/weather-station-01/data" \
  --profile weather-platform

# Publish test data
aws iot-data publish \
  --topic "weather/weather-station-01/data" \
  --payload '{
    "deviceId": "weather-station-01",
    "timestamp": "2024-01-15T10:30:00Z",
    "sensors": {
      "temperature": {"value": 25.5, "unit": "celsius"},
      "humidity": {"value": 60, "unit": "percent"}
    }
  }' \
  --profile weather-platform
```

### Verify Data Flow

1. **Check S3 Bucket**: Verify data appears in raw-data folder
2. **Monitor CloudWatch Logs**: Check IoT Rule execution logs
3. **Test Dashboard**: Confirm real-time data updates
4. **Validate Alerts**: Test threshold-based notifications

## Security Best Practices

### Certificate Management

- **Rotate Certificates**: Regularly update device certificates
- **Secure Storage**: Store private keys securely on devices
- **Certificate Revocation**: Disable compromised certificates immediately
- **Backup Certificates**: Maintain secure certificate backups

### Network Security

- **TLS Encryption**: All MQTT communication uses TLS 1.2+
- **VPC Endpoints**: Use VPC endpoints for private connectivity
- **IP Whitelisting**: Restrict device access by IP ranges
- **Rate Limiting**: Configure appropriate message rate limits

### Access Control

- **Least Privilege**: Grant minimal required permissions
- **Policy Variables**: Use IoT policy variables for dynamic permissions
- **Regular Audits**: Review and audit device permissions
- **Monitoring**: Enable CloudTrail for API call logging

## Troubleshooting Common Issues

### Connection Problems

```bash
# Check device connectivity
aws iot describe-thing --thing-name "weather-station-01" --profile weather-platform

# Verify certificate status
aws iot describe-certificate --certificate-id "abc123..." --profile weather-platform

# Test MQTT endpoint
aws iot describe-endpoint --endpoint-type iot:Data-ATS --profile weather-platform
```

### Data Flow Issues

```bash
# Check IoT Rules
aws iot list-topic-rules --profile weather-platform

# View rule details
aws iot get-topic-rule --rule-name "WeatherDataToS3" --profile weather-platform

# Monitor CloudWatch metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/IoT \
  --metric-name PublishIn.Success \
  --start-time 2024-01-15T00:00:00Z \
  --end-time 2024-01-15T23:59:59Z \
  --period 3600 \
  --statistics Sum \
  --profile weather-platform
```

## Monitoring and Alerts

### CloudWatch Metrics

Key IoT Core metrics to monitor:

- **Connect.Success**: Successful device connections
- **PublishIn.Success**: Successful message publications
- **RuleMessageThrottled**: Throttled rule executions
- **RuleNotFound**: Messages to non-existent rules

### Custom Alerts

Set up CloudWatch alarms for:

- Device disconnection events
- High message failure rates
- Unusual data patterns
- Certificate expiration warnings

{{% notice success %}}
AWS IoT Core is now configured for the Weather Platform. Devices can securely connect, publish sensor data, and receive configuration updates through the MQTT protocol.
{{% /notice %}}

## Next Steps

With IoT Core configured, you can:

1. **Deploy physical weather stations** with the generated certificates
2. **Monitor device connectivity** through the dashboard
3. **Configure custom alerts** for environmental thresholds
4. **Scale device management** as you add more weather stations
5. **Integrate with additional AWS services** for advanced analytics

The IoT Core infrastructure provides a secure, scalable foundation for your weather monitoring network.
