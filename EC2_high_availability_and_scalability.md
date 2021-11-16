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

### Duration-based stickiness

Duration-based stickiness routes requests to the same target in a target group using a load balancer generated cookie (`AWSALB`). The cookie is used to map the session to the target.

When a load balancer first receives a request from a client, it routes the request to a target (based on the chosen algorithm), and generates a cookie named `AWSALB`. It encodes information about the selected target, encrypts the cookie, and includes the cookie in the response to the client.

In subsequent requests, the client should include the `AWSALB` cookie. When the load balancer receives a request from a client that contains the cookie, it detects it and routes the request to the same target. If the cookie is present but cannot be decoded, or if it refers to a target that was deregistered or is unhealthy, the load balancer selects a new target and updates the cookie with information about the new target.

### Application-based stickiness

Application-based stickiness gives you the flexibility to set your own criteria for client-target stickiness. When you enable application-based stickiness, the load balancer routes the first request to a target within the target group based on the chosen algorithm. The **target** is expected to **set a custom application cookie** that matches the cookie configured on the load balancer to enable stickiness. This custom cookie can include any of the cookie attributes required by the application.

When the **Application Load Balancer** receives the custom application cookie from the target, it **automatically generates a new encrypted application cookie** to capture stickiness information. This load balancer generated application cookie captures stickiness information for each target group that has application-based stickiness enabled.

The load balancer generated application cookie does not copy the attributes of the custom cookie set by the target. It has its own expiry of 7 days which is non-configurable. In the response to the client, the Application Load Balancer only validates the name with which the custom cookie was configured at the target group level and not the value or the expiry attribute of the custom cookie. As long as the name matches, the load balancer sends **both cookies**, the **custom cookie set by the target, and the application cookie generated by the load balancer**, in the response to the client.

In subsequent requests, clients have to **send back both cookies** to maintain stickiness. The load balancer decrypts the application cookie, and checks whether the configured duration of stickiness is still valid. It then uses the information in the cookie to send the request to the same target within the target group to maintain stickiness. The load balancer also proxies the custom application cookie to the target without inspecting or modifying it.

**Reference:**

[Sticky sessions for your Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/sticky-sessions.html)

## 66 ELB SSL certificates

**SSL termination: **ELB can terminate TLS traffic. The external traffic talks to the load balancer using HTTPS. ELB can do SSL termination, so ELB can talk to the backend EC2 instance using HTTP. But the traffic goes over the VPC, which is private network, it is secure even without HTTPS. 

You can also load your own certificate to ACM if you want to. 

A **listener** is a process that checks for connection requests, using the protocol and port that you configure.

**SNI:** Using SNI, you are able to have multiple target groups for different web sites for using different SSL certificate. 

### Certificate type

* Choose a certificate from ACM
* Choose a certificate from IAM
* Import manually

## CLB vs ALB vs NLB

**CLB**

* CLB works at Network both layer 4 (TCP, secure TCP) and 7 (HTTP and HTTPS)
* HTTP headers for source IP, port and protocol

* can **only forward traffic to EC2 instances**
* There is **no network mapping** for CLB
* **Health checks** are set at CLB level
* **Security group** for downstream instances:  HTTP source is the security of CLB
* Support only **one SSL certificate**, so serve **only one domain name**. Must use multiple CLB for multiple hostname with multiple SSL certificates

**ALB**

* Network layer 7 only, support HTTPS, HTTP and websocket (HTTP/2)
* **Distribute requests based on** multiple variables, from the **network layer to the application layer**. It is **context-aware**
* HTTP headers for source IP, port and protocol

* **Forward traffic to different target groups**, the type of target group can be EC2 instances, but also include other types, such as Lambda function, private IP address, ECS etc
* Can forward traffic to **multiple applications**, or to multiple target group, each target group is an application. 
* **Network mapping** for ALB
* **Health checks** are set at target group level
* **Security group** for instances in target group:  HTTP source is the security of ALB
* **Protocol to target group:** HTTP. You can check the protocol and port from ELB in target group.
*  Supports multiple listeners with **multiple SSL certificates**. Uses Server Name Indication (SNI) to make it work
* ALB supports **Server Name Indication (SNI)**, which allows it to serve **many domain names** with different SSL certificate for each. There is a limit, however, to the number of certificates you can attach to an ALB, [namely 25 certificates](https://aws.amazon.com/blogs/aws/new-application-load-balancer-sni/) plus the default certificate.
* **Use case:** ALBs are typically used for web applications. If you have a microservices architecture, ALB can be used as an internal load balancer in front of EC2 instances or Docker containers that implement a given service. You can also use them in front of an application implementing a REST API, although [AWS API Gateway](https://aws.amazon.com/api-gateway/) would generally be a better choice here.

**NLB**

* NLB works at **layer 4** only and can handle both TCP and UDP, as well as TCP connections encrypted with TLS.
* **Distribute traffic based on network variables, such as IP address and destination ports.** It is not designed to take into consideration anything at the application layer such as content type, cookie data, custom headers, user location, or the application behavior. It is **context-less**, caring only about the network-layer information
* NLB natively **preserves the source IP address** in TCP/UDP packets

* **Forward traffic to different target groups**, the type of target group can be EC2 instances, but also include other types,  private IP address or ALB.
* **Network mapping** for NLB
* **Health checks** are set at target group level
* **Security group** for instances in target group:  HTTP source anywhere, because **NLB does not have security group**, it forward the external TCP traffic directly to target group. 
* **Protocol to target group:** TCP
*  Supports **multiple SSL certificates**. Uses Server Name Indication (SNI) to make it work
* **Use case:** NLBs would be used for anything that ALBs don’t cover. A typical use case would be a near real-time data streaming service (video, stock quotes, etc.) Another typical case is that you would need to use an NLB if your application uses non-HTTP protocols.

**Reference:**

1. [ELB vs. ALB vs. NLB: Choosing the Best AWS Load Balancer for Your Needs](https://iamondemand.com/blog/elb-vs-alb-vs-nlb-choosing-the-best-aws-load-balancer-for-your-needs/)
2. [AWS — Difference between Application load balancer (ALB) and Network load balancer (NLB)](https://medium.com/awesome-cloud/aws-difference-between-application-load-balancer-and-network-load-balancer-cb8b6cd296a4)

## 69 ELB monitoring and trouble shooting

### Error code

* 5xx: The error on server side, it could be error on load balancer or backend EC2 instances

* 503: EC2 instance is not available to send back the response to load balancer. It is an important code to know. 

### CloudWatch metrics

**BackendConnectionError:** to monitor if EC2 instance is erroring out. 

Latency: How long it takes for client to get the response after a request is sent. 

RequestCount: Request count overall

RequestCountPerTarget: Request received per instance on average. A good metric to monitor for scaling EC2 instance. 

### Connection idle timeout

For each request that a client makes through a load balancer, the load balancer maintains two connections. 

* The front-end connection is between a client and the load balancer. 
* The backend connection is between the load balancer and a target. 

The load balancer has a configured **idle timeout period** that applies to its connections. If no data has been sent or received by the time that the idle timeout period elapses, the load balancer closes the connection. 

For backend connections, in order to avoid to get 504 error: 

* You enable the HTTP keep-alive option for your EC2 instances. You can enable HTTP keep-alive in the web server settings for your EC2 instances. If you enable HTTP keep-alive, the load balancer can reuse backend connections until the keep-alive timeout expires. 
* We also recommend that you configure the idle timeout of your application to be larger than the idle timeout configured for the load balancer.

By default, Elastic Load Balancing sets the idle timeout value for your load balancer to 60 seconds.

**Reference:**

[Connection idle timeout](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/application-load-balancers.html#connection-idle-timeout)

## Target group attribute

### Slow start mode

A way for load balance to send traffic **gradually** to a target. 

* Using slow start mode gives targets **time to warm up** before the load balancer sends them a full share of requests.
* After you enable slow start for a target group, its targets enter slow start mode when they are **considered healthy** by the target group.
* Slow start is very useful for applications that depend on cache and need a warm-up period before being able to respond to requests with optimal performance.

**Reference:**

[Application Load Balancer Announces Slow Start Support for its Load Balancing Algorithm](https://aws.amazon.com/about-aws/whats-new/2018/05/application-load-balancer-announces-slow-start-support/)

### Request routing algorithms

* **Least outstanding requests**: The next instance to receive request is the instance that is less busy - has lowest number of pending or unfinished requests. This request algorithm is used for ALB and CLB.
* **Round robin:** Choose the target one after another, regardless how many outstanding requests the target has. This request algorithm is used for ALB and CLB.
* **Flow hash:** Each TCP/UDP connection is going to routed to the single target for the life of the connection, which is sort of equivalent to the sticky sessions. Whenever a use make a request to EC2 instance, all the information will be hashed through **flow hash algorithm**. Thanks to the hash number, the same request from the same user to the same EC2 instance as long as TCP connection is open. Flow hash is **specific for NLB**.

## ALB - Listener rules

* Process in order: Default rule is going to be the last one to be processed

* Supported actions:

  * Forward: to another target group
  * Redirect: to other URL
  * Fixed-response

* Rule conditions:

  * Host-header
  * Http-request-method: get, put, post ...
  * path-pattern: /example1, /example2
  * Source-IP: where is the request coming from
  * HTTP-header
  * Query-string

  

  ## 72/73 Auto Scaling Groups (ASG)

  If an instance is marked unhealthy by ELB, ASG **does not restart or stop** the instance, but **terminate** the instance

  ### Auto scaling alarms

  An alarm monitors a metric. 

  **Note:**

  **Metric are computed for the overall ASG instances** - Metric value does not from the minimum or the maximum but from the average.

  ### Bound ASG with ELB

  **Automatically Register new instances to a load balancer:** If you bound an ASG with ELB, the instances from ASG are going to be automatically registered to the target group of the ELB.  

## 74/75 Auto scaling policies

### Policy types

* **Dynamic scaling policies**: Scaling based on cloud watch alarms, which in turn based on the metrics
  * **Target tracking scaling**: We just need to give the metric, AWS create a CloudWatch alarm for us. This one is the simplest. 
  * **Simple scaling**: We need to create a alarm before hand based on a metric. 
  * **Step scaling**: We need to create a alarm before hand based on a metric. Step scaling is similar with simple scaling. The difference is that step scaling has more than 1 steps. The step is set using the alarm value. 
* **Predictive scaling policies**: Analyze the previous usage for a selected metric for a period of time, using machine learning algorithm to analyze the usage pattern. Then scaling ahead of the time according to the pattern. 
* **Scheduled action**: Base on time

### Good metrics to scale on

* The **RequestCountPerTarget** metric value indicates the average number of requests received by each target in a target group associated with an Application Load Balancer during a specified time period.
* **Average network in/out:** If you application needs a lot of upload and download, the network will be the bottleneck for your EC2 instance, you may scale based on average network in/out to make sure to set the threshold for ASG to scale in and out. 

## ASG for SysOps

### ASG Health Checks

* **EC2 status checks**: enabled by default, you cannot disable it.

* **ELB health checks**: If the ASG is bound to ELB, target group will do health check to EC2 instance. You need to enable this one. 

* **Custom Health checks**: using cli or AWS SDK

  **CLI used for custom health check:**

  ```
  set-instance-health
  terminate-instance-in-auto-scaling-group
  ```

### Troubleshooting ASG issues

**<number of instances> instance(s) are already running. Launching EC2  instance failed.**

* The Auto Scaling group has reached the limit set by the MaximumCapacity
  parameter. Update your Auto Scaling group by providing a new value for the
  maximum capacity.

* The account reach the maximum capacity for the AZ

  

