# CloudFormation for SysOps

## 86 CloudFormation create stack hands on

### CloudFormation concept - template & stack

When you use AWS CloudFormation, you work with *templates* and *stacks*. 

* You create **templates** to describe your AWS resources and their properties. A **template** is a JSON or YAML file that contains configuration information about the AWS resources you want to include in the stack. Template is stored in S3 bucket. Templates have to be uploaded in S3 and then referenced in
  CloudFormation.

* Every **stack** is based on a template. Whenever you create a **stack**, CloudFormation provisions the resources that are described in your template. You can see the template tab under stack. 

  

### Process to create a cloudFormation stack

If you upload the cloud formation template into S3, it will create a bucket and upload the template right away, even you cancel to create a stack.

You see after I click upload a template file and uploaded the file, the template source change to Amazon S3 url and show the url of S3 bucket

![create stack](/CloudFormation_images/create_stack.jpg)



## 87 CloudFormation Update and Delete stack

**Update a stack**

1 way is **Edit template in designer**

![update stack](/CloudFormation_images/update_stack1.png)

2 way: select **replace current template**. After I selected the file from my computer, the file was uploaded to the same bucket as before, but a new template file

![update stack](/CloudFormation_images/update_stack2.png)



**CloudFormation templates from one stack in S3**

You can see 2 designer and two yaml files in the same bucket. As long as the stack name is the same, all the template files related to it were uploaded in the same bucket

![update stack](/CloudFormation_images/s3_bucket_for_cf_template.png)

**Note for stack update: **

-	Parameter value is added at the time when you create/update a template
-	**Change set preview**: before you submit the update for the stack, you have a chance to review the change set, which is CloudFormation figure out what are the differences between the original template and new template, and what needs to change.
-	When I updated the stack, second template was uploaded, all resources in the first template will be deleted. The resources for the second template will be newly created. This does not apply to all the update. **If the Replacement is True in Change set preview, then the resource will be replaced**. 

**Delete a stack**

When you want to delete AWS resources created from CloudFormation, you go to the stack and delete it. Then all the resources provisioned by the stack is removed. 

## 89 CloudFormation Resources

When you create a resources, must have **Type** and **Properties**

AWS Resources documentation:

**Resource type** identifiers always take the following form:

```
service-provider::service-name::data-type-name
```

The following link contains reference information for all AWS resource and property types that are supported by AWS CloudFormation:

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html

## 90 CloudFormation Parameters

**NoEcho** (boolean value): If you pass a secret as a parameter, set NoEcho to True.

AWS  parameters is called **Pseudo parameters**. You can reference them directly.

## 91 CloudFormation Mapping

**Parameter vs mapping**

* Parameter is user specific, you don't know the value in advance

* Mapping: You know all the values in advance, all the values in map need to be explicitly written out in CloudFormation template.

## 94 CloudFormation Intrinsic Functions

In the exam, only the yaml will be tested, not Json

### Fn::GetAtt

I use EC2 instance as a example. 

You can find which attributes you can get from EC2 instance,

\-      Go to cloudFormation Template Reference, https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-reference.html

\-     Go to the first one, Resource and Property reference, then select EC2, then select AWS::EC2::Instance, click returned values from the right hand side. You can see what is the **return value** if you use Ref and Fn:GetAtt function from EC2 instance

#### Return values for EC2 instance

##### Ref

When you pass the logical ID of this resource to the intrinsic `Ref` function, `Ref` returns the instance ID. For example: `i-1234567890abcdef0`.

For more information about using the `Ref` function, see [Ref](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html).

##### Fn::GetAtt

The `Fn::GetAtt` intrinsic function returns a value for a specified attribute of this type. The following are the available attributes and sample return values.

For more information about using the `Fn::GetAtt` intrinsic function, see [Fn::GetAtt](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getatt.html).

* **AvailabilityZone**

The Availability Zone where the specified instance is launched. For example: `us-east-1b`.

You can retrieve a list of all Availability Zones for a Region by using the [Fn::GetAZs](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getavailabilityzones.html) intrinsic function.

* **PrivateDnsName**

The private DNS name of the specified instance. For example: `ip-10-24-34-0.ec2.internal`.

* **PrivateIp**

The private IP address of the specified instance. For example: `10.24.34.0`.

* **PublicDnsName**

The public DNS name of the specified instance. For example: `ec2-107-20-50-45.compute-1.amazonaws.com`.

* **PublicIp**

The public IP address of the specified instance. For example: `192.0.2.0`

### Fn::Sub 

#### Fn::Sub yaml example with a mapping

```
Name: !Sub
  - www.${Domain}
  - { Domain: !Ref RootDomainName }
```

#### Fn::Sub yaml example without a mapping

The following example uses Fn::Sub with the `AWS::Region` and `AWS::AccountId` pseudo parameters and the `vpc` resource logical ID to create an Amazon Resource Name (ARN) for a VPC.

```
!Sub 'arn:aws:ec2:${AWS::Region}:${AWS::AccountId}:vpc/${vpc}'
```

**Reference:**

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-sub.html

## CloudFormation user data

We pass the entire user data script to function **Fn::Base64**, that will convert the script to a Base64 string.

### Create a stack in CloudFormation console

Using CloudFormation template 3-user-data.yaml

* Upload the template file 
* SSH key pair name as parameter
* Create the stack

Go to **Resources** tab under the stack to confirm that both EC2 instance and security group have been created. 

![update stack](/CloudFormation_images/EC2_with_user_data.png)

### Check user data logs in EC2 instance

* **SSH to EC2 instance using the keypair above from parameter value**

```
[ec2-user@ip-172-31-14-10 ~]$ ssh -i Devkeypair.pem ec2-user@44.195.81.63

Result:
The authenticity of host '44.195.81.63 (44.195.81.63)' can't be established.
ED25519 key fingerprint is SHA256:MDVkUEIOrZ7bil0likwVr56TrQgkQ9XvQSKzbnO9VEY.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '44.195.81.63' (ED25519) to the list of known hosts.
   __|  __|_  )
   _|  (     /   Amazon Linux 2 AMI
  ___|\___|___|
```

* **Check user data script log is in /var/log/cloud-init-output.log**

```
[ec2-user@ip-172-31-14-10 ~]$ sudo cat /var/log/cloud-init-output.log

Result: 
...
+ yum update -y
...
Complete!
+ yum install -y httpd
...
Complete!
+ systemctl start httpd
+ systemctl enable httpd
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
+ echo 'Hello World from user data'
```

## 101 CloudFormation nested stacks

### Basic concept for nested stacks

**Nested stack**: A stack is a part of other stacks

**Parent stack/Root stack**: A stack contains other stacks.

Take the CloudFormation template file 7-nestedstacks.yaml as example, to see how to add nested stack in parent stack. What is the special for create, update and delete parent stacks. 

### Define nested stack in resources

If you see a rescoure with **Type: AWS::CloudFormation::Stack**, it is a nested stack.

```
Resources:
  myStack:
    Type: AWS::CloudFormation::Stack
```

### Create a parent stack with nested stack inside

You will see 2 stacks are created in CloudFormation, one the parent stack and the other one is the nested stack.

Open the nested stack, you can see this stack is marked as **nested** in the description.

### Delete/update a parent stack with nested stack inside

For deletion, you just need to delete the parent stack, the nested stack will get deleted automatically.

For update, never touch the nested stack, just do update in the parent stack. 

## CloudFormation Drift

**How to detect a drift for a stack?**

Select a stack> **Stack action** dropdown> Select **Detect drift**

Then go back to **Stack action** dropdown > select **View drift results**

In **Drifts** page, you can also access Drift menu under Stacks:

If a stack is drifted, **Stack drift status** appears as **DRIFTED** 

You can see what rescource is drifted in **Resource drift status**. Select one resource, click **View drift details**.
