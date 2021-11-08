# EC2 for SysOps

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

