# Security Controls

## Overview

Security was a fundamental consideration throughout the design and implementation of the **Serverless AWS Cost Monitoring Dashboard**. The solution follows AWS security best practices by protecting sensitive billing data, enforcing least-privilege access, and limiting direct exposure of AWS resources.

Rather than relying on a single security mechanism, the architecture applies multiple complementary controls that protect the application throughout the data collection, storage, and presentation lifecycle.


# Security Objectives

The solution was designed to achieve the following security objectives:

- Protect AWS billing information
- Prevent unauthorized access to Amazon S3
- Enforce least-privilege permissions
- Secure communication between components
- Eliminate AWS credentials from the frontend
- Reduce the attack surface
- Support monitoring and auditing
- Follow AWS security best practices

Security is applied at each layer of the architecture.

```
Amazon EventBridge
        │
        ▼
Cost Fetcher Lambda
        │
        ▼
AWS Cost Explorer
        │
        ▼
Private Amazon S3 Bucket
        │
        ▼
Cost API Lambda
        │
        ▼
Lambda Function URL (HTTPS)
        │
        ▼
Browser Dashboard
```


# Amazon S3 Security

Amazon S3 stores both historical billing reports and the latest dashboard dataset.

Because this data contains financial information, the bucket is configured as **private**.

Implemented controls include:

- Block Public Access enabled
- No anonymous access
- IAM-only access
- Private object storage
- Separate read and write permissions

The browser never communicates directly with Amazon S3.

# IAM Least Privilege

The project uses dedicated IAM roles for each Lambda function.

## Cost Fetcher Role

Permissions include:

- Read AWS Cost Explorer
- Upload reports to Amazon S3
- Write CloudWatch logs

The role cannot:

- Read dashboard data unnecessarily
- Serve browser requests
- Perform administrative operations

## Cost API Role

Permissions include:

- Read `data/latest.json`
- Write CloudWatch logs

The role cannot:

- Upload objects
- Delete reports
- Query AWS Cost Explorer
- Modify bucket contents

Separating permissions reduces the potential impact of credential compromise.

# Browser Security

The browser acts only as the presentation layer.

The dashboard:

- Does not contain AWS credentials
- Does not communicate directly with Amazon S3
- Does not query AWS Cost Explorer

Instead, it retrieves only the data required for visualization through the Lambda Function URL.

This minimizes the exposure of AWS resources to end users.

# Lambda Function URL Security

The Cost API Lambda is exposed through a Lambda Function URL.

Security controls include:

- HTTPS encryption
- Read-only API responses
- Limited application functionality
- IAM-protected backend resources

The Function URL exposes only the latest dashboard data rather than the underlying storage.

# Data Protection

The solution separates historical and operational data.

## Historical Reports

```
costs/
```

Characteristics:

- Permanent archive
- Append-only
- Not exposed through the dashboard

## Dashboard Dataset

```
data/latest.json
```

Characteristics:

- Updated daily
- Read-only for the API Lambda
- Used exclusively for dashboard visualization

This separation limits unnecessary exposure of archived financial records.

# Secure Communication

Communication between AWS services occurs over AWS-managed secure channels.

External communication with the dashboard uses HTTPS.

Protected communications include:

- Browser → Lambda Function URL
- Lambda → Amazon S3
- Lambda → AWS Cost Explorer
- EventBridge → Lambda

Encryption in transit protects data from interception during transmission.

# Logging and Monitoring

Operational visibility is provided through Amazon CloudWatch.

CloudWatch records:

- Lambda execution logs
- Errors
- Warnings
- Invocation history
- Performance metrics

Logging supports:

- Troubleshooting
- Operational monitoring
- Security investigations
- Auditing


# Event Security

Amazon EventBridge invokes the Cost Fetcher Lambda through a resource-based policy.

Only the configured EventBridge rule is authorized to trigger the Lambda.

This prevents unauthorized invocation from external sources.


# Separation of Responsibilities

The architecture separates responsibilities across multiple services.

| Component | Responsibility |
|------------|----------------|
| EventBridge | Scheduling |
| Cost Fetcher Lambda | Data collection |
| Amazon S3 | Secure storage |
| Cost API Lambda | Data retrieval |
| Browser Dashboard | Data visualization |

This separation reduces the impact of failures or compromised components.

# Defense in Depth

The project follows a **defense-in-depth** approach by implementing multiple layers of protection.

Examples include:

- Private Amazon S3 bucket
- IAM least privilege
- HTTPS communication
- Dedicated Lambda functions
- CloudWatch logging
- Browser isolation
- Resource-based invocation policies

No single control is relied upon to secure the application.

# Security Benefits

The implemented controls provide several advantages.

- Protects sensitive billing information
- Minimizes exposed AWS resources
- Prevents direct access to storage
- Supports least-privilege access
- Improves operational visibility
- Reduces attack surface
- Simplifies security management

# Alignment with AWS Best Practices

The solution aligns with several AWS security recommendations, including:

- Principle of Least Privilege
- Defense in Depth
- Secure by Default
- Separation of Duties
- Private Resource Access
- Encryption in Transit
- Continuous Monitoring

These practices contribute to a secure and maintainable serverless architecture.

# Summary

The **Serverless AWS Cost Monitoring Dashboard** incorporates multiple security controls that protect sensitive billing data while maintaining operational simplicity.

By combining private storage, least-privilege IAM, secure communication, dedicated serverless functions, and centralized monitoring, the architecture delivers a secure, scalable, and cloud-native solution aligned with AWS security best practices.
