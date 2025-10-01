---
title: "IoT Rule Engine"
weight: 3
chapter: false
pre: " <b> 4.3 </b> "
---

## IoT Rule Engine for S3 Data Lake

The IoT Rule Engine routes incoming weather telemetry messages from IoT devices directly to the S3 data lake for storage and processing.

### Rule Purpose

This rule automatically:

- Captures all weather telemetry messages from `weatherPlatform/telemetry/*` topics
- Routes messages to the S3 data lake bucket (`itea-weather-data-lake-storage`)
- Organizes data by location in the correct folder structure

### Create IoT Rule

1. **Navigate to AWS IoT Core Console**
2. **Go to Message routing** → **Rules**
3. **Click "Create rule"**
4. **Configure rule settings**:

   - **Rule name**: `WeatherTelemetryToS3Rule`
   - **Description**: `Route weather telemetry data to S3 data lake`

5. **SQL statement**:

   ```sql
   SELECT *, topic(3) as location
   FROM 'weatherPlatform/telemetry/+'
   ```

6. **Rule actions** → **Add action**:

   - **Action type**: **S3**
   - **S3 bucket**: `itea-weather-data-lake-storage-yourname` (use your unique bucket name)
   - **Key**: `raw-data/weatherPlatform/telemetry/${location}/${timestamp()}.json`

7. **Create or select IAM role**:

   - **Role name**: `IoTRuleToS3Role`
   - **Attach policy**: Allow S3 PutObject permissions

8. **Click "Create rule"**

### SQL Statement Explained

```sql
SELECT *, topic(3) as location
FROM 'weatherPlatform/telemetry/+'
```

- `SELECT *`: Captures the entire message payload
- `topic(3)`: Extracts the location from topic `weatherPlatform/telemetry/{location}`
- `FROM 'weatherPlatform/telemetry/+'`: Matches all topics under telemetry (+ is wildcard)

### S3 Key Pattern

```
raw-data/weatherPlatform/telemetry/${location}/${timestamp()}.json
```

**Result structure**:

```
itea-weather-data-lake-storage-yourname/
└── raw-data/
    └── weatherPlatform/
        └── telemetry/
            ├── itea-lab-room-a/
            │   ├── 1696118400000.json
            │   └── 1696118460000.json
            └── outdoor-station-1/
                ├── 1696118400000.json
                └── 1696118460000.json
```

### IAM Role Policy

The IoT Rule requires permissions to write to S3. Create a policy with:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject"],
      "Resource": [
        "arn:aws:s3:::itea-weather-data-lake-storage-yourname/raw-data/weatherPlatform/telemetry/*"
      ]
    }
  ]
}
```

### Test the Rule

1. **Use IoT Core Message Client**:

   - **Topic**: `weatherPlatform/telemetry/test-location`
   - **Message**:
     ```json
     {
       "deviceId": "test-device-01",
       "timestamp": "2024-01-15T10:30:00Z",
       "temperature": 25.5,
       "humidity": 60.2,
       "location": "test-location"
     }
     ```

2. **Check S3 bucket** for the stored message at:
   ```
   raw-data/weatherPlatform/telemetry/test-location/{timestamp}.json
   ```

{{% notice warning %}}
**Important**: Replace `itea-weather-data-lake-storage-yourname` with your actual unique bucket name in both the S3 action and IAM policy.
{{% /notice %}}

{{% notice info %}}
This rule ensures all weather device messages are automatically stored in the data lake, ready for AWS Glue processing.
{{% /notice %}}

### Verification

After creating the rule, verify:

- ✅ Rule is **Enabled**
- ✅ SQL statement syntax is correct
- ✅ S3 action points to your unique bucket name
- ✅ IAM role has proper S3 permissions

The IoT Rule Engine now automatically routes all weather telemetry to your S3 data lake for storage and further processing.
