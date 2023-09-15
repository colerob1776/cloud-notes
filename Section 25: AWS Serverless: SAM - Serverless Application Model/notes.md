# SAM Overview

- Serverless Application Model
- Framework for developing and deploying serverless apps
- All the config is YAML code
- Generate complex CloudFormation from simple SAM YAML file
- Supports anything from CloudFormation: Outputs, Mappings, Parameters, Resources...
- Only 2 commands to deploy to AWS
- SAM can use CodeDeploy to deploy Lambda functions
- SAM can help you run Lambda, API Gateway, DynamoDB locally

### SAM - Recipy

- **Transform Header indicates it's SAM template**:
    - **Transform: 'AWS::Serverless-2016-10-31'**

- **Write Code**
    - **AWS::Serverless::Function**
    - **AWS::Serverless::Api**
    - **AWS::Serverless::SimpleTable**

- **Package & Deploy**
    - **sam package**
    - **sam deploy**

### SAM - CLI Debugging

- Locally build,test and debug your serverless apps that are defined using AWS SAM templates
- Provides a lambda-like execution env locally
- SAM CLI + AWS Toolkites => step-through and debug your code
- Supported IDEs: Cloud9, VSCode, JetBrains, PyCharm, IntelliJ, ...
- **AWS Toolkits**: IDE plugins which allows you to build, test, debug, deploy and invoke Lambda functions built using AWS SAM

# SAM Policy Templates

- List of Templates to apply permissions to your Lambda functions
- Important Examples:
    - **S3ReadPolicy**: Gives read only perms to objects in S3
    - **SQSPollerPolicy**: Allows to poll an SQS Queue
    - **DynamoDBCrudPolicy**: CRUD = create/read/update/delete

# SAM and CodeDeploy

- **SAM framework natively uses CodeDeploy to update Lambda functions**

- Traffic Shifting feature
- Pre and Post traffic hook features to validate deployment (before the traffic shift starts and after it ends)
- Easy & automated rollback using CloudWatch Alarms

- **AutoPublishAlias**
    - Detects when new code is being deployed
    - Creates and publishes an updated version of that function with the latest code
    - Points the alias to the updated version of the Lambda
- **DeploymentPreference**
    - Canary, Linear, AllAtOnce
- **Alarms**
    - Alarms that can trigger a rollback
- **Hooks**
    - Pre and Post traffic shifting Lambda functions to test your deployment

# SAM - Local Capabilities

- **Locally start AWS Lambda**
    - **sam local start-lambda**
    - starts a local endpoint that emulates AWS Lambda
    - Can run automated tests against this local endpoint

- **Locally invoke Lambda Function**
    - **sam local invoke**
    - Invoke Lambda function with payload once and quit after invocation completes
    - Helpful for generating test cases
    - If the function makes API calls to AWS, make sure you are using the correct **--profile** option

- **Locally start an API Gateway Endpoint**
    - **sam local start-api**
    - Starts a local HTTP server that hosts all your functions
    - Changes to functions are automatically reloaded

- **Generate AWS Events fo Lambda Functions**
    - **sam local generate-event**
    - Generate sample payloads for event sources
    - S3, API Gateway, SNS, Kinesis, DynamoDB...

# SAM Section Summary

- SAM is built on CloudFormation
- SAM requires the **Transform** and **Resources** sections
- Commands to know:
    - sam build: fetch deps and create local deployment artifacts
    - sam package: package and upload to S3, generate CF templates
    - sam deploy: deploy to CF
- SAM Policy Templates for easy IAM policy definition
- SAM is integrated with CodeDeploy to do deploy to Lambda aliases

# Serverless Application Repository (SAR)

- managed repo for serverless apps
- **The apps are packaged using SAM**
- Build and publish apps that can be re-used by orgs
    - Can share publicly
    - Can share with specific AWS Accounts
- This prevents duplicate work, and just go straight to the publishing phase
- App settings and behavior can be customized using **Environment Variables**   