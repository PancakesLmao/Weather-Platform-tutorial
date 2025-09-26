---
title: "Prerequisites & Setup"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

## Development Environment Setup

### Required Software

Before starting, ensure you have the following installed:

- **Node.js 18 or higher** - [Download from nodejs.org](https://nodejs.org/)
- **pnpm package manager** - Install with `npm install -g pnpm`
- **AWS CLI** - [Installation guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- **Git** - For cloning the repository

### Verify Installation

```bash
# Check Node.js version
node --version  # Should be 18+

# Check pnpm
pnpm --version

# Check AWS CLI
aws --version

# Check Git
git --version
```

## AWS Account Setup

### Step 1: AWS Account Requirements

- An AWS Account with appropriate permissions
- Access to create IAM users and policies
- Billing alerts configured (recommended)

### Step 2: Create IAM User for Development

1. **Go to AWS Console** → IAM → Users → "Create user"

2. **User details:**

   - Username: `ws1-amplify`
   - Select "Provide user access to the AWS Management Console" if needed

3. **Set permissions:**

   - Choose "Attach policies directly"
   - Add these managed policies:
     - `AmplifyBackendDeployFullAccess`
     - `AWSCloudFormationReadOnlyAccess`
     - `IAMFullAccess` (for development only)
     - `AWSIoTFullAccess`

4. **Create Access Keys:**
   - Go to user → Security credentials → "Create access key"
   - Choose "Command Line Interface (CLI)"
   - Save the **Access Key ID** and **Secret Access Key**

{{% notice warning %}}
Store your AWS credentials securely and never commit them to version control!
{{% /notice %}}

### Step 3: Configure AWS CLI Profile

Set up a named profile for this project:

```bash
# Configure AWS CLI with your profile
aws configure --profile weather-platform

# Enter when prompted:
# AWS Access Key ID: [your-access-key-id]
# AWS Secret Access Key: [your-secret-access-key]
# Default region name: us-east-1 (or your preferred region)
# Default output format: json
```

Verify the profile:

```bash
aws sts get-caller-identity --profile weather-platform
```

## Edge Device Setup (Optional)

{{% notice info %}}
You can either use physical IoT devices or simulate data for development. Physical devices are optional for learning the platform.
{{% /notice %}}

### Hardware Options

- **ESP32 Development Board** - For sensor data collection
- **DF Robot Weather Module** - Complete weather sensor kit
- **Raspberry Pi Model 3B+** or higher - Edge computing device

{{% notice info %}}
Your Raspberry Pi must be from model 3B or higher to run the provided source code!
{{% /notice %}}

### Mock Data Alternative

For development and testing, you can write your own script to send messages over Wi-Fi, acting as a publisher in the MQTT network.

Make sure any message published to the Raspberry Pi (or any edge device running the provided source code) follows this format:
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

## Content

- [Setup Edge Device](2.1-setupedge/)
- [Setup Cloud Environment](2.2-setupcloud/)
