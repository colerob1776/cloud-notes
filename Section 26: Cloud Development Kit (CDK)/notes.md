# Cloud Development Kit

- Define your cloud infra using a familiar language:
    - JS/TS, Python, Java, and .NET
- Contains high level components called **constructs**
- The code is "compiled" into a CloudFormation template (JSON/YAML)
- **You can therefore deploy infra and app runtime code together**
    - Great for Lambda
    - Gread for Docker in ECS/EKS

### CDK vs SAM

- SAM:
    - Serverless focused
    - Write your template declaratively in JSON or YAML
    - Great for quickly getting started with Lambda
    - Leverages CloudFormation

- CDK:
    - All AWS Services
    - Write infra in a programming language JS/TS, Python, Java and .NET
    - Leverages CloudFormation

- **CDK + SAM**
    - you can use SAM CLI to locally test your CDK apps
    - **You must first run cdk synth**

# CDK - Constructs

- CDK Construct is a component that encapsulates everything CDK needs to create the final CloudFormation Stack
- Can represent a single AWS resource or multiple related resources (eg worker queue with compute)
- **AWS Construct Libary**
    - collection of Constructs included in AWS CDK which contains Constructs for every AWS resource
    - Contains 3 different levels of Constructs available (L1, L2, L3)
- **Construct Hub** - contains additional Constructs from AWS, 3rd parties, and open-source, CDK community

### CDK Constructs - Layer 1 Constructs (L1)

- Can be called **CFN Resources** which represents all resources directly available in CloudFormation
- Constructs are periodically generated from **CloudFormation Resource Specification**
- Construct names start with **Cfn** (eg **CfnBucket**)

### CDK Constructs - Layer 2 Constructs (L2)

- Represents AWS resources with a higher level (intent-based API)
- Similar functionality as L1 but with convenient defaults and boilerplate
    - dont need to know all the details about the resource properties
- Provide methods that make it simpler to work with the resource (eg **bucket.addLifeCycleRule()**)

### CDK Constructs - Layer 3 Constructs (L3)

- Can be called ***Patterns*** which represents multiple related resources
- Helps you complete common tasks in AWS
- Ex:
    - **aws-apigateway.LambdaRestApi** represents an API Gateway backed by a Lambda Function
    - **aws-ecs-patterns.ApplicationLoadBalancerFargateService** which represents an arch that includes Fargate cluster with ALB

# CDK - Commands & Bootstrapping

- **Important Commands to Know**
    - **npm install -g aws-cdk-lib**: install the CDK CLI and libraries
    - **cdk init app**: create new CDK project from specified templates
    - **cdk synth**: synthesizes and prints the CloudFormation template
    - **cdk bootstrap**: deploys the CDK Toolkit staging Stack
    - **cdk deploy**: Deploys the Stack(s)
    - **cdk diff**: view differences of local CDK and deployed Stack
    - **cdk destroy**: destrol the Stack(s)

### CDK - Bootstrapping

- the process of provisioning resources for CDK before you can deploy CDK apps into an AWS env
- **AWS Environment = account & region**
- CloudFormation Stack calld **CDKToolkit** is created and contains:
    - **S3 Bucket** - to store files
    - **IAM Roles** - to grant perms to perform deployments

- You must run the following command for each new env:
    - **cdk bootstrap aws://\<aws_account\>/\<aws_region\>**
- Otherwise, you will get an **error "Policy contains a statement with one or more invalid principal"**

# CDK - Unit Testing

- To test CDK apps, use **CDK Assertions Module** combined with popular test frameworks such as Jest (JS) or Pytest 
- Verify we have specific resources, rules, conditions, parameters...
- Two types of tests:
    - **Fine-grained Assertions (common)** - test specific aspects of the CloudFormation template
    - **Snapshot Tests** - test the synthesized CloudFormation template against a previously stored baseline template

- To import a template (EXAM):
    - Template.fromStack(MyStack) : stack built in CDK
    - Template.fromString(mystring) : stack built outside CDK