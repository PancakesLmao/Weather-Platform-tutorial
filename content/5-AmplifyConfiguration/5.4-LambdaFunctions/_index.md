---
title: "Lambda Functions Setup"
weight: 4
chapter: false
pre: " <b> 5.4 </b> "
---

## Lambda Functions Overview

The Weather Platform uses several Lambda functions to manage IoT devices and provide APIs for data access.

### Function Overview

**IoT Device Management:**

- `addThing`: Register new weather devices
- `deleteThing`: Remove weather devices
- `fetchThings`: List all registered devices
- `getIoTEndpoint`: Get IoT Core endpoint URL

**Data Access:**

- `getDataset`: Retrieve processed weather datasets
- `getTotalReadings`: Get telemetry statistics

### Key Functions Detailed

#### addThing Function

**Purpose**: Register new IoT weather devices
**Trigger**: API Gateway (POST)
**Functionality**:

- Creates IoT Thing in AWS IoT Core
- Adds device to ITeaWeatherHub Thing Group
- Attaches WeatherStationPolicies
- Returns device credentials

#### getTotalReadings Function

**Purpose**: Get statistics from the data lake
**Integration**: Connects to `itea-weather-data-lake-storage`
**Functionality**:

- Scans S3 bucket for telemetry files
- Counts total readings per location
- Returns aggregated statistics

#### getDataset Function

**Purpose**: Retrieve processed weather datasets
**Integration**: Connects to processed data bucket
**Functionality**:

- Lists available datasets
- Provides download URLs
- Handles CloudFront integration

### Deployment and Configuration

Lambda functions are automatically deployed with the Amplify backend:

```bash
# Functions deploy with sandbox
npx ampx sandbox --profile ws1-amplify
```

### Environment Variables

Functions automatically receive:

- IoT Core endpoint URL
- S3 bucket names
- Cognito User Pool information
- AWS region configuration

{{% notice info %}}
Lambda functions are fully integrated with Amplify's authentication and authorization system.
{{% /notice %}}
