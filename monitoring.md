# Monitoring

## Overview

Monitoring is essential for maintaining the reliability, availability, and operational health of cloud applications. The **Serverless AWS Cost Monitoring Dashboard** uses **Amazon CloudWatch** to monitor Lambda executions, collect logs, track performance metrics, and assist with troubleshooting.

By leveraging CloudWatch, the solution provides visibility into the automated cost collection workflow, helping administrators identify failures, investigate issues, and verify successful daily operations.


# Monitoring Objectives

The monitoring strategy was designed to:

- Monitor automated Lambda executions
- Track application performance
- Capture operational logs
- Detect execution failures
- Support troubleshooting
- Improve operational visibility
- Validate scheduled workflows


# Monitoring Architecture

<img width="1219" height="581" alt="akwannya2" src="https://github.com/user-attachments/assets/663ac446-79a5-497d-803b-9e583d133657" />


```
Amazon EventBridge
        │
        ▼
Cost Fetcher Lambda
        │
        ▼
Amazon CloudWatch Logs
        ▲
        │
Cost API Lambda
        │
        ▼
Browser Dashboard
```

CloudWatch acts as the centralized monitoring service for both Lambda functions.

# Amazon CloudWatch

Amazon CloudWatch is AWS's managed monitoring and observability service.

In this project, CloudWatch provides:

- Centralized logging
- Lambda execution metrics
- Error tracking
- Performance monitoring
- Invocation history

These capabilities help ensure the serverless application operates reliably.

# Lambda Monitoring

The project monitors two Lambda functions.

## Cost Fetcher Lambda

CloudWatch captures:

- Invocation count
- Execution duration
- Memory usage
- Success and failure status
- Runtime errors

This information confirms that scheduled billing data collection executes successfully.

## Cost API Lambda

CloudWatch monitors:

- API invocations
- Response execution time
- Runtime errors
- Log output

These metrics help verify that dashboard requests are processed correctly.

# CloudWatch Logs

Each Lambda function writes execution logs to a dedicated CloudWatch Log Group.

Example:

```
/aws/lambda/cost-fetcher

/aws/lambda/cost-api
```

Typical log entries include:

- Function start
- Cost Explorer query status
- Amazon S3 upload status
- Request processing
- Error messages
- Execution completion

These logs provide detailed insight into application behavior.

# Performance Metrics

CloudWatch automatically records several operational metrics.

| Metric | Description |
|---------|-------------|
| Invocations | Number of function executions |
| Duration | Time required to complete execution |
| Errors | Failed function executions |
| Throttles | Requests rejected due to limits |
| Concurrent Executions | Number of simultaneous executions |

These metrics help evaluate application performance and reliability.

# Monitoring the Scheduled Workflow

The automated workflow is validated by monitoring each execution step.

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
```

Administrators can verify:

- EventBridge successfully triggered the Lambda
- Billing data was retrieved
- Reports were written to Amazon S3
- No execution errors occurred

# API Monitoring

CloudWatch also monitors requests to the Lambda Function URL.

Administrators can review:

- Request frequency
- Execution duration
- Failed requests
- Runtime exceptions

This helps ensure the dashboard remains responsive and available.

# Error Detection

CloudWatch simplifies error identification by recording runtime exceptions.

Examples include:

- Cost Explorer API failures
- Amazon S3 access errors
- IAM permission issues
- Missing objects
- Invalid JSON responses
- Unexpected application exceptions

Captured logs provide the information required for investigation.

# Operational Visibility

Monitoring provides visibility into the overall health of the application.

Administrators can determine:

- Whether scheduled executions occurred
- Whether reports were generated
- Whether the API responded successfully
- Whether users received current billing data

This improves confidence in automated operations.

# Monitoring Benefits

Using Amazon CloudWatch provides several operational advantages.

- Centralized logging
- Managed monitoring
- Automatic metric collection
- Improved troubleshooting
- Faster incident investigation
- Better operational visibility
- Minimal administrative overhead

# Security Monitoring

CloudWatch contributes to security by recording operational events.

Examples include:

- Failed Lambda executions
- Repeated API failures
- Permission-related errors
- Unexpected invocation behavior

Although CloudWatch is not a dedicated security service, these logs can assist in identifying abnormal application behavior.

# Reliability

Continuous monitoring improves application reliability by enabling administrators to detect and resolve issues quickly.

Examples include:

- Identifying failed scheduled executions
- Detecting increased execution times
- Investigating API failures
- Verifying successful report generation

This helps maintain uninterrupted operation of the serverless workflow.

# Future Monitoring Enhancements

Future improvements could include:

- CloudWatch Alarms for failed Lambda executions
- Amazon SNS notifications for operational alerts
- CloudWatch Dashboards for visual monitoring
- AWS X-Ray for distributed tracing
- AWS CloudTrail integration for account activity auditing
- AWS Health Dashboard integration
- Automated anomaly detection

These enhancements would strengthen operational monitoring and reduce response times.

# Best Practices Applied

The monitoring strategy follows several AWS operational best practices.

- Centralized log collection
- Continuous performance monitoring
- Automatic metric collection
- Operational visibility
- Managed monitoring services
- Support for troubleshooting
- Event-driven observability

These practices improve the maintainability and reliability of the application.

# Summary

Amazon CloudWatch provides comprehensive monitoring for the **Serverless AWS Cost Monitoring Dashboard** by collecting logs, recording operational metrics, and tracking Lambda executions.

Through centralized observability, administrators can validate automated workflows, investigate failures, monitor API performance, and maintain the overall health of the serverless application. This monitoring strategy supports reliable operations while aligning with AWS operational best practices.
