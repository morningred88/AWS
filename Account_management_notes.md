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

## Control Tower

It is best practice to set up multiple accounts using Control Power in your accounts. 

It already adds compliances (Guardrails) and access control to the Control Tower from AWS. So you don't need to setup all from scratch like you do for AWS Organization. 

When we create a landing zone in Control Tower, it automatically show up in AWS Organization console. We should not manage it in AWS Organization.

## AWS Service Catalog

We allow users choose from the cloudFormation templates organized in portfolios, and launch them safely. Users does not any other AWS access at all except Service Catalog.

### Administration

* Add products - using CloudFormation with all the defined configurations
* Create portfolios - each portfolio is a collective products with access for specified users, groups or roles. For example, portfolio for Web developers, the administrator can add EC2 instances including security group, or any other AWS resources for web development.

