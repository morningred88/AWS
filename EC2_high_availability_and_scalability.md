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

### Target group

ALB forward traffic to **different target groups** instead of instances directly like in CLB.

**Target type of Target group** 

* private IP addresses
* ECS tasks
* EC2 instances 
* Lambda functions. 

If you choose a target type as Instances, then all the registered target in the target group must be EC2 instances.

You can **Register Targets** or **Deregister Target** to manage the number of targets.



