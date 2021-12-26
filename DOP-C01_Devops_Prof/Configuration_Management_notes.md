# Configuration Management and Infrastructure as Code

## CloudFormation

### Resources

When you create a resource,   type and properties are required. 

### Outputs

CloudFormation uses Outputs to link two stacks, one stack can reference the value from another stack

First stack: Output component, define **Export name** and value

Second stack: To reference the output from the first stack, using function **fn:ImportValue**, the Export name as the value of the function 