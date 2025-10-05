---
title: "Resource Cleanup"
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

## AWS Resource Cleanup

Properly clean up all Weather Platform resources to avoid ongoing charges. Amplify's CloudFormation-based infrastructure makes cleanup straightforward with simple commands.

### Quick Cleanup Commands

#### Development Sandbox Cleanup

```bash
# Single command to delete entire sandbox environment
npx ampx sandbox delete --profile ws1-amplify
```

#### Production Environment Cleanup

```bash
# Delete production deployment
npx ampx pipeline-deploy --branch main --appId <your-app-id> --delete --profile ws1-amplify
```

### What Gets Cleaned Up

**Amplify automatically removes:**

- All CloudFormation stacks
- Lambda functions and their execution roles
- S3 buckets and contents (if configured for auto-deletion)
- Cognito User Pools and Identity Pools
- AWS Glue databases, crawlers, and jobs
- CloudFront distributions
- EventBridge rules
- IAM roles and policies created by Amplify

### Manual Cleanup Required

Some resources may need manual deletion:

#### IoT Core Resources

```bash
# Delete IoT policies
aws iot delete-policy --policy-name WeatherStationPolicies --profile ws1-amplify
aws iot delete-policy --policy-name WeatherPlatformPubSubPolicy --profile ws1-amplify

# Delete IoT Rule
aws iot delete-topic-rule --rule-name WeatherTelemetryToS3Rule --profile ws1-amplify

# Delete Thing Group
aws iot delete-thing-group --thing-group-name ITeaWeatherHub --profile ws1-amplify
```

#### Data Lake Bucket

```bash
# Empty and delete data lake bucket
aws s3 rm s3://itea-weather-data-lake-storage-yourname --recursive --profile ws1-amplify
aws s3 rb s3://itea-weather-data-lake-storage-yourname --profile ws1-amplify
```

### Verification Steps

After cleanup, verify all resources are deleted:

1. **AWS Console Verification:**

   - **Amplify**: Check no apps remain
   - **CloudFormation**: Verify stacks are deleted
   - **S3**: Confirm buckets are removed
   - **Lambda**: Check no functions remain
   - **Cognito**: Verify User Pools are deleted

2. **CLI Verification:**

   ```bash
   # Check remaining S3 buckets
   aws s3 ls --profile ws1-amplify | grep weather

   # Check remaining Lambda functions
   aws lambda list-functions --profile ws1-amplify | grep weather

   # Check CloudFormation stacks
   aws cloudformation list-stacks --profile ws1-amplify | grep amplify
   ```

### Cost Monitoring

**Post-cleanup verification:**

1. **AWS Cost Explorer**: Check for any remaining charges
2. **Billing Dashboard**: Monitor for unexpected costs over next few days
3. **Budget Alerts**: Set up alerts for unusual spending

{{% notice info %}}
**Amplify Advantage**: Unlike manual AWS setups, Amplify's CloudFormation-based approach ensures complete and reliable cleanup of all related resources.
{{% /notice %}}

### Emergency Cleanup

If standard cleanup fails:

```bash
# Force delete CloudFormation stacks
aws cloudformation delete-stack --stack-name amplify-<app-name> --profile ws1-amplify

# List and manually delete remaining resources
aws resourcegroupstaggingapi get-resources --tag-filters Key=amplify:project-name --profile ws1-amplify
```

{{% notice success %}}
Complete cleanup ensures you avoid the platform monthly cost. For detailed cost analysis, see the [Pricing Plan section](../7-pricingplan/).
{{% /notice %}}
