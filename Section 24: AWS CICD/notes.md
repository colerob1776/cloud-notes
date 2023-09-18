# CodeCommit - Overview

- **Version Control** service (uses **git**)
- **Private Git repos**
- No size limit on repos
- Fully managed, HA
- Code only in AWS Cloud account => increased security and compliance
- Integrated with Jenkins, CodePipeline...


- **Security**
    - encyrpted repos
    - standard HTTPS or Keys auth
    - IAM policies to manage access

# CodePipeline

- Visual Workflow to orchestrate you CI/CD
- **Source** - CodeCommit, ECR, S3, GitHub
- **Build** - CodeBuild, Jenkins...
- **Test** - CodeBuild, 3rd party tools
- **Deploy** - CodeDeploy, CloudFormation, Beanstalk...
- **Invoke** - Lambda, Step Functions

- Consists of **Stages**, which consist of multiple **Action Groups**

### CodePipeline - Artifacts

- Each pipeline can create **artifacts**
- stored in S3 bucket and passed to the next stage

### CodePipeline - Troubleshooting

- use **CW Events (EventBridge)**:
    - create events for failed events or cancelled stages

- If CodePipeline fails a stage, your pipeline stops, and you get information in the console
- If pipeline can't perform an action, make sure the "IAM Service Role" attached has enough IAM perms (IAM Policy)

- CloudTrail can be used to audit AWS API calls

### CodePipeline - Events vs Webhooks vs Polling

- **Events**
    - EventBridge (only available in within AWS)
    - CodeStar Source connection

- **Webhooks**
    - Trigger and listen to CodePipeline with HTTPS requests

- **Polling**
    - listen to CodePipeline by polling HTTPS requests (not recommended)

### CodePipeline - Action Type Constraints for Artifacts

- **Owner**
    - **AWS** - for AWS services
    - **3rd Party** - GitHub or Alexa Skilled Kit
    - **Custom** - Jenkins

- **Action Type**
    - **Source** - S3, ECR, GitHub..
    - **Build** - CodeBuild, Jenkins
    - **Test** - CodeBuild, Device Farm, Jenkins
    - **Approval** - **Manual**
    - **Invoke** - Lambda, Step Functions
    - **Deploy** - S3, CloudFormation, CodeDeploy...

### CodePipeline - Manual Approval Stage

- ***Important*: Owner is "AWS", Action is "Manual"**

# CodeBuild - Overview

- **Source** - CodeCommit, S3, GitHub...
- **Build Instructions**: Code file **buildspec.yml** or insert manually in Console
- **Output Logs**: can be stored in S3 & CW Logs
- Use CW Metrics to monitor build statistics
- Use EventBridge to detect failed builds and trigger notifications
- Use CW Alarms to notify if you need "thresholds" for failures

- **Build Projects can be defined within CodePipeline or CodeBuild**

### CodeBuild - Supported Envs

- Java
- Ruby
- Python
- Go
- Node
- Android
- .NET core
- PHP
- Custom Docker Images

### CodeBuild - buildspec.yml

- **buildspec.yml** - must be at **root** of your code
- **env** - define env variables
    - **variables** - plaintext variables
    - **parameter-store** - variables stored in SSM Parameter Store
    - **secrets-manager** - variables stored in AWS Secrets Manager
- **phases** - specify commands to run
    - **install** - install deps
    - **pre_build**
    - **Build**
    - **post_build**
- **artifacts** - what to upload to S3 (encrypted with KMS)
- **cache** - files to cache to S3 for future build speedup (dependencies, etc..)

### CodeBuild - Local Builds

- Use CodeBuild Agent docker container

### CodeBuild - Inside VPC

- By default, CodeBuild containers are launched outside your VPC
    - It cannot access resources in a VPC

- You can specify a VPC config:
    - VPC ID
    - Subnet IDs
    - SG IDs

- Then your build can access resources in your VPC (eg, RDS, ElastiCache, EC2, ALB, ...)
- Use cases:
    - integration testing
    - data query
    - internal load balancers

# CodeDeploy

- Deployment service that automates app deployment
- Deploy new apps to EC2 Instances, on-premises servers, Lambda, ECS Services..
- Automated Rollback, or trigger CW Alarm
- Gradual deployment control
- A file named **appspec.yml** controls CodeDeploy

### CodeDeploy - EC2/On-premise Platform

- Perform in-place deployments or blue/green deployments
- Must run the **CodeDeploy Agent** on the target instances
- Define deployment speed
    - AllAtOnce: most downtime
    - HalfAtATime: reduced capacity by 50%
    - OneAtATime: Slowest, lowest availability impact
    - Custom: define your %

### CodeDeploy Agent

- The CodeDeploy Agent must be running on the EC2 instances as a pre-requisite
- It can be installed and updated automatically if you're using Systems Manager
- The Instances must have sufficient IAM perms to access S3 to get deployment bundles

### CodeDeploy - Lambda Platform

- **CodeDeploy** can help you automate traffic shift for Lambda aliases
- Feature is integrated within the SAM framework

- Traffic shifting strategies:
    - **Linear**
    - **Canary**
    - **AllAtOnce**

### CodeDeploy - ECS Platform

- automate the deployment of a new ECS Task Definition
- ONLY Blue/Green deployments (spin up new target group before shutting down old target group)

- Traffic shifting strategies for target groups:
    - **Linear**
    - **Canary**
    - **AllAtOnce**

# CodeDeploy for EC2 and ASG

### Deployments to EC2

- Define how to deploy the app using **appspec.yml** + Deployment Strategy
- Will do in-place update to your fleet of Instances
- Can use hooks to verify the deployment after each deployment phase

### Deployments to an ASG

- **In-place Deployment**
    - updates existing EC2
    - newly created EC2 instances by an ASG will also get automated deployments

- **Blue/Green Deployment**
    - new ASG is created (settings are copied)
    - Choose how long to keep the old EC2 Instances (old ASG)
    - Must be using an ELB

- **Redeploy & Rollbacks**
    - Rollback = redeploy a previous version
    - Deployments can be rolled back:
        - **Automatically** - rollback when a deployment fails or a CW Alarm threshold is met
        - **Manually**
    - Can disable rollbacks
    - **if a rollback happens, CodeDeploy redeploys the last known good version (*new deployment*)**

# CodeDeploy - Extras

### Troubleshooting

- **Deployment Error: "InvalidSignatureException" - Signature expired: [time] is now earlier than [time]**
    - For CodeDeploy to perform its operations, it required accurate time reference
    - if the date and time of your EC2 instances are not set correctly, they may not match the signature date of your deployment request, which CodeDeploy rejects
- Check log files to understand deployment issues
    - For Amazon Linux, Ubuntu and RHEL log files sotred at **/opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log**

# CodeStar - Overview

- An integrated solution that groups: GitHub, CodeCommit, CodeBuild, CodeDeploy, CloudFormation, CodePipeline, CloudWatch..
- Quickly create "CI/CD ready" projects for EC2, Lambda, Beanstalk
- Supported Languages
    - C#
    - Go
    - HTML 5
    - Java
    - Node
    - PHP
    - Python
    - Ruby
- Issue tracking with JIRA / GitHub Issues
- Ability to integrate with Cloud9 to obtain a web IDE (not all regions)
- One dashboard to view all your components
- Free service, pay only for the underlying services
- Limited Customization 

# CodeArtifact - Overview

- Storing and retrieving dependencies is called **artifact management**
- Traditionally you need to setup your own artifact management system
- **CodeArtifact** is a secure, scalable, and cost-effective Artifact Management for software development
- Works with common dependency management tools such as Maven, Gradle, npm, yarn, twine, pip, and NuGet
- **Developers and CodeBuild can then retrieve dependencies straight from CodeArtifact**

# CodeArtifact - EventBridge Integration

- **Event is created when a Package version is created, modified, or deleted**

# CodeArtifact - Resource Policy

- Can be used to authorize another account to access CodeArtifact
- A given principal can either read all of the packages in a repo or none of them

# CodeArtifact - Upstream Repositories & Domains

- A CodeArtifact repo can have other CodeArtifact repos as **Upstream Repositories**
- Allows a package manager client to access the packages that are contained in more than one repo using a single repo endpoint
- Up to 10 Upstream Repos
- Only one external connection

### CodeArtifact - External Connection

- An External Connection is a connection b/w a CodeArtifact Repo and an external/public repo (npm, PyPi, NuGet...)
- Allows you to fetch packages that are not already present in your CodeArtifact Repo
- A repo has a max of 1 external connection
- Create as many repos for many external connections

### CodeArtifact - Retention

- If a requested package version is found in an Upstream Repo, a reference to it is retained and is always available from the Downstream Repo
- The retained package version is not affected by changes to the Upstream Repo (deleting it, updating the package,...)
- Intermediate repos do not keep the package

### CodeArtifact - Domains

- **Deduplicated Storage** - asset only needs to be stored once in a domain, even if it's available in many repos (only pay once for storage)
- **Fast Copying** - only metadata records are updated when you pull packages from an Upstream Repo into a Downstream
- **Easy Sharing Across Repos and Teams** - all the assets and metadata in a domain are encrypted with a single AWS KMS Key
- **Apply Policy Across Multiple Repos** - domain admin can apply policy across the domain such as:
    - Restricting which accounts have access to repos in the domain
    - Who can configure connections to public repos to use as sources of packages

# CodeGuru - Overview

- An ML-powered service for **automated code reviews** and **application performance recommendations**
- Provides 2 functionalities
    - **CodeGuru Reviewer**: auto code reviews for static code analysis (development)
    - **CodeGuru Profiler**: visibility/recommendations about app performance during runtime (production)

### CodeGuru Reviewer

- Identify critical issues, security vulnerabilities, and hard-to-find bugs
- Ex: common coding best practices, resource leaks, security detection, input validation
- Uses ML and automated reasoning
- Hard-learned lessons across millions of code reviews on 1000s of open-source and AWS repos
- Supports Java and Python
- Integrates with GitHub, Bitbucket and CodeCommit

### CodeGuru Profiler

- Helps understand the runtime behavior of your app
- Ex: identify if you app is consuming excessive CPU capacity on a logging routine
- Features: 
    - Identify and remove code inefficiencies
    - improve app perf
    - Decrease compute costs
    - Provides heap summary (memory profiling)
    - Anomaly Detection
- Support apps running on AWS or on-premise
- Minimal overhead on app

# CodeGuru - Agent Configuration

- **MaxStackDepth** - the max depth of the stacks in the code that is represented in the profile
    - Ex: if CodeGuru finds a method A, which calls method B, which calls method C, which calls Method D, the depth is 4
    - If MaxStackDepth is set to 2, the profiler evaluates A and B
- **MemoryUsageLimitPercent** - the memory percentage used by the profiler
- **MinimumTimeForReportingMilliseconds** - the minimum time between sending reports (millis)
- **ReportingIntervalInMilliseconds** - the reporting interval used to report profiles (millis)
- **SamplingIntervalInMilliseconds** - the sampling interval that is used to profile samples (millis)
    - Reduce to have higher sampling rate

# Cloud9 - Overview
 
- Cloud-based IDE
- Code editor, debugger, terminal in browser
- Work on your projects from anywhere with internet connection
- Prepackaged with essential tools for popular languages (JS, Python, PHP, ...)
- Share your dev env with team (pair programming)
- Fully integrated with SAM & Lambda to easily build serverless apps