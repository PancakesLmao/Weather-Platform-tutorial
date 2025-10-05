---
title: "Thing Groups Configuration"
weight: 1
chapter: false
pre: " <b> 4.1 </b> "
---

AWS IoT Thing Groups provide a mechanism to manage multiple devices collectively. All weather devices are organized under a central Thing Group for simplified policy management and device organization.

## Create Thing Group

1. Navigate to **AWS IoT Core** Console
2. Go to **Manage** → **Thing Groups**
3. Click **Create thing group**
4. Choose **Create static thing group**
5. Enter Thing Group name: `ITeaWeatherHub`
6. Optionally add description and tag fcj_workshop1 - FCJ Workshop 1 (For resource management)
7. Click **Create thing group**

![Search IoT Core](/images/4-iotcore/1.png)
![Configure Thing Group](/images/4-iotcore/2.png)
![Create Thing Group](/images/4-iotcore/3.png)

{{% notice tip %}}
Thing Groups allow you to apply policies and manage multiple devices as a single unit, making device management much easier.
{{% /notice %}}

{{% notice info %}}
The Thing Group is now ready to accept weather station devices. When devices are registered through the platform, they will automatically be added to this group.
{{% /notice %}}
