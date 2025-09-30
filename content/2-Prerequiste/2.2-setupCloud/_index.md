---
title : "Setup Cloud Environment"
weight : 2
chapter : false
pre : " <b> 2.2 </b> "
---

In this step, we'll set up the environment to interact with AWS resources from the developer's terminal using AWS CLI

## AWS Account Setup

### Step 1: AWS Account Requirements

- You will first need an AWS Account with appropriate permissions
- Give it access to create IAM users and policies
- Billing alerts configured (recommended)

### Step 2: Create IAM User for Development

1. **Go to AWS Console** → IAM → Users → "Create user"

2. **User details:**

   - Username: `ws1-amplify`
   - Select "Provide user access to the AWS Management Console" if needed

3. **Set permissions:**

   - Choose "Attach policies directly"
   - Add these managed policies:
     - `AmplifyBackendDeployFullAccess` (for deploying Amplify backend resources)
     - `AWSCloudFormationReadOnlyAccess` (to check resource creation and debug stack builds)
     - `IAMFullAccess` (to manage IAM policies and verify permissions)
     - `AWSIoTFullAccess` (for IoT Core interactions via CLI and attaching IoT policies)

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