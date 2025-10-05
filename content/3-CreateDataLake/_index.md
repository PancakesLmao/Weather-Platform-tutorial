---
title: "Create Data Lake"
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

## S3 Data Lake for IoT Messages

The Weather Platform uses Amazon S3 as a simple data lake to store IoT messages from weather sensors. This is the primary storage for raw sensor data.

### What is the Data Lake?

The data lake is an S3 bucket named `itea-weather-data-lake-storage` that stores:

- Raw IoT sensor messages (JSON format)
- AWS Glue ETL scripts for data processing
### Create S3 Bucket
![Create Bucket](/images/3-datalake/1.png)
![Create Bucket](/images/3-datalake/2.png)
![Create Bucket](/images/3-datalake/3.png)
![Create Bucket](/images/3-datalake/4.png)
Click **Create bucket** to finish.
### Data Lake Bucket Structure

The data lake is organized with the following folder structure:

```
itea-weather-data-lake-storage/
├── raw-data/                   # Raw IoT sensor messages
│   └── weatherPlatform/        # Weather platform data
│       └── telemetry/          # Telemetry data organized by location
│           ├── {location-1}/   # e.g., "itea-lab-room-a"
│           │   └── sensor-data.json
│           ├── {location-2}/   # e.g., "outdoor-station-1"
│           │   └── sensor-data.json
│           └── {location-n}/   # Additional locations
│               └── sensor-data.json
└── glue-scripts/               # AWS Glue ETL scripts
    └── weather-transform.py    # Data transformation script
```

### IoT Message Storage

When IoT devices send weather data, the messages are stored in:

```
raw-data/weatherPlatform/telemetry/{location}/
```

Where `{location}` is the specific location identifier (e.g., `itea-lab-room-a`, `outdoor-station-1`).

### Important Notes

{{% notice warning %}}
**Critical**: S3 bucket names must be globally unique across all AWS accounts. Add your name or timestamp to ensure uniqueness: `itea-weather-data-lake-storage-yourname`

**Backend Code Update Required**: The Amplify backend code has hardcoded bucket names that must be updated to match your unique bucket name.
{{% /notice %}}

{{% notice info %}}
This data lake bucket is separate from the processed dataset bucket. Raw IoT sensor data flows into this bucket, while processed datasets are stored in a different bucket.
{{% /notice %}}

### Update Backend Code

After creating your unique bucket name, you **must** update the hardcoded references in the Amplify backend:

**Files to Update:**

1. **`amplify/backend.ts`** (Lines 233, 272, 273, 318):

   ```typescript
   // Change from:
   sourceBucketName: "itea-weather-data-lake-storage";

   // To your unique name:
   sourceBucketName: "itea-weather-data-lake-storage-yourname";
   ```

2. **`amplify/functions/getTotalReadings/handler.ts`** (Line 16):

   ```typescript
   // Change from:
   const bucket = "itea-weather-data-lake-storage";

   // To your unique name:
   const bucket = "itea-weather-data-lake-storage-yourname";
   ```

3. **`amplify/custom/WeatherDataGlue/resource.ts`** (Lines 65, 66, 195):

   ```typescript
   // Change all S3 ARNs and paths from:
   arn:aws:s3:::itea-weather-data-lake-storage
   s3://itea-weather-data-lake-storage/glue-scripts/

   // To your unique name:
   arn:aws:s3:::itea-weather-data-lake-storage-yourname
   s3://itea-weather-data-lake-storage-yourname/glue-scripts/
   ```

{{% notice tip %}}
Use Find & Replace in your code editor to quickly update all instances of `itea-weather-data-lake-storage` to your unique bucket name.
{{% /notice %}}

### Next Steps

With the S3 data lake configured:

- IoT devices send messages that get stored in `raw-data/weatherPlatform/telemetry/{location}/`
- AWS Glue scripts in `glue-scripts/` process the raw data
- Processed results are stored in a separate dataset bucket