---
title: "Production Deployment"
weight: 8
chapter: false
pre: " <b> 5.8 </b> "
---

## Deploy to Production

Deploy the Weather Platform to production using AWS Amplify Hosting with CloudFormation-managed infrastructure.

### Prerequisites

- Sandbox environment tested and working
- All hardcoded bucket names updated in backend code
- Git repository pushed to GitHub/GitLab
- AWS profile `ws1-amplify` configured with deployment permissions

### Step 1: Prepare for Production

**Build and test locally:**

```bash
# Build the Next.js application
pnpm build

# Test the production build locally
pnpm start
```

**Commit all changes:**

```bash
# Ensure all backend configuration changes are committed
git add .
git commit -m "Update backend configuration for production deployment"
git push origin main
```

### Step 2: Deploy via Amplify Console

**Method A: AWS Amplify Console (Recommended)**

1. **Go to [AWS Amplify Console](https://console.aws.amazon.com/amplify/)**
2. **Click "New app" → "Host web app"**
3. **Connect your Git repository:**

   - Choose GitHub as source
   - Authorize AWS Amplify to access your repository
   - Select the Weather Platform repository
   - Choose `prod` branch

![Create Amplify App](/images/5-amplifyConfiguration/5.8-amplify-deployment/1.png)
![Amplify Connect Repo](/images/5-amplifyConfiguration/5.8-amplify-deployment/2.png)
![Add Repo and Branch](/images/5-amplifyConfiguration/5.8-amplify-deployment/3.png)
![App Settings](/images/5-amplifyConfiguration/5.8-amplify-deployment/4.png)
![Create Service Role](/images/5-amplifyConfiguration/5.8-amplify-deployment/5.png)
![Deploy Amplify App](/images/5-amplifyConfiguration/5.8-amplify-deployment/6.png)

4. **Configure build settings** (auto-detected):

```yaml
version: 1
backend:
  phases:
    preBuild:
      commands:
        - export NODE_OPTIONS="--max-old-space-size=3072"
    build:
      commands:
        - corepack enable
        - pnpm install --frozen-lockfile
        - pnpm exec ampx pipeline-deploy --branch $AWS_BRANCH --app-id
          $AWS_APP_ID
frontend:
  phases:
    preBuild:
      commands:
        - export NODE_OPTIONS="--max-old-space-size=4096"
    build:
      commands:
        - pnpm run build
  artifacts:
    baseDirectory: .next
    files:
      - "**/*"
```

5. **Environment variables** (if needed):

   - `NODE_ENV`: `production`
   - Any custom environment variables

6. **Deploy:** Click "Save and deploy"

### Step 3: Production Resources

**Amplify automatically creates:**

- **Production CloudFormation stacks** for all backend resources
- **CloudFront distribution** for global content delivery
- **SSL certificates** with automatic renewal
- **Custom domain support** (optional)
- **Branch-based deployments** for different environments
- **Atomic deployments** with instant rollback capability

**Backend resources deployed:**

- Production versions of all Lambda functions
- S3 buckets with production naming (`weather-dataset-{account-id}-main`)
- Cognito User Pools and Identity Pools
- AWS Glue databases, crawlers, and ETL jobs
- CloudFront distribution for processed datasets
- EventBridge rules for scheduled processing

### Step 4: Post-Deployment Configuration

#### IoT Policy Attachment

For each user, manually attach IoT policies (see [Section 5.7](../5.7-authenticationiot/)):

```bash
# Get user's Cognito Identity ID and attach policy
aws iot attach-policy --policy-name WeatherPlatformPubSubPolicy \
  --target "us-east-1:12345678-1234-1234-1234-123456789012" \
  --profile ws1-amplify
```

#### Custom Domain (Optional)

1. **In Amplify Console:** Go to "Domain management"
2. **Add custom domain:** `weather.yourdomain.com`
3. **Configure DNS:** Add CNAME record as instructed
4. **SSL certificate:** Automatically provisioned by AWS

### Step 5: Verify Production Deployment

**Frontend verification:**

- Website accessible via Amplify domain
- Authentication working (sign up/sign in)
- Dashboard loading correctly
- Real-time IoT data streaming

**Backend verification:**

- Lambda functions responding
- S3 buckets created with correct names (Test run by activating Step functions)
- Glue ETL pipeline running
- CloudFront serving processed data

### Monitoring and Maintenance

**CloudWatch dashboards:**

- Application performance metrics
- Error rates and latency
- Cost tracking and optimization

**Amplify Console features:**

- Build history and logs
- Performance monitoring
- User analytics

### Troubleshooting Production Issues

**Common deployment issues:**

1. **Build failures:**

   ```bash
   # Check build logs in Amplify Console
   # Verify dependencies in package.json
   ```

2. **Backend deployment errors:**

   ```bash
   # Check CloudFormation events
   # Verify IAM permissions
   ```

3. **Domain issues:**
   ```bash
   # Verify DNS configuration
   # Check SSL certificate status
   ```

{{% notice success %}}
**CloudFormation Advantage**: Amplify uses CloudFormation for infrastructure deployment, ensuring consistent, repeatable deployments with automatic rollback on failures.
{{% /notice %}}

{{% notice info %}}
Production deployment typically takes 10-15 minutes for initial setup. Subsequent deployments take 5-7 minutes for frontend changes only.
{{% /notice %}}

### Next Steps

- Monitor application performance
- Set up production monitoring and alerts
- Configure backup and disaster recovery
- Plan for [resource cleanup](../../6-resourcecleanup/) when done
