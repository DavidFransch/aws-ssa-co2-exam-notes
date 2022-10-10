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

## AWS Organisations
- Management account used to create organisation and contains payment method
- Root account (structure)
- Consolidated billing - removes financial overhead
- Consolidation of reservations and volume accounts
- Creating a new account is a single step
- Inviting an existing account needs confirmation

## Service Control Policies (SCP)
- **Policy document** (JSON) which can be attached to:
  -  root container or
  -  one or more organisation units or
  -  one or more accounts
- Management account is unaffected by SCP
- SCPs are **account permissions boudaries**
- They limit what the account (**including the account root user**) can do
- **DON'T grant permissions**
- Default = FullAWSAccess (hence SCPs are **deny list architecture**)
- Implementing allow lists -> remove FullAWSAccess -> AllowS3EC2 (more admin overhead)
- If allowed by SCP still **requires Identity Policy**

## CloudWatch Logs
- Store, monitor and access logging data
- Public service and can also be utilised in an on-premises environment and even from other public cloud platforms.
- AWS integrations - EC2, VPC Flow Logs, Lamda, CloudTrail, R53 and more.
- Can generate metrics based on logs - metric filter

## CloudTrail
- Logs **API** calls/activities as a **CloudTrail Event**
- 90 days stored by default in **Event History**
- Enabled **by default** - no cost for **90 day** history
- To customise the service .. create 1 or more Trails
  - **TRAILS** are how you configure S3 and CWLogs
- **Management** Events and Data Events (not enabled by default and extra cost)
