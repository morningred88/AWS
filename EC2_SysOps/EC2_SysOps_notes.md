# EC2 for SysOps

## Enhanced networking

ENA is enabled in Amazon Linux 2. 

We launch T2 micro and T3 micro instance both using Amazon Linux 2 AMI.

*  T2 micro does not use ENA
* T3 uses ENA

**To leverage the enhanced networking, you need to use newer generation of EC2 instance**, such as T3 micro.

## EC2 shutdown behavior & Termination protection

* Shutdown behavior: decide only the shutdown behavior using the **OS**. 

  If the shutdown behavior is set as Terminate and you shut down the instance from console, the instance will be shut down but not terminated. 

* Termination protection: decide only the termination protection from **Console or CLI**. 

  If the termination protection is enabled, you can still terminate the instance using OS. 

## Spot instances  

If you want to terminate the spot instance and not have them relaunch, you need first cancel a spot request, then terminate the spot instances. 

## EC2 launch trouble shooting

### Instance volume limits

The maximum number of volumes that your instance can have depends on the operating system and instance type. When considering how many volumes to add to your instance, you should consider whether you need increased I/O bandwidth or increased storage capacity.

#### Nitro System volume limits

Most of these instances support a maximum of 28 attachments. For example, if you have no additional network interface attachments on an EBS-only instance, you can attach up to 27 EBS volumes to it. If you have one additional network interface on an instance with 2 NVMe instance store volumes, you can attach 24 EBS volumes to it.

#### Linux-specific volume limits

Attaching more than 40 volumes can cause boot failures. This number includes the root volume, plus any attached instance store volumes and EBS volumes. If you experience boot problems on an instance with a large number of volumes, stop the instance, detach any volumes that are not essential to the boot process, and then reattach the volumes after the instance is running.

**Reference:**

[AWS documentation> Amazon EC2>Instance volume limits](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/volume_limits.html)

## EC2 Instance Type deep dive
**How to accumulate CPU Credit for burstable T2 and T3 instances?**

T type instance are burstable instance. Burst means a boost of power. If the machine burst, it utilize "burst credit". The burst credit can be monitored in CloudWatch:

- `CPUCreditUsage` – The number of CPU credits spent during the measurement period.
- `CPUCreditBalance` – The number of CPU credits that an instance has accrued. This balance is depleted when the CPU bursts and CPU credits are spent more quickly than they are earned.

CPU credit can be earned per hour. The big the instance is, the more credits can be earn per hour. 
For example:
T2.micro – 6 CPU credits eared per hour
T2.large – 36
T2.2xlarge – 81

**Reference:**

[Monitor your CPU credits](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-performance-instances-monitoring-cpu-credits.html)

## Burstable instances

**AWS News Blog**

T2 Unlimited – Going Beyond the Burst with High Performance

https://aws.amazon.com/blogs/aws/new-t2-unlimited-going-beyond-the-burst-with-high-performance/

The blog explains the pictures.

**CPUSurplusCreditBalance**: T2 Unlimited instances have the ability to borrow an entire day’s worth of future credits, allowing them to perform additional bursting. This borrowing is tracked by the new **CPUSurplusCreditBalance** CloudWatch metric.  If there’s nothing more to borrow from the future, and any further CPU usage is charged at the end of the hour



## 21. CloudWatch Matrics for EC2

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

## EC2 instance connect

You don't need to provide or create new key pair when you launch a new instance. 

EC2> Instances >Check the selected instance> Connect tab > EC2 instance connect > User name: **ec2-user**, you will ssh to the instance. 

## EC2 status checks

### System status checks

System status checks monitor AWS systems on which your instance runs.

Examples of the problem with the underlying host: 

* Loss of network connectivity
* Loss of system power
* Software issue on the physical host
* Hardware issue on the physical host that impact network reachbility

**Solution:**

Either wait for AWS to fix the host or move EC2 instance to a new host by stopping and starting the instance

### Instance status checks

Instance status checks monitor the software and network configuration of your individual instance.

Example of issues:

* Incorrect networking or startup configuration
* Exhausted memory
* Corrupted file system
* Incompatible kernel

**Solution:**

Requires user's involvement to fix

* Restart the EC2 instance
* Or change the configuration of EC2 instance

