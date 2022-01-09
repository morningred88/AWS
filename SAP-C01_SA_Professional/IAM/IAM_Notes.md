## IAM

### Access Advisor vs Access Analyzer

**Access Advisor**: in under an user or a role, shows permission granted and when last accessed.

![Access Advisor](/IAM_images/Access_Advisory.png)



**Access Analyzer**: For a AWS account or an organization, finds the resources can be accessed by external entity.

![Access_Analyzer](\IAM_images\Access_Analyzer.png)



### IAM policy

#### Power user policy: less privileged than administrator

![Power_User_Policy](\IAM_images\Power_User_Policy.png)

**Note: Use "NotAction" instead of "Deny"**

- If we use deny for the first policy, then the second allow policy will never in effect, because explicit deny takes the precedence. 
- If we want to allow some of the actions but deny the rest, we can use "NotAction"

### IAM Permission Boundary

Permission boundary is  an IAM policy, which defines the maximum permission for a user or role. 

#### Use a scenario to explain the permission boundary

Background: A lambda developer need the permission to create role to do some EC2 and S3 work, like stop and start EC2 instance, retrieve files from S3. 

##### Attach policy without permission boundary to an user

Admin creates an DeveloperPolicy that gives the developer the permission to do all the work related to Lambda service, also to create roles used by Lambda service. Then attach the DeveloperPolicy to the Lambda developer (IAM user).

The following is the json format of the created DeveloperPolicy.

DeveloperPolicy

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "iam:CreatePolicy",
                "iam:PassRole",
                "iam:Get*",
                "iam:List*",
                "lambda:*",
                "iam:CreateRole",
                "iam:UpdateRole",
                "iam:AttachRolePolicy"
            ],
            "Resource": "*"
        }
    ]
}
```

**Drawback of the DeveloperPolicy above**: Developer can give a Lambda function the Admin permission,  the Lambda can execute the Admin permission not only to EC2 and S3, but to any of AWS service, such as databases, etc.

##### Attach policy with permission boundary to an user

**Step1: Create boundary policy in IAM**

Creating boundary policy is the same procedure as you create any IAM policy. 

AWS Console>IAM> Policy>Create policy

The following is the json format of the created boundary policy. We will add it as conditional for DeveloperPolicy

BoundaryPolicy

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "s3:Get*",
                "s3:List*",
                "ec2:Get*",
                "ec2:Describe*"
            ],
            "Resource": "*"
        }
    ]
}
```

**Step 2: Update DeveloperPolicy with permission boundary**

* Take the 2 actions for IAM service out of DeveloperPolicy: `"iam:CreateRole"` and  `"iam:AttachRolePolicy"`. Then add these 2 action to a newly added IAM service again, but with condition by clicking **Add condition**. 

![Add_permission_boundary_step1](\IAM_images\Add_permission_boundary_step1.png)

* Add created BoundaryPolicy as a condition, note that value is the ARN of BoundaryPolicy.

![Add_permission_boundary_step2](\IAM_images\Add_permission_boundary_step2.png)

The following the json format of updated DeveloperPolicy with permission boundary:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "iam:CreateRole",
                "iam:AttachRolePolicy"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "iam:PermissionsBoundary": "arn:aws:iam::745361488260:policy/BoundaryPolicy"
                }
            }
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "iam:CreatePolicy",
                "iam:PassRole",
                "iam:Get*",
                "iam:List*",
                "lambda:*",
                "iam:UpdateRole"
            ],
            "Resource": "*"
        }
    ]
}
```

The user only allows to create roles when user attach the boundary policy. 

##### Create roles by the user with permission boundary

* Whenever user create a role, a permission boundary requires to be added, by click set permissions boundary below:

![Create_role_with_permission_boundary1](\IAM_images\Create_role_with_permission_boundary1.png)

* Then choose the created BoundaryPolicy above

![Create_role_with_permission_boundary2](\IAM_images\Create_role_with_permission_boundary2.png)

**Reference:**

IAM Permissions Boundary - Full Configuration

https://www.youtube.com/watch?v=gLQwzsqpSFA

### Identity federation

#### SAML 2.0 Federation – AWS API Access

* Identity provider: SMAL 2.0 IdP
* Identity store: LDAP-based Identity Store
* **AssumeRoleWithSAML** API
* User get temporary security credentials for AWS API access

#### SAML 2.0 Federation – AWS Console Access

Difference between SAML 2.0 Federation – AWS API Access and AWS Console Access:

* **AWS Sign-in Endpoint for SAML** (https://signin.aws.amazon.com/saml) instead of AssumeRoleWithSAML API call
* User gets sign-in url for AWS console

#### SAML 2.0 Federation –Active Directory FS (ADFS)

Difference between SAML 2.0 Federation – AWS Console Access and ADFS:

* Identity provider: ADFS
* Identity store: Active Directory

#### Custom Identity Broker Application

Difference between SAML 2.0 Federation and Custom Identity Broker:

Custom Identity Broker has Admin power, can directly get temporary security credential from STS by:

* The Identity Broker must determine the appropriate IAM Role
* Uses the STS API AssumeRole or GetFederationToken, back to Custom Identity Broker itself
* Then Custom Identity Broker passes user the token or URL

### Web Identity Federation

Allow user identified by third party Idp, such as Google, Amazon or Facebook

#### Without Cognito

The old way to treat the third party identity token with temporary security credentials from STS by **AssumeRoleWithWebIdentity** API

#### With Cognito

Not directly treat he third party identity token with temporary security credentials from STS, but treat with **Cognito token** instead. This way, Web Identity Federation can consume the advantage of Amazon Cognito, such as MFA. 