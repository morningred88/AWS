# AWS Account Management

## Organization 

AWS Organization is a **global service**, because Organization group the multiple accounts together.

You can **invite an existing account** to the Organization or **create new account** directly from there. 

### Management account

Management account is the account you use it to create an Organization. 

You just need to got AWS Organization console > click on **Create Organization**

It is best practice to leave the management account under the root OU. But if you want, you can also move it to any OUs under Root OU.

### Organizational units

**Create Organizational unit:**

Login to your management account > **AWS Organization** console> check the **Root** OU> click on **Action** > Create new> Give **Organizational unit name** > **Create organization unit**.

Use the process above you can add Dev, Test and Prod OUs

You can create OUs with OU, such as you can create HR and Finance department under Prod.

The reason we do it because we want to have service control policies (SCP)

### SCP

Similar with IAM policies in JSON.

**Create SCP policy**:

**AWS Organization** console> Policies > service control policies (SCP) > **Enable**, then you will have **AWSFullAccess** policy for your organization root OU. 

You can create any SCP policy and attach it to the OU

### Tag Policies

You can use AWS Organization Tag Policies to standardize tags across resources in all AWS Accounts inside your AWS Organization. 



## Control Tower

It is best practice to set up multiple accounts using Control Power in your accounts. 

It already adds compliances (Guardrails) and access control to the Control Tower from AWS. So you don't need to setup all from scratch like you do for AWS Organization. 

When we create a landing zone in Control Tower, it automatically show up in AWS Organization console. We should not manage it in AWS Organization.



## AWS Service Catalog

We allow users choose from the cloudFormation templates organized in portfolios, and launch them safely. Users does not any other AWS access at all except Service Catalog.

### Administration

* Add products - using CloudFormation with all the defined configurations
* Create portfolios - each portfolio is a collective products with access for specified users, groups or roles. For example, portfolio for Web developers, the administrator can add EC2 instances including security group, or any other AWS resources for web development.

### TagOptions library

A TagOption is a key-value pair managed in AWS Service Catalog. It is not an AWS tag, but serves as a template for creating an AWS tag based on the TagOption.

#### Create TagOptions

First go to TagOptions library, enable it. Then you can start to add key value pairs as TagOptions

You can see that I created 3 TagOptions:

![Create_new_TagOption](/Account_Management_SysOps/Account_Management_images/Create_new_TagOption.png)

#### How to use TagOptions

Administrators can associate TagOptions with portfolios and products. During a product launch (provisioning), AWS Service Catalog aggregates the associated **portfolio** and **product** TagOptions, and applies them to the provisioned product, as shown in the following diagram.

![TagOption-library](/Account_Management_SysOps/Account_Management_images/TagOption-library.png)

**Reference:**

https://docs.aws.amazon.com/servicecatalog/latest/adminguide/tagoptions.html



## AWS Billing Alarms

You can set a billing threshold for a **specified time period**, such as 6 hours or 1 week. 

In order to set AWS billing alarms, you need first to enable billing alert.

### Enable billing alert 

Account name > **Billing dashboard** > **Billing preferences** > Check **Receive Billing Alert**

You need to wait about 15 minute for your data collection to be active, the billing metric will populate to CloudWatch metrics.

### Create billing alarm

You need to go to us-east-1 region, billing metrics is only available at the region

CloudWatch console > Alarms > Billing > Create billing alarm > Select Billing name space > you can choose the metric at **two levels**:  either for different AWS services, or total charge for all services in your account. The metric name is EstimatedCharges.

## AWS Budget

You set a **monthly** budget amount, either money, usage, saving plan or reservation base as threshold to create a alert. 

### Four types of budgets

* Cost budget: Money based, either for a specific service or total amount

* Usage budget: Track the usage for a specific metric, such as EC2:EBS -Snapshot, how many GB per month

* Saving plan budget: How much you utilize your saving plan

* Reservation budget: For reserved instances, for EC2, ElastiCache, RDS, Redshift, Elasticsearch
