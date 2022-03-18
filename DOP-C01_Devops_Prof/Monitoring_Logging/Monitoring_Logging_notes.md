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
