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



