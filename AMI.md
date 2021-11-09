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

### Steps of AMI deletion

**Step1: Deregister the AMI**

**Step2: Delete snapshot created during AMI creation process**

When you deregister an Amazon EBS-backed AMI, it doesn't affect the snapshot(s) that were created for the volume(s) of the instance during the AMI creation process. You'll continue to incur storage costs for the snapshots. Therefore, if you are finished with the snapshots, you should delete them.

### Console procedure of AMI deletion

**To clean up your Amazon EBS-backed AMI using the console**

1. Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.

2. **Deregister the AMI**

   1. In the navigation pane, choose **AMIs**.

   2. Select the AMI to deregister, and take note of its ID — this can help you find the snapshots to delete in the next step.

   3. Choose **Actions**, and then **Deregister**. When prompted for confirmation, choose **Continue**.

      **Note**

      It might take a few minutes before the console removes the AMI from the list. Choose **Refresh** to refresh the status.

3. **Delete snapshots that are no longer needed**

   1. In the navigation pane, choose **Snapshots**.
   2. Select a snapshot to delete (look for the AMI ID from the prior step in the **Description** column).
   3. Choose **Actions**, and then choose **Delete**. When prompted for confirmation, choose **Yes, Delete**.

**Reference:**

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/deregister-ami.html#clean-up-ebs-ami

## EC2 image builder

You can build both AMI or docker image by allocating the components by using EC2 image builder.

EC2 image builder is free, you just need to pay the underlying resources. 

The image creation process has gone through 4 states: Pending>Building>testing>Distributing. It takes quite a while > 10 minutes. 

### Testing the build AMI

* Launch a EC2 instance using the build AMI

* EC2 instance connect to the EC2 instance
* Testing commands for the 2 components:

```
aws --version
Result: aws-cli/2.3.7

java -version
Result: Openjdk version "11.0.13"
```





