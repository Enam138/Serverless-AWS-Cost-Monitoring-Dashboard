# Request Flow

## Overview

The **Serverless AWS Cost Monitoring Dashboard** follows an event-driven workflow that automates AWS cost collection, securely stores billing data, and delivers the latest cost information to a browser-based dashboard.

The architecture separates **data collection** from **data presentation**, allowing each component to perform a single, well-defined responsibility while following the principles of least privilege and serverless computing.

# Request Flow Diagram

<img width="1219" height="581" alt="akwannya2" src="https://github.com/user-attachments/assets/bf074bbb-55e2-4486-8fd2-e3aedab23036" />



# End-to-End Workflow

The solution operates in two independent workflows:

1. **Automated Cost Collection Workflow**
2. **Dashboard Request Workflow**

Separating these workflows improves maintainability, scalability, and security.

# Workflow 1 – Automated Cost Collection

This workflow executes automatically once every day.

## Step 1 – EventBridge Schedule

Amazon EventBridge triggers the **cost-fetcher** Lambda according to the scheduled rule:

```
daily-cost-fetch
```

using the cron expression:

```
cron(0 6 * * ? *)
```

Execution occurs daily at **06:00 UTC**.

No manual intervention is required.

## Step 2 – Lambda Invocation

The EventBridge rule invokes the **cost-fetcher** Lambda through a resource-based policy.

The Lambda execution environment starts only for the duration of the task.

Responsibilities include:

- Initialize the application
- Authenticate with AWS services
- Retrieve billing information
- Process cost data

## Step 3 – Query AWS Cost Explorer

The cost-fetcher Lambda sends requests to the AWS Cost Explorer API.

The API returns billing information including:

- Daily costs
- Costs grouped by AWS service
- Rolling 30-day history
- Credits and refunds

The returned dataset becomes the source for both dashboard visualization and historical reporting.

## Step 4 – Process Cost Data

The Lambda processes the Cost Explorer response and generates two outputs.

### Historical Report

```
CSV
```

Purpose:

- Long-term archive
- Historical reporting
- Trend analysis

Example location:

```
costs/year=2026/month=07/day=17.csv
```

### Dashboard Dataset

```
JSON
```

Purpose:

- Current dashboard data
- Browser visualization
- Rolling 30-day window

Example location:

```
data/latest.json
```

Unlike the CSV archive, this file is overwritten during every scheduled execution.

## Step 5 – Store Data in Amazon S3

The generated files are uploaded to Amazon S3.

Storage strategy:

### Historical Archive

```
costs/
```

Characteristics:

- Append-only
- Permanent storage
- Never modified

### Current Dashboard Data

```
data/latest.json
```

Characteristics:

- Single object
- Updated daily
- Read by the API Lambda

Amazon S3 Block Public Access remains enabled throughout the process.


## Cost Collection Workflow Summary

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
Generate CSV + JSON
        │
        ▼
Amazon S3
```

# Workflow 2 – Dashboard Request

This workflow begins whenever a user opens the dashboard.

## Step 1 – User Opens Dashboard

The user opens the dashboard in a web browser.

The dashboard loads:

```
index.html
```

along with its associated CSS and JavaScript resources.

No AWS credentials are embedded in the application.

## Step 2 – Browser Sends HTTPS Request

After loading, the dashboard sends an HTTPS request to the configured Lambda Function URL.

Example:

```
Browser
      │
 HTTPS GET
      ▼
Lambda Function URL
```

The browser communicates only with the Function URL.

It never communicates directly with Amazon S3.

## Step 3 – Lambda Function URL

The Function URL invokes the **cost-api** Lambda.

Responsibilities include:

- Accept HTTPS requests
- Trigger Lambda execution
- Return JSON responses

The Function URL provides a lightweight API endpoint without requiring Amazon API Gateway.

## Step 4 – Cost API Lambda

The **cost-api** Lambda receives the request.

Responsibilities include:

- Read `data/latest.json`
- Retrieve the latest dashboard dataset
- Add CORS headers
- Return JSON to the browser

The Lambda has read-only access to Amazon S3.

It cannot:

- Upload objects
- Modify historical reports
- Query AWS Cost Explorer

## Step 5 – Read from Amazon S3

The API Lambda performs a single operation:

```
GetObject
```

Target:

```
data/latest.json
```

Only this object is required for dashboard visualization.

Historical CSV files are never accessed during browser requests.

## Step 6 – Return JSON Response

The Lambda returns:

- HTTP response
- JSON payload
- CORS headers

Example response flow:

```
Amazon S3
      │
GetObject
      ▼
Cost API Lambda
      │
JSON Response
      ▼
Browser
```

## Step 7 – Dashboard Rendering

The browser receives the JSON response.

JavaScript processes the data to display:

- Total AWS costs
- Daily spending
- Cost by AWS service
- Charts
- Trends

Because all processing occurs after the JSON response is returned, the dashboard remains lightweight and responsive.

# Dashboard Request Summary

```
Browser
     │
HTTPS Request
     ▼
Lambda Function URL
     │
     ▼
Cost API Lambda
     │
GetObject
     ▼
Amazon S3
     │
JSON Response
     ▼
Browser Dashboard
```

# Security Throughout the Request Flow

Security is enforced at every stage of the workflow.

## EventBridge

- Invokes Lambda using a resource-based policy
- Prevents unauthorized invocation

## Cost Fetcher Lambda

Permissions:

- Cost Explorer Read
- Amazon S3 PutObject
- CloudWatch Logs

Cannot:

- Serve browser requests

## Amazon S3

Configured with:

- Block Public Access
- IAM-only access
- Private object storage

Anonymous users cannot access stored billing data.

## Cost API Lambda

Permissions:

- Amazon S3 GetObject
- CloudWatch Logs

Cannot:

- Write objects
- Modify data
- Query Cost Explorer

## Browser

The browser:

- Never stores AWS credentials
- Never communicates directly with Amazon S3
- Accesses only the Lambda Function URL

This significantly reduces the attack surface.

# Design Advantages

Separating data collection from dashboard presentation provides several architectural benefits.

## Separation of Responsibilities

Each component performs a single task.

- EventBridge schedules
- Cost Fetcher collects data
- S3 stores data
- Cost API serves data
- Browser visualizes data

## Improved Security

The browser cannot directly access cloud storage.

Sensitive billing data remains protected through IAM permissions and private Amazon S3 storage.

## Better Scalability

Because each component is independently managed by AWS:

- Lambda scales automatically
- S3 provides virtually unlimited storage
- EventBridge reliably schedules execution
- CloudWatch automatically captures logs

## Reduced Operational Overhead

The architecture requires:

- No virtual machines
- No server maintenance
- No cron servers
- No load balancers

AWS managed services handle the operational complexity.

# Summary

The request flow demonstrates how AWS managed services work together to automate cloud cost monitoring while maintaining strong security and operational simplicity.

By separating automated cost collection from dashboard delivery, the architecture achieves:

- Fully automated daily processing
- Secure storage of billing information
- Lightweight browser access
- Least-privilege security
- Scalable serverless execution
- Reliable operational monitoring

This event-driven workflow provides an efficient and maintainable foundation for cloud cost visibility and FinOps reporting.
