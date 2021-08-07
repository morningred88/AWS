# EC2 Storage and Data Management

## EBS Overview

* **EBS is locked to AZ**: EC2 instance is bound to AZ, so is the EBS volume. 

* It is possible for us to create EBS volume and leave them **unattached**. They are not necessary to attach to an EC2 instance. 

* **Delete on termination attribute**: When we launch an EC2 instance and add EBS storage to the instance, by default **Delete on termination** is ticked for root volume but not for other attached EBS volumes. When EC2 instance is terminated, the root volume will be deleted. But if you need to preserve the root EBS volume, you are able to just untick the attribute (attribute disabled). 

## EBS Volume Types

**gp2**: Size of the volume and IOPS are linked. You can only select volume, IOPS will be calculated based on the provisioned volume. 

**gp3**: Size of the volume and IOPS are not linked. You can select the volume and IOPS independently. 

**Notes:**
Only gp2 supports bursting, up to 3,000 iops. gp3 has baseline of 3,000 iops.



 

