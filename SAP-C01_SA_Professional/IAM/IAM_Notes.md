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