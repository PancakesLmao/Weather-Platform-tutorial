---
title: "IoT Security Policies"
weight: 2
chapter: false
pre: " <b> 4.2 </b> "
---

Two security policies are required for proper platform operation: Device Policy for weather stations and Platform Policy for user access.

## Device Policy (WeatherStationPolicies)

This policy enables weather devices to connect and publish telemetry data to IoT Core.

### Step 1: Create Device Policy

1. Navigate to **AWS IoT Core** Console
2. Go to **Secure** → **Policies**
3. Click **Create policy**
4. Enter Policy name: `WeatherStationPolicies`
5. Add tag (optional): fcj_workshop1 - FCJ Workshop 1
6. Copy and paste the following policy document:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iot:Publish",
      "Resource": "arn:aws:iot:<region>:<account_ID>:topic/weatherPlatform/telemetry/*"
    },
    {
      "Effect": "Allow",
      "Action": "iot:Connect",
      "Resource": "arn:aws:iot:<region>:<account_ID>:client/*"
    }
  ]
}
```
![Configure Policy](/images/4-iotcore/4.png)
![Create Policy](/images/4-iotcore/5.png)
7. Click **Create**

### Step 2: Attach Policy to Thing Group

1. Navigate to IoT Core → **All devices** → **Thing Groups**
2. Select **ITeaWeatherHub** Thing Group
3. Go to **Policies** tab → **Manage Policies**
4. Click **Attach policy**
5. Select **WeatherStationPolicies**
6. Click **Attach**

![Configure Policy](/images/4-iotcore/6.png)
![Update Policy](/images/4-iotcore/7.png)

{{% notice info %}}
This policy assignment automatically applies to all devices within the Thing Group.
{{% /notice %}}

## Platform Policy (WeatherPlatformPubSubPolicy)

This policy enables the web platform to interact with IoT devices, receive telemetry data, and send notifications.

### Step 1: Create Platform Policy

1. In **AWS IoT Core** Console, go to **Secure** → **Policies**
2. Click **Create policy**
3. Enter Policy name: `WeatherPlatformPubSubPolicy`
4. Add tag (optional): fcj_workshop1 - FCJ Workshop 1
5. Copy and paste the following policy document:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iot:Connect",
      "Resource": "arn:aws:iot:<region>:<account_ID>:client/*"
    },
    {
      "Effect": "Allow",
      "Action": "iot:Receive",
      "Resource": "arn:aws:iot:<region>:<account_ID>:topic/weatherPlatform/*"
    },
    {
      "Effect": "Allow",
      "Action": "iot:Subscribe",
      "Resource": [
        "arn:aws:iot:<region>:<account_ID>:topicfilter/weatherPlatform/telemetry/*",
        "arn:aws:iot:<region>:<account_ID>:topicfilter/weatherPlatform/notifications"
      ]
    },
    {
      "Effect": "Allow",
      "Action": "iot:Publish",
      "Resource": "arn:aws:iot:<region>:<account_ID>:topic/weatherPlatform/notifications"
    }
  ]
}
```

6. Click **Create**

{{% notice warning %}}
This policy is attached to individual Cognito Identity IDs to grant authenticated users access to IoT Core resources. See the User IoT Policy Attachment section for detailed implementation steps.
{{% /notice %}}

## Verification

After creating both policies, you should see:

- **WeatherStationPolicies**: Attached to ITeaWeatherHub Thing Group
- **WeatherPlatformPubSubPolicy**: Ready for user assignment

## Next Steps

- Complete Amplify backend deployment
- Set up user authentication and policy attachment
