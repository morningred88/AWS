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

## SSM Parameter Store

### SSM Parameter reference

#### Referencing AWS Secrets Manager secrets

```
/aws/reference/secretsmanager/secret_ID_in_Secrets_Manager
```

Parameter Store, a capability of AWS Systems Manager, is integrated with Secrets Manager so that you can retrieve Secrets Manager secrets when using other AWS services that already support references to Parameter Store parameters. These services include Amazon Elastic Compute Cloud (Amazon EC2), Amazon Elastic Container Service (Amazon ECS), AWS Lambda, AWS CloudFormation, AWS CodeBuild, AWS CodeDeploy, and other Systems Manager capabilities.

**Reference:** 

https://docs.aws.amazon.com/systems-manager/latest/userguide/integration-ps-secretsmanager.html

