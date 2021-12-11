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

  





