# Security

## CloudTrail

### CloudTrail storage

A trail stored in S3 is required.

![CloudTrail_Storage_S3](Security_Images/CloudTrail_Storage_S3.png)

 But it is optional to save it in CloudWatch logs.

![](Security_Images/CloudTrail_Storage_CloudWatch_Log.png)

### CloudTrail events

* Management event
  -  free
  - Can choose write or/and read event
  - Can exclude KMS events
* Data event
  - for S3 and Lambda function
  - Can choose write or/and read event
  - need to pay
* Insight event
  - no option to set 
  - Need to pay