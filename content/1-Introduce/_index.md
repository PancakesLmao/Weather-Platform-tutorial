---
title: "Introduction"
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

## Problem Statement

Current weather stations at the ITea Lab rely on manual data collection, which is inefficient when managing multiple units. The absence of a centralized system for real-time data access and analytics, combined with the high costs and complexity of third-party platforms, limits research capabilities and operational efficiency.

## Solution Architecture

The IoT Weather Platform addresses these challenges through a serverless AWS architecture. Raspberry Pi edge devices, equipped with SEN0186 module, collect weather data and transmit it via MQTT to AWS IoT Core. Data is stored in an Amazon S3 data lake, processed using AWS Glue for cataloging and ETL jobs, and analyzed for insights.

A Next.js web application, hosted on AWS Amplify, provides a user interface for real-time dashboards and trend analysis, with access secured by Amazon Cognito. The system supports multiple weather stations, with scalability to 15, and maintains low operational costs at approximately under $5 per month.

![CloudArchitecture](/images/full-architecture.jpg)

## Key Features

### Real-time Dashboard

- **Live Weather Data**: Temperature, humidity, pressure, wind, rainfall
- **Multiple Visualizations**: Cards, charts, and real-time indicators
- **Connection Status**: Visual feedback for IoT connection health
- **Smart Device Monitoring**: UI indicators respond within 3 seconds, notifications sent after 30 seconds offline

### IoT Device Management

- **Device Registration**: Register and manage weather stations
- **Status Monitoring**: Automated offline detection with notifications
- **Multi-location Support**: Switch between weather stations seamlessly

### Data Analytics

- **Historical Data**: Weather data with filtering and export capabilities
- **Automated Processing**: Daily ETL pipeline for data transformation
- **Global Distribution**: CloudFront CDN for dataset access

### Technologies Used

- **Frontend**: Next.js 15, React 19, TypeScript, Shadcn
- **Backend**: AWS Amplify Gen 2, Lambda functions
- **IoT**: AWS IoT Core with PubSub messaging
- **Storage**: Amazon S3 with CloudFront CDN
- **Database**: AWS Glue for data transformation
- **Authentication**: Amazon Cognito
- **Charts**: Recharts for data visualization

