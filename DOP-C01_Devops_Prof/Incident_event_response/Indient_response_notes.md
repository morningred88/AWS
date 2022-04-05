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
