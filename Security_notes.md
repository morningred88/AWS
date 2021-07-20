# Security and Compliance for SysOps

## KMS key rotation

### Automatic key rotation

* Key rotation every 1 year
* No change to CMK ID

### Manual key rotation

* Key rotation every 90 days, 180 days
* New key has a different CMK ID
* Better to use alias

## CloudHSM 

* Unlike KMS, you have dedicated hardware to create Customer Master key (CMK).
* HSM: Hardware Security Module
* Need to install software for CloudHSM to work. And the software is used to manage the user access control. IAM is used for CRUD of CMK keys
* Customer can import **asymetric** keys to CloudHSM.

## AWS Certificate Manager (ACM)

* If you want to make your webpage have secured connection, you need to have a SSL certificate for IT. 
* You can get both private and public SSL certificate from AWS ACM for your domain name. Public SSL is free. 
* After you get the certificate, use Route 53 to manage the DNS name. 

### Example of Creating and using the SSL certificate

* Create a SSL certificate from ACM for you domain name

* you use Beanstalk to deploy an application with high availabilty. 
* Go to Route 53,  you can create a CName record for the URL created by Beanstock, set the website DNS as the one you got SSL certificate. 
*  Go to ALB HTTPS listener, you can the SSL certificate used, because SSL certificate from ACM integrated with ELB. It is also possible to change the certificate.

