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



## CloudFormation Update and Delete stack

**Update a stack**

1 way is **Edit template in designer**

![update stack](/CloudFormation_images/update_stack1.png)

2 way: select **replace current template**. After I selected the file from my computer, the file was uploaded to the same bucket as before, but a new template file

![update stack](/CloudFormation_images/update_stack2.png)

