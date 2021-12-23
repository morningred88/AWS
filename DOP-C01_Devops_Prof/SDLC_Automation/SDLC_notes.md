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

### Overview

* CodeBuild is AWS managed service
* Serverless, leverage the docker container

### First build 

#### Create a build project

In the AWS console > CodeBuild> Create project> Configure:

* Source provider
  - Amazon S3
  - AWS CodeCommit
  - GitHub
  - Bitbucket
  - GitHub Enterprise
* Reference for CodeCommit:
  - Branch
  - Git tag: Different versions of an application
  - Commit ID: The single commit

* Environment: Choose either **Managed image** or **Custom image**

  - **Managed image:** Use an image managed by AWS CodeBuild

  - **Custom image:** specify our own docker image if we need specific software installed in the image to perform the build

* Additional configuration

  - Time out: Time out is between 5 minutes and 8 hours, which is much better than Lambda to run the tests. Lambda has time out for 15 minutes.  You cannot do many things in Lambda. 
  - Computer: computer capacity (memory, vCPUs) for CodeBuild docker container.

* Buildspec

* Artifact: What artifact we want to push in the end
* Logs: send logs to CloudWatch and/or S3. The reason we want to send logs to CloudWatch or S3 is because we don't want to lose logs after the docker container is gone. We need to keep the logs for debugging purpose. 

**Note:** 

We have to create single CodeBuild project for each branch, or git tag, or commit ID.

#### Build history

After building the project, you can see if the build is success or failed in Build history, also the duration. Such as my build is 51 seconds, that is how much we have billed for the build. The docker container is gone after the build, then we will not pay it any more. 

### Environment variables and parameter store

**How to add environment variables?**

* In buildspec.yml

  see buildspec syntax, https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html

* Add environment variable to CodeBuild Enviroment

  CodeBuild> Build projects > CodeBuild project name (MyWebAppCodeBuildMaster)> Edit drop down > Select Environment > Additional configuration > Environment variables 

### Artifact and S3 - Upload an artifact from CodeBuild to S3

#### What is artifact? 

When you build a project, we will generate new files out of it. For example, if we build a java project, we will generate a jar file. This build output is an artifact.

The artifact need to be uploaded somewhere, such as S3, in order to be consumed by other programs and then deploy to whatever you want. 

#### IAM permission update for CodeBuild service role

IAM permission will be needed, to allow CodeBuild to write into S3

#### Build ID

Build ID: In a code build project, you will get a new build id every time when you run a build. 

#### Debugging of CodeBuild error

When I update the CodeBuild project and try to upload the artifact to S3, I got the following error message:

```
Error in UPLOAD_ARTIFACTS phase: AccessDenied: No AWSAccessKey was presented.
    status code: 403, request id: EWJVCNHW386KXWP4, host id: scOqv0WG4j8sRnjvpAUmfGnaIFXCH9NABUQRK1fPmWPapJmGRCvrQQm6OE+1E5bQYEoRgqErwkU=
```

But I had added IAM permission to the CodeBuild service role. 

Then I got the solution from Stack Overflow.

It looks to be S3 Bucket access issue. You need to check below things to make this working :

1. Your bucket is in the same region where your codebuild is running.
2. Your bucket has a policy which allows codebuild to upload the objects.
3. Your CodeCommit, CodeBuild has IAM role (Policy) attached to it which has access to S3 bucket.

I checked my bucket is in a different region as CodeBuild.

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

**Create a CodeDeploy application**: MyCodeDeploy

Inside MyCodeDeploy application **create a deployment group**: MyDevelopmentInstances. Deployment group is to define where the application will be deployed, such as EC2 instance, ECS or Lambda. 

* Use tag to select instance/a group of instances
* IAM role: CodeDeployRole 

Inside MyDevelopmentInstances deployment group **create a Deployment**.  Deployment will define the revision location,  actual application code stored in S3. 

#### Compute platform for CodeDeploy

* EC2/On-premises
* AWS Lambda
* Amazon ECS

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

#### Use the register-on-premises-instance command (IAM user ARN) to register an on-premises instance

Launch an EC2 instance as premise instance. Do not give it any IAM role, because it should work as on-premise VM and has nothing AWS on it. 

Then following the steps

https://docs.aws.amazon.com/codedeploy/latest/userguide/register-on-premises-instance-iam-user-arn.html

##### Step 1-3: Create an IAM user for the on-premises instance

* We need to give IAM user programmatically access (IAM user credentials), for step 4, 5
* Assign Permission to the IAM user: S3 read only 

##### Step 4: Add a configuration file to the on-premises instance

```
ssh -i Devkeypair.pem ec2-user@ec2-44-202-14-234.compute-1.amazonaws.com

sudo mkdir -p /etc/codedeploy-agent/conf

sudo nano /etc/codedeploy-agent/conf/codedeploy.onpremises.yml
```

**Note**: If  you restart the instance, public IP v4 address and DNS will change. 

**Content of codedeploy.onpremises.yml :**

```
---
aws_access_key_id: secret-key-id
aws_secret_access_key: secret-access-key
iam_user_arn: iam-user-arn
region: supported-region
```

##### Step 5: Install and configure the AWS CLI in the on-premises instance

We user AWS Linux 2, AWS Cli already pre-installed with Linux 2. But we need to configure it, using the same credentials as in step 4.

Configure  the AWS CLI in the on-premises instance using `aws configure`, like we do it with the local computer.

```
$ aws configure
AWS Access Key ID [None]: secret-key-id
AWS Secret Access Key [None]: secret-access-key
Default region name [None]: us-east-1
Default output format [None]: json
```

##### Step 6: Set the AWS_REGION environment variable (not required, can skip this step)

##### Step 7: Install the CodeDeploy agent to the on-premises instance

In my-webpage, CODEDEPLOY.md file

```
sudo yum update -y
sudo yum install -y ruby wget
wget https://aws-codedeploy-us-east-1.s3.us-east-1.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto
sudo service codedeploy-agent status
```

 We don't need to run one command after another , can paste all the command once.

##### Step 8: Register the on-premises instance with CodeDeploy using AWS CLI

The command dose not run from on-prem instance. But from the CLI of the local computer or AWS CLI. 

```
aws deploy register-on-premises-instance --instance-name AssetTag12010298EX --iam-user-arn arn:aws:iam::745361488260:user/CodeDeployUser-OnPrem
```

Just need to change the IAM user ARN from the instruction.

--instance-name: The name of instance is the name appear in CodeDeploy on-premises instances. You should give a specific name for you to recognize in the future. 

##### Step 9: Tag the on-premises instance

Go to AWS console > CodeDeploy > On-premises instances, you can see the instance with the name of AssetTag12010298EX

## Trouble shooting of registering on-premises instance to CodeDeploy

#### Uninstall the CodeDeploy agent from Amazon Linux

I installed CodeDeploy agent from wrong region, so I need to uninstall the current one.

```
sudo yum erase codedeploy-agent
```

Then I reinstall the CodeDeploy agent from correct region, using the following command to check CodeDeploy agent status:

```
$ sudo service codedeploy-agent status
The AWS CodeDeploy agent is running as PID 3517
```

But the deployment was still failed. 

#### Deregister on-premises instance from CodeDeploy

Then I launched a new EC2 instance, repeat all the steps. 
An error occurred (IamUserArnAlreadyRegisteredException) when calling the RegisterOnPremisesInstance operation: The on-premises instance could not be registered because the request included an IAM user ARN that has already been used to register an instance. Include either a different IAM user ARN or IAM session ARN in the request, and then try again.

Then I have to deregister the previous on-premises instance from CodeDeploy. 

```
aws deploy deregister-on-premises-instance --instance-name AssetTag12010298EX
```

It is working. 

**Note:**
* You can give any instance name for on-premises instances, don't need to be AssetTag12010298EX.
* One IAM user can only be used for one on-premises instance

## CodePipeline - CodeCommit, CodeBuild & CodeDeploy

### Create a CodePipeline

#### Artifact store

**Default location:** Create a default S3 Bucket in your account. It will create a S3 bucket for each pipeline.

**Customer location:** Customer defined S3 bucket. It is best practise to centralize all the CodePipeline in a centralized location. 

If you start to create many many CodePipeline, and keep choosing default location, then you will create many many S3 buckets. You may run into the limit. If you want to centralize all the CodePipelines, then you can choose a customer location. I will use my S3 bucket aws-devops-lily1234.

#### Add source stage

**Source provider:**

* AWS CodeCommit
* Amazon ECR
* Amazon S3
* GitHub
* Bitbucket
* GitHub Enterprise

Here we have Amazon ECR compared to CodeBuild.

One CodePipeline for each branch in our CodeCommit repository.

**Detection option**

* Amazon CloudWatch Events: It is recommended way. When we push a commit into CodeCommit, a CloudEvent rule will trigger the pipeline. 
* AWS CodePipeline:  CodePipeline checks the changes periodically. If CodePipleline check every 30 seconds, then we might have 30 seconds delay when the code get pushed and pipeline get triggered. That could be an issue. 

#### Add deploy stage

We can use CodePipeline in one region, but use CodeDeploy in another region.

We can use one CodePipeline to trigger many code deploys into multiple regions.

As soon as we created pipeline, the source get triggered.  

#### CloudWatch event rule for triggering CodePipeline

Below is the event rule created for triggering the CodPipeline, you can get it from CloudWatch.

Rule name: codepipeline-mywebp-main-833110-rule

Pattern:

```
{
  "source": ["aws.codecommit"],
  "detail-type": ["CodeCommit Repository State Change"],
  "resources": ["arn:aws:codecommit:us-east-1:745361488260:my-webpage"],
  "detail": {
    "event": ["referenceCreated", "referenceUpdated"],
    "referenceType": ["branch"],
    "referenceName": ["main"]
  }
}
```

#### Summary of the created CodePipeline

CodePipeline name: CodePipeline-Lily

3 stages: 

* Source: from CodeCommit, myweb, main branch
* Test: test using CodeBuild from previous lab, update IAM policy to give access to S3 bucket aws-devops-lily1234
* Deploy: CodeDeploy from previous lab

### CodePipeline - Artifacts, encryption and S3

#### Artifacts

Artifacts is the way for the stages in the CodePipeline to communicate one another. Artifacts are passed around between each stages. 

Artifacts produced from Pipeline will be stored in S3 bucket you specified when you create the pipleline. So S3 is the backbone of CodePipeline. CodePipeline interacts with S3 many times to store and retrieve the artifacts. 

#### Artifact stored pattern  in S3

Open the bucket,  where the artifacts of created CodePipeline have been stored a folder named the same as the CodePipeline. There are 2 folders inside:

AWS S3 > buckets> aws-devops-lily1234 > CodePipeline-Lily>SourceArti/

AWS S3 > buckets> aws-devops-lily1234 > CodePipeline-Lily>TestResult/

The folder name SourceArti from pipeline stage name Source, all artifacts are stored in this folder after the Source stage was run successfully

The folder name TestResult from pipeline stage name Test, all artifacts are stored in this folder after the Test stage was run successfully

#### CodePipeline flow through the stages

* CodePipeline pulls the code from CodeCommit and stored in S3 SourceArti folder. 
* TestResultThen CodeBuild will pull that file from S3, test it and put the file into S3 TestResult folder. 
* CodeDeploy will pull that file from S3 and deploy it to EC2 instance. 

#### Difference of artifact in CodeBuild and CodePipeline

Artifact in CodeBuild and CodePipeline are different. 

In buildspec.yaml file, we let my-webpage app to store an artifact into s3 bucket cicd-lilyma-devops.  Even it is in the buildspec.yaml file, Codepipeline does not store the artifact into the bucket cicd-lilyma-devops, but the bucket aws-devops-lily1234.

### CodePipeline - CloudWatch event integration

#### CodePipeline event types 

* **Pipeline execution state change**

  Event pattern

  ```
  {
    "source": ["aws.codepipeline"],
    "detail-type": ["CodePipeline Pipeline Execution State Change"],
    "detail": {
      "state": ["SUCCEEDED"]
    }
  }
  ```

* **Stage execution state change**

  Event pattern

  ```
  {
    "source": ["aws.codepipeline"],
    "detail-type": ["CodePipeline Stage Execution State Change"],
    "detail": {
      "state": ["FAILED"]
    }
  }
  ```

  

* **Action execution state change**

  Event pattern 

  ```
  {
    "source": ["aws.codepipeline"],
    "detail-type": ["CodePipeline Action Execution State Change"],
    "detail": {
      "state": ["FAILED"]
    }
  }
  ```

  

#### Sample event for CodePipeline

Sample event shows what information can be retrieved from the event, so can be passed to the target, such as a Lambda function. 

```
{
  "version": "0",
  "id": "CWE-event-id",
  "detail-type": "CodePipeline Stage Execution State Change",
  "source": "aws.codepipeline",
  "account": "123456789012",
  "time": "2017-04-22T03:31:47Z",
  "region": "us-east-1",
  "resources": ["arn:aws:codepipeline:us-east-1:123456789012:pipeline:myPipeline"],
  "detail": {
    "pipeline": "myPipeline",
    "version": "1",
    "execution-id": "01234567-0123-0123-0123-012345678901",
    "stage": "Prod",
    "state": "STARTED"
  }
}
```

### CodePipeline - Stage actions

#### Action group, parallel and sequential actions

In the console, you can specify a serial sequence for an action by choosing **Add action group** at the level in the stage where you want it to run, or you can specify a parallel sequence by choosing **Add action**. *Action group* refers to a run order of one or more actions at the same level.

Parallel actions are in the same action groups. 

Sequential action are in different action groups. 

Only all the parallel action completed, then CodePipeline will start to start the next action groups. 

#### Runorder

If you manually create or edit a JSON file to create a pipeline or update a pipeline from the AWS CLI,  uer `Runorder` to define the action execution order. 

The default `runOrder` value for an action is 1. The value must be a positive integer (natural number). You cannot use fractions, decimals, negative numbers, or zero. To specify a serial sequence of actions, use the smallest number for the first action and larger numbers for each of the rest of the actions in sequence. To specify parallel actions, use the same integer for each action you want to run in parallel. 

For example, if you want three actions to run in sequence in a stage, you would give the first action the `runOrder` value of 1, the second action the `runOrder` value of 2, and the third the `runOrder` value of 3. However, if you want the second and third actions to run in parallel, you would give the first action the `runOrder` value of 1 and both the second and third actions the `runOrder` value of 2.

**Reference:**

https://docs.aws.amazon.com/codepipeline/latest/userguide/reference-pipeline-structure.html
