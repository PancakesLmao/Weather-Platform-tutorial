---
title: "Amplify Configuration"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

## AWS Amplify Backend Setup

This section covers cloning the Weather Platform project, understanding its architecture, and deploying the Amplify backend with the correct configuration.

### Overview

The Weather Platform uses AWS Amplify Gen 2 with a custom backend configuration that includes:

- Next.js 15 frontend with App Router
- Custom CDK constructs for advanced AWS services
- Lambda functions for IoT device management
- AWS Glue for data processing
- CloudFront for global content delivery

### What You'll Learn

In this section, you will:

- Clone the Weather Platform repository
- Inspect the project structure and Amplify backend configuration
- Set up the development environment with pnpm
- Deploy the backend using AWS Amplify with the `ws1-amplify` profile
- Understand how custom CDK constructs integrate with Amplify

{{% notice info %}}
This section uses the `ws1-amplify` AWS profile. Make sure your profile is configured before proceeding.
{{% /notice %}}

### Prerequisites

Before starting, ensure you have:

- AWS CLI configured with `ws1-amplify` profile
- Node.js 18+ installed
- pnpm package manager installed
- Git for cloning the repository

### Section Structure

- **[5.1: Project Setup and Inspection](5.1-projectsetup/)** - Clone repository and explore project structure
- **[5.2: Amplify Backend Configuration](5.2-backendconfiguration/)** - Deploy sandbox with `ws1-amplify` profile
- **[5.3: Custom CDK Constructs Overview](5.3-customcdkresources/)** - Understanding advanced AWS integrations
- **[5.4: Lambda Functions Setup](5.4-lambdafunctions/)** - IoT device management and data APIs
- **[5.5: Data Processing Pipeline](5.5-dataprocessing/)** - AWS Glue ETL automation
- **[5.6: Content Distribution Setup](5.6-cloudfrontsetup/)** - CloudFront global delivery
- **[5.7: User Authentication and IoT Policy Management](5.7-authentication/)** - Cognito and IoT policies
- **[5.8: Production Deployment](5.8-productiondeployment/)** - Amplify Hosting deployment

{{% notice info %}}
Amplify backend runs CloudFormation under the hood, making it quick to deploy sandbox infrastructure and easily integrate with existing projects. The sandbox environment allows rapid development and testing before production deployment.
{{% /notice %}}

{{% notice info %}}
**Development Cleanup**: For sandbox environments, you can easily delete all resources with a single command: `npx ampx sandbox delete --profile ws1-amplify`. See [Section 6: Resource Cleanup](../6-resourcecleanup/) for complete cleanup procedures.
{{% /notice %}}
