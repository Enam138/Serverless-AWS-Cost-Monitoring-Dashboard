# Deployment Guide

## Overview

This guide provides a step-by-step walkthrough for deploying the **Serverless AWS Cost Monitoring Dashboard** on AWS.

The deployment provisions the components shown in the project architecture:

- Amazon S3 Bucket
- IAM Roles and Policies
- Cost Fetcher Lambda
- Cost API Lambda
- Lambda Function URL
- Amazon EventBridge Rule
- Amazon CloudWatch Logs
- Browser Dashboard

After deployment, the solution will automatically collect AWS cost data once per day and make the latest results available through a browser-based dashboard.


# Deployment Architecture
<img width="1219" height="581" alt="akwannya2" src="https://github.com/user-attachments/assets/72c2a78b-37ed-4c55-a477-090af292a3b3" />


# Deployment Workflow

Deploy the resources in the following order:

```
1. Configure AWS CLI
2. Create Amazon S3 Bucket
3. Create IAM Roles
4. Deploy Cost Fetcher Lambda
5. Deploy Cost API Lambda
6. Configure Lambda Function URL
7. Configure EventBridge Rule
8. Upload Dashboard
9. Validate Deployment
```

Deploying in this order ensures that dependent resources are available when required.


# Step 1 – Configure AWS CLI

Verify the AWS CLI installation.

```bash
aws --version
```

Configure your credentials.

```bash
aws configure
```

Provide:

- Access Key ID
- Secret Access Key
- Default Region
- Output Format

Verify the identity.

```bash
aws sts get-caller-identity
```

# Step 2 – Create the Amazon S3 Bucket

Create a bucket that will store both historical reports and dashboard data.

Example structure:

```
Bucket
│
├── costs/
│
└── data/
      └── latest.json
```

Configure the bucket to:

- Enable Block Public Access
- Disable public object access
- Allow access only through IAM
- Enable versioning (recommended)

# Step 3 – Configure IAM

Create two separate IAM roles.

## Cost Fetcher Role

Permissions:

- AWS Cost Explorer
- Amazon S3 PutObject
- CloudWatch Logs

Responsibilities:

- Retrieve cost data
- Generate reports
- Upload reports


## Cost API Role

Permissions:

- Amazon S3 GetObject
- CloudWatch Logs

Responsibilities:

- Read dashboard data
- Return JSON responses

Using separate IAM roles follows the Principle of Least Privilege.

# Step 4 – Deploy the Cost Fetcher Lambda

Create a Lambda function.

Configuration:

| Setting | Value |
|---------|-------|
| Runtime | Python 3.x |
| Architecture | x86_64 (or arm64) |
| Timeout | 30–60 seconds |
| Memory | 256–512 MB |
| Trigger | Amazon EventBridge |

Assign the **Cost Fetcher IAM Role**.

Deploy the function code.

Verify the function deploys successfully.

# Step 5 – Deploy the Cost API Lambda

Create a second Lambda function.

Configuration:

| Setting | Value |
|---------|-------|
| Runtime | Python 3.x |
| Architecture | x86_64 (or arm64) |
| Timeout | 10–30 seconds |
| Memory | 128–256 MB |
| Trigger | Lambda Function URL |

Assign the **Cost API IAM Role**.

Deploy the function code.

Verify the deployment.

# Step 6 – Configure the Lambda Function URL

Enable a Function URL for the **Cost API Lambda**.

Configuration:

- HTTPS enabled
- GET requests
- CORS configured for the dashboard

The dashboard JavaScript will send requests to this endpoint.

Example:

```
Browser
      │
 HTTPS
      ▼
Lambda Function URL
```


# Step 7 – Configure Amazon EventBridge

Create a scheduled rule.

Rule Name:

```
daily-cost-fetch
```

Schedule:

```
cron(0 6 * * ? *)
```

Target:

```
Cost Fetcher Lambda
```

Grant EventBridge permission to invoke the Lambda function.

Once configured, EventBridge will automatically execute the data collection workflow every day.

# Step 8 – Test the Cost Fetcher Lambda

Invoke the function manually.

Verify that:

- Cost Explorer API returns data.
- CSV reports are generated.
- `latest.json` is created.
- Files are uploaded to Amazon S3.
- CloudWatch logs show successful execution.

# Step 9 – Verify Amazon S3

After execution, the bucket should contain:

```
costs/
```

Example:

```
year=2026/
   month=07/
      day=17.csv
```

and

```
data/
   latest.json
```

Confirm that:

- Historical reports are retained.
- `latest.json` reflects the most recent execution.
- Objects remain private.


# Step 10 – Test the Cost API Lambda

Invoke the Lambda Function URL.

Expected response:

```json
{
  "services": [...],
  "totalCost": "...",
  "date": "..."
}
```

Confirm:

- JSON is returned.
- HTTP status code is 200.
- CORS headers are present.
- No AWS credentials are exposed.

# Step 11 – Deploy the Dashboard

Upload the dashboard files.

Typical structure:

```
dashboard/

index.html

style.css

app.js
```

Update the dashboard configuration so that API requests use the Lambda Function URL.

Open the dashboard in a browser.

# Step 12 – End-to-End Validation

Verify the complete workflow.

```
EventBridge
      │
      ▼
Cost Fetcher Lambda
      │
      ▼
AWS Cost Explorer
      │
      ▼
Amazon S3
      │
      ▼
Cost API Lambda
      │
      ▼
Lambda Function URL
      │
      ▼
Browser Dashboard
```

Confirm:

- EventBridge executes successfully.
- Cost data is collected.
- Files are stored in Amazon S3.
- API returns valid JSON.
- Dashboard displays the latest AWS cost data.

# Monitoring

Open Amazon CloudWatch.

Verify log groups exist for:

- cost-fetcher
- cost-api

Review logs for:

- Successful execution
- Errors
- Duration
- Memory usage

# Security Verification

Confirm the following security controls:

- Amazon S3 Block Public Access enabled
- Separate IAM roles for each Lambda
- Least-privilege permissions
- Lambda Function URL uses HTTPS
- Browser never accesses Amazon S3 directly
- Browser contains no AWS credentials

# Troubleshooting

If deployment issues occur, verify:

- AWS CLI credentials
- IAM permissions
- Lambda execution roles
- EventBridge schedule
- S3 bucket permissions
- Cost Explorer availability
- CloudWatch logs
- Lambda Function URL configuration

Refer to:

```
troubleshooting/common-issues.md
```

for detailed troubleshooting guidance.

# Deployment Summary

After completing the deployment, the solution provide:

- Automated daily AWS cost collection
- Historical cost archive
- Latest dashboard dataset
- Secure API access
- Browser-based cost visualization
- CloudWatch monitoring
- Fully serverless operation

The deployment results in a scalable, secure, and low-maintenance FinOps solution built entirely with AWS managed services.
