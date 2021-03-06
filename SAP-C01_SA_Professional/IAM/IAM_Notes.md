## IAM

## Instance profile (EC2 instance role)

* Use an instance profile to pass an IAM role to an EC2 instance.  

* **You can attach only one role to ec2 instance**.

* Instance profiles are usually recommended over configuring a static access key as they are considered more secure and easier to maintain.

### Instance Profiles Working principle

EC2 shares the credentials with the application running on EC2 through the [metadata service](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html). 

The instance profile credentials are exposed on `http://169.254.169.254/latest/meta-data/iam/security-credentials/`. When you `curl` this URL on an EC2 instance, you will get the name of the instance profile attached to the instance. When you `curl` the same URL with the instance profile name at the end, you get the temporary credentials as JSON. The metadata service will return access key id, secret access key, a token, and the expiration date of the temporary credentials.

If you are going to use these credentials manually, remember that the token is required. Normal user access keys don’t have a token, but temporary credentials require it.

### Managing instance profiles (console)

If you use the AWS Management Console to create a role for Amazon EC2, the console automatically creates an instance profile and gives it the same name as the role. When you then use the Amazon EC2 console to launch an instance with an IAM role, you can select a role to associate with the instance. In the console, the list that's displayed is actually a list of instance profile names. The console does not create an instance profile for a role that is not associated with Amazon EC2.

Reference:

https://kichik.com/2020/09/08/how-does-ec2-instance-profile-work/#:~:text=The%20instance%20profile%20credentials%20are,profile%20attached%20to%20the%20instance.

https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html

## Access Advisor vs Access Analyzer

**Access Advisor**: in under an user or a role, shows permission granted and when last accessed.

![Access Advisor](IAM_images/Access_Advisory.png)



**Access Analyzer**: For a AWS account or an organization, finds the resources can be accessed by external entity.

![Access_Analyzer](IAM_images/Access_Analyzer.png)



## IAM policy

### Power user policy: less privileged than administrator

![Power_User_Policy](IAM_images/Power_User_Policy.png)

**Note: Use "NotAction" instead of "Deny"**

- If we use deny for the first policy, then the second allow policy will never in effect, because explicit deny takes the precedence. 
- If we want to allow some of the actions but deny the rest, we can use "NotAction"

## IAM Permission Boundary

Permission boundary is  an IAM policy, which defines the maximum permission for a user or role. 

### Use a scenario to explain the permission boundary

Background: A lambda developer need the permission to create role to do some EC2 and S3 work, like stop and start EC2 instance, retrieve files from S3. 

#### Attach policy without permission boundary to an user

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

#### Attach policy with permission boundary to an user

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

![Add_permission_boundary_step1](IAM_images/Add_permission_boundary_step1.png)

* Add created BoundaryPolicy as a condition, note that 
  * Condition key: iam-PermissionBoundary
  * Value is the ARN of BoundaryPolicy

![Add_permission_boundary_step2](IAM_images/Add_permission_boundary_step2.png)

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

#### Create roles by the user with permission boundary

* Whenever user create a role, a permission boundary requires to be added, by click set permissions boundary below:

![Create_role_with_permission_boundary1](IAM_images/Create_role_with_permission_boundary1.png)

* Then choose the created BoundaryPolicy above

![Create_role_with_permission_boundary2](IAM_images/Create_role_with_permission_boundary2.png)

**Reference:**

IAM Permissions Boundary - Full Configuration

https://www.youtube.com/watch?v=gLQwzsqpSFA

## STS

### Cross account access with STS

#### Step 1: Create a role in the Production Account

You create the role in the **Production** account and specify the **Development** account as a trusted entity. You also limit the role permissions to only read and write access to the `productionapp` bucket. Anyone granted permission to use the role can read and write to the `productionapp` bucket.

Notes: 

* Create **read-write-app-bucket** policy 

  ```
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": "s3:ListAllMyBuckets",
        "Resource": "*"
      },
      {
        "Effect": "Allow",
        "Action": [
          "s3:ListBucket",
          "s3:GetBucketLocation"
         ],
        "Resource": "arn:aws:s3:::productionapp"
      },
      {
        "Effect": "Allow",
        "Action": [
          "s3:GetObject",
          "s3:PutObject",
          "s3:DeleteObject"
        ],
        "Resource": "arn:aws:s3:::productionapp/*"
      }
    ]
  }
  ```

* Create  **UpdateApp**  role

  * AWS AMI console > Roles > Create a role>Choose the **An AWS account** role type
  * For **Account ID**, type the **Development** account ID.

* d

#### Step 2: Grant access to the role in **Development** account

Add inline policy for  User groups **Developers** to allow them to switch to the **UpdateApp** role in **Production** account.

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": "sts:AssumeRole",
    "Resource": "arn:aws:iam::PRODUCTION-ACCOUNT-ID:role/UpdateApp"
  }
```



## Identity federation

### SAML 2.0 Federation – AWS API Access

* Identity provider: SMAL 2.0 IdP
* Identity store: LDAP-based Identity Store
* **AssumeRoleWithSAML** API
* User get temporary security credentials for AWS API access

### SAML 2.0 Federation – AWS Console Access

Difference between SAML 2.0 Federation – AWS API Access and AWS Console Access:

* **AWS Sign-in Endpoint for SAML** (https://signin.aws.amazon.com/saml) instead of AssumeRoleWithSAML API call
* User gets sign-in url for AWS console

### SAML 2.0 Federation –Active Directory FS (ADFS)

Difference between SAML 2.0 Federation – AWS Console Access and ADFS:

* Identity provider: ADFS
* Identity store: Active Directory

### Custom Identity Broker Application

For IdP without SAML 2.0

**Difference between SAML 2.0 Federation and Custom Identity Broker:**

Custom Identity Broker has Admin power, can directly get temporary security credential from STS by 

* The Identity Broker must determine the appropriate IAM Role
* Uses the STS API AssumeRole or **GetFederationToken**, back to Custom Identity Broker itself
* Then Custom Identity Broker passes user the token or URL

### Web Identity Federation

Allow user identified by third party Idp, such as Google, Amazon or Facebook

#### Without Cognito

The old way to treat the third party identity token with temporary security credentials from STS by **AssumeRoleWithWebIdentity** API

#### With Cognito

Not directly treat he third party identity token with temporary security credentials from STS, but treat with **Cognito token** instead. This way, Web Identity Federation can consume the advantage of Amazon Cognito, such as MFA and anonymous mode. 

## AWS Directory Service

### AWS Managed Microsoft AD

* Deploy Microsoft AD in your AWS VPC
* Standalone or setup trust with on-premises
* If setup trust, two places where user are defined - on premises and in the cloud.

### AD Connector

Users are solely managed by on-premises Microsoft AD, no possibility of setting up a trust

### Simple AD

AWS AD product, not from Microsoft AD, users only can be managed in AWS.

## AWS Organizations

Management account: Account used to create an organization. It is best practice to leave the management account under root. 

### OrganizationAccountAccessRole

We have **2 options to add a member account**  in an AWS Organization:

* **Create an AWS account**

  You can see OrganizationAccountAccessRole is **automatically** created in the member account creating process, see below. Management account assume this role in the member account to manage all the member accounts.  

* **Invite an existing account**

  OrganizationAccountAccessRole needs to be **manually** added in order to allow Organization to management the member account. 

![Create_Member_Account_In_Organization](IAM_images/Create_Member_Account_In_Organization.png)

## Session policies

**Session policies** are advanced policies that you pass as a **parameter** when you programmatically (AWS CLI or AWS API) create a temporary session for a role or federated user. 

* You can create role session and pass session policies programmatically using the `AssumeRole`, `AssumeRoleWithSAML`,  `AssumeRoleWithWebIdentity` or `GetFederationToken` API operations.

* Session policies **limit permissions** for a created session, **but do not grant permissions**. The maximum permissions that a session can have are the permissions that are allowed by the role’s identity-based policies. 
* You can pass an inline session policy and ARNs of up to 10 managed policies in the same role session. 

### Inline session policy 

 You can pass a single inline session policy programmatically by using the **policy** parameter with the [AssumeRole](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html), [AssumeRoleWithSAML](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html), [AssumeRoleWithWebIdentity](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithWebIdentity.html), and [GetFederationToken](https://docs.aws.amazon.com/STS/latest/APIReference/API_GetFederationToken.html) API operations.

**Example - Passing inline session policy using the AWS CLI:**

```
aws sts assume-role 
--role-arn "arn:aws:iam::111122223333:role/SecurityAdminAccess" 
--role-session-name "s3-session" 
--policy file://policy.json 
```

### Using IAM managed policies as session policies

You can now pass up to 10 IAM managed policies as session policies. This gives you the ability to further restrict session permissions. The managed policy you pass can be AWS managed or customer managed. To pass managed policies as session policies, you need to specify the [Amazon Resource Name](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html) (ARN) of the IAM policies using the **policy-arns** parameter in the `AssumeRole`, `AssumeRoleWithSAML`, `AssumeRoleWithWebIdentity`, or `GetFederationToken` API operations.

**Example - Passing the managed policies as session policy using the AWS CLI:**

```
aws sts assume-role 
--role-arn “arn:aws:iam::*444455556666*:role/AppDev” 
--role-session-name “-session” 
--policy-arns arn=”arn:aws:iam::*444455556666*:policy/DevCalifornia”
```

**Reference:**

https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html#policies_session

Create fine-grained session permissions using IAM managed policies, https://aws.amazon.com/blogs/security/create-fine-grained-session-permissions-using-iam-managed-policies/

## AWS Single Sign-On (SSO)

SSO is acting your own identity provider, and SSO will perform the heavy lifting as well of talking to STS service. It is much easier to set up.

**AWS SSO is preferred way to for identity federation than SAML2.0. **

## AWS Control Tower

AWS Control Tower provides the easiest way to set up and govern a **secure, multi-account** AWS environment, called a **landing zone**. It creates your landing zone **using AWS Organizations**. 

**Control Tower is a free service**. But user needs to pay AWS accounts and services created in AWS Control Tower.

### Create landing Zone in Control tower

It takes around one hour to create landing zone in control tower. When we created a Landing Zone, you can see 2 OUs, 3 shared accounts, SSO access and 20 preventive guardrails are created.


  ![Creating Landing Zone](IAM_images/Creating_Landing_Zone.png)

After landing zone is created, you can check in in AWS Organization. And you also continue to manage the AWS Organization in Control tower, instead of managing it in AWS Organization directly.

### Account Factory

The AWS Control Tower enables automate the account provisioning workflow using an account factory. 

![Account_Factory](IAM_images/Account_Factory.png)



### User Access in a Landing Zone

Control tower uses SSO to manage all the users to the accounts. You can see the portal URL for users to sign in.

![Landing_Zone_SSO](IAM_images/Landing_Zone_SSO.png)

The way to handle user identity management is AWS SSO directory. 

![LandingZone_SSO_Directory](IAM_images/LandingZone_SSO_Directory.png)