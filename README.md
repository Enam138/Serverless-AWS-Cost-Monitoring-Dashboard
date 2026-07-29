# Serverless AWS Cost Monitoring Dashboard

![AWS](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazonaws)
![Lambda](https://img.shields.io/badge/AWS_Lambda-Serverless-FF9900?logo=awslambda)
![Amazon S3](https://img.shields.io/badge/Amazon_S3-Storage-green?logo=amazons3)
![Amazon EventBridge](https://img.shields.io/badge/EventBridge-Scheduler-purple)
![CloudWatch](https://img.shields.io/badge/CloudWatch-Monitoring-orange)
![Cost Explorer](https://img.shields.io/badge/AWS_Cost_Explorer-FinOps-blue)
![Python](https://img.shields.io/badge/Python-3.12-blue?logo=python)
![License](https://img.shields.io/badge/License-MIT-green)

A fully serverless AWS FinOps solution that automatically collects AWS Cost Explorer data, securely stores historical and current cost reports in Amazon S3, and exposes a lightweight API for a browser-based dashboard.

The solution demonstrates event-driven automation, Infrastructure as Code principles, least-privilege IAM, secure data access, and cloud cost visibility using AWS managed services.


# Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Project Highlights](#project-highlights)
- [Business Scenario](#business-scenario)
- [Solution Architecture](#solution-architecture)
- [AWS Services Used](#aws-services-used)
- [Repository Structure](#repository-structure)
- [Features](#features)
- [Security Highlights](#security-highlights)
- [Deployment Overview](#deployment-overview)
- [Validation](#validation)
- [Documentation](#documentation)
- [Screenshots](#screenshots)
- [Skills Demonstrated](#skills-demonstrated)
- [Future Improvements](#future-improvements)
- [Lessons Learned](#lessons-learned)
- [References](#references)
- [Author](#author)

# Project Overview

Cloud cost management is an important aspect of operating workloads in AWS. Organizations need timely visibility into spending trends to support budgeting, identify unexpected increases, and make informed financial decisions.

This project implements a fully serverless cost monitoring solution that automatically retrieves AWS Cost Explorer data every day, stores both historical and current reports in Amazon S3, and serves the latest cost information through a secure Lambda-based API for visualization in a browser dashboard.

The solution demonstrates how AWS managed services can be combined to build a scalable, low-maintenance, and cost-effective FinOps application.

# Architecture

> **Solution Architecture**

<img width="1219" height="581" alt="akwannya2" src="https://github.com/user-attachments/assets/0ec15993-8b60-4eba-99bb-abbdc245b2a8" />


### High-Level Workflow

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
 ├── Historical CSV Archive
 └── Latest JSON
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

# Project Highlights

- Fully serverless architecture
- Event-driven automation with Amazon EventBridge
- Daily AWS Cost Explorer data collection
- Historical cost archive stored in Amazon S3
- Latest cost data served through a Lambda Function URL
- Browser dashboard with no embedded AWS credentials
- Principle of Least Privilege using IAM
- Block Public Access enabled on Amazon S3
- CloudWatch logging for operational monitoring
- Low operational overhead using AWS managed services

# Business Scenario

Cloud costs can increase rapidly as organizations deploy additional services and workloads. Without regular monitoring, identifying spending trends or unexpected charges becomes difficult.

This solution was designed to automate daily AWS cost collection and provide a lightweight dashboard for visualizing cloud spending while maintaining secure access to cost data and minimizing operational complexity.

# Solution Architecture

The solution follows an event-driven serverless architecture.

| Layer | AWS Service |
|---------|-------------|
| Scheduling | Amazon EventBridge |
| Compute | AWS Lambda |
| Cost Data | AWS Cost Explorer API |
| Storage | Amazon S3 |
| API | Lambda Function URL |
| Monitoring | Amazon CloudWatch |
| Identity | AWS IAM |

The architecture separates data collection from data presentation. One Lambda function is responsible for collecting and storing cost data, while a second Lambda function serves only the latest JSON file to the dashboard.


# AWS Services Used

| Service | Purpose |
|---------|---------|
| Amazon EventBridge | Schedule daily cost collection |
| AWS Lambda | Serverless compute |
| AWS Cost Explorer API | Retrieve AWS cost data |
| Amazon S3 | Store historical and current reports |
| Lambda Function URL | Expose dashboard API |
| Amazon CloudWatch | Logging and monitoring |
| AWS IAM | Secure permissions |

# Repository Structure

```
.
├── architecture/
├── deployment/
├── security/
├── finops/
├── monitoring/
├── troubleshooting/
├── diagrams/
├── screenshots/
├── README.md
├── lessons-learned.md
```


# Features

- Automated daily cost collection
- Serverless architecture
- Historical cost archive
- Rolling 30-day dashboard data
- Browser-based visualization
- Secure API access
- IAM least-privilege permissions
- CloudWatch monitoring
- Block Public Access on S3
- Cost Explorer integration

# Security Highlights

Security was incorporated throughout the solution:

- Amazon S3 Block Public Access enabled
- IAM least-privilege roles
- Separate read and write Lambda permissions
- Browser never accesses S3 directly
- Browser contains no AWS credentials
- Lambda Function URL serves only required dashboard data
- CloudWatch logs support operational visibility
- Resource-based policy allows only EventBridge to invoke the data collection Lambda

Detailed security documentation is available in the **security/** directory.

# Deployment Overview

Deployment consists of:

1. Configure IAM permissions
2. Create the Amazon S3 bucket
3. Deploy the Lambda functions
4. Configure EventBridge schedule
5. Configure Lambda Function URL
6. Upload the dashboard application
7. Validate daily cost collection
8. Verify dashboard functionality

Detailed deployment instructions are available in:

`deployment/deployment-guide.md`


# Validation

The solution was validated to confirm:

- Daily EventBridge execution
- Successful Cost Explorer queries
- Historical CSV generation
- Latest JSON generation
- Amazon S3 object storage
- API responses through Lambda Function URL
- Dashboard rendering
- CloudWatch logging
- Block Public Access enforcement

Additional validation details are available in:

`deployment/validation.md`


# Documentation

## Architecture

- Architecture Overview
- AWS Services
- Request Flow
- Serverless Design

## Deployment

- Prerequisites
- Deployment Guide
- Validation

## Security

- Security Controls
- IAM Security
- API Security
- Threat Model
- AWS Best Practices

## FinOps

- AWS Cost Explorer
- Dashboard Features
- Cost Monitoring

## Monitoring

- CloudWatch Monitoring

## Troubleshooting

- Common Issues


# Screenshots

The following screenshots will be included:

- Architecture Diagram
- EventBridge Rule
- Cost Fetcher Lambda
- Cost API Lambda
- Amazon S3 Bucket
- Cost Explorer Results
- Lambda Function URL
- Browser Dashboard
- CloudWatch Logs


# Skills Demonstrated

This project demonstrates experience with:

- AWS Serverless Architecture
- AWS Lambda
- Amazon EventBridge
- AWS Cost Explorer API
- Amazon S3
- Cloud Monitoring
- IAM Security
- FinOps Fundamentals
- Cloud Cost Optimization
- Python Development
- API Design
- Cloud Architecture
- Technical Documentation

# Future Improvements

Potential enhancements include:

- Amazon QuickSight integration
- Cost anomaly detection
- SNS budget notifications
- Multi-account cost aggregation
- AWS Organizations support
- Cost forecasting
- Daily email reports
- CI/CD deployment pipeline

# Lessons Learned

Key lessons from this project include:

- Serverless architectures reduce operational overhead.
- Event-driven automation simplifies recurring tasks.
- Least-privilege IAM improves security.
- Separating data collection from presentation improves maintainability.
- Cloud cost visibility is essential for FinOps practices.
- AWS managed services enable scalable and reliable solutions with minimal infrastructure management.

A more detailed discussion is available in `lessons-learned.md`.

# References

- AWS Cost Explorer Documentation
- AWS Lambda Documentation
- Amazon EventBridge Documentation
- Amazon S3 Documentation
- Amazon CloudWatch Documentation
- AWS Well-Architected Framework

# Author

**Manyo Sampson**

Cloud Security | Cybersecurity - SOC Analyst | AWS & Azure 
