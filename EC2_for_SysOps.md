# EC2 for SysOps

## EC2 shutdown behavior & Termination protection

* Shutdown behavior: decide only the shutdown behavior using the **OS**. 

  If the shutdown behavior is set as Terminate and you shut down the instance from console, the instance will be shut down but not terminated. 

* Termination protection: decide only the termination protection from **Console or CLI**. 

  If the termination protection is enabled, you can still terminate the instance using OS. 

## EC2 launch trouble shooting

### Instance volume limits

The maximum number of volumes that your instance can have depends on the operating system and instance type. When considering how many volumes to add to your instance, you should consider whether you need increased I/O bandwidth or increased storage capacity.

#### Nitro System volume limits

Most of these instances support a maximum of 28 attachments. For example, if you have no additional network interface attachments on an EBS-only instance, you can attach up to 27 EBS volumes to it. If you have one additional network interface on an instance with 2 NVMe instance store volumes, you can attach 24 EBS volumes to it.

#### Linux-specific volume limits

Attaching more than 40 volumes can cause boot failures. This number includes the root volume, plus any attached instance store volumes and EBS volumes. If you experience boot problems on an instance with a large number of volumes, stop the instance, detach any volumes that are not essential to the boot process, and then reattach the volumes after the instance is running.

**Reference:**

[AWS documentation> Amazon EC2>Instance volume limits](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/volume_limits.html)

## CloudWatch Matrics for EC2

### Aws Provided metrics

* CPU: CPU Utiliztion
  * If we have T2/T3 burst instance, we've got 2 metrics - Credit Usage/Balance 
* Network: Network In/Out

* Status Check: Check if the instance is healthy
  * Instance status: Check if EC2 VM is working
  * System status: Check if the underlying hardware is working

* Disk: Read/write operations Byte for the **instance store**. 

If the instance is attached a EBS volume, the Disk metrics are all zero



## CloudWatch - Unified CloudWatch agent

EC2 can send both the logs and custom metrics to CloudWatch by installing unified CloudWatch agent.

If you want to collect additional system-level metrics for RAM, the only way it to use CloudWatch unified agent.

If you want to configure your agent, you can configure it by using SSM parameter store and storing the configuration in a central place, or you can specify a configuration file alternatively. 



## EC2 Hibernate 

### Supported instance families

- Xen: C3, C4, I3, M3, M4, R3, R4, T2
- Nitro: C5, C5d, M5, M5a, M5ad, M5d, R5, R5a, R5ad, R5d, T3, T3a

**Reference:**

[AWS documentation: Hibernation prerequisites](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/hibernating-prerequisites.html#hibernation-prereqs-supported-instance-families)

### Stop a instance with/without hibernating

If you decide that you no longer need an instance, you can terminate it. As soon as the state of an instance changes to `shutting-down` or `terminated`, we stop charging for that instance. 

You're not charged for instance usage for a hibernated instance when it is in the `stopped` state. However, you are charged for instance usage while the instance is in the `stopping` state, while the contents of the RAM are transferred to the EBS root volume. (This is different from when you stop an instance above without hibernating it.)

