# Elastic Beanstalk for SysOps

## Application and environment

**Application** is the **center** part of beanstalk. When you delete an application, all environments from this application will be deleted too. 

* When you create a new application in Beanstalk, you also create an environment for the application. 
* An application can have multiple environments, such as dev, qa and prod. Select you application, click create a new environment, we can create a second environment for the application.  You can select the **environment tier - Web server environment or Worker environment**.
* Under Applications Tab, select one of the application, expand the tab on the left hand side, you can see **Application versions** tab. An Application can have multiple versions.

### Application environments

Under Environments tab, you can see all the environment you created. Some of them are from the same application. **Environment is a part of application**. 

#### Environment tiers

* Web server environment
* Worker environment

Worker tier instances do not run web server processes (apache, nginx, etc). As such, they do not directly respond to customer requests. Instead, they can be used to offload long-running processes from your web tier.

The tiers communicate with each other via SQS. 
