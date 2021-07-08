# Monitoring, Auditing and performance

## CloudWatch Metrics

We are required to provide all of the identifiers, **Namespace, Name, and Dimensions** to uniquely identify a single Metric

### Metric namespace

Namespace is based on AWS services, such as ELB, EC2, RDS ...

As Amazon CloudWatch is a regional service, Namespaces are unique within a Region.

![Metrics_Namespace](/Monitor_Audit_SysOps/CloudWatch_CloudTrail_images/Metrics_Namespace.png)

### Metric name

Names are unique within a Namespace, e.g., *CPUUtilization* is distinct from *DiskReadOps*

### Metric dimension

The key to distinguishing between Metrics with the same Name are Dimensions

*A dimension is a name/value pair that is part of the identity of a metric. You can assign up to 10 dimensions to a metric.*

![Metric_name_search](/Monitor_Audit_SysOps/CloudWatch_CloudTrail_images/Metric_name_search.png)

When I search a metric using name CPUUtilization,  it shows up in different dimensions under different namespaces. 

**Dimension for CPUUtilization in EC2: InstanceId**

Then I open EC2>Per-Instance Metrics. I can see 15 metrics, for 15 different InstanceID. 

Here the dimension name for metric CPUUtilization in namespace EC2 is InstaceID.

![Metric_name_search](/Monitor_Audit_SysOps/CloudWatch_CloudTrail_images/Dimension_InstanceId.png)

**Dimension for CPUUtilization in RDS: DBInstanceIdentifier**

You can see the same metric name under namespace RDS with dimension name DBInstanceIdentifier.

![Dimension_DBInstanceIdentifier](/Monitor_Audit_SysOps/CloudWatch_CloudTrail_images/Dimension_DBInstanceIdentifier.png)



### CloudWatch Metrics Hierarchy - 3 Level structure

CloudWatch console > Metrics> All metrics

*  **Namespace** level,  Such as  EC2
  * **Dimensions level**, such as By AutoScaling group 
    * **Metrics level**, such as NetworkOut



## CloudWatch Custom metrics

CloudWatch **Unified agent** does use PutMetricData API call to push metric data to CloudWatch regularly. 

## CloudWatch logs

### Metric filter

**Filter**: A pattern of regular expression that can be filtered in the log streams.

Metric Filter: A custom metric for a filter in a log group.

You can use metric filters to monitor events in a log group as they are sent to CloudWatch Logs. You can monitor and count specific terms or extract values from log events and associate the results with a metric.

### Subscription filter

Specify a filter and data in a log group will be streamed by one of the following AWS services:

* Elastic searching with AWS managed lambda function
* Kinesis firehose
* Kinesis data stream
* Lambda function

You can create 2 subscription filters per log group. 

## EventBridge 

EventBridge is its own service. It builds upon and extends CloudWatch Events.

You cannot create any rules in CloudWatch Events any more. You will be redirected to EventBridge. 

### Eventbus

A Eventbus can have lots of rules. 

You can use both default or custom event bus. I tried to created CodePipeline rule and put it both in default and my own event bus. Both works. The good things is you can also select third party service or your own application to create some events. 

## CloudTrail

* You can add multiple accounts in your organization to one CloudTrail. So you can mange everything directly from CloudTrail severice. 

* CloudTrail is not “real-time”
  * It might take up to 15 minutes for the events to appear in CloudTrail. 
  * Delivers log files to an S3 bucket every 5 minutes

* Create a trail to export event to S3 or CloudWatch

### Export a trail to S3: 

* It will create a S3 bucket, and add a folder for each region. You need to go your region to find the events. 
* You cannot read the events directly from S3, you can download and unzip it, the event is written in json. 
* But you can use Athena to analyze the CloudTrail events.

### Export a trail to CloudWatch logs

* If you don't have cloudtrail log group in CloudWatch, it will create a log group. The event in a CloudTrail is going to be written in one log stream. 

* You can read the event directly

## AWS Config

AWS config is not a part of free tier. The more resources you recorded, the more you have to pay.

After the rescources have been discovered, you can go to AWS Config console> Resources menu> You can see **resource timeline** or **manage rescources**.

### AWS Config rule

We can define rules to check the resources if you are complied to the rule. 

For example, rule restricted-ssh, whether security groups that are in use disallow unrestricted incoming SSH traffic. If SSH inbound rule is opened to everyone 0.0.0.0/0, the security group is **Noncompliant**.

#### AWS managed rule

The rules are created by AWS.

Rule restricted-ssh is AWS managed rule.

#### Custom rule

Create custom rules and add them to AWS Config. **Associate each custom rule with an AWS Lambda function**, which contains the logic that evaluates whether your AWS resources comply with the rule. 

### Setup remediation action for noncompliant resources

Resources menu> Select noncompliant resources> Rule applied > Select a rule that the compliance status is noncompliant> click Action> Select **manage remediation**:

* **Automatic remediation**: The remediation action gets triggered automatically when the recources in scope become noncompliant. 
* **Manual remediation**: You need to define a remediation action, either choose SSM automation document or create your own. 
