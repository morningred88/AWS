# AMI - Amazon Machine Image

## EC2 instance migration using AMIs

### EC2 instance migration between AZ

EC2> Instances> right click the instance ID or  select Action tab on the top>**Create AMI**, then launch instance, select the expect AZ

### Cross-region instance migration

EC2>AMI> right click AMI ID or select Action tab on the top> **copy AMI** option, you can define:

* Destination region
* Name
* Description
* Encryption: Encrypt target EBS snapshot

### Cross-Account AMI Sharing

Sharing an AMI does **not affect the ownership**, which means you still own the AMI. 

You can share the image with other AWS account by selecting **Modify Image Permission**. 

If you set you image as Private, you can **Add Permission** to the specific AWS Account Number. 

If you add **create volume** permission, the account that you share with is allowed to make the own copy of the shared image.

## Delete the created AMI 

**Step1: Deregister the AMI**

**Step2: Delete snapshot created during AMI creation process**

When you deregister an Amazon EBS-backed AMI, it doesn't affect the snapshot(s) that were created for the volume(s) of the instance during the AMI creation process. You'll continue to incur storage costs for the snapshots. Therefore, if you are finished with the snapshots, you should delete them.

Reference:

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/deregister-ami.html#clean-up-ebs-ami

