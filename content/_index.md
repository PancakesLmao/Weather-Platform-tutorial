---
title: "Building a centralized IoT Platform for Weather station"
weight: 1
chapter: false
---

# Building a centralized IoT Platform for Weather station

### Overall

This workshop introduces the IoT Weather Platform, a serverless AWS-based solution developed for the ITea Lab in Ho Chi Minh City to enhance weather data collection and analysis for research purposes. The tutorial is designed for lab members and community participants to understand and implement the platform systematically.

The platform is powered by modern tools such as Next.js 15, AWS Amplify Gen 2, IoT Core, and a comprehensive data pipeline that supports both real-time weather monitoring and historical data analysis.

![Overview](/images/arc-log.png)

### Content

1. [Introduction](1-introduce/)
2. [Prerequisites & Setup](2-prerequiste/)
3. [Create Data Lake](3-createdatalake/)
4. [Setup AWS IoT Core](4-setupiotcore/)
5. [Amplify Configuration](5-amplifyconfiguration/)
6. [Resource Cleanup](6-resourcecleanup/)
7. [Pricing Plan](7-pricingplan/)

### Architecture Overview

The Weather Platform provides:

- **IoT Device Management**: Register and manage weather sensors
- **Real-time Data Collection**: Stream weather data via MQTT
- **Data Lake Storage**: Raw sensor data storage in S3
- **ETL Processing**: Transform raw data with AWS Glue
- **Global Distribution**: Serve datasets via CloudFront
- **Web Dashboard**: Next.js 15 application for data visualization

### Key Technologies

- **Frontend**: Next.js 15 with App Router, TypeScript, Tailwind CSS
- **Backend**: AWS Amplify Gen 2 with custom CDK constructs
- **IoT**: AWS IoT Core with MQTT protocol
- **Data Processing**: AWS Glue for ETL operations
- **Storage**: Amazon S3 for data lake and processed datasets
- **CDN**: CloudFront for global content delivery
- **Authentication**: AWS Cognito User Pools
