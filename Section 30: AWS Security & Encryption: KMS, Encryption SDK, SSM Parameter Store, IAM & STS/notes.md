# Encryption 101

### Encryption in flight (SSL)

- Data encrypted before sending and decrypted after receiving
- SSL certs help with encryption (HTTPS)
- Encryption in flight ensures no MITM (man in the middle) attacks

### Encryption at rest

- Data is encrypted after being received by the server
- only server knows how to encrypt/decrypt
- The encryption / decryption keys must be managed somewhere and the server must have access to it

### Client Side Encryption

- data will be decrypted by the receiving client
- the server should never be able to decrypt the data
- Could leverage Envelope Encryption

# KMS Overview

- Key Management Service
- AWS manages encryption keys
- Fully integrated with IAM for auth
- Easy way to control access to your data
- able to audit KMS Key usage using CloudTrail
- Seamlessly integrated into most AWS services
- **Never store your secrets in plaintext, especially in code**
    - KMS Key Encryption also available through API calls (SDK, CLI)
    - Encrypted Secrets can be stored in the code / env variables

### KMS Key Types

- **KMS Keys is the new name of KMS Customer *Master* Key**
- **Symmetric (AES-256 keys)**
    - Single key that is used to Encrypt / Decrypt data
    - services that are integrated with KMS use Symmetric CMKs
    - You never get access to the KMS key unencrypted (must call KMS API to use)
- **Asymmetric (RSA & ECC key pairs)**
    - PK pair
    - the public key is downloadable, but you cant access the Private key unencrypted
    - Use case: encryption outside AWS by users who cant call the KMS API

- **Types of KMS Keys**:
    - AWS Owned Keys (free): SSE-S3, SSE-SQS, SSE-DDB (default)
    - AWS Managed Keys: **Free** (aws/service-name, ex: aws/rds or aws/ebs)
    - Customer managed keys created in KMS: **$1/month**
    - Customer managed keys imported (must be symmetric): **$1/month**
    - + pay for API call to KMS ($0.03 / 10k calls)

    - Automatic Key rotation:
        - AWS-managed KMS Key: automatic every 1 year
        - Customer-managed KMS Key: (must be enabled) automatic every 1 year
        - Imported KMS Key: only manual rotation possible using alias

### KMS Key Policies

- control access to KMS keys, "similar" to S3 bucket policies
- Difference: you cannot control access without them

- **Default KMS Key Policy**:
    - Created if you don't provide a specific KMS Key Policy
    - Complete access to the key to the root user = entire AWS account
- **Custom KMS Key Policy**: 
    - Define users, roles that can access the KMS key
    - Define who can administer the key

### Copying Snapshots across accounts

1. Create a Snapshot, encrypted with your own KMS Key
2. **Attach a KMS Key Policy to authorize cross-account access**
3. Share the encrypted snapshot
4. (in target) Create a copy of the Snapshot, encrypt it with a CMK in your account
5. Create a volume from the snapshot

# KMS Encryption Patterns and Envelope Encryption

### Envelope Encryption

- KMS Encrypt call has a limit 4 KB
- If you want to encrypt >4 KB, we need to use **Envelope Encryption**
- The main API that will help us is the **GenerateDataKey** API

- **For the exam: anything over 4 KB of data that needs to be encrypted must use the Envelope Encryption == GenerateDataKey API**

### Encryption SDK

- Encryption SDK implemented Envelope Encryption for us
- The Encryption SDK also exists as a CLI tool we can install
- Implementations for Java, Python, C, JS

- **Feature - Data Key Caching**:
    - re-use data keys instead of creating new ones for each encryption
    - Helps with reducing the number of calls to KMS with a security trade-off
    - Use LocalCryptoMaterialsCache (max age, max bytes, max number of messages)

### API Summary

- **Encrypt**: encrypt up to 4 KB of data through KMS
- **GenerateDataKey**: generates a unique symmetric data key (DEK)
    - returns a plaintext copy of the data key
    - AND a copy that is encrypted under the CMK that you specify
- **GenerateDataKeyWithoutPlaintext**:
    - Generate a DEK to use at some point
    - DEK that is encrypted under the CMK that you specify (must use decrypt later)
- **Decrypt**: decrypt up to 4 KB of data (including DEK)
- **GenerateRandom**: returns a random byte string

# KMS Limits

### Request Quotas

- When you exceed a request quota, you get a **ThrottlingException** (exponential backoff)
- for cryptographic ops, they share a quota
- This includes requests made by AWS on your behalf (ex: SSE-KMS)
- For GenerateDataKey, consider using DEK caching to reduce API calls
- **you can request a Request Quota increase through API or AWS support ticket**


# S3 Bucket Key

### SSE-KMS Encryption

- New setting to decrease...
    - Number of API calls made to KMS from S3 by 99%
    - Costs of overall KMS encryption with S3 by 99%
- This leverages data keys
    - a "S3 bucket key" is generated
    - That key is used to encrypt KMS objects with new data keys
- You will see **less KMS CloudTrail events in CloudTrail**

# CloudHSM Overview

- KMS => AWS manages the software for encryption
- CloudHDM => AWS provisions encryption **hardware**
- Dedicated Hardware (HSM = Hardware Security Module)
- You manage your own encryption keys entirely (not AWS)- Supports both symmetric and **asymmetric** encryption (SSL/TLS keys)
- No free tier available
- Must use the CloudHSM Client Software
- Redshift supports CloudHSM for db encryption and key management
- **Good option to use with SSE-C encryption**

- **IAM Perms**
    - CRUD an HSM Cluster

- **CloudHSM Software**
    - Manage the Keys
    - Manage the Users

- clusters spread across multi-AZ ( High Availability )

### Integration with AWS Services

- Through integration with KMS
- configure KMS Custom Key Store with CloudHSM
- Ex: EBS, S3, RDS...

# SSM Parameter Store Overview

- Secure storage for configs and secrets
- Optional Seamless Encryption using KMS
- Serverless, scalable, durable easy SDK
- Version tracking 
- Security w/ IAM
- Notifications with EventBridge
- Integration with CloudFormation

### Hierarchy

- /my-department/
    - my-app/
        - dev/
            - db-url
            - db-password
        - prod/
            - db-url
            - dp-password
- /other-department/
    - etc...

### Parameter Policies (advanced)

- Allow to assign a TTL to a parameter (expiration date) to force updating or deleting sensitive data such as passwords
- Can assign multiple policies at a time

# Secrets Manager - Overview

- Newer service, meant for storing secrets
- Capability to force the **rotation of secrets** every X days
- Automate generation of secrets on rotation (uses Lambda)
- Integration with **RDS** (MySQL, PostreSQL, Aurora)
- Secrets are encrypted with KMS

- Mostly meant for RDS

### Multi-Region Secrets

- Replicate Secrets across multiple AWS Regions
- Secrets Manager keeps read replicas in sync with the primary Secret
- Ability to promote a read replica Secret to a standalone Secret
- Use cases: multi-region apps, disaster recovery strategies, multi-region DB...

# Secrets Manager - CloudFormation Integration

### RDS & Aurora

- ***ManageMasterUserPassword** - creates admin secret implicitly
- RDS, Aurora will manage the secret in Secrets Manager and its rotation

# SSM Parameter Store vs Secrets Manager

- **Secrets Manager ($$$)**:
    - Automatic rotation of secrets with Lambda
    - Lambda is provided for RDS, Redshift, DocumentDB
    - KMS encryption is mandatory
    - Can integrate with CloudFormation
- ***SSM Parameter Store ($)**:
    - Simple API
    - No secret rotation (can enable rotation using Lambda triggered by CW Events)
    - KMS encryption is optional
    - Can integrate with CloudFormation
    - Can pull a Secrets Manager secret using the SSM Parameter Store API

# CloudWatch Logs Encryption

- you can encrypt CW Logs with KMS keys
- enabled at the log group level, by associating a CMK with a log group, either when you create the log group or after it exists
- You cannot associate a CMK with a log group using the CW console

- You must use the CW Logs API:
    - **associate-kms-key**: if the log group already exists
    - **create-log-group**: if the log group doesnt exist yet

# CodeBuild Security

- to access resources in your VPC, make sure you specify VPC config for your CodeBuild

- Secrets in CodeBuild:
- **Dont store them as plaintext env variables**
    - Env variables can reference Parameter Store parameters
    - Env variables can reference Secrets Manager secrets

# AWS Nitro Enclaves

- Process highly sensitive data in an isolated compute environment
    - Personally Identifiable Information (PII), healthcare, financial, ...
- Fully isolated VMs, hardened, and highly constrained
    - Not a container, not persistent storage, no interactive access, no external networking
- Helps reduce the attack surface for sensitive data processing apps
    - **Cryptographic Attestation** - only authorized code can be running in your Enclave
    - only Enclaves can access sensitive data (integration with KMS)
- Use cases: securing private keys, processing credit cards, secure multi-party computation...

