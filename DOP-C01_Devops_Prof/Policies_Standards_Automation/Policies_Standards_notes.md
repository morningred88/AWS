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