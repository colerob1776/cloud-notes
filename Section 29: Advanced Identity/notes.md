# AWS STS - Security Token Service

- Allows to grant limited and temp access to AWS resources (up to 1 hour)
- **AssumeRole**: Assume roles within your account or cross account
- **AssumeRoleWithSAML**: return creds for users logged with SAML
- **AssumeRoleWithWebIdentity**
    - return creds for users logged with IdP
    - AWS recommends against using this, and using **Cognito Identity Pools** instead
- **GetSessionToken**: for MFA, from a user or AWS account root user
- **GetFederationToken**: obtain temp creds for a federated user
- **GetCallerIdentity**: return details about the IAM user or role used in the API call
- **DecodeAuthorizationMessage**: decode err message when an AWS API is denied

### Using STS to Assume a Role

- Define an IAM role within your account or cross-account
- Define which principals can access this IAM Role
- Use aws STS to retrieve creds and impersonate the IAM Role you have access to (**AssumeRole API**)
- Temp creds can be valid between 15min to 1hour

### STS with MFA

- Use **GetSessionToken** from STS
- Appropriate IAM policy using IAM Conditions
- **aws:MultiFactorAuthPresent:true**
- Reminder, GetSessionToken returns:
    - Access ID
    - Secret Key
    - Session Token
    - Expiration Date

# Advanced IAM

### Authorization Model Evaluation of Policies, simplified

1. If there's an explicit DENY, end decision with DENY
1. If there's an explicit ALLOW, end decision with ALLOW
3. Else DENY

### IAM Policies & S3 Bucket Policies

- IAM Policies are attached to users, roles, groups
- S3 Bucket Policies are attached to buckets
- When evaluating if an IAM Principal can perform an operation X on a bucket, the **union** of its assigned IAM Policies and S3 Bucket Policies will be evaluated

### Dynamic Policies with IAM

- How do you assign each user a /home/\<user\> folder in an S3 bucket?
- Option 1:
    - Create an IAM policy allowing george to have access to /home/george
    - Create an IAM policy allowing sarah to have access to /home/sarah
    - ...One Policy per user (doesnt scale!)
- Option 2:
    - Create one dynamic policy with IAM
    - Leverage the special policy variable ${aws:username}

### Inline vs Managed Policies

- AWS Managed Policy
    - maintained by AWS
    - Good for power users and admins
    - Updated in case of new services / new APIs
- Customer Managed Policies
    - Best Practice, re-usable, can be applied to many principals
    - Version controlled + rollback, central change management
- Inline:
    - Strict one-to-one relationship b/w policy and principal
    - Policy is deleted if you delete the IAM principal

# Granting User Permissions to Pass a Role to an AWS Service

- To configure many AWS services, you must **pass** an IAM role to the service (this happens only once during setup)
- The service will later assume the role and perform actions
- Example of passing a role:
    - To an EC2 instance
    - To a Lambda function
    - To an ECS task
    - To CodePipeline to allow it to invoke other services

- For this you need the IAM permission **iam:PassRole**
- It often comes with iam:GetRole to view the role being passed

### Can a role be passed to any service?

- **No: Roles can only be passed to what their *trust* allows**
- A *trust policy* for the role that allows the service to assume the role

# AWS Directory Services

- Microsoft Active Directory (AD):
    - Found on any Windows Server with AD Domain Services
    - Database of **objects**: User Accounts, Computers, Printers, File Shares, Security Groups
    - Centralized security management, create account, assign permissions
    - Objects are organized in **trees**
    - A group of trees is a **forest**

- AWS Directory Services:
    - **AWS Managed Microsoft AD**:
        - Create your own AD in AWS, manage users locally, supports MFA
        - Establish "trust" connections with your on-premise AD

    - **AD Connector (Proxy)**:
        - Directory Gateway (proxy) to redirect to on-premise AD, supports MFA
        - Users are managed on the on-premise AD
    
    - **Simple AD**:
        - AD-compatible managed directory on AWS
        - Cannot be joined with on-premise AD