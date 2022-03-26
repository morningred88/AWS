# CloudTrail

## Overview

### Create a Trail

* By default, CloudTrail logs event in all AWS regions
* By default, it is encrypted, by SSE-S3
* By default, log file validation is enabled (Details see Integrity validation)
* CloudTrail can be stored in S3 or CloudWatch logs, 
* For S3, I just need to define my bucket **aws-devops-lily1234**. After the trail was created, the trial created bucket key as **aws-devops-lily1234/AWSLogs/MyAccountID**

### Features about CloudTrails

* CloudTrails in S3 - folder patter:

  bucket name> regions>date>Trails

* Not real time, 15 min delay. If you want something real time, you need CloudWatch Event.

* CloudTrail in CloudWatch event is way easier to read and search. We will get one log line per event

* A bucket policy are added automatically when we create a new trial, so make sure CloudTrail can write to S3 bucket, but resource is the bucket key, not the whole bucket

### CloudTrail log files

CloudTrail log file is json file, it includes:

* Who made the request: userIdentity
* When and to where: eventTime, eventSource
* API call: eventName
* What is the request: requestParameters
* What is the response: responseElements

### CloudTrail log file example

The following example shows that an IAM user named Alice used the AWS CLI to call the Amazon EC2 `StartInstances` action by using the `ec2-start-instances` command for instance `i-ebeaf9e2`.

```
{"Records": [{
    "eventVersion": "1.0",
    "userIdentity": {
        "type": "IAMUser",
        "principalId": "EX_PRINCIPAL_ID",
        "arn": "arn:aws:iam::123456789012:user/Alice",
        "accessKeyId": "EXAMPLE_KEY_ID",
        "accountId": "123456789012",
        "userName": "Alice"
    },
    "eventTime": "2014-03-06T21:22:54Z",
    "eventSource": "ec2.amazonaws.com",
    "eventName": "StartInstances",
    "awsRegion": "us-east-2",
    "sourceIPAddress": "205.251.233.176",
    "userAgent": "ec2-api-tools 1.6.12.2",
    "requestParameters": {"instancesSet": {"items": [{"instanceId": "i-ebeaf9e2"}]}},
    "responseElements": {"instancesSet": {"items": [{
        "instanceId": "i-ebeaf9e2",
        "currentState": {
            "code": 0,
            "name": "pending"
        },
        "previousState": {
            "code": 80,
            "name": "stopped"
        }
    }]}}
}]}
```

Reference:

https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-log-file-examples.html

## CloudTrail log file integrity validation 

To determine whether a log file was modified, deleted, or unchanged after CloudTrail delivered it, you can use CloudTrail log file integrity validation. This feature is built using industry standard algorithms: SHA-256 for hashing and SHA-256 with RSA for digital signing. 

You can enable log file integrity validation by using the AWS Management Console, AWS Command Line Interface (AWS CLI), or CloudTrail API. In AWS Console, choose **Yes** for the **Enable log file validation** option when you create or update a trail. By default, this feature is enabled for new trails. 

### How it works

When you enable log file integrity validation, CloudTrail creates a hash for every log file that it delivers. Every hour, CloudTrail also creates and delivers a file that references the log files for the last hour and contains a hash of each.

### Validate the integrity using AWS CLI

To validate the integrity of CloudTrail log files, you can use the AWS CLI or create your own solution.

Reference:

https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-log-file-validation-enabling.html

https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-log-file-validation-intro.html?icmpid=docs_console_unmapped

## CloudTrail - cross account logging

### Receiving CloudTrail log files from multiple accounts

Very simple, just need 2 steps:

* In s3 bucket, add the resource to the bucket policy, just need to change the account name

  ```
  "Resource": [
          "arn:aws:s3:::myBucketName/optionalLogFilePrefix/AWSLogs/111111111111/*",
          "arn:aws:s3:::myBucketName/optionalLogFilePrefix/AWSLogs/222222222222/*"
        ]
  ```

* The create trial in all other account: Use the same bucket name from first account

**Reference:**

https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-receive-logs-from-multiple-accounts.html

https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-set-bucket-policy-for-multiple-accounts.html

### Sharing CloudTrail log files between AWS accounts

To share log files between multiple AWS accounts, you must perform the following general steps. These steps are explained in detail later in this section.

- Create an IAM role for each account that you want to share log files with.

- For each of these IAM roles, create an access policy that grants read-only access to the account you want to share the log files with.

- Have an IAM user in each account programmatically assume the appropriate role and retrieve the log files.

  1. Sign in to the AWS Management Console and open the IAM console.

  2. Choose the user whose permissions you want to modify.

  3. Choose the **Permissions** tab.

  4. Choose **Custom Policy**.

  5. Choose **Use the policy editor to customize your own set of permissions**.

  6. Type a name for the policy.

  7. Copy the following policy into the space provided for the policy document.

     ```
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Action": ["sts:AssumeRole"],
           "Resource": "arn:aws:iam::account-A-id:role/Test"
         }
       ]
     }
     ```

**Reference:**

https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-sharing-logs.html

https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-sharing-logs-assume-role.html#cloudtrail-sharing-logs-assume-role-create-policy

# Kinesis

## Kinesis Firehose

**What is Streaming ETL?**

Streaming ETL is the processing and movement of real-time data from one place to another. ETL is short for the database functions extract, transform, and load. Extract refers to collecting data from some source. Transform refers to any processes performed on that data. Load refers to sending the processed data to a destination, such as a warehouse, a datalake, or an analytical tool.

**What is Amazon Kinesis Data Firehose?**

Kinesis Data Firehose is a streaming ETL solution. It is the easiest way to load streaming data into data stores and analytics tools. It can capture, transform, and load streaming data into Amazon S3, Amazon Redshift, Amazon OpenSearch Service, and Splunk

Reference:

https://aws.amazon.com/kinesis/data-firehose/faqs/

# CloudWatch

## Metrics need to know

### EC2 metrics

* Disk reads, disk read operations, disk write, disk write operations are only available for instance store EC2. If you have EBS EC2, you only get the disk information directly from EBS service. 
* CPU Credit usage and CPU Credit balance only for T2/T3 instances

But we cannot get RAM and process information

## EBS metrics

* Read bandwidth, Write bandwidth
* Read throughput, write throughput
* Average Queue Length: How much operation has been queued for write into EBS volume, large queue is bad. 
* Time Spent Idle: What percent of time that the disks do nothing
* Average read size
* Average write size: Shows the IO big or small
* Average read latency
* Average write latency
* Burst balance: For gp2 type of volume 

But we cannot get how much is the space left in EBS

### ASG metrics

* Minimum group size
* Maximum group size
* Desired capacity
* In service instance (count)
* Pending instance (count)
* Standby instance (count)
* Terminating instance (count)
* Total instance (count)

Group metrics is the same as for the EC2 instance in ASG, but aggregated. It needs to be enabled. 

### Load balancers metrics

* Target response time
* Request (count)
* HTTP 5xx (count): 5xx errors
* HTTP 4xx (count): 4xx errors
* ELB 4xx (count): 4xx errors
* ELB 5xx (count): 5xx errors
* Client TLS Negotiation error: SSL error
* Target TLS Negotiation error: SSL error
* Target connection error
* Sum rejected connections
* **Active connect count**
* **New connect count**
* Processed bytes
* Consumed Load Balancer capacity units: This is how AWS bill you

### RDS metrics

* CPU Utilization
* Database connection
* Bin Log Disk usage: How much disk has been used
* Freeable Memory
* Free storage space
* Read IOPS
* Read latercy
* Read Throughput
* Swap usage

Swap is hard disk space used as RAM. It is (relatively speaking) **very** slow, but stops computers from crashing when they are trying to deal with more data then their RAM can handle.

To stop processes from using swap — install more RAM.

**Reference:**

https://serverfault.com/questions/48486/what-is-swap-memory

## Custom metric

* Standard resolution: one minute granularity
* High resolution: one second granularity

* Up to 10 dimensions in one custom metric
* We use AWS CLI to push these metrics

### put-metric-data Api

* Create custom metrics and custom name spaces
* Push single data point
* Publishing statistic set, such as sum, minimum, maxmum

## Export of CloudWatch metrics

We can export CloudWatch metrics to other AWS service, such as s3, using Api call `get-metric-statistics`.

If it is just one Time use, we can just use AWS cli

If we want to export regularly, then we need to automate it by creating a cron job and invoke Lambda function. Lambda function will have 2 api calls:

* `get-metric-statistics` to get metric data
* Put the data into s3

## CloudWatch alarms

* We can create CloudWatch alarms based on the CloudWatch metrics
* Any metric can be used to create alarm, including custom metric
* We cannot create a rule for CloudWatch alarm in CloudWatch Event

### Creating a CloudWatch alarm

* One alarm can only use **one** metric

* Alarm condition can be based on **statistic** or **anomaly detection**
* Alarm notification:
  * SNS topic
  * Auto scaling action 
    - ASG group
    - ECS
  * EC2 action: only for EC2 metrics
    * Terminate instance
    * Stop instance
    * Reboot instance

## Unified CloudWatch agent 

* It is called unified agent, because it can push both metrics and logs to CloudWatch metrics and CloudWatch logs. 
* After the agent is installed and configured in EC2 instance, we saved the configuration file into SSM parameter store. 
* The we all the instances run the userdata can fetch the parameter to install CloudWatch agent.

## Metric filter 

* You can create a custom metric based on the pattern in logs, such as finding 404 error.
* Metric filter is on the log group level

## Exporting CloudWatch logs to S3

AWS console> select a log group> Action> Export data to S3 >

If you want to automate the process:

EventBridge> Cronejob> Invoke Lambda function to call the api above.

Drawback: may have delay depending the crone job interval

We also use CloudWatch log subscription to get the log delivery to S3 a bit quickly, but also more expensive.

## CloudWatch log subscription

* a **real-time feed** of log events from CloudWatch Logs

* You can set a **subscription filter** to filter out the log 

* The logs can be delivered to other services such as:

  * an Amazon Kinesis stream, **real time** processing
  * an Amazon Kinesis Data Firehose stream, **near real time**, Firehose need to wait buffer to be filled, depending on the Firehose setting we made. 
  * or AWS Lambda for custom processing, **real time** processing

  We also see Open Search in the console, it is actually use underlying Lamba function to deliver the logs to Open Search

## All kind of logs

* Application logs
* Operating System logs
* Access logs

### Access logs

#### What is an access log?

An access log is a list of all requests for individual files -- such as [Hypertext Markup Language](https://www.theserverside.com/definition/HTML-Hypertext-Markup-Language) files, their embedded graphic images and other associated files that get transmitted -- that people or [bots](https://www.techtarget.com/whatis/definition/bot-robot) have made from a website.

#### What AWS service has access logs?

* S3
* CloudFront
* ELB
* Route 53
* Web server, such as HTTPD access logs

## Creating event rule with CloudTrail API 

* Events from API actions that start with the keywords List, Get, or Describe are not processed by EventBridge. 
* We can create a event rule based on the API call.
* **Detail-type** -  **AWS API Call via CloudTrail** as the value for . 
* Data events (for example, for Amazon S3 object level events, DynamoDB, and AWS Lambda) must have trails configured to receive those events

Note:

A event rule can have multiple targets.

## S3 event notification

* Unrelated to CloudWatch event
* All object level operation
* Targets:
  * SNS
  * SQS
  * Lambda function
* Does not include all S3 event, for the rest you need to use CloudWatch event:
  * Bucket level API call via CloudTrail
  * Object level API call via CloudTrail: must have trail created in CloudTrail
