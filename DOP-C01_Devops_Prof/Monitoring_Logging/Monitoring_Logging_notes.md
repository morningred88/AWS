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
