# CloudTrail

## Overview

### Create a Trail

* By default, CloudTrail logs event in all AWS regions
* By default, it is encrypted, by SSE-S3
* By default, log file validation is enabled (Details see Integrity validation)
* CloudTrail can be stored in S3 or CloudWatch logs, 
* For S3, I just need to define my bucket **aws-devops-lily1234**. After the trail was created, the trial created bucket key as **aws-devops-lily1234/AWSLogs/MyAccountID**

