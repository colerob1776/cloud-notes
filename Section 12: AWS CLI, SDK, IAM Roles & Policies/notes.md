## EC2 Instance Metadata (IMDS)

- AWS EC2 Instance Metadata (IMDS) is powerful but one of the least known features to developers
- It allow EC2 instances to "learn about themselve" without using an IAM Role for that purpose
- The URL is *{instance_url}/latest/metadata*
- You can retrieve the IAM Role name from the metadata, but you CANNOT retrieve the IAM Policy
- Metadata = Info about hte EC2 instance
- Userdata = launch script of the Instance

### IMDSv2 vs. IMDSv1

- **IMDSv1** is accessing *{instance_url}/latest/metadata* directly

- **IMDSv2** is more secure and is done in 2 steps:
    1. **Get Session Token (limited validity)** - using headers & PUT
        - `TOKEN='curl -X PUT "{instance_url}/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
    2. **Use Session Token in IMDSv2 calls** - using headers
        - `curl {instance_url}/latest/meta-data/profile -H "X-aws-ec2 metadata-token: $TOKEN`


## AWS CLI Profiles

- Creating:
    1.  `aws configure --profile {profile_name}`
    2.  when prompted input **AWS Access Key ID**, **AWS Secret Access Key**, **Default Region Name**, and **Default Output Format**

- Using:
    - with any command add the `--profile {profile_name}` flag option

## MFA with CLI

- To use MFA with CLI, you must use create a temporary session
- To do so, you must run the **STS GetSessionToken** API call

- `**aws sts get-session-token** --serial-number {arn_of_the_mfa_device} --token-code {code_from_token} --duration-seconds 3600`

## AWS SDK Overview

- Official SDKs are:
    - Java
    - .NET
    - Node.js
    - PHP
    - Python (named boto3/botocore)
    - Go
    - Ruby
    - C++

- We have to use the SDK when coding against AWS Services such as DynamoDB
- Fun fact... the CLI uses the boto3 SDK
- **The exam expects you to know when you should use an SDK**

- Good to know: if you don't specify/configure a default region, then us-east-1 will be chosen by default

## Expontial Backoff & Service Limit Increase

- API Rate Limits:
    - **DescribeInstances** APi for EC2 has a limit of 100 calls per second
    - **GetObject** on S3 has a limit of 5500 GET per second per prefix
    - For Intermittent Errors: implement **Exponential Backoff**
    - For Constistent Errors: request an API throttling limit increase

- Service Quotas (Service Limits):
    - Running On-Demand Standard Instances: 1 152 vCPU
    - You can request a service limit increase by **opening a ticket**
    - You can request a service quota increase by using the **Service Quotas API**

### EXAM: Exponential Backoff (any AWS Service)

- Exponential Backoff is a retry mechanism where the more you try, the more you wait `time_wait = prev_time_wait^2`
- If you get **ThrottlingExeption** intermittently, use exponential backoff
- Retry mechanism already included in AWS SDK API calls
- Must implement yourself if using the AWS API as-is or in specific cases
    - **Must only implement the retries on *5xx server errors* and throttling**
    - Do not implement on 4xx client errors

- Exam question: Which type of errors should you implement exponential backoff? **5xx server errors** and **throttling**
- Exam question: Do you need to implement exponential backoff when using the SDK? **NO**

## AWS CLI Credentials Provider Claim (Exam)

- The CLI will look for credentials in this order:
    1. **Command line options**: --region, --output, --profile
    2. **Environment variables**: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_SESSION_TOKEN
    3. **CLI credentials file** - aws configure
    4. **CLI configuration file** - aws configure
    5. **Container credentials** - for ECS containers
    7. **Instance profile credentials** - for EC2 Instances

## AWS SDK Default Credentials Provider Chain (Exam)

- The SDKs will look for credentials in this order:
    1. **System Properties** - aws.accessKeyId and aws.secretKey
    2. **Environment variables**: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY
    3. **default credentials profile file** - at `~/.aws/credentials``
    4. **ECS Container credentials** -for ECS containers
    7. **Instance profile credentials** - used on EC2 instances

### AWS Credentials Best Practices

- **NEVER STORE AWS CREDS IN YOUR CODE**
- Best practice is for creds to be inherited from the credentials chain

- **If using working within AWS, use IAM Roles**
    - EC2 Instance Roles for EC2 Instances
    - ECS Roles for ECS tasks
    - Lambda Roles for Lambda functions

- If working outside of AWS, use env variables / named profiles

## AWS Signature v4 Signing

- When you call hte AWS HTTP API, you sign the request so that AWS can identify you, using your AWS creds (access key & secret key)
- Note: some requests to S3 don't need to be signed
- If you use the SDK or CLI, the HTTP requests are signed for you

- You should sign an AWS HTTP request using Signature v4 (SigV4)

