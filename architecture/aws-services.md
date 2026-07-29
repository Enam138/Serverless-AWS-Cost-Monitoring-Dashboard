# AWS Services

## Overview

The **Serverless AWS Cost Monitoring Dashboard** leverages a collection of fully managed AWS services to automate cloud cost collection, securely store financial data, expose cost information through a lightweight API, and provide operational visibility.

Each service has a well-defined responsibility within the architecture, following the principles of separation of concerns, least privilege, and event-driven computing.

# AWS Services Architecture

| AWS Service | Role in the Solution |
|-------------|----------------------|
| Amazon EventBridge | Schedules automated cost collection |
| AWS Lambda | Executes serverless business logic |
| AWS Cost Explorer API | Retrieves AWS billing and usage data |
| Amazon S3 | Stores historical and current cost reports |
| Lambda Function URL | Exposes dashboard data to the browser |
| Amazon CloudWatch | Collects logs and operational metrics |
| AWS IAM | Controls secure access to AWS resources |

# Amazon EventBridge

## Purpose

Amazon EventBridge automates the execution of the data collection process by invoking the **cost-fetcher** Lambda function on a daily schedule.

This eliminates the need for manual execution or dedicated scheduling infrastructure.


## How It Is Used

The solution creates a scheduled EventBridge rule:

```
daily-cost-fetch
```

using the cron expression:

```
cron(0 6 * * ? *)
```

This executes every day at **06:00 UTC**, triggering the cost-fetcher Lambda.


## Benefits

- Fully managed scheduler
- Reliable event delivery
- No infrastructure to maintain
- Native integration with AWS Lambda
- Supports future automation workflows

# AWS Lambda

AWS Lambda provides the serverless compute layer for the application.

Instead of provisioning servers, Lambda executes code only when required.

The project uses **two independent Lambda functions**, each with a dedicated responsibility.

## Cost Fetcher Lambda

### Purpose

The **cost-fetcher** function serves as the data ingestion component.

Responsibilities include:

- Querying AWS Cost Explorer
- Processing billing information
- Generating CSV reports
- Generating dashboard JSON
- Uploading files to Amazon S3

### Permissions

The function receives only the permissions required to:

- Read AWS Cost Explorer data
- Write objects to Amazon S3
- Generate CloudWatch logs

It does **not** require permission to read dashboard data.

## Cost API Lambda

### Purpose

The **cost-api** function serves as the application's API layer.

Rather than exposing Amazon S3 directly, the browser retrieves dashboard data through this Lambda.

Responsibilities include:

- Reading `latest.json`
- Returning JSON responses
- Providing CORS headers
- Handling browser requests

### Permissions

The API Lambda receives read-only access to:

```
data/latest.json
```

It cannot:

- Write to Amazon S3
- Query AWS Cost Explorer
- Modify historical reports

This separation supports the principle of least privilege.

# AWS Cost Explorer API

## Purpose

AWS Cost Explorer provides detailed billing and usage information for AWS resources.

It acts as the authoritative data source for the dashboard.

## Data Retrieved

The application retrieves:

- Daily AWS costs
- Cost grouped by AWS service
- Rolling 30-day history
- Credits and refunds filtering

## Why Cost Explorer?

Using the Cost Explorer API provides:

- Accurate billing information
- Automated reporting
- Near real-time visibility
- No manual CSV exports
- Consistent financial data

# Amazon S3

## Purpose

Amazon S3 acts as the secure storage layer for the solution.

The bucket stores both historical reports and the latest dashboard dataset.

---

## Historical Archive

```
costs/
```

Example:

```
costs/year=2026/month=07/day=17.csv
```

Characteristics:

- Append-only
- Long-term storage
- Never overwritten
- Supports historical analysis

## Dashboard Dataset

```
data/latest.json
```

Characteristics:

- Updated daily
- Contains current dashboard data
- Read by the API Lambda
- Served to the browser

## Security Configuration

The bucket is configured with:

- Block Public Access enabled
- IAM-only access
- Separate read/write permissions
- Private storage architecture

This ensures billing data remains protected from unauthorized access.

# Lambda Function URL

## Purpose

The Lambda Function URL exposes the **cost-api** Lambda over HTTPS.

Instead of deploying Amazon API Gateway, this lightweight endpoint provides direct browser access to dashboard data.

## Responsibilities

- Receive HTTPS requests
- Invoke the API Lambda
- Return JSON responses
- Support browser access through CORS

## Benefits

- Simpler architecture
- Lower operational overhead
- Native Lambda integration
- Suitable for lightweight APIs

# Amazon CloudWatch

## Purpose

Amazon CloudWatch provides centralized logging and operational monitoring.

Each Lambda function automatically writes execution logs to CloudWatch.

## Monitoring Capabilities

CloudWatch enables administrators to:

- Review execution history
- Diagnose application errors
- Monitor successful executions
- Troubleshoot failures
- Audit serverless operations

## Operational Benefits

CloudWatch improves:

- Reliability
- Observability
- Troubleshooting
- Operational visibility

# AWS IAM

## Purpose

AWS Identity and Access Management (IAM) secures access to AWS resources.

The solution uses dedicated IAM roles for each Lambda function.

## Cost Fetcher Role

Permissions include:

- Cost Explorer Read
- Amazon S3 PutObject
- CloudWatch Logs

## Cost API Role

Permissions include:

- Amazon S3 GetObject
- CloudWatch Logs

The API role cannot modify or upload objects.

## Security Benefits

Using separate IAM roles provides:

- Least privilege
- Separation of responsibilities
- Reduced attack surface
- Improved security management

# Service Integration

The AWS services work together as follows:

```
Amazon EventBridge
        │
        ▼
Cost Fetcher Lambda
        │
        ▼
AWS Cost Explorer API
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

Each service performs a dedicated function, creating a modular and maintainable serverless architecture.

# Why These Services Were Selected

The selected AWS services were chosen because they align with the project's goals of automation, scalability, security, and cost efficiency.

| Requirement | AWS Service |
|-------------|-------------|
| Automated scheduling | Amazon EventBridge |
| Serverless compute | AWS Lambda |
| Cost reporting | AWS Cost Explorer API |
| Durable storage | Amazon S3 |
| Secure API access | Lambda Function URL |
| Monitoring | Amazon CloudWatch |
| Access control | AWS IAM |

Together, these managed services eliminate the need to provision or maintain servers while providing a reliable platform for automated cloud cost monitoring.

# Summary

The Serverless AWS Cost Monitoring Dashboard demonstrates how AWS managed services can be integrated to build a secure, scalable, and automated FinOps solution.

Each service has a clearly defined responsibility, contributing to a modular architecture that emphasizes automation, least-privilege security, operational simplicity, and cloud cost visibility.
