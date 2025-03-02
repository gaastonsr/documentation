---
title: Cloud Cost Management
kind: documentation
further_reading:
- link: "https://docs.datadoghq.com/infrastructure/cloud_cost_management/"
  tag: "Blog"
  text: "Gain visibility and control of your cloud spend with Datadog Cloud Cost Management"
---
## Overview

Cloud Cost Management provides insights for engineering and finance teams to see how changes to infrastructure can affect costs. It enables you to understand trends, allocate spend across your organization, and identify inefficiencies.
Datadog ingests your cloud cost data and transforms it into queryable metrics. If costs rise, you can correlate the change with usage metrics to determine the root cause.

To use Cloud Cost Management, you must have an AWS account with access to Cost and Usage Reports (CURs), and have the AWS integration installed in Datadog.

## Setup

To setup Cloud Cost Management in Datadog, you need to generate a Cost and Usage report.

### Prerequisite: generate a Cost and Usage Report

Follow AWS instructions for [Creating Cost and Usage Reports][1], and select the following content options for use with Datadog Cloud Cost Management:

* **Include resource IDs**
* **Automatically refresh your Cost & Usage Report**

Select the following Delivery options:

* Time granularity: **Hourly**
* Report versioning: **Create new report version**
* Compression type: **GZIP**
* Format: `text/csv`

### Configure the AWS integration

Select your AWS management account from the dropdown menu, allowing Datadog to display tags associated with this account. If you have multiple similarly-named management accounts, view the tags associated with a selected account to ensure you have selected the specific account you want.

**Note**: Datadog recommends sending a Cost and Usage Report from an [AWS **management account**][2] for cost visibility into related **member accounts**. If you decide to send a Cost and Usage report from an AWS **member account**, ensure that you have selected the following options in your **management account's** [preferences][3] to allow Datadog to have full visibility into the member account:

* **Linked Account Access**
* **Linked Account Refunds and Credits**
* **Linked Account Discounts**

### Locate the Cost and Usage Report

If you have navigated away from the report that you created in the setup prerequisites section, follow AWS documentation to find and [view your Cost and Usage Reports details][4].

To enable Datadog to locate the Cost and Usage Report, complete the fields with their corresponding details:

* **Region**: This is the region your bucket is located. For example, `us-east-1`.
* **Bucket Name**: This is the name of the s3 bucket that the CUR is saved to.
* **Report Path Prefix**: This is the folder name. If viewing **Report path prefix** from the AWS details page, this is the first section of the path. For example, if **Report path prefix** is displayed as `cur-report-dir/cost-report`, you would enter `cur-report-dir`.
* **Report Name**: This is the name you entered when you generated the report in the prerequisite section. If viewing the **Report path prefix** from the AWS details page, this is the second half of the path. For example, if **Report path prefix** is displayed as `cur-report-dir/cost-report`, you would enter `cost-report`.

### Configure access to the Cost and Usage Report

Configure AWS to ensure Datadog has permissions to access the CUR and the s3 bucket it is stored in by [creating a policy][5] using the following JSON:

{{< code-block lang="yaml" collapsible="true" >}}
{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Sid": "DDCloudCostReadBucket",
          "Effect": "Allow",
          "Action": [
              "s3:ListBucket"
          ],
          "Resource": "arn:aws:s3:::BUCKETNAME"
      },
      {
          "Sid": "DDCloudCostGetBill",
          "Effect": "Allow",
          "Action": [
              "s3:GetObject"
          ],
          "Resource": "arn:aws:s3:::BUCKETNAME/REPORT_PREFIX/REPORT_NAME/*"
      },
      {
          "Sid": "DDCloudCostCheckAccuracy",
          "Effect": "Allow",
          "Action": [
              "ce:Get*"
          ],
          "Resource": "*"
      },
      {
          "Sid": "DDCloudCostListCURs",
          "Action": [
              "cur:DescribeReportDefinitions"
          ],
          "Effect": "Allow",
          "Resource": "*"
      },
      {
          "Sid": "DDCloudCostListOrganizations",
          "Action": [
              "organizations:Describe*",
              "organizations:List*"
          ],
          "Effect": "Allow",
          "Resource": "*"
      }
  ]
}
{{< /code-block >}}

**Tip:** Make note of the name you created for this policy for next steps.

### Attach the policy to the Datadog integration role

Attach the new S3 policy to the Datadog integration role.

1. Navigate to **Roles** in the AWS IAM console.
2. Locate the role used by the Datadog integration. By default it is named **DatadogIntegrationRole**, but the name may vary if your organization has renamed it. Click the role name to open the role summary page.
3. Click **Attach policies**.
4. Enter the name of the S3 bucket policy created above.
5. Click **Attach policy**.

**Note:** Data can take up to 48 to 72 hours after setup to stabilize in Datadog.
## Cost types

You can visualize your ingested data using the following cost types:

| Cost Type            | Description           |
| -------------------- | --------------------- |
| `aws.cost.amortized` | Cost based on applied discount rates plus the distribution of pre-payments across usage for the discount term (accrual basis). |
| `aws.cost.unblended` | Cost shown as the amount charged at the time of usage (cash basis).|
| `aws.cost.blended`   | Cost based on the average rate paid for a usage type across an organization's member accounts.|
| `aws.cost.ondemand`  | Cost based on the list rate provided by AWS. |

## Tag enrichment

Datadog adds out-of-the-box tags to the ingested cost data to help you further break down and allocate your costs. These tags are derived from your [Cost and Usage Report (CUR)][6].

The following out-of-the-box tags are also available for filtering and grouping data:

| Tag                          | Description       |
| ---------------------------- | ----------------- |
| `aws_product`                | The AWS service being billed.|
| `aws_product_family`         | The category for the AWS service being billed (for example, Compute or Storage).|
| `aws_management_account_name`| The AWS management account name associated with the item.|
| `aws_management_account_id`  | The AWS management account ID associated with the item.|
| `aws_member_account_name`    | The AWS memmber account name associated with the item.|
| `aws_member_account_id`      | The AWS member account ID associated with the item.|
| `aws_cost_type`              | The type of charge covered by this item (for example, Usage, or Tax).|
| `aws_pricing_term`           | Whether the usage is Reserved, Spot, or On-Demand.|
| `aws_reservation_arn`        | The ARN of the Reserved Instance that the item benefited from.|
| `aws_savings_plan_arn`       | The ARN of the Savings Plan the item benefited from.|
| `aws_usage_type`             | The usage details of the item (for example, BoxUsage:i3.8xlarge).|
| `aws_operation`              | The operation associated with the item (for example, RunInstances).|
| `aws_region`                 | The region associated with the item.|
| `aws_availability_zone`      | The availability zone associated with the item.|
| `aws_resource_id`            | The resource ID associated with the item.|
| `aws_instance_type`          | The instance types associated with your items.|
| `aws_instance_family`        | The instance family associated with your item (for example, Storage optimized).|
| `is_aws_ec2_compute`         | Whether the usage is related to EC2 compute.|
| `is_aws_ec2_compute_on_demand`| Whether the usage is on-demand.|
| `is_aws_ec2_compute_reservation`| Whether the usage is associated with a Reserved Instance.|
| `is_aws_ec2_capacity_reservation`| Whether the usage is associated with a Capacity Reservation.|
| `is_aws_ec2_spot_instance`   | Whether the usage is associated with a Spot Instance.|
| `is_aws_ec2_savings_plan`    | Whether the usage is associated with a Savings Plan.|

## Cloud costs in dashboards

Visualizing infrastructure spend alongside related utilization metrics can help you spot potential inefficiencies and savings opportunities. You can add cloud costs to widgets in Datadog dashboards by selecting the *Cloud Cost* data source.

{{< img src="infrastructure/cloudcost/cloud_cost_data_source.png" alt="Cloud Cost available as a data source in dashboard widget creation"  >}}

## Further reading

{{< partial name="whats-next/whats-next.html" >}}

[1]: https://docs.aws.amazon.com/cur/latest/userguide/cur-create.html
[2]: https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/consolidated-billing.html
[3]: https://us-east-1.console.aws.amazon.com/cost-management/home?region=us-east-1#/settings
[4]: https://docs.aws.amazon.com/cur/latest/userguide/view-cur.html
[5]: https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create-console.html
[6]: https://docs.aws.amazon.com/cur/latest/userguide/data-dictionary.html
