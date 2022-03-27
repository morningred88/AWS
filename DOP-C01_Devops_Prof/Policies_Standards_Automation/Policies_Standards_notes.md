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