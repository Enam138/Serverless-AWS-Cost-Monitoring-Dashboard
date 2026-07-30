# AWS Cost Explorer Integration

## Overview

AWS Cost Explorer is the primary data source for the **Serverless AWS Cost Monitoring Dashboard**. It provides detailed cost and usage information that enables organizations to understand, monitor, and optimize their AWS spending.

In this project, AWS Cost Explorer is queried automatically each day by the **Cost Fetcher Lambda**. The retrieved billing data is processed, stored in Amazon S3, and presented through a web-based dashboard.

This automated workflow eliminates the need for manual cost reporting while providing near real-time visibility into cloud spending.

# Objectives

The integration with AWS Cost Explorer was designed to:

- Automate AWS cost collection
- Eliminate manual billing exports
- Generate historical cost reports
- Provide current cost information for the dashboard
- Support cloud cost optimization
- Demonstrate serverless FinOps automation


# Architecture

<img width="1219" height="581" alt="akwannya2" src="https://github.com/user-attachments/assets/6124e987-32f7-4ff6-b4e3-c4372d600337" />


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
Dashboard
```

AWS Cost Explorer serves as the authoritative source for billing information.

# What is AWS Cost Explorer?

AWS Cost Explorer is a managed AWS service that provides detailed information about cloud spending.

It allows organizations to:

- Analyze AWS costs
- View service-level spending
- Track historical trends
- Filter costs by time period
- Group costs by AWS service
- Identify spending patterns

The service is accessed programmatically through the AWS Cost Explorer API.

# Data Collection Process

The project retrieves billing information through an automated process.

1. Amazon EventBridge triggers the Cost Fetcher Lambda.
2. The Lambda calls the AWS Cost Explorer API.
3. Cost data is retrieved for the configured reporting period.
4. The data is transformed into CSV and JSON formats.
5. Reports are stored in Amazon S3.
6. The dashboard displays the latest JSON dataset.

This process executes without manual intervention.

# Billing Data Retrieved

The project collects cost information such as:

- Total AWS cost
- Cost by AWS service
- Daily spending
- Reporting period
- Currency
- Service names

These metrics provide a high-level overview of cloud spending.

# Example Cost Explorer Response

A simplified response from the Cost Explorer API may resemble:

```json
{
  "ResultsByTime": [
    {
      "TimePeriod": {
        "Start": "2026-07-01",
        "End": "2026-07-31"
      },
      "Total": {
        "UnblendedCost": {
          "Amount": "125.84",
          "Unit": "USD"
        }
      }
    }
  ]
}
```

The project processes this response into formats suitable for storage and visualization.

# Data Processing

After retrieving billing information, the Cost Fetcher Lambda performs several processing steps.

These include:

- Parsing API responses
- Calculating total costs
- Organizing costs by AWS service
- Generating historical CSV reports
- Creating the latest dashboard JSON

The processed data is optimized for dashboard consumption.

# Historical Reporting

Historical billing data is preserved within Amazon S3.

Example structure:

```
costs/
    year=2026/
        month=07/
            day=30.csv
```

This approach enables long-term cost analysis without overwriting previous reports.

# Dashboard Dataset

The dashboard consumes a simplified JSON file.

Example:

```json
{
  "date": "2026-07-30",
  "totalCost": "125.84",
  "currency": "USD",
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

This format is lightweight and efficient for frontend visualization.

# Benefits of AWS Cost Explorer

Using AWS Cost Explorer provides several advantages:

- Managed billing service
- Accurate cost reporting
- Historical trend analysis
- Service-level visibility
- API-based automation
- Integration with serverless applications

These capabilities make it well suited for FinOps automation.


# Security Considerations

Access to AWS Cost Explorer is restricted through IAM.

Only the Cost Fetcher Lambda is authorized to query billing information.

Additional controls include:

- Dedicated IAM execution role
- Least-privilege permissions
- Temporary AWS credentials
- CloudWatch logging

These measures help protect sensitive financial data.


# Limitations

Although AWS Cost Explorer provides valuable billing insights, several limitations should be considered.

- Cost data is not real-time and may have a delay before appearing.
- Historical data availability depends on AWS billing records.
- API usage requires appropriate IAM permissions.
- Organizations with multiple AWS accounts may require consolidated billing configuration.

These limitations were considered during the design of the project.

# Alignment with FinOps

The integration supports several FinOps principles.

- Cost visibility
- Financial accountability
- Automated reporting
- Historical analysis
- Continuous cost monitoring

Providing timely billing information enables informed cloud cost management decisions.

# Summary

AWS Cost Explorer is the core data source for the **Serverless AWS Cost Monitoring Dashboard**.

By integrating Cost Explorer with Amazon EventBridge, AWS Lambda, and Amazon S3, the project automates the collection, storage, and presentation of cloud billing information. This serverless approach reduces manual effort while improving cost visibility and supporting FinOps best practices.
