# Common Issues and Troubleshooting

## Overview

This document provides guidance for diagnosing and resolving common issues encountered when deploying or operating the **Serverless AWS Cost Monitoring Dashboard**.

The troubleshooting procedures cover problems related to AWS Lambda, Amazon EventBridge, AWS Cost Explorer, Amazon S3, IAM permissions, Lambda Function URLs, and Amazon CloudWatch.

# Troubleshooting Approach

When investigating an issue, follow this process:

1. Identify the affected component.
2. Review Amazon CloudWatch logs.
3. Verify IAM permissions.
4. Validate service configuration.
5. Test the affected component individually.
6. Verify end-to-end functionality.

# Issue 1: EventBridge Does Not Trigger Lambda

## Symptoms

- Daily reports are not generated.
- No new files appear in Amazon S3.
- CloudWatch shows no Lambda invocations.

## Possible Causes

- EventBridge rule is disabled.
- Incorrect cron expression.
- Lambda target not configured.
- Missing resource-based policy.

## Resolution

- Verify the EventBridge rule is enabled.
- Confirm the correct Lambda target is configured.
- Check the schedule expression.
- Verify that EventBridge has permission to invoke the Lambda.
- Review CloudWatch Logs for invocation attempts.

# Issue 2: Lambda Function Fails

## Symptoms

- Lambda execution returns an error.
- Reports are not generated.
- CloudWatch records runtime exceptions.

## Possible Causes

- Application error
- Missing environment variables
- Invalid configuration
- Dependency issues

## Resolution

- Review CloudWatch Logs.
- Check Lambda configuration.
- Verify deployment package contents.
- Test the function manually.
- Redeploy the latest application package if necessary.

# Issue 3: Access Denied to AWS Cost Explorer

## Symptoms

```
AccessDeniedException
```

or

```
UnauthorizedOperation
```

## Possible Causes

- Missing IAM permissions
- Incorrect execution role

## Resolution

Verify that the Cost Fetcher Lambda role includes permissions such as:

```text
ce:GetCostAndUsage
```

Confirm that the Lambda function is using the intended execution role.

# Issue 4: Amazon S3 Access Denied

## Symptoms

```
Access Denied
```

or

```
403 Forbidden
```

## Possible Causes

- Incorrect IAM policy
- Bucket policy restrictions
- Incorrect object path

## Resolution

Verify:

- S3 bucket name
- Object key
- IAM permissions
- Bucket policy
- Block Public Access configuration

Ensure the Cost Fetcher Lambda has write access and the Cost API Lambda has read access.

# Issue 5: Dashboard Displays No Data

## Symptoms

- Empty dashboard
- Charts do not render
- Cost values are missing

## Possible Causes

- Missing `latest.json`
- API failure
- Lambda execution failure
- JavaScript errors

## Resolution

Verify:

- `data/latest.json` exists
- Cost API Lambda executes successfully
- Browser Developer Tools for JavaScript errors
- Lambda Function URL response

# Issue 6: Lambda Function URL Returns Error

## Symptoms

```
500 Internal Server Error
```

or

```
404 Not Found
```

## Possible Causes

- Lambda execution failure
- Missing S3 object
- Incorrect Function URL configuration

## Resolution

- Review CloudWatch Logs.
- Verify Function URL settings.
- Confirm that `latest.json` exists.
- Test the Lambda function directly.

# Issue 7: CloudWatch Logs Missing

## Symptoms

- No execution logs
- Missing Log Groups

## Possible Causes

- Lambda never executed
- Missing IAM permissions
- Logging configuration issues

## Resolution

Verify the Lambda role includes:

```text
logs:CreateLogGroup
logs:CreateLogStream
logs:PutLogEvents
```

Then invoke the Lambda manually and confirm log creation.

# Issue 8: Cost Explorer Returns Empty Results

## Symptoms

- Dashboard shows $0.00
- Empty JSON response

## Possible Causes

- Incorrect reporting period
- Recently created AWS account
- No billable resource usage
- Incorrect API request

## Resolution

- Verify the requested date range.
- Confirm AWS resources generated billable usage.
- Validate Cost Explorer query parameters.
- Test the query independently.

# Issue 9: Incorrect Cost Values

## Symptoms

- Dashboard totals do not match the AWS Billing Console.
- Unexpected service costs.

## Possible Causes

- Different reporting periods
- Different cost metrics
- Data processing errors

## Resolution

- Compare reporting periods.
- Verify Cost Explorer API parameters.
- Review JSON transformation logic.
- Compare results with the AWS Billing Console.

# Issue 10: CORS Errors

## Symptoms

Browser displays:

```
Access-Control-Allow-Origin
```

or

```
CORS Policy Error
```

## Possible Causes

- Missing CORS headers
- Incorrect Function URL configuration

## Resolution

Configure the Lambda Function URL to return appropriate CORS headers.

Example:

```text
Access-Control-Allow-Origin
Access-Control-Allow-Headers
Access-Control-Allow-Methods
```

Then clear the browser cache and retest.

# Issue 11: IAM Permission Errors

## Symptoms

```
AccessDenied
```

during Lambda execution.

## Possible Causes

- Missing IAM policy
- Incorrect execution role
- Resource ARN mismatch

## Resolution

- Review attached IAM policies.
- Confirm resource ARNs.
- Verify least-privilege permissions.
- Test the IAM role using the IAM Policy Simulator if required.

# Diagnostic Checklist

Before escalating an issue, verify the following:

| Component | Verification |
|-----------|--------------|
| EventBridge | Rule enabled and scheduled correctly |
| Lambda | Function deployed successfully |
| IAM | Correct execution roles attached |
| AWS Cost Explorer | API permissions available |
| Amazon S3 | Bucket accessible with expected permissions |
| Function URL | HTTPS endpoint responding |
| CloudWatch | Logs generated successfully |
| Dashboard | Displays latest JSON data |

# Useful AWS Services for Troubleshooting

| AWS Service | Purpose |
|-------------|---------|
| Amazon CloudWatch | Review logs and metrics |
| AWS Lambda Console | Test function execution |
| Amazon S3 Console | Verify stored reports |
| IAM Console | Review roles and permissions |
| EventBridge Console | Validate scheduled rules |
| AWS Cost Explorer | Compare billing information |

# Best Practices

To reduce operational issues:

- Use least-privilege IAM roles.
- Monitor Lambda executions with CloudWatch.
- Validate EventBridge schedules after deployment.
- Verify S3 object paths before testing.
- Keep deployment packages up to date.
- Test each component independently before end-to-end testing.
- Document configuration changes.
- Review logs before making infrastructure changes.

# Summary

Most operational issues within the **Serverless AWS Cost Monitoring Dashboard** can be resolved by reviewing Amazon CloudWatch logs, validating IAM permissions, confirming EventBridge scheduling, and verifying Amazon S3 object availability.

By troubleshooting each AWS service individually before testing the complete workflow, administrators can quickly identify root causes and restore normal application operation. This structured approach improves reliability, reduces downtime, and supports efficient maintenance of the serverless architecture.
