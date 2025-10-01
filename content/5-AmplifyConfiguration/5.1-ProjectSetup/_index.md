---
title: "Project Setup and Inspection"
weight: 1
chapter: false
pre: " <b> 5.1 </b> "
---

## Clone the Weather Platform Project

Start by cloning the Weather Platform repository and exploring its structure.

### Step 1: Clone Repository

```bash
# Clone the Weather Platform project
git clone https://github.com/Itea-Lab/Weather-Platform.git
cd Weather-Platform
```

### Step 2: Install Dependencies

The project uses **pnpm** as the package manager:

```bash
# Install dependencies with pnpm
pnpm install
```

### Step 3: Project Structure Inspection

Explore the key directories:

```
Weather-Platform/
├── amplify/                    # Amplify backend configuration
│   ├── backend.ts             # Main backend definition
│   ├── auth/                  # Cognito authentication
│   ├── custom/                # Custom CDK constructs
│   │   ├── CloudFront/        # CDN configuration
│   │   ├── EventBridge/       # Scheduled data processing
│   │   ├── GetDataset/        # Dataset API
│   │   ├── WeatherDataGlue/   # ETL processing
│   │   └── WeatherDatasetStorage/ # S3 storage
│   └── functions/             # Lambda functions
│       ├── addThing/          # Add IoT devices
│       ├── deleteThing/       # Remove IoT devices
│       ├── fetchThings/       # List IoT devices
│       ├── getDataset/        # Retrieve processed data
│       ├── getIoTEndpoint/    # Get IoT endpoint URL
│       └── getTotalReadings/  # Get telemetry statistics
├── src/                       # Next.js frontend
│   ├── app/                   # App Router pages
│   ├── components/            # React components
│   ├── contexts/              # React contexts
│   ├── hooks/                 # Custom hooks
│   ├── lib/                   # Utility libraries
│   └── types/                 # TypeScript type definitions
│   └── utils/                 # Utility functions
├── public/                    # Static assets
├── package.json               # Dependencies and scripts
└── amplify_outputs.json       # Generated after deployment
```

### Step 4: Understand Backend Architecture

The `amplify/backend.ts` file defines:

**Core Services:**

- **Authentication**: Cognito User Pool and Identity Pool
- **Storage**: Custom S3 bucket with lifecycle policies
- **Functions**: Lambda functions for IoT device management
- **Data Processing**: AWS Glue for ETL operations
- **Content Delivery**: CloudFront distribution

**Custom CDK Constructs:**

- **WeatherDatasetStorage**: Advanced S3 configuration
- **WeatherDataGlue**: ETL processing pipeline
- **CloudFront**: Global content delivery
- **EventBridge**: Scheduled data processing

### Step 5: Environment Configuration

Create `.env.local` for development:

```bash
# Copy environment template
cp .env.example .env.local

# Edit with your specific values
# DEFAULT_REGION=us-east-1
```

{{% notice info %}}
The `amplify_outputs.json` file is **generated** after backend deployment and contains all the necessary configuration for the frontend.
{{% /notice %}}

{{% notice warning %}}
Remember to update the hardcoded bucket names in the backend code as mentioned in the Data Lake section.
{{% /notice %}}