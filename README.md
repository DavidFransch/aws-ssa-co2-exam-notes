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
- **Regional** service
  - One region trail
  - All region trail
- IAM, STS, CLoudFront => Global Service Events
- **NOT REALTIME** - There is a delay (about 15min)

## Simple Storage Service (S3)

### S3 Security (Resource Policies & ACLs)
- S3 is private by default (anyone **other than root user** needs permission)
- S3 bucket policies
  - A form of **resource policy**
  - Like identity policies but attached to a bucket
  - Resource perspective permissions
  - ALLOW/DENY same or **different** accounts
  - ALLOW/DENY **Anonymous** principals
  - Usecase for bucket policies:
    - Who can access objects
- Access Control Lists (ACLs)
  - ACLs on objects and bucket
  - A subresource
  - Legacy (recommended to not use)
  - Inflexible & Simple permissions (can't write objects)
- Block public access (apply to anonymous principal)

Example bucket policy:
(**Principal** specfied -> way to identify that it's a bucket policy)
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicRead",
            "Effect": "Allow",
            "Principal": "*",
            "Action": ["s3:GetObject"],
            "Resource": ["arn:aws:s3:::secretcatproject/*"]
        }
    ]
}
```

Example: Blocks access if your IP is not 1.3.3.7/32
```json
{
    "Version": "2012-10-17",
    "Id": "BlockUnLeet",
    "Statement": [
        {
            "Sid": "IPAllow",
            "Effect": "Deny",
            "Principal": "*",
            "Action": ["s3:*"],
            "Resource": ["arn:aws:s3:::secretcatproject/*"],
            "Condition": {
                "NotIpAddress": {"aws:SourceIp": "1.3.3.7/32"}
            }
        }
    ]
}
```

Exam Powerup:
- If you are managing **many different resources** across an AWS account, then **resources policies don't make sense** (not every service supports resource policies and one would need a policy for every service) RATHER USE **identity policy**
- When managing on a **specific product** i.e. AWS S3 USE **bucket/resource policy**
- **Never** use ACLs, unless you must

### S3 Static Hosting
- Normal access is via AWS API's
- Feature allows access via HTTP - e.g Blogs..
- Index and Error documents are set
- Website Endpoint is created
- Custom Domain via R53 - Bucketname Matters

### Object Versioning and MFA
Versioning let's you store **multiple versions** of objects within a bucket. Operations which would modify objects **generate a new version.**

```
Disabled -> Enabled -> Suspended ..

Enabled <- Suspended
```
- **Enabled cannot go to disabled**
- **Cannot be switched** off - only suspended
- Space is consumed by ALL versions
- Billed for ALL versions
- Only way to 0 costs - is to delete the bucket
- MFA Delete
  - Enabled in **versioning configuration**
  - MFA is required to **change** bucket **versioning state**
  - MFA required to **delete versions**
  - Serial number (MFA) + Code passed with API calls

### S3 Performance Optimization
- **Single PUT Upload** 
  - Single data stream to S3
  - Stream fails - upload fails
  - Requires full restart
  - Speed & reliability = limit of 1 stream
  - Any upload up to **5GB**

- **Multipart Upload**
  - Data is broken up
  - **Min** data size **100MB** for multipart
  - **10,000 max parts**, 5MB -> 5GB
  - Last part can be smaller than 5MB
  - Parts can fail, and be restarted

- **S3 Transfer Acceleration**
  - Uses AWS **edge locations**
  - Need to enable the S3 bucket (i.e. **default off**)
  - Restrictions
    - No periods in name
    - DNS compatible
  - Faster and lower consistent latency
  - Can test with s3-accelerate-speedtest tool
