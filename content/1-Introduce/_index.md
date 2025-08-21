---
title: "Introduction"
weight: 1
chapter: false
pre: " <b> 1. </b> "
---
## Problem statement
Current weather stations at the ITea Lab rely on manual data collection, which is inefficient when managing multiple units. The absence of a centralized system for real-time data access and analytics, combined with the high costs and complexity of third-party platforms, limits research capabilities and operational efficiency.
## Solution Architecture
The IoT Weather Platform addresses these challenges through a serverless AWS architecture. Raspberry Pi edge devices, equipped with ESP32 sensors, collect weather data and transmit it via MQTT to AWS IoT Core. Data is stored in an Amazon S3 data lake, processed using AWS Glue for cataloging and ETL jobs, and analyzed for insights. A Next.js web application, hosted on AWS Amplify, provides a user interface for real-time dashboards and trend analysis, with access secured by Amazon Cognito for up to five lab members. The system supports five weather stations, with scalability to 15, and maintains low operational costs at approximately under $5 per month.

![ConnectPrivate](/images/arc-log.png)

