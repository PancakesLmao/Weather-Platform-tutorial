---
title: "Setup Amplify Weather Platform"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

In this section, we'll deploy the Weather Platform using AWS Amplify Gen 2. This will create all the necessary backend infrastructure including Lambda functions, IoT Core, S3 storage, Cognito authentication, and the data processing pipeline.

{{% notice info %}}
AWS Amplify Gen 2 uses a modern approach with CDK-based custom constructs for advanced infrastructure components while maintaining simplicity for core features.
{{% /notice %}}

![Amplify Architecture](/images/amplify-architecture.png)

## Development Deployment (Sandbox)

### Step 1: Verify Prerequisites

Ensure you have completed the prerequisites from Section 2:

```bash
# Verify you're in the project directory
pwd  # Should show Weather-Platform directory

# Verify AWS CLI profile
aws sts get-caller-identity --profile weather-platform

# Verify dependencies are installed
pnpm list --depth=0
```

### Step 2: Deploy to Amplify Sandbox

The sandbox environment creates isolated AWS resources perfect for development and testing:

```bash
# Deploy to sandbox environment
npx ampx sandbox --profile weather-platform
```

{{% notice tip %}}
The sandbox deployment typically takes 5-10 minutes. You'll see real-time progress as AWS resources are created.
{{% /notice %}}

This command will create:

- **Lambda Functions**: Device management, data processing, IoT endpoint configuration
- **IoT Core**: Device registry, certificates, policies, and MQTT topics
- **S3 Bucket**: Weather data storage with CloudFront CDN
- **Cognito**: User pools and identity pools for authentication
- **Glue ETL**: Data transformation pipeline
- **EventBridge**: Scheduled data processing events
- **Step Functions**: Orchestrated workflows for data processing

### Step 3: Monitor Deployment Progress

During deployment, you'll see output similar to:

```
✨ Amplify Sandbox

✅ Amplify project setup complete!
✅ Building backend...
✅ Deploying backend...

📋 Amplify outputs:
- Auth: Cognito User Pool created
- Functions: 6 Lambda functions deployed
- Storage: S3 bucket with CloudFront distribution
- Custom: IoT Core and Glue pipeline configured

🌐 Sandbox URL: https://sandbox.amplifyapp.com/[app-id]
```

### Step 4: Configure Environment

After successful deployment, Amplify automatically generates `amplify_outputs.json` with your backend configuration:

```json
{
  "auth": {
    "user_pool_id": "us-east-1_xxxxxxxxx",
    "user_pool_client_id": "xxxxxxxxxxxxxxxxxx",
    "identity_pool_id": "us-east-1:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  },
  "custom": {
    "iot_endpoint": "xxxxxxxxxx-ats.iot.us-east-1.amazonaws.com",
    "s3_bucket": "amplify-weather-platform-xxxxxxxx"
  }
}
```

{{% notice warning %}}
Never commit `amplify_outputs.json` to version control as it contains sensitive configuration data.
{{% /notice %}}

## Local Development

### Step 1: Start Development Server

With the sandbox running, start the frontend development server:

```bash
# Start Next.js development server
pnpm dev
```

### Step 2: Access the Application

Open your browser and navigate to:

- **Root**: http://localhost:3000
- **Dashboard**: http://localhost:3000/dashboard
- **Devices**: http://localhost:3000/devices
- **Dataset**: http://localhost:3000/dataset

### Step 3: Create Your First User

1. Navigate to http://localhost:3000
2. Click **"Sign Up"** to create a new account
3. Enter your email and password
4. Verify your email address
5. Sign in to access the dashboard

{{% notice info %}}
The first user you create will have admin privileges and can manage IoT devices.
{{% /notice %}}

## Testing IoT Functionality

### Step 1: Register a Test Device

1. Go to **Devices** page (http://localhost:3000/devices)
2. Click **"Add Device"**
3. Enter device details:
   - **Device Name**: `test-weather-station-01`
   - **Location**: `Lab Room A`
   - **Description**: `Test weather station for development`
4. Click **"Register Device"**

### Step 2: Simulate Weather Data

For development, you can use the built-in data simulator:

```bash
# Run the mock data generator (if available)
npm run simulate-data

# Or manually publish test data using AWS CLI
aws iot-data publish \
  --topic "weather/test-weather-station-01/data" \
  --payload '{"temperature":25.5,"humidity":60,"pressure":1013.25,"timestamp":"2024-01-15T10:30:00Z"}' \
  --profile weather-platform
```

### Step 3: Verify Real-time Updates

1. Navigate to the **Dashboard** (http://localhost:3000/dashboard)
2. Select your test device from the dropdown
3. You should see real-time data updates
4. Verify connection status indicators

## Production Deployment

### Step 1: Deploy Backend to Production

```bash
# Deploy backend to production branch
npx ampx pipeline-deploy --branch prod --outputs-format json --profile weather-platform
```

### Step 2: Set Up Amplify Hosting

1. Go to [AWS Amplify Console](https://console.aws.amazon.com/amplify/)
2. Click **"Create app"** → **"Host web app"**
3. Choose **GitHub** and select your repository
4. Configure build settings:
   - **App name**: `weather-platform-prod`
   - **Branch**: `prod`
   - **Build command**: `pnpm build`
   - **Build output directory**: `.next`
   - **Node.js version**: `18`

### Step 3: Environment Variables

Add these environment variables in Amplify Console:

- `DEFAULT_REGION`: `us-east-1` (or your region)
- `NODE_ENV`: `production`

### Step 4: Deploy and Access

1. Click **"Save and deploy"**
2. Wait for deployment (5-10 minutes)
3. Access your production app at: `https://prod.[app-id].amplifyapp.com`

## Troubleshooting

### Common Issues

**Deployment Fails:**

```bash
# Check Amplify status
npx ampx info

# View detailed logs
npx ampx sandbox --verbose
```

**Authentication Issues:**

- Verify Cognito user pool configuration
- Check `amplify_outputs.json` is generated correctly
- Ensure proper IAM permissions

**IoT Connection Problems:**

- Verify IoT endpoint in configuration
- Check device certificates and policies
- Test MQTT connection manually

### Getting Help

```bash
# Get troubleshooting information
npx ampx info

# View Amplify documentation
npx ampx docs
```

## Next Steps

With Amplify successfully deployed, you can now:

1. **Configure IoT devices** to send real weather data
2. **Set up data processing** for historical analytics
3. **Customize the dashboard** for your specific needs
4. **Add more weather stations** to expand coverage

{{% notice success %}}
Congratulations! You've successfully deployed the Weather Platform using AWS Amplify Gen 2. The platform is now ready for real-time weather monitoring and data analysis.
{{% /notice %}}
