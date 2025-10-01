---
title: "Pricing Plan"
weight: 7
chapter: false
pre: " <b> 7. </b> "
---

## Weather Platform Cost Analysis

Understanding the cost structure of the Weather Platform helps with budget planning and optimization decisions.

### Total Cost Summary

| Period      | Cost (USD) |
| ----------- | ---------- |
| **Monthly** | **$0.74**  |
| **Annual**  | **$8.88**  |
| **Upfront** | **$0.00**  |

{{% notice info %}}
Cost estimate based on AWS Pricing Calculator as of October 1, 2025. Actual costs may vary based on usage patterns and AWS pricing changes.
{{% /notice %}}

### Cost Breakdown by Service

#### Core Platform Services

**AWS Amplify - $0.19/month ($2.28/year)**

- **Usage**: Web app hosting and CI/CD pipeline
- **Build time**: 5 minutes per month
- **Instance**: Standard (8 GB Memory, 4 vCPUs)
- **Storage**: 0.03 GB data stored
- **SSR requests**: 20 per day
- **Data served**: 0.003 GB per month

**AWS Glue ETL - $0.36/month ($4.32/year)**

- **Usage**: Data processing and transformation
- **Apache Spark DPUs**: 2 units
- **Python Shell DPUs**: 0.0625 units
- **Processing**: Daily ETL jobs for weather data

#### Data Storage & Processing

**AWS Glue Crawlers - $0.07/month ($0.84/year)**

- **Usage**: Automatic schema discovery
- **Crawlers**: 1 crawler for data lake

**AWS Glue Data Catalog - $0.03/month ($0.36/year)**

- **Usage**: Metadata storage and access
- **Requests**: 3,000 access requests per month
- **Objects**: 3,000 objects stored per month

**S3 Storage - $0.05/month ($0.60/year total)**

- **Data Lake (US East N. Virginia)**: $0.04/month
  - Storage: 1 GB standard storage
  - Requests: 1,000 PUT/COPY/POST, 20,000 GET requests
- **Dataset (US East Ohio)**: $0.01/month
  - Storage: 0.3 GB standard storage
  - Requests: 100 PUT/COPY/POST, 1,000 GET requests

#### Content Delivery & IoT

**Amazon CloudFront - $0.02/month ($0.24/year)**

- **Usage**: Global content delivery for datasets
- **HTTPS requests**: 20,000 per month
- **Data transfer**: 0.01 GB per month

**AWS IoT Core (MQTT) - $0.01/month ($0.12/year)**

- **Usage**: Weather sensor connectivity
- **Devices**: 3 IoT devices
- **Messages**: 1 message per device
- **Message size**: 430 bytes average
- **Rules**: 1 rule triggered per message

#### Compute & Authentication

**AWS Lambda - $0.01/month ($0.12/year)**

- **Usage**: Backend API functions
- **Architecture**: x86
- **Requests**: 1,000 per month
- **Ephemeral storage**: 512 MB

**Amazon Cognito - $0.00/month (Free Tier)**

- **Usage**: User authentication and authorization
- **Monthly Active Users**: 4 users
- **Advanced security**: Disabled (basic security included)

**AWS Step Functions - $0.00/month (Free Tier)**

- **Usage**: Dataset update orchestration
- **Workflow requests**: 4 per month
- **State transitions**: 70 per workflow

### Cost Optimization Strategies

#### Development vs Production

**Development (Sandbox)**:

- Use `npx ampx sandbox` for development
- Automatically shuts down when not in use
- Minimal data processing costs

**Production**:

- Fixed monthly costs as shown above
- Scales with actual usage

#### Free Tier Benefits

**Services with generous free tiers:**

- **Cognito**: 50,000 MAUs free (currently using 4)
- **Step Functions**: 4,000 state transitions free per month
- **Lambda**: 1M requests + 400,000 GB-seconds free per month
- **IoT Core**: 2.25M messages free per month

#### Scaling Considerations

**Cost scales with:**

- Number of IoT devices and message frequency
- Data processing volume (Glue ETL runtime)
- CloudFront requests and data transfer
- Storage growth over time

**Cost remains fixed:**

- Amplify hosting (unless build time increases)
- Basic infrastructure components

### Regional Cost Differences

**US East (N. Virginia)** - Primary region:

- Most services hosted here
- Typically lowest AWS pricing
- Total regional cost: ~$0.73/month

**US East (Ohio)** - Secondary region:

- Dataset storage only
- Slightly different S3 pricing
- Total regional cost: ~$0.01/month

#### Annual Budget

**Current scale**: ~$10/year

- Small lab environment
- 3-4 weather stations
- Regular development activity

### Cost Monitoring

#### Cost Tags

**Tag all resources for tracking:**

- Project: WeatherPlatform
- Environment: Development/Production
- Owner: ITea-Lab
- Purpose: Research

### Legal Disclaimer

{{% notice warning %}}
AWS Pricing Calculator provides only an estimate of AWS fees and doesn't include any taxes that might apply. Your actual fees depend on various factors, including your actual usage of AWS services.
{{% /notice %}}

**Cost estimate details:**

- **Currency**: USD
- **Locale**: en_US
- **Created**: October 1, 2025
- Visit the [Repo](https://github.com/PancakesLmao/Weather-Platform-tutorial) to view the Pricing file