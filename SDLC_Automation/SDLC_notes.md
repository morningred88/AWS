# SDLC

## CodeCommit

### Pull request

By creating a pull request, we allow people to review our code, and make sure everything looks fine before the code get merged.

#### Ways to create pull request

* CodeCommit > Repository > Branch >Open a branch> Click Create pull request> You will redirect to **Pull requests** of **Repository**
* CodeCommit > Repository > Directly to **Pull requests**

### Triggers & Notifications

We can create notification rules, triggers or CloudWatch event rules for automation by adding SNS or lambda as target.

Notification rule's target is SNS

Trigger's target: SNS, Lambda

### Trigger AWS Lambda

Create an AWS Lambda function, triggered by CodeCommit, using AWS documentation

Step 1: Create a Lambda function - python, add AWSLambdaBasicExecutionRole

Step 2: Add a trigger to the Lambda function from CodeCommit - set any changes of the repository will trigger the Lambda function

Step 3: Copy the sample code from the AWS documentation

Step 4: Test - go to CodeCommit repository, edit a file and make a commit. Go to CloudWatch, you can see the log file

**Note:** The trigger created in the Lambda function can be found in the Triggers of the CodeCommit repository - Go to repository > Setting> Trigger 

Reference:

https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-notify-lambda.html

## CodeBuild

### Environment variables and parameter store

**How to add environment variables?**

* In buildspec.yml

  see buildspec syntax, https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html

* Add environment variable to CodeBuild Enviroment

  CodeBuild> Build projects > CodeBuild project name (MyWebAppCodeBuildMaster)> Edit drop down > Select Environment > Additional configuration > Environment variables 


### Artifact and S3

Build ID: In a code build project, you will get a new build id every time when you run a build. 

When I update the CodeBuild project and try to upload the artifact to S3, I got the following error message:

```
Error in UPLOAD_ARTIFACTS phase: AccessDenied: No AWSAccessKey was presented.
    status code: 403, request id: EWJVCNHW386KXWP4, host id: scOqv0WG4j8sRnjvpAUmfGnaIFXCH9NABUQRK1fPmWPapJmGRCvrQQm6OE+1E5bQYEoRgqErwkU=
```

I added IAM permission to the CodeBuild service role. 

Then I got the solution from Stack Overflow.

It looks to be S3 Bucket access issue. You need to check below things to make this working :

1. Your bucket is in the same region where your codebuild is running.
2. Your bucket has a policy which allows codebuild to upload the objects.
3. Your CodeCommit, CodeBuild has IAM role (Policy) attached to it which has access to S3 bucket.

**Reference:**

https://stackoverflow.com/questions/57900770/aws-codebuild-upload-artifacts-client-error-error-uploading-artifacts-access

## CodeDeploy

### EC2 Setup

**EC2 provision:**

CodeDeploy does not provision AWS service, so we need to launch an EC2 instance. 

**IAM role**

EC2 instance need an IAM role to fetch the code and artifact from S3. 

Role name: EC2RoleForCodeDeploy

Permission: S3 read only permission

**CodeDeploy agent**

AWS documentation: Install the CodeDeploy agent for Amazon Linux 

https://docs.aws.amazon.com/codedeploy/latest/userguide/codedeploy-agent-operations-install-linux.html

**Automate the installation of  CodeDeploy agent**

Option 1: You can also use SSM to install CodeDeploy agent when you create Deployment group. It would be convenient if you have multiple instances in a deployment group. But you need to first install SSM agent into all instances. 

Option 2: For single instance, you can also add user data to automatically install CodeDeploy agent. 

### Application, Deployment groups

Create a CodeDeploy application: MyCodeDeploy

Inside MyCodeDeploy application create a deployment group: MyDevelopmentInstances

* Use tag to select instance/a group of instances
* IAM role: CodeDeployRole 

Inside MyDevelopmentInstances deployment group create a Deployment. 

### Deployment groups discussion

You can have multiple deployment group under one application, such as development, qa and production. You can have multiple instances in one deployment group. The deployment groups can be separated by the tags of the instances. 

### Deployment configuration

#### Deployment Type

* In place: Don't need to create new EC2 instances
* Blue/green: Need to create new EC2 instances. This is perfect for auto scaling group. It is painful if manually provision instances, which is not good option for Devops, we want the thing automated. For blue/green, we also need to enable load balancer as well. 

#### Deployment configuration/setting

Deployment configuration defines the strategy how an application deployed to  EC2 instances, such as all at once, one at a time or half at a time. You can also customize the deployment configuration by percentage or instance numbers. 

It is selected when you create a deployment group. You can override a deployment configuration when you create a deployment. 

### CloudWatch event vs Trigger

You can integrate CodeDeploy through CloudWatch event with many other services, such as SMS, SNS, Lambda, Kinesis stream, etc.

You can send messages using SNS topic through Triggers directly  in CodeDeply.

### On-premises instance setup

#### Working with on-premises instances for CodeDeploy

- **Step 1** – Configure each on-premises instance, register it with CodeDeploy, and then tag it.

  Tagging on-premise instance is only available for CodeDeploy

- **Step 2** – Deploy application revisions to the on-premises instance.

https://docs.aws.amazon.com/codedeploy/latest/userguide/instances-on-premises.html

#### Register an on-premises instance with CodeDeploy

* Use an IAM user ARN to authenticate requests.

  - Suitable for registering a small number of on-premises instances

  - Need to create an IAM user for each on-premise instance

* Use an IAM role ARN to authenticate requests.

  Best for registering a large number of on-premises instances. 

  This way is the most secure way. But it is more painful to do.  So we choose to do hands on using IAM user to register an on-premises instance

https://docs.aws.amazon.com/codedeploy/latest/userguide/on-premises-instances-register.html
