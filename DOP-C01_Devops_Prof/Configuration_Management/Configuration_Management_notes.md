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

AWS CloudFormation now offers a **third option: continue update rollback**, which you can initiate from the AWS CloudFormation console or with the **continue-update-rollback** command in the AWS Command Line Interface (CLI). This functionality is enabled for all the stacks in the UPDATE_ROLLBACK_FAILED state.

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

#### Create HelloWorld  and deploy it to Elastic Beanstalk using EB CLI

EB CLI in Gitbash: 

##### Step1: Create a project folder, initiate Elastic Beanstalk

**EB command used in this step:**

```
mkdir HelloWorld
cd HelloWorld
eb init
```

**EB CLI:**

```
xldu@DESKTOP-HJA61V6 MINGW64 ~/local-repository/AWS/DOP-C01_Devops_Prof/Configuration_Management (main)
$ mkdir HelloWorld

xldu@DESKTOP-HJA61V6 MINGW64 ~/local-repository/AWS/DOP-C01_Devops_Prof/Configuration_Management (main)
$ cd HelloWorld

xldu@DESKTOP-HJA61V6 MINGW64 ~/local-repository/AWS/DOP-C01_Devops_Prof/Configuration_Management/HelloWorld (main)
$ eb init
```

**Make selections for eb init:**

Region: us-east-1

Application name: default, the same as directory name, enter

Platform: PHP

Platform branch: latest version as default, PHP 8.0 running on 64bit Amazon Linux 2x, enter

SSH Keypair: Yes, Devkeypair

**elasticbeanstalk and config.yml**:

Now, a `.elasticbeanstalk` folder has created under directory HelloWorld, and `config.yml` has been created in `.elasticbeanstalk` folder.

**Below are content of config.yml file:**

```
branch-defaults:
  default:
    environment: dev-env
    group_suffix: null
global:
  application_name: HelloWorld
  branch: null
  default_ec2_keyname: Devkeypair
  default_platform: PHP 8.0 running on 64bit Amazon Linux 2
  default_region: us-east-1
  include_git_submodules: true
  instance_profile: null
  platform_name: null
  platform_version: null
  profile: null
  repository: null
  sc: null
  workspace_type: Application
```

##### Step 2: Add index.html and deploy the application to EB

**EB command used in this step:**

```
echo "Hello World" > index.html
eb create dev-env
```

**EB CLI:**

```
xldu@DESKTOP-HJA61V6 MINGW64 ~/local-repository/AWS/DOP-C01_Devops_Prof/Configuration_Management/HelloWorld (main)
$ echo "Hello World" > index.html
xldu@DESKTOP-HJA61V6 MINGW64 ~/local-repository/AWS/DOP-C01_Devops_Prof/Configuration_Management/HelloWorld (main)
$ eb create dev-env
Creating application version archive "app-220407_135501069544".
Uploading HelloWorld/app-220407_135501069544.zip to S3. This may take a while.
Upload Complete.
Environment details for: dev-env
  Application name: HelloWorld
  Region: us-east-1
  Deployed Version: app-220407_135501069544
  Environment ID: e-thmbbrreia
  Platform: arn:aws:elasticbeanstalk:us-east-1::platform/PHP 8.0 running on 64bit Amazon Linux 2/3.3.12
  Tier: WebServer-Standard-1.0
  CNAME: UNKNOWN
  Updated: 2022-04-07 18:55:05.410000+00:00
Printing Status:
2022-04-07 18:55:03    INFO    createEnvironment is starting.
2022-04-07 18:55:05    INFO    Using elasticbeanstalk-us-east-1-745361488260 as Amazon S3 storage bucket for environment data.
2022-04-07 18:55:32    INFO    Created security group named: sg-08c92983fa5978356
2022-04-07 18:55:47    INFO    Created load balancer named: awseb-e-t-AWSEBLoa-1BL95F1FUNMH1
2022-04-07 18:55:47    INFO    Created security group named: awseb-e-thmbbrreia-stack-AWSEBSecurityGroup-1IBUPFC3J2RWG
2022-04-07 18:56:04    INFO    Created Auto Scaling launch configuration named: awseb-e-thmbbrreia-stack-AWSEBAutoScalingLaunchConfiguration-hPdnFoTdNszA
2022-04-07 18:56:51    INFO    Created Auto Scaling group named: awseb-e-thmbbrreia-stack-AWSEBAutoScalingGroup-U53NS9NCV57I
2022-04-07 18:56:51    INFO    Waiting for EC2 instances to launch. This may take a few minutes.
2022-04-07 18:57:06    INFO    Created Auto Scaling group policy named: arn:aws:autoscaling:us-east-1:745361488260:scalingPolicy:b033b3a3-504e-487e-af44-3e0a037f911b:autoScalingGroupName/awseb-e-thmbbrreia-stack-AWSEBAutoScalingGroup-U53NS9NCV57I:policyName/awseb-e-thmbbrreia-stack-AWSEBAutoScalingScaleUpPolicy-ADCU8B52CZFB
2022-04-07 18:57:06    INFO    Created Auto Scaling group policy named: arn:aws:autoscaling:us-east-1:745361488260:scalingPolicy:aea6d1b6-828a-4513-963e-c4ea9410bd56:autoScalingGroupName/awseb-e-thmbbrreia-stack-AWSEBAutoScalingGroup-U53NS9NCV57I:policyName/awseb-e-thmbbrreia-stack-AWSEBAutoScalingScaleDownPolicy-14KH58NEHDS7R
2022-04-07 18:57:06    INFO    Created CloudWatch alarm named: awseb-e-thmbbrreia-stack-AWSEBCloudwatchAlarmHigh-23B0M2NGD32K
2022-04-07 18:57:06    INFO    Created CloudWatch alarm named: awseb-e-thmbbrreia-stack-AWSEBCloudwatchAlarmLow-1HBZEFBF8JHJK
2022-04-07 18:57:10    INFO    Instance deployment: You didn't include a 'composer.json' file in your source bundle. The deployment didn't install Composer dependencies.
2022-04-07 18:57:13    INFO    Instance deployment completed successfully.
2022-04-07 18:57:44    INFO    Application available at dev-env.eba-3iycvrr3.us-east-1.elasticbeanstalk.com.
2022-04-07 18:57:45    INFO    Successfully launched environment: dev-env
```

As soon as I entered `eb create dev-env`, it is going to zipup the content in my directory and upload to Elastic Beanstalk, and create dev environment. 

When you go to Elastic Beanstalk console, you can see the application.

##### Step 3: test the application

`eb open` to access the web application 

```
xldu@DESKTOP-HJA61V6 MINGW64 ~/local-repository/AWS/DOP-/Configuration_Management/HelloWorld (main)
$ eb open
```

#### Important EB commands

##### eb status

`eb status`: Show the status of the application, HelloWorld application status is ready, health is green.

```
xldu@DESKTOP-HJA61V6 MINGW64 ~/local-repository/AWS/DOP-C01_Devops_Prof/Configuration_Management/HelloWorld (main)
$ eb status
Environment details for: dev-env
  Application name: HelloWorld
  Region: us-east-1
  Deployed Version: app-220407_135501069544
  Environment ID: e-thmbbrreia
  Platform: arn:aws:elasticbeanstalk:us-east-1::platform/PHP 8.0 running on 64bit Amazon Linux 2/3.3.12
  Tier: WebServer-Standard-1.0
  CNAME: dev-env.eba-3iycvrr3.us-east-1.elasticbeanstalk.com
  Updated: 2022-04-07 18:57:45.097000+00:00
  Status: Ready
  Health: Green
```

##### eb health

`eb health` and `eb health --refresh` : Show the overall health of all the instances. 

`eb health --refresh`  refreshes the result every 10 seconds. It is equivalent to the **Health** tab under dev-env in AWS console.

```
xldu@DESKTOP-HJA61V6 MINGW64 ~/local-repository/AWS/DOP-C01_Devops_Prof/Configuration_Management/HelloWorld (main)
$ eb health

 dev-env                         Ok                         2022-04-07 14:47:02
WebServer                        PHP 8.0 running on 64bit Amazon Linux 2/3.3.12
  total      ok    warning  degraded  severe    info   pending  unknown
    1        1        0        0        0        0        0        0

  instance-id           status     cause                               health
    Overall             Ok
  i-03e7bd7b36f2ea4eb   Ok

  instance-id           r/sec    %2xx   %3xx   %4xx   %5xx      p99      p90
    Overall             0.0         -      -      -      -         -        -       -       -       -
  i-03e7bd7b36f2ea4eb   0.0         -      -      -      -         -        -       -       -       -

  instance-id           type       az   running     load 1  load 5      user %
  i-03e7bd7b36f2ea4eb   t2.micro   1b   51 mins        0.0     0.0         0.0     0.0       0.0   100.0        0.0

  instance-id           status     id   version                   ago
  i-03e7bd7b36f2ea4eb   Deployed   1    app-220407_135501069544   49 mins



xldu@DESKTOP-HJA61V6 MINGW64 ~/local-repository/AWS/DOP-C01_Devops_Prof/Configuration_Management/HelloWorld (main)
$ eb health --refresh
```

##### eb logs

`eb logs`: shows all the logs of the application. It is really helpful to debug the application, without doing ssh to it. It is equivalent to the **Logs** tab under dev-env in AWS console. 

##### eb deploy

`eb deploy`: deploy new version of the application

I change Hello World to Hello world v2 in index.html and redeploy to EB.

It will create a new application archive and will upload it to S3, then update the environment. 

```
xldu@DESKTOP-HJA61V6 MINGW64 ~/local-repository/AWS/DOP-C01_Devops_Prof/Configuration_Management/HelloWorld (main)
$ eb deploy
Creating application version archive "app-220407_150126188796".
Uploading HelloWorld/app-220407_150126188796.zip to S3. This may take a while.
Upload Complete.
2022-04-07 20:01:27    INFO    Environment update is starting.
2022-04-07 20:01:32    INFO    Deploying new version to instance(s).
2022-04-07 20:01:35    INFO    Instance deployment: You didn't include a 'composer.json' file in your source bundle. The deployment didn't install Composer dependencies.
2022-04-07 20:01:43    INFO    Instance deployment completed successfully.
2022-04-07 20:01:50    INFO    New application version was deployed to running EC2 instances.
2022-04-07 20:01:50    INFO    Environment update completed successfully.
```

You can see now 2 application versions are available under Application. 

![Application_version](Configuration_Management_images/Application_versions.png)

#####  eb terminate

`eb terminate`: Will delete the environment. 

### Elastic Beanstalk - saved configuration

#### Saved configuration

In AWS console, there is concept called Saved configuration, a tab on the left hand side under an application:

* Application version
* Saved configuration

#### Advantage of saved configuration

We can quickly use saved configuration to create new environment, or even reconfigure them directly using code. 

We like to save the application configuration in Elastic Beanstalk, for backup and also to reproduce the configuration in another account or another region. So we love to save the configuration as code. 

We can do it by going to CloudFormation and get the CloudFormation template created by Elastic Beanstalk. But it is not native to Elastic Beanstalk. We can create saved configuration using EB CLI.

#### Create, update saved configuration and apply it to EB using EB CLI

**Tasks to practice**:

Step 1: Create a saved configuration: initial-configuration

Step 2: Update the current configuration by adding an environment variable

Step 3: Create a saved configuration for the updated configuration: prod

Step 4: Update the current configuration by adding auto scaling rules

Step 5: Update the saved configuration prod

Step 6: Apply the updated saved configuration prod to the environment

##### Step 1- Backs up the current dev environment configuration as initial-configuration

```
xldu@DESKTOP-HJA61V6 MINGW64 ~/local-repository/AWS/DOP-C01_Devops_Prof/Configuration_Management/HelloWorld (main)
$ eb config save dev-env --cfg initial-configuration

Configuration saved at: C:\Users\xldu\local-repository\AWS\DOP-C01_Devops_Prof\Configuration_Management\HelloWorld\.elasticbeanstalk\saved_configs\initial-configuration.cfg.yml
```

**Notes:**

* Only save the non-default settings of the configuration
* Initial-configuration also appears in AWS console under Saved configuration tab. 

##### Step 2- Create an environment variable 

Create a new environment variable ENABLE_COOL_NEW_FEATURE, set its value as true. 

```
xldu@DESKTOP-HJA61V6 MINGW64 ~/local-repository/AWS/DOP-C01_Devops_Prof/Configuration_Management/HelloWorld (main)
$ eb setenv ENABLE_COOL_NEW_FEATURE=true
2022-04-08 00:27:09    INFO    Environment update is starting.
2022-04-08 00:27:17    INFO    Updating environment dev-env's configuration settings.
2022-04-08 00:27:54    INFO    Instance deployment: You didn't include a 'composer.json' file in your source bundle. The deployment didn't install Composer dependencies.
2022-04-08 00:28:02    INFO    Instance deployment completed successfully.
2022-04-08 00:28:37    INFO    Successfully deployed new configuration to environment.
```

You can the environment variable in configuraton under environment in AWS console.

![Configuration_add_env_variable](Configuration_Management_images/Configuration_add_env_variable.png)



##### Step 3 - save our config from the current state of our environment as prod

```
xldu@DESKTOP-HJA61V6 MINGW64 ~/local-repository/AWS/DOP-C01_Devops_Prof/Configuration_Management/HelloWorld (main)
$ eb config save dev-env --cfg prod

Configuration saved at: C:\Users\xldu\local-repository\AWS\DOP-C01_Devops_Prof\Configuration_Management\HelloWorld\.elasticbeanstalk\saved_configs\prod.cfg.yml
```

A new saved configuration is created. Now we have 2 saved configuration files in saved_configs folder inside .elasticbeanstalk folder. You can confirm this in AWS console.

![saved_configs4](Configuration_Management_images/saved_configs.png)



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
$ eb config put prod
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

![Configuration_updated_auto_scaling](Configuration_Management_images/Configuration_updated_auto_scaling.png)

### .ebextensions for configs

#### Option settings

Elastic Beanstalk defines a large number of configuration options that you can use to configure your environment's behavior and the resources that it contains. Configuration options are organized into namespaces like `aws:autoscaling:asg`, which defines options for an environment's Auto Scaling group.

You can use the `option_settings` key to modify the Elastic Beanstalk configuration and define variables that can be retrieved from your application using environment variables. 

**Reference:**

https://docs.amazonaws.cn/en_us/elasticbeanstalk/latest/dg/command-options.html

https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ebextensions-optionsettings.html

#### Precedence for configuration

During environment creation, configuration options are applied from multiple sources with the following precedence, from highest to lowest:

- Settings applied directly to the environment
- Saved Configurations
- Configuration Files (.ebextensions)
- Default Values

**Reference:**

https://docs.amazonaws.cn/en_us/elasticbeanstalk/latest/dg/command-options.html

### .ebextensions for resources

We can use .config file to add resources, if it does not exist at the EB creation time. For example, to create a DynamoDB table or SNS topic. 

You can not see the resources created in EB, but you can see it in CloudFormation resources. 
