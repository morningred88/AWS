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



 ## EBS Operation: Volume resizing

* Go to **Volume** menu in EC2 console, select a EBS volume, right click then select Modify volume, you can not only change the volume size, but also change the volume type, for example, change from st1 to gp2. 
* As soon as the volume status changes to "Optimization", you can start repartItion the EBS volume to make the new added volume available to OS.

## EBS Operation: Snapshot

## Snapshot options

* Create volume from snapshot
* Create Image from snapshot
* Copy Snapshot
* Manage fast snapshot restore

![snapshot options](/EBS_EFS_images/snapshot.png)

## Create a volume from snapshot

When you create volume from a snapshot, you are able to: 

* change volume type and size as well
* also select AZ 
* Encryption

![snapshot options](/EBS_EFS_images/snapshot_create_volume.png)

### Snapshot lifecycle policy from a EBS volume

#### Policy types

* EBS snapshot policy
* EBS-backed AMI policy
* Cross-account copy event policy

### Fast snapshot restore (FSR)

FSR is a useful but expensive service. It will be billed per minutes per snapshot. 

The **real world use case** is: create a snapshot from EBS volume > Enable FSR > Restore snapshot to EBS volume > disable FSR



 
