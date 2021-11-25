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

