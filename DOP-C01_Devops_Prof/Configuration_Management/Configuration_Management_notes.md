# Configuration Management and Infrastructure as Code

## CloudFormation

### Resources

When you create a resource,   type and properties are required. 

### Outputs

CloudFormation uses Outputs to link two stacks, one stack can reference the value from another stack

First stack: Output component, define **Export name** and value

Second stack: To reference the output from the first stack, using function **fn:ImportValue**, the Export name as the value of the function 

### cfn-init

* cfn-init  (**AWS::CloudFormation::Init**) must be in the Metadata of a resource 

* cfn-init script is run in UserData

* UserData logs go to /var/log/cfn-init.log. 
  * If UserData is defined in cfn-init file, you can get UserData output in log  /var/log/cloud-init-output.log file. 
  * If UserData has been defined in cfn-init file, you can find UserData output in log /var/log/cfn-init.log file. cloud-init-output.log still exists, but UserData output is not in it. 

### cfn-signal and wait conditions

cfn-signal are also is run in UserData

**Wait condition** is a resource. When the wait condition receives the success signal, it will complete the creation. Then the stack goes to the completion stage.

### CloudFormation rollbacks

When you create a stack in the console, you can set **Stack creation option** as enabled or disabled in the **Advanced options**. The default setting is enabled. But if we need to debug the stack creation failure, then we need to change the setting as disabled. 

### CloudFormation - nested stack

```
Resources:
  myStack:
    Type: AWS::CloudFormation::Stack
```

**Note:**

Never make change in the nested stack, but in root stack. 

### CloudFormation Change sets

#### How to create change sets

We can create change sets in 2 ways in AWS console:

* Change sets tab on the left hand side> Click Create change sets button 
* Stack> select the stack you want to create change sets>Action> Create change sets for current stack

####  Change sets outcomes

After you reviewed the change sets, you can choose

* Delete - delete all the updates to the CloudFormation template
* Execute - Update the current CloudFormation template, then the stack

#### Change sets in a stack update

When you update a stack and before you finish the update process, you can view the change sets. 

### CloudFormation - Deploying Lambda functions

#### Lambda Code in the CloudFormation - in line

Lambda code is directly in the CloudFormation template

Good for small piece of code.

Restrictions:

* Maximum 4,000 character

* Cannot have dependencies

#### Lambda Code in the CloudFormation - zipped in S3

Lambda code is not in the CloudFormation template, but zipped and stored in S3. The  S3 location url is in the CloudFormation template. 

Advantage:

* Good for big piece of Lambda code
* Can have dependencies

#### Issue of Lambda function not updated (code zipped in S3) 

**Description:**

A Lambda function code is stored in S3 bucket as my-bucket/my-lambda.zip, we use CloudFormation to deploy the Lambda function. The code has been updated and reload to S3, then run CloudFormation to deploy the Lambda. But the Lambda function was not updated. 

**Solution:**

When you upload the file with same name to S3, it will overwrite the existing file. 

The issue is that CloudFormation does not detect a new file has been uploaded to S3 unless one of these parameters change: 

* S3Bucket
* S3Key
* S3ObjectVersion

### CloudFormation -Customer resources

#### Type and use cases

The type of customer resource starts with **Cutomer**::xxxx. You can use customer resources to do anything. But it can mainly be used for the 4 use cases:

- An AWS resource is yet not covered in CloudFormation (new service for example)
- An On-Premise resource
- Emptying an S3 bucket before being deleted
- Fetch an AMI id

#### Customer resource example - empty S3 bucket 

```
Resources:
  myBucketResource:
    Type: AWS::S3::Bucket

  LambdaUsedToCleanUp:
    Type: Custom::cleanupbucket
    Properties:
      ServiceToken: !ImportValue EmptyS3BucketLambda
      BucketName: !Ref myBucketResource
```

ServiceToken is the Lamba ARN,  imported value from another stack, the Export name is EmptyS3BucketLambda in that stack.

### CloudFormation status code

Need to familiar all status code, most important one is update_rollback_failed

#### UPDATE_ROLLBACK_FAILED state

A stack’s state is set to UPDATE_ROLLBACK_FAILED when CloudFormation cannot roll back all changes during an update.

##### Troubleshoting the failure

* Failed to receive the required number of signals: Use the [signal-resource](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/signal-resource.html) command to manually send the required number of successful signals to the resource that's waiting for them
* Changes to a resource were made outside of AWS CloudFormation: Manually sync resources so that they match the original stack's template
* Insufficient permissions: grant permission
* Invalid security token
* Limitation error: Delete resources that you don't need or request a quota increase
* Resource didn't stabilize: A resource didn't respond because the operation might have exceeded the AWS CloudFormation timeout period or an AWS service might have been interrupted. No change is required.

##### How to resolve a stack in UPDATE_ROLLBACK_FAILED state

After a stack was in the UPDATE_ROLLBACK_FAILED state, you had only **two options**: Delete the stack or contact AWS Support to return the stack to a working state. In many cases (for example, if it is running production workloads), deleting the stack is not an acceptable option.

AWS CloudFormation now offers a **third option: continue update rollback**, which you can initiate from the AWS CloudFormation console or with the **[continue-update-rollback](http://docs.aws.amazon.com/cli/latest/reference/cloudformation/continue-update-rollback.html)** command in the AWS Command Line Interface (CLI). This functionality is enabled for all the stacks in the UPDATE_ROLLBACK_FAILED state.

Depending on the cause of the failure (See **Troubleshooting the failure** above), you can manually fix the error and continue the rollback. By continuing the rollback, you can return your stack to a working state (the `UPDATE_ROLLBACK_COMPLETE` state), and then try to update the stack again. 

##### continue update rollback in CloudFormation console

1. Open the [CloudFormation console](https://console.aws.amazon.com/cloudformation/).

2. From the navigation pane, choose **Stacks**.

3. From the **Stack name** column, select the stack that's stuck in UPDATE_ROLLBACK_FAILED status.

4. If you don't want to skip resources, choose **Stack** **Actions**, and then choose **Continue update rollback**.

-or-

If you want to skip FAILED resources during rollback, complete the following:

1. From the **Stack name** column, select the stack that's stuck in UPDATE_ROLLBACK_FAILED status.

2. Choose **Stack Actions**, and then choose **Continue update rollback**.

3. In the **Continue update rollback** dialog box, expand **Advanced troubleshooting**.

4. In the **Resources to skip - optional** section, select the resources that you want to skip.

5. Choose **Continue update rollback**.

**Reference:**

[Continue Rolling Back an Update for AWS CloudFormation stacks in the UPDATE_ROLLBACK_FAILED state](https://aws.amazon.com/blogs/devops/continue-rolling-back-an-update-for-aws-cloudformation-stacks-in-the-update_rollback_failed-state/)

https://aws.amazon.com/premiumsupport/knowledge-center/cloudformation-update-rollback-failed/

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/troubleshooting.html#troubleshooting-errors-update-rollback-failed

### CloudFormation - InsufficientCapabilitiesException

This except can be thrown if CloudFormation need to create IAM resources but no assigned capabilities, CAPABILITY_IAM` and `CAPABILITY_NAMED_IAM.

#### CAPABILITY_IAM and CAPABILITY_NAMED_IAM

Some stack templates might include resources that can affect permissions in your AWS account; for example, by creating new AWS Identity and Access Management (IAM) users. For those stacks, you must explicitly acknowledge this by specifying one of these capabilities.

- If you have IAM resources, you can specify either capability.
- If you have IAM resources with custom names, you *must* specify `CAPABILITY_NAMED_IAM`.
- If you don't specify either of these capabilities, AWS CloudFormation returns an `InsufficientCapabilities` error.

### cfn-hup

We can use cfn-hup to update EC2 metadata.

If EC2 metadata change, cfn-hup ensure all the changes will be applied to EC2. 

We can change EC2 instance directly from CloudFormtion without replacing EC2 instance

### CloudFormation - Stack policies

#### What is stack policy

Stack policy defines the resources that you want to protect from unintentional updates during a stack update. It is a json document that  defines the allow/deney update action to the resources. 

#### Stack policy features

* After you set a stack policy, all of the resources in the stack are protected by default. To allow updates on specific resources, you specify an explicit `Allow` statement for those resources in your stack policy. 
* You can define only one stack policy per stack, but, you can protect multiple resources within a single policy.

* It is similar with IAM policy. A stack policy applies only during stack updates. It doesn't provide access controls like an AWS Identity and Access Management (IAM) policy.

* It is in Advanced options in AWS console under CloudFormation.  
* After you apply a stack policy, you can't remove it from the stack, but you can use the AWS CLI to modify it.

#### Stack policy example

```
{
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "Update:*",
            "Principal": "*",
            "Resource": "*"
        },
        {
            "Effect": "Deny",
            "Action": "Update:*",
            "Principal": "*",
            "Resource": "LogicalResourceId/CriticalSecurityGroup"
        },
        {
            "Effect" : "Deny",
            "Action" : "Update:*",
            "Principal": "*",
            "Resource" : "*",
            "Condition" : {
              "StringEquals" : {
                "ResourceType" : ["AWS::RDS::DBInstance"]
              }
            }
        }
    ]
}
```

CriticalSecurityGroup and any DBInstance are not allowed to be changed. 

#### Modifying a stack policy

We need to update the resources if it is not allowed to be updated in stack policy by modifying the stack policy. 

We can perform the modification of stack policy at update time. This will temporarily change the stack policy to allow update the rescource. CloudFormation is going to revert the stack policy as it was after the update. 

## Elastic Beanstalk

### EB CLI

#### EB CLI manual installation

* Open git bash

* Verify that Python and `pip` are both installed correctly by using the following commands.

  ```
  xldu@DESKTOP-HJA61V6 MINGW64 ~
  $ python --version
  Python 3.9.6
  
  xldu@DESKTOP-HJA61V6 MINGW64 ~
  $ pip --version
  pip 22.0.4 from c:\users\xldu\appdata\local\programs\python\python39\lib\site-packages\pip (python 3.9)
  ```

* Install the EB CLI using `pip`.

  ```
  xldu@DESKTOP-HJA61V6 MINGW64 ~
  $ pip install awsebcli --upgrade --user
  ```

* Add the executable path

  To modify your `PATH` variable (Windows):

  1. Press the Windows key, and then enter `environment variables`.
  2. Choose **Edit environment variables for your account**.
  3. Choose **PATH**, and then choose **Edit**.
  4. Add paths to the **Variable value** `C:\users\xldu\AppData\Roaming\Python\Python39\Scripts\`

* Restart a new git bash window

* Verify that the EB CLI is installed correctly.

  ```
  xldu@DESKTOP-HJA61V6 MINGW64 ~
  $ eb --version
  EB CLI 3.20.3 (Python 3.9.6)
  
  ```

**Reference:**

https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install-windows.html

##### Step 3 - save our config from the current state of our environment as prod

```
xldu@DESKTOP-HJA61V6 MINGW64 ~/local-repository/AWS/DOP-C01_Devops_Prof/Configuration_Management/HelloWorld (main)
$ eb config save dev-env --cfg prod

Configuration saved at: C:\Users\xldu\local-repository\AWS\DOP-C01_Devops_Prof\Configuration_Management\HelloWorld\.elasticbeanstalk\saved_configs\prod.cfg.yml

```

A new saved configuration is created. Now we have 2 saved configuration files in saved_configs folder inside .elasticbeanstalk folder. You can confirm this in AWS console.



##### Step 4 - Add auto scaling rules to prod config

Using text editor to add the auto scaling rule to prod.cfg.yml file

```
AWSEBAutoScalingScaleUpPolicy.aws:autoscaling:trigger:
    UpperBreachScaleIncrement: '2'
  AWSEBCloudwatchAlarmLow.aws:autoscaling:trigger:
    LowerThreshold: '20'
    MeasureName: CPUUtilization
    Unit: Percent
  AWSEBCloudwatchAlarmHigh.aws:autoscaling:trigger:
    UpperThreshold: '50'
```

##### Step 5 - Update saved configuration prod to EB

```
xldu@DESKTOP-HJA61V6 MINGW64 ~/local-repository/AWS/DOP-C01_Devops_Prof/Configuration_Management/HelloWorld (main)
$ eb config put pr
```

##### Step 6 - Apply  the updated saved configuration prod to dev_env 

```
xldu@DESKTOP-HJA61V6 MINGW64 ~/local-repository/AWS/DOP-C01_Devops_Prof/Configuration_Management/HelloWorld (main)
$ eb config dev-env --cfg prod
Printing Status:
2022-04-08 01:08:34    INFO    Environment update is starting.
2022-04-08 01:08:42    INFO    Updating environment dev-env's configuration settings.
2022-04-08 01:09:50    INFO    Successfully deployed new configuration to environment.
```
You can see the auto scaling rule is added in the configuration in AWS console.
![Configuration_updated_auto_scaling](Configuration_Management\Configuration_Management_images\Configuration_updated_auto_scaling.png)
