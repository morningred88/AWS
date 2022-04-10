# ASG

## ASG - From launch configuration

### Creating a launch configuration

* Instances: All spot instances or all on-demand instances
* When you want to update, it will create new configuration
* Tags: If the new instance is check, then the tags created in Launch configuration will be pass be the new instances. 

## ASG - From launch template

Advanced way to launch configuration. From template, we can launch: 

* Launch an instance
* Launch an ASG
* Launch spot fleet

### Creating a launch template

* When you want to update, it will create new template or new template **version**
* You can define source template, if you want to create template from the existing template as parent. It can create hierarchy. 
* We have both instance tags and template tags

### Creating ASG with launch template

Fleet composition: You can choose `Combine purchase options and instance types`, which means you can choose a mix of on-demand instances and spot instances and multiple instance types. 

## ASG Scaling policies

### Important parameters

#### Default cooldown

The number of second after a scaling activity complete before another can begin. 

Default: 300 s

#### Instance warm up 

How long needs to wait for the value comes to CloudWatch metrics. For example, we want to track Average CPU utilization for a target tracking scaling. When the new instance launched, the CPU utilization for this instance is 0. 

Default: 60 s

If you set this number bigger than cooldown, it will happen that more instances will be launched. 

#### Disable scale-in

No termination instances, only create instances

### Policies

#### Target tracking

AWS create CloudWatch alarms based on the metrics we selected and value we entered.

**Metrics to track:**

* Average network in
* Average network out
* Average CPU utilization

#### Simple scaling policy

We can based on any alarm we created in advanced add or remove instances. 

#### Step scaling

More steps of simple scaling

## ASG - ALB integration

Step 1: Create ALB, set:

* Instance
* New target group: demo-target-group
* Health check

Step 2: Update launch template

* Add **Target group**: demo-target-group
* Change Health Check Type: From **EC2** to **ELB**
* Add Health check grace period: 60 s

Step 3: Edit target group demo-target-group

* Change attribute **Slow start duration** to 60 s. That means from 0 to 60 s, load balancer will gradually send traffic to the instances. So the instance can warm up the cache.  After 60 s, load balancer will send full traffic to it. 

  ![ASG_Attributes](Incident_response_images\ASG_Attributes.png)

## ASG - HTTPS on ALB

### Requesting a SSL Certification

Step 1: Go to Route 53, create a domain name: muhutech.com

Step 2: Go to ACM, request a certificate 

* Add a domain name in ACM: app.muhutech.com
* Select verification method: DNS validation
* ACM created a CName record for Route 53 to validate

Step 3: Add CName record to Route 53, in order to verify that we do have the right to use the domain name

Step 4:  After verified by Route 53, ACM issued the certificate. 

### Adding HTTPS to ALB

#### Adding listener

AWS console > ELB> To our ALB>**Listener** setting>**Add Listener** button>Protocol - HTTPS, port - 443, SSL certificate from above, Default action - Forward to demo-target-group>save

#### Updating security group

Add port 443 for HTTPS protocol from anywhere

#### How to force to use HTTPS instead of HTTP

AWS console > ELB> To our ALB>**Listener** setting> select Listen HTTP:80> Change Default action: remove forward to demo-target-group, add redirect to HTTPS, 443>save

## ASG- Suspending process

suspend and then resume one or more of the processes for your Auto Scaling group. It is good for troubleshooting. For example, all the instances in ASG becomes unhealthy, we can suspend `ReplaceUnhealthy`, so we can troubleshoot what is the reason. 

### Suspend and resume processes (console)

You can suspend and resume individual processes or all processes.

**To suspend and resume processes**  

1. Open the Amazon EC2 Auto Scaling console at https://console.aws.amazon.com/ec2autoscaling/.

2. Select the check box next to the Auto Scaling group.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected.

3. On the **Details** tab, choose **Advanced configurations**, **Edit**.

4. For **Suspended processes**, choose the process to suspend.

   To resume a suspended process, remove it from **Suspended processes**.

5. Choose **Update**.

Reference:

https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-suspend-resume-processes.html#as-suspend-resume

### Processes to suspend

![Suspend_processes](Incident_response_images\Suspend_processes.png)

* Launch: suspend to terminate instances

* Terminate: suspend to terminate instances 

* Add to Load balancer: 

  * It does not affect the instances currently in ELB Target group. 

  * The new added instances will not be added to target group, but in ASG. When you remove the suspending, the instance will not be added automatically. You need manually register the instances to Target group. 

### Instance setting in ASG

![ASG_Instance_Setting](Incident_response_images\ASG_Instance_Setting.png)

#### Detach instance

If you detach this instance, the detachment **cannot be undone**. Proceeding with this action will:

- Remove this instance from the Auto Scaling group 
- Replace this instance with a new running instance within the Auto Scaling group 

Add a new instance to the Auto Scaling group to balance the load - Did not check this one

Result: Instance is not more in ASG, desired capacity of ASG changed from 2 to 1. But I can see the instance in EC2 console. 

#### Set to Standby

Proceeding with this action will:

* Remove this instance from the Elastic Load Balancers associated with Auto Scaling group 
* Increases the load on other instances in Auto Scaling group 

Add a new instance to the Auto Scaling group to balance the load - Did not check this one

Result: Instance is still in ASG, desired capacity of ASG changed from 2 to 1. But when I set the instance in service, desired capacity of ASG changed back from 1 to 2. 

This setting is good for trouble shooting, then we can put it back to service. 

## Termination policy

### Default termination policy

* It first determines which **Availability Zones** have the most instances, and it finds at least one instance that is not protected from scale in. 
* Within the selected Availability Zone, the following default termination policy behavior applies: the instances eligible for termination use the **oldest launch template or launch configuration**. 
* After applying the preceding criteria, if there are multiple unprotected instances to terminate, determine which instances are **closest to the next billing hour**. 
* If there are multiple unprotected instances closest to the next billing hour, terminate one of these instances at **random**.

Reference:

https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-termination-policies.html#default-termination-policy

## ASG - Integration with SQS

### Senario 1: Scaling base on SQS backlog 

We can use ASG to process the messages from SQS queue. ASG needs to scale in response to activity in an Amazon SQS queue. Amazon SQS metric like `ApproximateNumberOfMessagesVisible`can not be a metric for ASG scaling, because the number of instance keep changing. 

We need to create a custom CloudWatch metric **Backlog per instance**, **acceptable backlog per instance** as target for the metric.

Reference:

https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-using-sqs-queue.html

### Senario 2: Scale in protection for EC2 instance in ASG

If EC2 instance needs to process long run task, SQS queue can send the EC2 instance a script to enable scale-in protection as long as the task is not done, and remove scale-in protection when the task is completed. 

### Integration with CloudWatch

CloudWatch event, has duplicated event as for SNS, and much more. CloudWatch event's target is much more than SNS. So AWS recomment use CloudEvent than SNS. 

ASG does not integrate with CloudWatch Log. You can install CloudWatch Agent in EC2 instance of ASG to get logs delivered to CloudWatch Log.

## ASG - CloudFormation creation policy

To make sure ASG creation is successful, we need each EC2 instances in AG to send a signal.  

Creation policy: in ASG resource, define how many signal should receive and time out

cfn-signal: In launch configuration resource, send out the signal

## ASG - CloudFormation Update policy

Update policy: in ASG resource, define the update attribute

launch configuration:  A change has been made to it. 

### Update policy attribute

- For `AWS::AutoScaling::AutoScalingGroup` resources, CloudFormation invokes one of three update policies depending on the type of change you make or whether a scheduled operation is associated with the Auto Scaling group.

  - The `AutoScalingReplacingUpdate` and `AutoScalingRollingUpdate` policies apply *only* when you complete one or more of the following:

    - Change the Auto Scaling group's `AWS::AutoScaling::LaunchConfiguration`.
    - Change the Auto Scaling group's `VPCZoneIdentifier` property.
    - Change the Auto Scaling group's `LaunchTemplate` property.
    - Update an Auto Scaling group that contains instances that don't match the current `LaunchConfiguration`.

    If both the `AutoScalingReplacingUpdate` and `AutoScalingRollingUpdate` policies are specified, setting the `WillReplace` property to `true` gives `AutoScalingReplacingUpdate` precedence.

     `AutoScalingReplacingUpdate` : create new ASG group, same as blue/green in CodeDeploy, Immutable in EB.

     `AutoScalingReplacingUpdate` : Change part of instance, such as 1 or 2 at a time

  - The `AutoScalingScheduledAction` policy applies when you update a stack that includes an Auto Scaling group with an associated scheduled action.

Reference:
https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-updatepolicy.html

## ASG - CodeDeploy integration

We want to use CodeDeploy to deploy the application to ASG

Step 1: Create a CloudFormation template for ASG, install CodeDeploy agent in EC2 instances in ASG, Agent file in in S3

Step 2: Create CodeDeploy application, deployment group, deployment

In deployment group, choose target as ASG, so when the new EC2 instance added to ASG, CodeDeploy will automatically deploy the application to the instance. 

Blue/green deployment type for ASG: Use launch configuration to create a new ASG group and new instances, and we need load balancer for this type of deployment. After verifying the new ASG group is working, ELB will reroute the traffic from the old ASG to the new ASG. 

## ASG- CodeDeploy integration troubleshooting

Problem: When new Amazon EC2 instances are launched as part of an Amazon EC2 Auto Scaling group, CodeDeploy can deploy your revisions to the new instances automatically.

2 solutions:

* Redeploy CodeDeploy 
* Set Suspend processes to Launch, so no instance can be launched during CodeDeploy deployment. 

Reference:

https://docs.aws.amazon.com/codedeploy/latest/userguide/integrations-aws-auto-scaling.html

# DynamoDB
## Overview

### Primary key

### Calculate WCU, and RCU

## LSI & GSI

LSI: Local secondary index, the same principle key. It can only be created at table creation time. 

GSi: Global secondary index. 

## Global table

Global table: Allow replications DynomoDB table for multiple regions.

Table can be updated in both regions.

Table must be empty, and DynamoDB stream is enabled. 

## DynamoDB Stream

The underlying of DynamoDB stream is Kinesis stream.

Limitation: No more than 2 most should be read from the same streams shard at the same time. That means, we cannot connect 3 Lambda function to DynamoDB. We will get throttling. 

Solution: We can connect 1 Lambda function to DynamoDB stream, and write the stream to SNS. Then we can have many Lambda functions connect to SNS. 

# S3

## 3 kinds of logs for S3

1. **CloudTrail**: Which logs almost all API calls at Bucket level 
2. **CloudTrail Data Events**: Which logs almost all API calls at Object level 
3. **S3 server access logs**: Which logs almost all ([best effort server logs delivery](https://docs.aws.amazon.com/AmazonS3/latest/dev/ServerLogs.html#LogDeliveryBestEffort)) access calls to S3 objects. 

2 and 3 seem similar functionalities but they have some differences which may prompt users to use one or the other or both:

- Both works at different levels of granularity. e.g. CloudTrail data events can be set for all the S3 buckets for the AWS account or just for some folder in S3 bucket. Whereas, S3 server access logs would be set at individual bucket level
- The S3 server access logs seem to give more comprehensive information about the logs like BucketOwner, HTTPStatus, ErrorCode,

* CloudTrail does not deliver logs for requests that fail authentication (in which the provided credentials are not valid). However, it does include logs for requests in which authorization fails (AccessDenied) and requests that are made by anonymous users.
* If a request is made by a different AWS Account, you will see the CloudTrail log in your account only. The logs for the same request will however be delivered in the server access logs of your account.

Reference:

https://stackoverflow.com/questions/34136861/aws-s3-bucket-logs-vs-aws-cloudtrail

# CloudFormation stack set

* **Maximum concurrent action**: Set how CloudFormation deployed. If you set the value as 1, that means you want to deploy CloudFormation only one region or one account at a time. When it is done, then deploy to another region or account.
* Failure tolerance: Do you allow the CloudFormation failed in any region or any account? If you set the value as 1, that means you allow CloudFormation deployment failed in one region or one account.  
