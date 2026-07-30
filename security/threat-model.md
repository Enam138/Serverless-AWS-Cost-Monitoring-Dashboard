# Threat Model

## Overview

This document identifies potential security threats to the **Serverless AWS Cost Monitoring Dashboard** and describes the controls implemented to reduce those risks.

The threat model follows a structured approach by identifying:

- Protected assets
- Potential threat actors
- Attack vectors
- Security controls
- Residual risks

The objective is to evaluate how the architecture protects sensitive AWS billing information while maintaining a simple, serverless design.

# Security Objectives

The solution was designed to achieve the following security goals:

- Protect AWS billing information
- Prevent unauthorized access to cloud resources
- Protect stored cost reports
- Secure communication between services
- Enforce least-privilege access
- Minimize the attack surface
- Support operational monitoring

# Architecture

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
Private Amazon S3
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

# Protected Assets

The following assets require protection.

| Asset | Importance |
|---------|------------|
| AWS Cost Data | High |
| Historical CSV Reports | High |
| Latest Dashboard JSON | High |
| Amazon S3 Bucket | High |
| Lambda Functions | High |
| IAM Roles | High |
| Lambda Function URL | Medium |
| CloudWatch Logs | Medium |


# Threat Actors

Potential threat actors include:

- Anonymous internet users
- Unauthorized AWS users
- Compromised AWS credentials
- Malicious insiders
- Automated bots
- Accidental administrators

Although this project is intended as a portfolio solution, the security model assumes production-style risks.

# Threat 1 – Unauthorized Amazon S3 Access

## Description

An attacker attempts to access billing reports stored in Amazon S3.

Potential impact:

- Exposure of financial information
- Disclosure of historical spending
- Loss of confidentiality

## Mitigations

The architecture implements several protections.

- Block Public Access enabled
- Private bucket configuration
- IAM-only access
- No public bucket policy
- Browser never accesses S3 directly

Risk Level:

**Low**

# Threat 2 – Excessive IAM Permissions

## Description

A Lambda function receives unnecessary permissions.

Potential impact:

- Unauthorized resource access
- Privilege escalation
- Data modification

## Mitigations

The project implements:

- Dedicated execution roles
- Least-privilege IAM policies
- Separate read and write permissions
- No administrative privileges

Risk Level:

**Low**

# Threat 3 – Unauthorized Lambda Invocation

## Description

An attacker attempts to invoke the Cost Fetcher Lambda outside the scheduled workflow.

Potential impact:

- Unnecessary Cost Explorer requests
- Unexpected report generation
- Increased operational costs

## Mitigations

The solution uses:

- EventBridge resource-based permissions
- Controlled Lambda invocation
- IAM authorization

Only the configured EventBridge rule can invoke the scheduled Lambda.

Risk Level:

**Low**

# Threat 4 – Direct Browser Access to Amazon S3

## Description

A user attempts to bypass the application and retrieve billing reports directly from Amazon S3.

Potential impact:

- Exposure of sensitive financial data
- Unauthorized downloads

## Mitigations

The browser:

- Never communicates directly with Amazon S3
- Uses only the Lambda Function URL

Amazon S3:

- Blocks public access
- Requires IAM authorization

Risk Level:

**Low**

# Threat 5 – API Misuse

## Description

An attacker repeatedly sends requests to the Lambda Function URL.

Potential impact:

- Increased Lambda invocations
- Higher AWS costs
- Reduced availability

## Mitigations

The API:

- Returns only a single JSON document
- Performs read-only operations
- Does not expose administrative functionality

CloudWatch logs support detection of abnormal request patterns.

Risk Level:

**Medium**

# Threat 6 – Exposure of AWS Credentials

## Description

AWS credentials become embedded within frontend code.

Potential impact:

- Complete AWS account compromise
- Unauthorized resource access

## Mitigations

The dashboard:

- Contains no AWS credentials
- Uses HTTPS requests only
- Communicates exclusively with the Lambda Function URL

Lambda uses temporary IAM role credentials.

Risk Level:

**Low**

# Threat 7 – Compromise of Cost Fetcher Lambda

## Description

An attacker compromises the Cost Fetcher Lambda.

Potential impact:

- Incorrect report generation
- Unauthorized uploads
- Operational disruption

## Mitigations

The Lambda role:

- Has limited permissions
- Cannot administer AWS resources
- Cannot modify IAM
- Cannot access unrelated services

CloudWatch logging supports incident investigation.

Risk Level:

**Medium**

# Threat 8 – Data Tampering

## Description

Historical reports are modified or deleted.

Potential impact:

- Loss of reporting accuracy
- Corrupted financial history

## Mitigations

The architecture:

- Separates historical data from dashboard data
- Uses append-only historical reporting
- Recommends Amazon S3 Versioning

Risk Level:

**Low**

# Threat 9 – Network Interception

## Description

An attacker intercepts communication between the browser and the backend.

Potential impact:

- Exposure of dashboard information
- Session manipulation

## Mitigations

All browser communication uses HTTPS.

AWS-managed TLS protects:

- Browser → Function URL
- Lambda → AWS services

Risk Level:

**Low**

# Threat 10 – Logging Sensitive Information

## Description

Sensitive billing information is accidentally written to CloudWatch Logs.

Potential impact:

- Information disclosure
- Compliance concerns

## Mitigations

Application logging should:

- Record operational events
- Avoid logging sensitive financial data
- Log only information required for troubleshooting

Risk Level:

**Low**


# Threat Summary

| Threat | Risk | Mitigation |
|---------|------|------------|
| Unauthorized S3 access | Low | Private bucket, Block Public Access, IAM |
| Excessive IAM permissions | Low | Least-privilege roles |
| Unauthorized Lambda invocation | Low | Resource-based policies |
| Direct browser access to S3 | Low | API layer, private bucket |
| API misuse | Medium | Minimal functionality, CloudWatch monitoring |
| Credential exposure | Low | IAM roles, no embedded credentials |
| Lambda compromise | Medium | Limited IAM permissions |
| Data tampering | Low | Separate storage strategy, versioning |
| Network interception | Low | HTTPS/TLS |
| Sensitive logging | Low | Secure logging practices |

# Residual Risks

Although the implemented controls significantly reduce risk, some residual risks remain.

Examples include:

- Excessive requests to the Function URL
- Misconfigured IAM policies
- Human configuration errors
- Unexpected AWS service failures

These risks can be further reduced through additional monitoring and governance.

# Future Security Enhancements

Potential improvements include:

- Amazon API Gateway with authentication
- AWS WAF protection
- Amazon Cognito for user authentication
- AWS Secrets Manager
- Amazon GuardDuty
- AWS Config
- AWS Security Hub
- AWS CloudTrail
- Amazon SNS security alerts

These services would strengthen security for a production deployment.

# Alignment with AWS Security Best Practices

The solution aligns with key AWS security principles.

- Principle of Least Privilege
- Defense in Depth
- Secure by Default
- Separation of Duties
- Private Resource Access
- Encryption in Transit
- Continuous Monitoring

These principles contribute to a secure, maintainable, and scalable serverless architecture.

# Summary

The threat model demonstrates that security considerations were incorporated throughout the design of the **Serverless AWS Cost Monitoring Dashboard**.

By protecting Amazon S3 with private access, separating Lambda responsibilities, enforcing least-privilege IAM permissions, using HTTPS for all external communication, and monitoring operations through Amazon CloudWatch, the solution reduces common security risks while maintaining a simple and efficient serverless architecture.

While no system is completely risk-free, the implemented controls provide a strong security foundation that aligns with AWS best practices and can be further enhanced as the application evolves.
