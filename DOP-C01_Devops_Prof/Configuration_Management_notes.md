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

The issue is that CloudFormation does not detect a new file has been uploaded to S3 unless one of these parameters change: 

* S3Bucket
* S3Key
* S3ObjectVersion