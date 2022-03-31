# SSM

SSM run automations, patches, commands, inventory at scale.

## SSM setup

### EC2 IAM instance profile

An instance profile is a container for an IAM role that you can use to pass role information to an EC2 instance when the instance starts.

Simple word, the IAM role attached to an EC2 instance is its instance profile. 

Currently it has 5 role to choose, but you can create your own IAM role

![EC2_instance_profile1](Policies_standards_images\EC2_instance_profile1.png)

![EC2_instance_profile2](Policies_standards_images\EC2_instance_profile2.png)

## SSM - EC2 setup

I launched an instance, select

* Platform: Amazon Linux instance
* InstanceProfile: EC2RoleForSSM

Then the instance appears on SSM Fleet manager.

Note: AWS managed instance start with "i"

## SSM - on premise setup

Step 1: Download and install SSM agent

Step 2: Hybrid activation in SSM

Step 3: Start SSM agent,  instance appears on SSM Fleet manager

Note: On-premise instance start with letter "mi"

## SSM - Resource group

2 types:

*  Tag based
*  CloudFormation stack based: We can create one resource group for all resources from one CloudFormation stack. 

## SSM- Patch manager

Step 1: Create **patch baseline**. Patch baseline is created based on the Operating System

Step 2: Create maintenance window/

Step 3: In the created maintenance Window, register target for maintenance window.

Step 4: In the created maintenance Window, register task - run command, **AWS-RunPatchBaseline**. This command will run all the patch baseball suitable for the target of the maintenance Window

### SSM - Inventory

Inventory is used to list all the staff running on the target instances tracked by SSM Agent. But more importantly, you are able to see what is available for each instances.

You can set up inventory filtered by resource groups, tags or inventory types.

### SSM - Automations

Labs: create a fully patched golden AMI, assess the AMI afterwards. 

Step 1: In SSM automation

We select automation document **AWS-UpdateLinuxAmi**, add all the parameters, such as resource ami ID, roles. In around 5 minutes, the process is completed, ami-ID of fully patched EC2 instance was created and show as output. 

Step 2: Create a CloudWatch event rule

* Service name: SSM
* Event type: Automation
* Specific detail type: EC2 automation execution status changed
* Status: Success
* Target: Inspector assessment template, to assess the created AMI

From above, you can see **Automation is fully integrated with CloudWatch events**. After automation is done, you can create a CloudWatch event to choose a target to run the test. 

The article below shows much complicated automation process and more AWS resources involved. 

**Reference:**

Building a Secure, Approved AMI Factory Process Using Amazon EC2 Systems Manager (SSM), AWS Marketplace, and AWS Service catalog

chrome-extension://efaidnbmnnnibpcajpcglclefindmkaj/viewer.html?pdfurl=https%3A%2F%2Fd1.awsstatic.com%2Fwhitepapers%2Faws-building-ami-factory-process-using-ec2-ssm-marketplace-and-service-catalog.pdf&clen=434558&chunk=true

## SSM- Session manager

When you start a session to a EC2 instance from session manager, you SSH to the instance. All the session is recorded,   you can come back to check the session history to know, who and when did what changes. 

You can also do session manager to on-premise instances.

# Config

## Overview

* Ensure instance has proper AWS configuration

* Config help us to audit, track the compliance state of the resources over the time. We can build automation to react and remediate on the compliance issues. 

### Dashboard

* Get all resources included
* How many rules we have
* How many resources compliant or noncompliant
* List of noncompliant rule, you can click to the resources to check the details and remediate the resource.

### Resources in Config:

For each resource in the config, show the compliance state for the Timeline:

* Configuration timeline
* Compliance timeline

In each time point of the timeline, you can track:

* Configuration details
* Change
* Relationship: With other AWS resources
* CloudTrail: If we want to know who changes what

Additionally, all the data is available in S3 bucket, we can use Athena to track it. 

You can export configuration items as Json document.

### Rules

Rules is most important part of Config. Config check if the resources is compliant with the rules. 

Each time when you add it a rule, it costs $1 per month.

You can use and edit AWS managed rule or create your own rule.

## Config automation

### CloudWatch Event

CloudWatch rule can be set to automatically react when the rule not compliant.

### Config rules

You can set remediation directly in config rules. This allow you to use SSM automation to execution remediation action. 

### Summary of automation

Both CloudWatch Event and Config rules are possible to achieve remediation. But CloudWatch event have more target possiblities. 

## Automation with CloudWatch event

We can use CloudWatch event for Trusted Advisor to trigger the automation.

Use case:  see github Trust Advisor tools

[Stop Amazon EC2 instances with low utilization](https://github.com/aws/Trusted-Advisor-Tools/blob/master/LowUtilizationEC2Instances)

[Create snapshots for EBS volumes with no recent backup](https://github.com/aws/Trusted-Advisor-Tools/blob/master/AmazonEBSSnapshots)

[Delete exposed IAM Keys and monitor usage](https://github.com/aws/Trusted-Advisor-Tools/blob/master/ExposedAccessKeys) This one can also be triggered using Health

Reference: 

https://github.com/aws/Trusted-Advisor-Tools