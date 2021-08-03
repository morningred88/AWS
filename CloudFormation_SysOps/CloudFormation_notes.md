# CloudFormation for SysOps

## 86 CloudFormation create stack hands on

### CloudFormation concept - template & stack

When you use AWS CloudFormation, you work with *templates* and *stacks*. 

* You create **templates** to describe your AWS resources and their properties. A **template** is a JSON or YAML file that contains configuration information about the AWS resources you want to include in the stack. Template is stored in S3 bucket. Templates have to be uploaded in S3 and then referenced in
  CloudFormation.

* Every **stack** is based on a template. Whenever you create a **stack**, CloudFormation provisions the resources that are described in your template. You can see the template tab under stack. 

  

