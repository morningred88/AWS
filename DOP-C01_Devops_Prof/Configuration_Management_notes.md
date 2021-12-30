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