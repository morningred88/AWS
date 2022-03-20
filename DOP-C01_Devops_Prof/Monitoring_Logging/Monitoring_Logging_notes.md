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
