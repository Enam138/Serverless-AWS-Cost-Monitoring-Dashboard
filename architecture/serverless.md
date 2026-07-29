# Serverless Architecture Design

## Overview

The **Serverless AWS Cost Monitoring Dashboard** was designed using AWS managed services to provide an automated, scalable, secure, and cost-effective solution for monitoring cloud spending.

Rather than relying on traditional server-based infrastructure, the solution adopts a **serverless architecture**, where AWS manages the underlying infrastructure, allowing the application to focus solely on business logic.

This design minimizes operational overhead while improving scalability, reliability, and maintainability.

# Design Objectives

The architecture was designed to achieve the following goals:

- Automate AWS cost collection
- Eliminate manual reporting
- Reduce infrastructure management
- Secure financial data
- Minimize operational costs
- Improve scalability
- Support future enhancements
- Follow AWS Well-Architected best practices


# Serverless Architecture

The solution uses event-driven serverless services.

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
Dashboard
```

Each service performs a single responsibility, creating a loosely coupled architecture that is easy to maintain and extend.

# Why Serverless?

Traditional applications typically require virtual machines or containers to remain running continuously.

For this project, that approach would introduce unnecessary complexity because cost data is collected only once per day and the dashboard is accessed only when needed.

Using AWS Lambda allows compute resources to run only during execution, reducing infrastructure costs and operational effort.

# Design Decision 1: Event-Driven Scheduling

## Selected Service

Amazon EventBridge

### Why?

Cost collection is a scheduled task rather than a continuous process.

EventBridge provides a fully managed scheduler that automatically invokes the cost-fetcher Lambda at the configured time each day.

### Benefits

- No cron server required
- Reliable scheduling
- Native Lambda integration
- Minimal maintenance
- Easily adjustable schedules

### Alternative Considered

Running scheduled tasks on an Amazon EC2 instance.

### Why It Was Not Selected

An EC2-based scheduler would require:

- Server provisioning
- Operating system maintenance
- Security patching
- Monitoring
- Continuous availability

EventBridge removes these operational responsibilities.


# Design Decision 2: Serverless Compute

## Selected Service

AWS Lambda

### Why?

The project requires two independent processing tasks:

- Collect AWS cost data
- Serve dashboard data

Using separate Lambda functions allows each task to be independently deployed, secured, and maintained.

### Benefits

- Automatic scaling
- Pay-per-use pricing
- No server management
- Stateless execution
- Fast deployment

### Alternative Considered

Amazon EC2

### Why It Was Not Selected

An EC2 instance would remain running continuously despite the workload executing only briefly each day.

Lambda provides a more cost-effective and operationally efficient solution.


# Design Decision 3: Amazon S3 for Storage

## Selected Service

Amazon S3

### Why?

The application stores two types of files:

- Historical CSV reports
- Latest dashboard JSON

These objects do not require a relational database.

Amazon S3 offers durable, scalable object storage that is well suited to static report files.

### Benefits

- High durability
- Low cost
- Virtually unlimited storage
- Native Lambda integration
- Fine-grained IAM access

### Alternative Considered

Amazon RDS

### Why It Was Not Selected

A relational database would introduce unnecessary complexity, including:

- Database administration
- Schema management
- Higher costs
- Backup management

Object storage better fits the project's requirements.


# Design Decision 4: Separate Lambda Functions

The architecture separates responsibilities into two Lambda functions.

## Cost Fetcher

Responsibilities:

- Retrieve billing data
- Process reports
- Upload objects

## Cost API

Responsibilities:

- Read dashboard data
- Return JSON
- Handle browser requests

### Benefits

- Easier maintenance
- Independent deployment
- Least-privilege IAM
- Better fault isolation
- Clear separation of concerns

# Design Decision 5: Lambda Function URL

## Selected Service

Lambda Function URL

### Why?

The dashboard requires a lightweight HTTPS endpoint to retrieve the latest JSON data.

Lambda Function URLs provide direct HTTPS access without introducing additional services.

### Benefits

- Simple configuration
- Lower operational overhead
- Native Lambda integration
- HTTPS endpoint
- Suitable for lightweight APIs

### Alternative Considered

Amazon API Gateway

### Why It Was Not Selected

API Gateway offers many advanced features, but the dashboard only requires a simple endpoint to retrieve a single JSON file.

For this use case, Lambda Function URLs reduce complexity while meeting functional requirements.

# Design Decision 6: Historical and Current Data Separation

The storage layer separates historical reporting from dashboard data.

## Historical Archive

```
costs/
```

Characteristics:

- Append-only
- Long-term storage
- Never overwritten

Supports:

- Trend analysis
- Auditing
- Historical reporting

## Dashboard Dataset

```
data/latest.json
```

Characteristics:

- Updated daily
- Optimized for dashboard access
- Small file size
- Fast retrieval

This separation prevents historical data from affecting dashboard performance.


# Design Decision 7: Least-Privilege IAM

Each Lambda function receives only the permissions necessary for its responsibilities.

## Cost Fetcher

Can:

- Read Cost Explorer
- Write to Amazon S3
- Generate CloudWatch logs

Cannot:

- Serve browser requests

## Cost API

Can:

- Read latest.json
- Generate CloudWatch logs

Cannot:

- Write objects
- Query Cost Explorer

This reduces the potential impact of compromised credentials.

# Design Decision 8: Private Amazon S3

Amazon S3 is configured with:

- Block Public Access
- IAM-only access
- Private objects

The browser never accesses S3 directly.

Instead, all requests pass through the API Lambda.

Benefits include:

- Reduced attack surface
- Better access control
- Improved protection of billing data

# Design Decision 9: CloudWatch Logging

CloudWatch automatically collects logs from both Lambda functions.

Benefits include:

- Operational visibility
- Debugging
- Error investigation
- Performance monitoring
- Audit support

No additional logging infrastructure is required.

# Scalability Considerations

The architecture automatically scales with demand.

AWS services handle:

- Lambda execution
- Object storage
- Event scheduling
- Logging

As cloud usage grows, the application can continue operating without infrastructure changes.

# Reliability

The architecture improves reliability through:

- Managed AWS services
- Durable Amazon S3 storage
- Stateless Lambda execution
- Automatic EventBridge scheduling
- Centralized CloudWatch logging

This minimizes operational risk while reducing maintenance effort.

# Cost Optimization

The architecture follows AWS cost optimization principles.

Examples include:

- Serverless compute
- Pay-per-request execution
- No idle infrastructure
- Managed storage
- Event-driven scheduling
- Lightweight API implementation

These design choices reduce operational costs while delivering the required functionality.

# Summary

The Serverless AWS Cost Monitoring Dashboard demonstrates how AWS managed services can be combined to build a secure, scalable, and maintainable FinOps solution.

The architectural decisions prioritize automation, least-privilege security, operational simplicity, and cost efficiency. By selecting serverless services over traditional infrastructure, the solution achieves reliable cloud cost monitoring while minimizing both administrative effort and infrastructure costs.
