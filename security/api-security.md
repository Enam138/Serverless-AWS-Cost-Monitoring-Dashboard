# API Security

## Overview

The **Serverless AWS Cost Monitoring Dashboard** exposes application data through an **AWS Lambda Function URL**, which acts as the backend API for the browser dashboard.

Rather than allowing users to access Amazon S3 directly, the application uses a dedicated API layer to retrieve only the required dashboard data. This approach protects sensitive billing information while maintaining a simple, serverless architecture.

The API is intentionally lightweight and follows the principles of **least privilege**, **secure communication**, and **controlled data exposure**.

# API Architecture

<img width="1219" height="581" alt="akwannya2" src="https://github.com/user-attachments/assets/9dcec54a-38e8-470c-813c-8b320316414a" />


The API request flow is shown below.

```
Browser Dashboard
        │
 HTTPS Request
        ▼
Lambda Function URL
        │
        ▼
Cost API Lambda
        │
        ▼
Amazon S3
        │
 GetObject
        ▼
data/latest.json
```

The browser communicates only with the Lambda Function URL.

Amazon S3 remains private and inaccessible from the public internet.

# API Objectives

The API was designed to:

- Securely expose dashboard data
- Prevent direct browser access to Amazon S3
- Eliminate AWS credentials from the frontend
- Return only the latest cost dataset
- Minimize the exposed attack surface
- Support HTTPS communication
- Follow serverless security best practices

# Lambda Function URL

The project uses a **Lambda Function URL** as the API endpoint.

Instead of deploying Amazon API Gateway, the Function URL provides a lightweight HTTPS endpoint that invokes the Cost API Lambda.

Benefits include:

- Native Lambda integration
- Simplified deployment
- Lower operational overhead
- HTTPS support
- Suitable for lightweight APIs

For this project, the API only needs to return a single JSON document, making a Lambda Function URL an appropriate choice.

# HTTPS Encryption

All communication between the browser and the API occurs over HTTPS.

HTTPS provides:

- Encryption in transit
- Data confidentiality
- Data integrity
- Protection against interception

This ensures that billing information cannot be easily intercepted while traveling across the network.

# API Request Flow

A typical request follows these steps:

1. The user opens the dashboard.
2. The browser sends an HTTPS GET request to the Lambda Function URL.
3. The Function URL invokes the Cost API Lambda.
4. The Lambda retrieves `data/latest.json` from Amazon S3.
5. The Lambda returns a JSON response.
6. The dashboard updates its visualizations.

The browser never communicates directly with Amazon S3.

# Data Exposure

The API intentionally exposes only a single resource:

```
data/latest.json
```

The endpoint does **not** expose:

- Historical CSV reports
- Amazon S3 bucket structure
- AWS Cost Explorer
- IAM information
- AWS account details

This minimizes the amount of information available through the public endpoint.

# Amazon S3 Protection

Amazon S3 remains completely private.

Security controls include:

- Block Public Access enabled
- IAM-only access
- No public bucket policy
- No direct browser access

The Cost API Lambda is the only component authorized to read the dashboard dataset.

# IAM Protection

The Cost API Lambda uses a dedicated execution role.

Permissions include:

```
s3:GetObject

logs:CreateLogGroup

logs:CreateLogStream

logs:PutLogEvents
```

The Lambda cannot:

- Upload objects
- Delete reports
- Modify bucket contents
- Query AWS Cost Explorer

This enforces the Principle of Least Privilege.

# CORS Configuration

The Lambda Function URL returns the required **Cross-Origin Resource Sharing (CORS)** headers so that the browser can retrieve dashboard data.

Typical CORS headers include:

```
Access-Control-Allow-Origin

Access-Control-Allow-Methods

Access-Control-Allow-Headers
```

Proper CORS configuration allows legitimate browser requests while reducing the risk of unauthorized cross-origin interactions.

# Response Format

The API returns structured JSON.

Example:

```json
{
  "date": "2026-07-29",
  "totalCost": "125.84",
  "services": [
    {
      "service": "Amazon EC2",
      "cost": "48.31"
    },
    {
      "service": "Amazon S3",
      "cost": "8.15"
    }
  ]
}
```

Returning JSON keeps the API lightweight and easy for the frontend to consume.

# Error Handling

The API should return appropriate HTTP status codes.

| Status Code | Meaning |
|-------------|---------|
| 200 | Request successful |
| 400 | Invalid request |
| 404 | Dashboard dataset not found |
| 500 | Internal server error |

Meaningful error responses simplify troubleshooting while avoiding unnecessary disclosure of internal implementation details.

# Logging

All API requests generate CloudWatch logs.

Logs include:

- Invocation time
- Execution duration
- Errors
- Warnings
- Request status

CloudWatch supports operational monitoring and incident investigation.

# Security Considerations

The API was intentionally designed to expose as little functionality as possible.

The endpoint:

- Does not authenticate with AWS credentials
- Does not expose Amazon S3
- Does not modify AWS resources
- Does not execute Cost Explorer queries
- Does not expose historical billing reports

Instead, it simply retrieves and returns the latest dashboard dataset.

This significantly reduces the application's attack surface.

# Why Lambda Function URL Instead of API Gateway?

For this project, a Lambda Function URL was selected because the application requires only a simple HTTPS endpoint for retrieving a single JSON document.

Compared to Amazon API Gateway, this approach:

- Reduces architectural complexity
- Simplifies deployment
- Lowers operational overhead
- Meets the functional requirements of the dashboard

If the application later requires features such as authentication, rate limiting, API versioning, or usage plans, Amazon API Gateway would be an appropriate enhancement.

# Security Benefits

The implemented API design provides several advantages:

- Secure HTTPS communication
- No browser access to Amazon S3
- Minimal exposed functionality
- Read-only API operations
- Least-privilege IAM permissions
- CloudWatch monitoring
- Lightweight serverless architecture

These controls improve the confidentiality and integrity of the application's billing data.

# Summary

The API layer provides a secure interface between the browser dashboard and the application's backend storage.

By using a Lambda Function URL, private Amazon S3 storage, HTTPS encryption, least-privilege IAM permissions, and CloudWatch logging, the solution delivers a secure, maintainable, and efficient serverless API that exposes only the information required for dashboard visualization while protecting sensitive AWS billing data.
