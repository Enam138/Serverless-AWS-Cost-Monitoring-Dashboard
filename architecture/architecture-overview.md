# Architecture Overview

## Overview

The **Serverless AWS Cost Monitoring Dashboard** is a cloud-native FinOps solution designed to automate the collection, storage, and presentation of AWS cost data using fully managed AWS services.

The architecture follows an **event-driven serverless design**, where scheduled events trigger automated cost collection, secure storage maintains historical and current reports, and a lightweight API delivers cost data to a browser-based dashboard.

By leveraging AWS managed services, the solution minimizes operational overhead while providing secure, scalable, and reliable access to cloud cost information.


# Architecture Diagram


<img width="1219" height="581" alt="akwannya2" src="https://github.com/user-attachments/assets/40f07ebd-9291-4925-9cc5-7ffa168217cc" />


# Architecture Goals

The solution was designed with the following objectives:

- Automate AWS cost collection
- Eliminate manual reporting processes
- Maintain historical cost records
- Provide near real-time dashboard data
- Secure cloud cost information
- Minimize operational overhead
- Implement least-privilege access controls
- Use fully managed AWS services
- Support future scalability and enhancements


# High-Level Architecture

The architecture consists of seven primary components:

```
Browser Dashboard
        ▲
        │
Lambda Function URL
        ▲
        │
Cost API Lambda
        ▲
        │
Amazon S3
        ▲
        │
Cost Fetcher Lambda
        ▲
        │
AWS Cost Explorer API
        ▲
        │
Amazon EventBridge
```

Each component performs a dedicated responsibility, resulting in a loosely coupled and maintainable serverless application.

# Architectural Components

## 1. Amazon EventBridge

Amazon EventBridge serves as the scheduler for the solution.

A scheduled rule executes once every day at **06:00 UTC**, automatically invoking the **cost-fetcher** Lambda function.

Responsibilities include:

- Scheduling daily execution
- Triggering automated workflows
- Eliminating manual intervention
- Supporting reliable event-driven processing

The EventBridge rule invokes the Lambda function through a resource-based policy, ensuring that only authorized AWS services can trigger execution.

## 2. Cost Fetcher Lambda

The **cost-fetcher** Lambda function is responsible for collecting AWS billing data.

During each scheduled execution, the function:

- Queries the AWS Cost Explorer API
- Retrieves approximately 30 days of cost information
- Groups costs by AWS service
- Filters credits and refunds
- Generates a historical CSV report
- Generates the latest JSON dataset
- Uploads both files to Amazon S3

This function acts as the **data ingestion layer** of the architecture.


## 3. AWS Cost Explorer API

The AWS Cost Explorer API provides detailed billing information used by the dashboard.

The solution retrieves:

- Daily cloud costs
- Service-level spending
- Rolling 30-day cost history

The Cost Explorer API serves as the authoritative source of financial data for the application.

## 4. Amazon S3

Amazon S3 acts as the central data repository.

The bucket stores two categories of data.

### Historical Archive

```
costs/
```

The historical archive contains CSV files generated during every execution.

Example:

```
costs/year=2026/month=07/day=17.csv
```

Characteristics:

- Append-only
- Never overwritten
- Long-term storage
- Supports trend analysis


### Dashboard Dataset

```
data/latest.json
```

This object represents the most recent dashboard dataset.

Characteristics:

- Updated every execution
- Read by the API Lambda
- Serves browser requests
- Maintains a rolling 30-day view


The bucket is configured with **Block Public Access**, ensuring that all objects remain private unless accessed through authorized IAM principals.

## 5. Cost API Lambda

The **cost-api** Lambda function serves as the application's API layer.

Rather than allowing the browser to access Amazon S3 directly, the API Lambda:

- Retrieves `latest.json`
- Returns JSON responses
- Adds CORS headers
- Responds to browser requests

This separation improves security by preventing direct access to the storage layer.

## 6. Lambda Function URL

The dashboard communicates with the backend through a Lambda Function URL.

The Function URL:

- Receives HTTPS requests
- Invokes the API Lambda
- Returns JSON responses
- Eliminates the need for API Gateway for this lightweight use case

This approach reduces architectural complexity while providing secure access to application data.

## 7. Browser Dashboard

The dashboard provides the user interface for visualizing AWS cost information.

The browser:

- Loads the dashboard application
- Sends HTTPS requests to the Function URL
- Receives JSON responses
- Renders charts and cost summaries

Importantly, the browser never communicates directly with Amazon S3 and does not require AWS credentials.

## 8. Amazon CloudWatch

Amazon CloudWatch provides operational visibility into the application.

Logs are automatically generated for both Lambda functions, enabling:

- Execution monitoring
- Error detection
- Performance analysis
- Troubleshooting
- Operational auditing

CloudWatch simplifies diagnosing issues while supporting ongoing maintenance.

# Data Flow

The solution follows a straightforward event-driven workflow.

1. EventBridge triggers the **cost-fetcher** Lambda.
2. The Lambda retrieves billing information from AWS Cost Explorer.
3. Historical CSV and current JSON files are stored in Amazon S3.
4. A browser requests dashboard data through the Lambda Function URL.
5. The **cost-api** Lambda retrieves `latest.json` from Amazon S3.
6. The JSON response is returned to the browser.
7. The dashboard renders visualizations using the latest cost data.

# Security Architecture

Security is integrated throughout the architecture.

Key controls include:

- Amazon S3 Block Public Access
- IAM least-privilege permissions
- Separate read and write Lambda roles
- Resource-based policy for EventBridge
- HTTPS communication
- No AWS credentials stored in the browser
- Controlled access to Amazon S3 through Lambda

These measures reduce the attack surface while protecting sensitive billing information.

# Scalability

The architecture is designed to scale with minimal operational effort.

AWS managed services automatically handle:

- Lambda execution
- Event scheduling
- Amazon S3 storage
- CloudWatch logging

As AWS usage grows, the dashboard can continue processing larger datasets without requiring dedicated server infrastructure.

# Reliability

The architecture improves reliability through:

- Managed AWS services
- Scheduled automation
- Durable Amazon S3 storage
- CloudWatch operational logs
- Stateless Lambda functions

Because no servers require maintenance, operational complexity remains low.

# Cost Optimization

The solution itself follows AWS cost optimization principles.

These include:

- Serverless compute
- Pay-per-use pricing
- No always-on infrastructure
- Managed storage
- Event-driven execution
- Lightweight API implementation

This architecture minimizes infrastructure costs while delivering the required functionality.

# Architectural Benefits

The selected design provides several advantages.

- Fully serverless architecture
- Event-driven automation
- Secure data storage
- Separation of responsibilities
- Low operational overhead
- High scalability
- Built-in monitoring
- Improved maintainability
- Cost-efficient execution
- Strong security posture

# Summary

The Serverless AWS Cost Monitoring Dashboard demonstrates how AWS managed services can be combined to build a secure, automated, and scalable FinOps solution.

By separating cost collection, storage, API delivery, and visualization into dedicated components, the architecture follows cloud-native design principles while remaining simple to operate and extend.
