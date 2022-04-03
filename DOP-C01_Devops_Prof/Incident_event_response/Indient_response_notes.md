# ASG

## ASG - From launch configuration

### Creating a launch configuration

* Instances: All spot instances or all on-demand instances
* When you want to update, it will create new configuration
* Tags: If the new instance is check, then the tags created in Launch configuration will be pass be the new instances. 

## ASG - From launch template

Advanced way to launch configuration. From template, we can launch: 

* Launch an instance
* Launch an ASG
* Launch spot fleet

### Creating a launch template

* When you want to update, it will create new template or new template **version**
* You can define source template, if you want to create template from the existing template as parent. It can create hierarchy. 
* We have both instance tags and template tags

