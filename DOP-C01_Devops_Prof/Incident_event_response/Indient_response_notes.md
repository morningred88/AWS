# ASG

## ASG - From launch configuration

### Creating a launch configuration

* Instances: All spot instances or all on-demand instances
* When you want to update, it will create new configuration
* Tags: If the new instance is check, then the tags created in Launch configuration will be pass be the new instances. 

## ASG - From launch template

Advanced way to launch configuration. From template, we can launch: 

* Launch an instance
* Launch an ASG
* Launch spot fleet

### Creating a launch template

* When you want to update, it will create new template or new template **version**
* You can define source template, if you want to create template from the existing template as parent. It can create hierarchy. 
* We have both instance tags and template tags

### Creating ASG with launch template

Fleet composition: You can choose `Combine purchase options and instance types`, which means you can choose a mix of on-demand instances and spot instances and multiple instance types. 

## ASG Scaling policies

### Important parameters

#### Default cooldown

The number of second after a scaling activity complete before another can begin. 

Default: 300 s

#### Instance warm up 

How long needs to wait for the value comes to CloudWatch metrics. For example, we want to track Average CPU utilization for a target tracking scaling. When the new instance launched, the CPU utilization for this instance is 0. 

Default: 60 s

If you set this number bigger than cooldown, it will happen that more instances will be launched. 

#### Disable scale-in

No termination instances, only create instances

### Policies

#### Target tracking

AWS create CloudWatch alarms based on the metrics we selected and value we entered.

**Metrics to track:**

* Average network in
* Average network out
* Average CPU utilization

#### Simple scaling policy

We can based on any alarm we created in advanced add or remove instances. 

#### Step scaling

More steps of simple scaling

## ASG - ALB integration

Step 1: Create ALB, set:

* Instance
* New target group: demo-target-group
* Health check

Step 2: Update launch template

* Add **Target group**: demo-target-group
* Change Health Check Type: From **EC2** to **ELB**
* Add Health check grace period: 60 s

Step 3: Edit target group demo-target-group

* Change attribute **Slow start duration** to 60 s. That means from 0 to 60 s, load balancer will gradually send traffic to the instances. So the instance can warm up the cache.  After 60 s, load balancer will send full traffic to it. 

  ![ASG_Attributes](Incident_response_images\ASG_Attributes.png)

## ASG - HTTPS on ALB

### Requesting a SSL Certification

Step 1: Go to Route 53, create a domain name: muhutech.com

Step 2: Go to ACM, request a certificate 

* Add a domain name in ACM: app.muhutech.com
* Select verification method: DNS validation
* ACM created a CName record for Route 53 to validate

Step 3: Add CName record to Route 53, in order to verify that we do have the right to use the domain name

Step 4:  After verified by Route 53, ACM issued the certificate. 

### Adding HTTPS to ALB

#### Adding listener

AWS console > ELB> To our ALB>**Listener** setting>**Add Listener** button>Protocol - HTTPS, port - 443, SSL certificate from above, Default action - Forward to demo-target-group>save

#### Updating security group

Add port 443 for HTTPS protocol from anywhere

#### How to force to use HTTPS instead of HTTP

AWS console > ELB> To our ALB>**Listener** setting> select Listen HTTP:80> Change Default action: remove forward to demo-target-group, add redirect to HTTPS, 443>save

## ASG- Suspending process

suspend and then resume one or more of the processes for your Auto Scaling group. It is good for troubleshooting. For example, all the instances in ASG becomes unhealthy, we can suspend `ReplaceUnhealthy`, so we can troubleshoot what is the reason. 

### Suspend and resume processes (console)

You can suspend and resume individual processes or all processes.

**To suspend and resume processes**  

1. Open the Amazon EC2 Auto Scaling console at https://console.aws.amazon.com/ec2autoscaling/.

2. Select the check box next to the Auto Scaling group.

   A split pane opens up in the bottom part of the **Auto Scaling groups** page, showing information about the group that's selected.

3. On the **Details** tab, choose **Advanced configurations**, **Edit**.

4. For **Suspended processes**, choose the process to suspend.

   To resume a suspended process, remove it from **Suspended processes**.

5. Choose **Update**.

Reference:

https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-suspend-resume-processes.html#as-suspend-resume

### Processes to suspend

![Suspend_processes](Incident_response_images\Suspend_processes.png)

* Launch: suspend to terminate instances

* Terminate: suspend to terminate instances 

* Add to Load balancer: 

  * It does not affect the instances currently in ELB Target group. 

  * The new added instances will not be added to target group, but in ASG. When you remove the suspending, the instance will not be added automatically. You need manually register the instances to Target group. 
