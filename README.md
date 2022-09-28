# AWS Solutions Architect

[IAM, Accounts and AWS Organisations](/section-A)

## IAM, Accounts and AWS Organisations

### IAM Policies

Deny and Allow rules:
1. Explicit **DENY** (takes priority)
2. Explicit **ALLOW**
3. Default **DENY** (implicit)
   
Example:
```json
{
    "Version": "2012-10-17"
    "Statement": [
        {
            "Sid": "FullAccess",
            "Effect": "Allow",
            "Action": ["s3:*"],
            "Resource": ["*"],
        },
        {
            "Sid": "DenyCatBucket",
            "Action": ["s3:*"],
            "Effect": "Deny",
            "Resource": ["arn:aws:s3:::catgifs", "arn:aws:s3:::catgifs/*"],
        }
    ]
}
```

#### Managed Policies:
- Reusable
- Low management overhead
#### IAM Users and ARNs

- IAM Users are an identity used for anything requiring long-term AWS access e.g. Humans, Applications or service accounts. 

- Principal makes the request
- Authentication = Username&Password or Access Keys
- Authorisation = IAM checks policy to see if service is allowed/denied
- Amazon Resource Name (ARN) - uniquely identify resources within AWS accounts
- Examples:
  - arn:partition:service:region:account-id:resource-id
  - arn:partition:service:region:account-id:resource-type/resource-id
  - arn:aws:s3:::catgifs  (bucket)
  - arn:aws:s3:::catgifs/* (objects in bucket)

#### Exam Power Up
  - **5000** IAM Users per account
  - IAM User can be a member of 10 groups
  - System design impacts.. 
    - Internet-scale applications
    - Large orgs & orgs merges
    - IAM Roles & Identity Federation fix this (more later)

#### IAM groups
- **Can't login and no credentials** (potential trick question)
- IAM user can be a member of multiple groups
- Benefits
  - admin style management
  - groups can have policies attached (inline and managed)
- Not all users group in IAM
- No nesting
- Limit of 300 groups (can be adjusted by logging ticket)
- Groups are NOT a true identity. They can't be referenced as a principal in policy.
- Simply a container for IAM users.

#### IAM Roles

- Used by unknown number of users or multiple principals
- If you can't identify the number 
- Temporary identity (any changes to permission policy results in new credentials as well)
- Policy types
  - Trust policy
  - Permissions policy 
- Temporary credentials -> `sts:AssumeRole`

#### When to use IAM Roles
- AWS Lambda requires access keys, instead of hardcoding these -> we can use a Lambda Execution Role
- When using an AWS service a role is generally a good idea
- Assigning emergency role (e.g. break glass scenario)
- **ID Federation**
  - Provide SSO for employees to use an AWS service (Active Directory -> Role -> s3)
  - App  1,000,000's of users (Web identity (i.e. Facebook) -> Role -> DynamoDB)
- AWS account with 1000s of identities -> role -> s3
