# Prerequisites

## Overview

This document outlines the software, AWS services, permissions, and configuration required to deploy the **Serverless AWS Cost Monitoring Dashboard**.

Before deployment, ensure that your AWS environment and development workstation are properly configured.

# AWS Account Requirements

You will need:

- An active AWS account
- Billing enabled
- AWS Cost Explorer enabled
- IAM permissions to create AWS resources

> **Note:** AWS Cost Explorer must be activated before cost data can be queried. Newly enabled Cost Explorer may take up to 24 hours before data becomes available.


# Required AWS Services

The solution uses the following AWS services.

| Service | Purpose |
|----------|---------|
| Amazon EventBridge | Schedule daily cost collection |
| AWS Lambda | Execute serverless functions |
| AWS Cost Explorer API | Retrieve AWS billing data |
| Amazon S3 | Store historical and current reports |
| AWS IAM | Identity and access management |
| Amazon CloudWatch | Logging and monitoring |
| Lambda Function URL | Expose dashboard API |


# IAM Permissions

The deployment user should have permission to create and configure:

- IAM Roles
- IAM Policies
- AWS Lambda Functions
- Amazon S3 Buckets
- Amazon EventBridge Rules
- Lambda Function URLs
- CloudWatch Log Groups

The deployment account also requires permission to access the AWS Cost Explorer API.

# Development Environment

Install the following tools before deployment.

## Python

Python 3.10 or later

Verify installation:

```bash
python --version
```

## AWS CLI

Install the latest AWS CLI.

Verify installation:

```bash
aws --version
```

Configure credentials:

```bash
aws configure
```

Required information:

- AWS Access Key ID
- AWS Secret Access Key
- Default Region
- Output Format (JSON recommended)


## Git

Install Git for version control.

Verify installation:

```bash
git --version
```

# Project Structure

A typical project structure is shown below.

```text
project/
│
├── dashboard/
│   ├── index.html
│   ├── app.js
│   └── styles.css
│
├── lambda/
│   ├── cost-fetcher/
│   └── cost-api/
│
├── deployment/
├── architecture/
├── security/
├── monitoring/
├── screenshots/
└── README.md
```


# Amazon S3 Bucket

Create an S3 bucket that will store:

## Historical Reports

```
costs/
```

Example:

```
costs/year=2026/month=07/day=17.csv
```


## Dashboard Dataset

```
data/latest.json
```

The bucket should:

- Enable Block Public Access
- Remain private
- Be accessible only through IAM permissions

# Lambda Functions

Deploy two independent Lambda functions.

## Cost Fetcher

Responsibilities:

- Query AWS Cost Explorer
- Generate reports
- Upload reports to Amazon S3

Runtime:

- Python 3.x

Trigger:

- Amazon EventBridge

## Cost API

Responsibilities:

- Read latest dashboard data
- Return JSON responses
- Handle browser requests

Runtime:

- Python 3.x

Trigger:

- Lambda Function URL

# EventBridge Configuration

Create a scheduled rule.

Example:

```
daily-cost-fetch
```

Schedule:

```
cron(0 6 * * ? *)
```

Execution Time:

06:00 UTC (Daily)

# CloudWatch

CloudWatch Log Groups are automatically created when the Lambda functions execute.

No additional configuration is required for basic logging.

# Browser Dashboard

The frontend should consist of static web assets, for example:

- index.html
- JavaScript
- CSS

The dashboard communicates only with the Lambda Function URL.

No AWS credentials should be stored in the browser.

# Security Requirements

Before deployment, verify that:

- Block Public Access is enabled on Amazon S3.
- IAM follows the principle of least privilege.
- Separate IAM roles exist for each Lambda function.
- Lambda Function URL uses HTTPS.
- AWS credentials are never embedded in frontend code.

# Estimated Deployment Time

| Task | Estimated Time |
|------|----------------|
| Configure AWS CLI | 5–10 minutes |
| Create IAM Roles | 10–15 minutes |
| Deploy Lambda Functions | 10–15 minutes |
| Create S3 Bucket | 5 minutes |
| Configure EventBridge | 5 minutes |
| Configure Function URL | 5 minutes |
| Test Deployment | 15–20 minutes |

**Estimated Total:** 1–2 hours

# Cost Considerations

This project is designed to minimize AWS costs by using serverless and managed services.

Expected costs depend on:

- Number of Lambda invocations
- Amount of stored S3 data
- CloudWatch log retention
- AWS Cost Explorer API usage

For personal projects and learning environments, the monthly cost is typically very low because the solution runs only once per day and stores small report files.

# Deployment Checklist

Before beginning deployment, confirm the following:

- AWS account is active
- AWS Cost Explorer is enabled
- AWS CLI is configured
- Python is installed
- Git is installed
- S3 bucket has been created
- IAM roles have been configured
- Lambda deployment packages are ready
- EventBridge schedule is defined
- Dashboard files are available

# Summary

Completing these prerequisites ensures that the AWS environment is prepared for deploying the Serverless AWS Cost Monitoring Dashboard.

With the required services, permissions, and tools in place, the deployment process can proceed smoothly while following AWS security and operational best practices.
