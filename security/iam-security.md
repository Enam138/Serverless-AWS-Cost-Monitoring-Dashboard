# IAM Security

## Overview

AWS Identity and Access Management (IAM) is the foundation of security within the **Serverless AWS Cost Monitoring Dashboard**. The solution uses IAM to control which AWS services can access resources, invoke functions, and retrieve sensitive billing information.

Rather than granting broad administrative permissions, each component is assigned only the permissions required to perform its specific task. This implementation follows the **Principle of Least Privilege**, reducing the attack surface and minimizing the impact of compromised credentials.

# IAM Design Principles

The IAM configuration was designed around the following principles:

- Principle of Least Privilege
- Separation of Responsibilities
- Temporary Credentials
- Service-to-Service Authentication
- Resource-Based Permissions
- No Long-Term Credentials in the Application

These principles help secure both the serverless infrastructure and the billing data stored within Amazon S3.

# IAM Architecture

The architecture uses dedicated IAM roles for each AWS Lambda function.

```
Amazon EventBridge
        │
        ▼
Resource-Based Policy
        │
        ▼
Cost Fetcher Lambda
        │
 IAM Role
        │
 ├── AWS Cost Explorer
 ├── Amazon S3 (Write)
 └── CloudWatch Logs

---------------------------------------

Browser
        │
        ▼
Lambda Function URL
        │
        ▼
Cost API Lambda
        │
 IAM Role
        │
 ├── Amazon S3 (Read)
 └── CloudWatch Logs
```

Each Lambda function assumes its own IAM role during execution.

# Cost Fetcher IAM Role

## Purpose

The **Cost Fetcher** Lambda retrieves AWS billing data and stores reports in Amazon S3.

Its IAM role grants only the permissions required to perform these operations.

## Permissions

The role is permitted to:

- Read AWS Cost Explorer data
- Upload objects to Amazon S3
- Create CloudWatch log streams
- Write CloudWatch logs

Typical IAM actions include:

```text
ce:GetCostAndUsage

s3:PutObject

logs:CreateLogGroup

logs:CreateLogStream

logs:PutLogEvents
```

## Restrictions

The role cannot:

- Read dashboard files unnecessarily
- Delete objects from Amazon S3
- Modify IAM resources
- Access unrelated AWS services
- Perform administrative actions

Restricting permissions reduces the potential impact of a compromised execution environment.

# Cost API IAM Role

## Purpose

The **Cost API** Lambda retrieves the latest dashboard dataset and returns it to the browser.

Because this function only serves existing data, it requires significantly fewer permissions than the Cost Fetcher.

## Permissions

The role can:

- Read `data/latest.json`
- Generate CloudWatch logs

Typical IAM actions include:

```text
s3:GetObject

logs:CreateLogGroup

logs:CreateLogStream

logs:PutLogEvents
```

## Restrictions

The API role cannot:

- Upload files
- Delete reports
- Query AWS Cost Explorer
- Modify Amazon S3
- Access historical billing archives
- Perform administrative operations

This strict separation helps contain potential security incidents.

# EventBridge Permissions

Amazon EventBridge invokes the Cost Fetcher Lambda according to the configured schedule.

Rather than using IAM user credentials, EventBridge is granted permission through a **resource-based policy** attached to the Lambda function.

This policy authorizes only the configured EventBridge rule to invoke the function.

Benefits include:

- Controlled invocation
- Reduced attack surface
- Secure service-to-service communication

# Amazon S3 Access Control

Amazon S3 stores both historical reports and the latest dashboard dataset.

Access is controlled exclusively through IAM.

## Write Access

Granted only to:

- Cost Fetcher Lambda

Operations:

```text
PutObject
```

## Read Access

Granted only to:

- Cost API Lambda

Operations:

```text
GetObject
```


## Public Access

Public access is disabled through:

- Block Public Access
- Private bucket configuration
- IAM-only authorization

No anonymous users can access stored billing information.

# CloudWatch Permissions

Both Lambda functions require permission to write execution logs.

The following permissions are granted:

```text
logs:CreateLogGroup

logs:CreateLogStream

logs:PutLogEvents
```

These permissions support operational monitoring without exposing additional AWS resources.

# Temporary Credentials

AWS Lambda automatically receives temporary credentials by assuming its assigned IAM role during execution.

Advantages include:

- No hardcoded AWS keys
- Automatic credential rotation
- Reduced credential management
- Improved security posture

This follows AWS best practices for serverless applications.

# Separation of Responsibilities

IAM supports separation of duties throughout the architecture.

| Component | Responsibility | IAM Access |
|-----------|----------------|------------|
| EventBridge | Schedule execution | Invoke Cost Fetcher Lambda |
| Cost Fetcher Lambda | Retrieve and store cost data | Cost Explorer, S3 Write, CloudWatch |
| Cost API Lambda | Serve dashboard data | S3 Read, CloudWatch |
| Browser Dashboard | Display cost information | No AWS IAM permissions |

Each component receives only the permissions required for its function.

# Principle of Least Privilege

The Principle of Least Privilege is implemented throughout the solution.

Examples include:

- Separate IAM roles for each Lambda
- Read-only access for the API Lambda
- Write-only access where appropriate
- Restricted service permissions
- No administrative privileges
- No wildcard resource permissions where avoidable

Applying least privilege reduces the likelihood and impact of unauthorized actions.

# Security Risks and Mitigations

| Risk | Mitigation |
|------|------------|
| Excessive IAM permissions | Least-privilege policies |
| Credential exposure | Temporary IAM role credentials |
| Unauthorized Lambda invocation | Resource-based policy |
| Unauthorized S3 access | Private bucket with IAM-only access |
| Accidental privilege escalation | Separate execution roles |
| Hardcoded AWS keys | IAM roles instead of access keys |


# IAM Best Practices Applied

The solution follows several AWS IAM best practices.

- Dedicated execution roles
- No embedded AWS credentials
- Least-privilege permissions
- Separation of responsibilities
- Temporary credentials
- Service-to-service authentication
- Resource-based permissions
- Private resource access

These practices improve the overall security and maintainability of the application.

# Summary

AWS Identity and Access Management plays a critical role in securing the **Serverless AWS Cost Monitoring Dashboard**.

By assigning dedicated execution roles, enforcing least-privilege permissions, using temporary credentials, and restricting access to sensitive AWS resources, the solution minimizes security risks while maintaining reliable serverless operation.

The IAM implementation demonstrates cloud security best practices and supports a secure, scalable, and maintainable FinOps architectu
