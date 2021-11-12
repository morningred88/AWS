# EC2 High Availability and Scalability

## What is high availability and Scalability

### Horizontal scaling

Horizontal scaling implies distributed system. It is very common for web applications/modern applications.

Horizontal scaling can be used for Auto Scaling Group and Load Balancer. 

**Note:**

Not every application is capable for horizontal scaling. 

### High availability

Run instance for the same application across multi AZ.

This is for Auto Scaling Group that has multi AZ enabled and Load Balancer that has multi AZ enabled as well. 

## Elastic load balancing (ELB) 

### Why use a load balancer?

Provide SSL termination for your websites - means that  you have HTTPS encrypted traffic for your website.

### Configure health check for CLB

```
Ping Protocol: HTTP

Ping port: 80

Ping path: /  
```

**Advanced Details:**

```
Response Timeout: 4 second

Interval: 5 second

Unhealthy threshold: 2

Healtht threshold: 3
```

**Explanation of the matrix:**

* **Ping path:** We need to give the correct path. If you give the input as */random*, but you check the path and  the page cannot be found, you will get unhealthy status.
* **Response Timeout:** The amount of the time to wait to receive a response from the health check. in seconds.

* **Interval:** the interval the CLB wait for doing another health check of an individual instance, in seconds. Interval time must be greater than response timeout.

* **Unhealthy threshold:** How many failed healthy checks in a row to decide the unhealthy status 
* **Healthy threshold:** How many successful healthy checks in a row to decide the healthy status 

**Reference:**

[Configure health checks for your Classic Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-healthchecks.html)

## 59/60 Application load balancer (ALB)

### Network mapping

The load balancer routes traffic to targets in the selected subnets, and in accordance with your IP address settings.

Select at least two Availability Zones and one subnet per zone. The load balancer routes traffic to target in these Availability Zones only. 

**Test:**

I launched 3 instances in us-est-1a,  us-est-1b,  us-est-1c, respectively. 

I created an ALB with networking mapping only for us-est-1a and us-est-1b,  but not for us-est-1c.

I create a target group for the ALB and added all 3 instances into the target group.

When you check the health status in the target group, the increase for us-est-1c grayed out, its Health Status is Unused, detailed message is: 

**Target is in an Availability Zone that is not enabled for the load balancer**.

### Security group editing inbound rule error

In AWS EC2, I start a Application Load Balancer in front of a target group containing EC2 instances . I want to change the inbound rule of Security Group of EC2 instance . A security group for Classical Load Balancer (it's name is **elb-sg**) was also started. when i am changing source of an inbound rule of security group for EC2 instances to the **elb-sg**, following error is prompted:

**You may not specify a referenced group id for an existing IPv4 CIDR rule.**

**Solution:**

I solved this from deleting the existing rule and creating a new rule.

### Target group

ALB forward traffic to **different target groups** instead of instances directly like in CLB.

**Target type of Target group** 

* private IP addresses
* ECS tasks
* EC2 instances 
* Lambda functions. 

If you choose a target type as Instances, then all the registered target in the target group must be EC2 instances.

You can **Register Targets** or **Deregister Target** to manage the number of targets.

## 61/62 NLB

### Static IP address per subnet

When you create an internet-facing load balancer, you can optionally specify one Elastic IP address **per subnet**. If you do not choose one of your own Elastic IP addresses, Elastic Load Balancing provides one Elastic IP address per subnet for you. These Elastic IP addresses provide your load balancer with **static IP addresses** that will not change during the life of the load balancer. You cannot change these Elastic IP addresses after you create the load balancer.

**Reference:**

[Network Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/network-load-balancers.html)

### Target group for NLB

* EC2 instances

* Private IP addresses

  Scenario: If you need NLB to forward traffic to your server on premise. You can allocate a static IP address for your local server, then connect it with NLB in the upstream. 

* ALB

  **Scenario:** If you need a static IP address in the front, but you still need ALB to manage the application servers, such as using the features of listener rules. 

## CLB vs ALB vs NLB

**CLB**

* can **only forward traffic to EC2 instances**
* Can only forward traffic to **single application**
* There is **no network mapping** for CLB
* **Health checks** are set at CLB level
* **Security group** for downstream instances:  HTTP source is the security of CLB

**ALB**

* **Forward traffic to different target groups**, the type of target group can be EC2 instances, but also include other types, such as Lambda function, private IP address, ECS etc
* Can forward traffic to **multiple applications**, or to multiple target group, each target group is an application. 
* **Network mapping** for ALB
* **Health checks** are set at target group level
* **Security group** for instances in target group:  HTTP source is the security of ALB
* **Protocol to target group:** HTTP. You can check the protocol and port from ELB in target group. 

**NLB**

* **Forward traffic to different target groups**, the type of target group can be EC2 instances, but also include other types,  private IP address or ALB.
* Can **forward traffic** to single application/single target group, or first forward traffic to ALB then multiple applications.
* **Network mapping** for NLB
* **Health checks** are set at target group level
* **Security group** for instances in target group:  HTTP source anywhere, because **NLB does not have security group**, it forward the external TCP traffic directly to target group. 
* **Protocol to target group:** TCP



## Sticky sessions for your Application Load Balancer

### General information about stickyness

* Application Load Balancers support both duration-based cookies and application-based cookies.
* Sticky sessions are enabled at the target group level.
* You can use a combination of duration-based stickiness, application-based stickiness, and no stickiness across all of your target groups.
* For both stickiness types, the Application Load Balancer resets the expiry of the cookies it generates after every request.
* If a cookie expires, the session is no longer sticky and the client should remove the cookie from its cookie store.

### Requirements

- An HTTP/HTTPS load balancer.
- At least one healthy instance in each Availability Zone.
