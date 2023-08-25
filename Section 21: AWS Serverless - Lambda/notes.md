
# What's Serverless

- Initially.. Serverless == FaaS (Functions as a Service)
- Now, serverless includes anything that's managed: "databases, messaging, storage, etc.."

- Serverless Services in AWS:
    - Lambda
    - DynamoDB
    - Cognito
    - API Gateway
    - S3
    - SNS & SQS
    - Kinesis Firehose
    - Aurora Serverless
    - Step Functions
    - Fargate

# Lambda Overview

- **Functions**
- **Short Executions** (15 minutes max)
- **On-demand**
- **Scaling is Automated**

### Benefits

- Easy Pricing:
    - Pay per request and compute time
    - Free tier of 1M requests and 400k GB-seconds of compute time

- Integrated with the whole AWS suite of services
- Integrated with many programming languages
- Easy monitoring through CloudWatch
- Easy to get more resources per functions (up to 10GB of RAM)
- Increasing RAM will also improve CPU and network!

- **Language Support**
    - Node.js
    - Python
    - Java
    - C#
    - Go
    - C#
    - Ruby
    - Custom Runtime API (community supported, ex. Rust)

    - Lambda Container Image (running Lambda in predefined docker images):
        - The container image must implement Lambda Runtime API
        - ECS / Fargate is preferred for running arbitrary Docker images

- **Pricing**
    - Pay per **calls**:
        - First 1M requests are free
        - $0.20 per 1M requests thereafter
    - Pay per **durations (in increments of 1ms)
        - 400k GB-seconds of compute time per month if FREE
        - == 400k seconds if function is 1GB RAM
        - == 3.2M seconds if function is 128 MB RAM
        - After that $1.00 for 600k GB-seconds
    - It us usually *very cheap* to run Lambda so it's *very popular*

# Lambda Synchronous Invocations

- Synchronous: CLI, SDK, API Gateway, ALB
    - Results are returned right away
    - Error handling must happen client side (retries, exponential backoff, etc..)

- **Services**
    - *User Invoked*:
        - ELB
        - API Gateway
        - CloudFront
        - S3 Batch
    - *Service Invoked*:
        - Cognito
        - Step Functions
    - *Other Services*:
        - Amazon Lex
        - Amazon Alexa
        - Kinesis Data Firehose

# Lambda & ALB

**Integration**
- To expose a Lambda as an HTTP(s) endpoint..
- You can use the **Application Load Balancer** (or an API Gateway)
- The Lambda function must be registered in a **target group**

- HTTP gets translated to a JSON document for the Lambda:
    - {
        "statusCode": 200,
        "statusDescription": "200 OK",
        "headers": {
            "Content-Type": "text/html; charset=utf-8"
        },
        "body": "\<h1>Hello World!\</h1>",
        "isBase64Encoded": false
    }

- **ALB Multi-Header Values (EXAM)**:
    - ALB can support multi header values (ALB setting)
    - When you enable multi-value headers, **HTTP headers and query string parameters** that are sent with **multiple values** are shown **as arrays** within the AWS Lambda event and response objects

# Lambda Asynchronous Invocations & DLQ

- S3, SNS, CloudWatch Events...
- The events are placed in an **Event Queue**
- Lambda attempts to retry on errors
    - 3 tries total
    - 1 minute wait after 1st, then 2 minutes wait
- Make sure the processing is **idempotent** (in case of retries)
- If the function is retried, you will see **duplicate logs entries in CloudWatch Logs**
- Can define a DLQ (dead-letter queue) - **SNS or SQS** - for failed processing (need correct IAM permissions)
- Asynchronous invokes allow you to speed up the processing if you don't need to wait for the result (ex: you need 1000 files processed)

- **Services**:
    - S3
    - SNS
    - CloudWatch Events / EventBridge
    - CodeCommit
    - CodePipeline
    - CloudWatch Logs
    - Simple Email Service
    - CloudFormation
    - Config
    - IoT
    - IoT Events

# Lambda & CloudWatch Events / EventBridge

1. **CRON** (scheduled triggers)
2. **Event based Triggers**

# S3 Event Notifications

- Use Case: generate thumbnail of images uploaded to S3

- S3 event notifications typically deliver events in seconds but can sometimes take a minute or longer
- If 2 writes are made to a single non-versioned object at the same time, it is possible that only a single event notification will be sent
- If you want to ensure that an event notification is sent for every successful write, you can enable versioning on your bucket

### Simple S3 Event Pattern - Metadata Sync

S3 -> Lambda (update metadata table) -> RDS/DynamoDB

# Lambda Event Source Mapping

- **Kinesis Data Streams / SQS / DynamoDB Streams**

- Common denominator: records need to be polled from the source
- **Your Lambda Function is invoked synchronously**

### Streams & Lambda (Kinesis & DynamoDB)

- An event source mapping creates an iterator for each shard, processes items in order
- Start with new items, from the beginning, or from timestamp
- Processed items aren't removed from the stream (other consumers can read them)
- Low traffic: use batch window to accumulate records before processing
- You can process multiple batches in parallel

### Streams & Lambda - Error Handling

- **By default, if you function returns an error, the entire batch is reprocessed until the function succeeds, or the items in the batch expire**
- To ensure in-order processing, processing for the affected shard is paused until the error is resolved
- You can configure the event source mapping to:
    - discard old events
    - restrict the number of retries
    - split the batch on error (to work around Lambda timeout issues)
- Discarded events can go to a **Destination**

### SQS & SQS FIFO

- Event Source Mapping will poll SQS (Long Polling)
- Specify **batch size** (1-10 messages)
- Recommended: Set the queue visibility timeout to 6x the timeout of your Lambda function
- **To use DLQ**:
    - **set-up on the SQS queue, not Lambda** (DLQ for Lambda is only for async invocations)
    - Or use a Lambda destination for failures

### Queues & Lambda

- Lambda supports in-order processing for FIFO queues, **scaling up to the number of active message groups**
- For standard queues, items aren't necessarily processed in order
- Lambda scales up to process a standard queue as quickly as possible

- When an error occurs, batches are returned ot the queue as individual items and might be processes in a different grouping than the original batch
- Occasionally, the event source mapping might receive the same item from the queue twice, even if no function error occurred
- Lambda deletes the items from the queue after they're processed successfully
- You can configure the source queue to send items to a dead-letter queue if they cant't be processed

### Lambda Event Mapper Scaling

- **Kinesis Data Streams & DynamoDB Streams**:
    - One Lambda invoke per stream shard
    - If you use parallelization, up to 10 batches processed per shard simultaneously

- **SQS Standard**:
    - Lambda adds 60 more instances per minute to scale up
    - Up to 1000 batches of messages processed simultaneously

- **SQS FIFO**:
    - Messages with the same GroupID will be processed in order
    - The Lambda scales to the number of active message groups

# Lambda Event & Context Objects

- **Event Object**:
    - JSON-formatted document contains data for the function to process
    - Contains information from the invoking service (eg. EventBridge, custom, ...)
    - Lambda runtime converts the event to an object
    - Ex: input args, invoking service args, ...

- **Context Object**:
    - Provides methods and properties that provide info about the invocation, function, and runtime environment
    - Passed to your function by Lambda at runtime
    - Ex: aws_request_id, function_name, memory_limit_in_mb, ...

# Lambda Destinations

- Can configure to send result to a **destination**
- **Asychronous Invocations** - can define destinations for successful and failed events:
    - SQS
    - SNS
    - Lambda
    - EventBridge

    - Note: AWS recommends you use destinations instead of DLQ now (but both can be used at the same time)

- **Event Source Mapping** - for discarded event batches:
    - SQS
    - SNS

    - Note: you can send events to a DLQ directly from SQS

# Lambda Permissions - IAM Roles & Resource Policies

### Lambda Exectuion Role (IAM Role - Give Lambda access to Services)

- Grants the Lambda function perms to AWS services / resources
- Sample managed policies for Lambda:
    - AWSLambdaBasicExecutionRole - Upload logs to CloudWatch
    - AWSLambdaKinesisExecutionsRole - Read from Kinesis
    - AWSLambdaDynamoDBExecutionRole - Read from DynamoDB Streams
    - AWSLambdaSQSQueueExecutionRole - Read from SQS
    - AWSLambdaVPCAccessExecutionRole - Deploy Lambda function in VPC
    - AWSLambdaXRayDaemonWriteAccess - Upload trace data to X-Ray

- **When you use an *event source mapping* to invoke your function, Lambda uses the execution role to read event data**
- Best Practice: create one Lambda Execution Role per function

### Lambda Resource Based Policies (Function Invocation Policies)

- Use resource-based policies to give other accounts and AWS services permission to use your Lambda resources
- Similar to S3 bucket policies for S3 bucket
- An IAM principal can access Lambda:
    - if the IAM policy attached to the principal authorizes it (eg. user access)
    - OR if the resource-based policy authorizes (eg. service access)

- When an AWS service like S3 calls your Lambda function, the resource-based policy gives it access


# Lambda Environment Variables

- Environment variable = key / value pair in "String" form
- Adjust the function behavior without updating code
- The environment variables are available to your code
- Lambda Service adds its own system environment variables as well

- Helpful to store secrets (encrypted by KMS)
- Secrets can be encrypted by the Lambda service key, or your own CMK

# Lambda Monitoring & X-Ray Tracing

### Lambda Logging & Monitoring

- CloudWatch Logs:
    - execution logs are stored in CloudWatch Logs
    - **Make sure you Lambda function has an execution role with an IAM policy that authorizes write to CloudWatch Logs

- CloudWatch Metrics:
    - metrics are displayed in CloudWatch Metrics
    - Invocations, Durations, Concurrent Executions
    - Error count, Success Rates, Throttles
    - Async Delivery Failures
    - Iterator Age (Kinesis & DynamoDB Streams)

### Lambda Tracing with X-Ray

- Enable in Lambda config (**Active Tracing**)
- Runs the X-Ray daemon for you
- Use AWS X-Ray SDK in Code
- Ensure Lambda Function has a correct IAM Execution Role
    - The managed policy is called AWSXRayDaemonWriteAccess
- Env Variables to comminicate with X-Ray
    - **_X_AMZN_TRACE_ID**: contains the tracing header
    - **AWS_XRAY_CONTEXT_MISSING**: by default, LOG_ERROR
    - **AWS_XRAY_DAEMON_ADDRESS**: the X-Ray Daemon IP_ADDRESS:PORT

# Lambda@Edge & CloudFront Functions

- Many modern apps execute some form of logic at the edge
- **Edge Function**:
    - code attached to the CloudFront distributions
    - Runs close to your users to minimize latency
- CloudFront provides 2 types: **CloudFront Functions & Lambda@Edge**

- Use Case: customize the CDN content
- Pay for what you use
- fully serverless

### Use Cases

- Website Security and Privacy
- Dynamic Web Apps at the Edge
- SEO
- Intelligently Route Across Origins and Data Centers
- Bot Mitigation at the Edge
- Real-time Image Transformation
- A/B Testing
- User Auth
- User Prioritization
- User Tracking & Analytics

### CloudFront Functions

- Lightweight functions written in JS
- For high-scale, latency-sensitive CDN Customizations
- Sub-ms startup times, **millions requests/secs**
- Used to change Viewer requests and responses:
    - **Viewer Request**: after CloudFront receives a requests from a viewer
    - **Viewer Response**: before CloudFront forwards the response to the viewer
- Native feature of CloudFront (manage code entirely within CloudFront)

### Lambda@Edge

- functions written in Node.js or Python
- Scales to **1000s of requests/second**
- Used to change CloudFront requests and responses:
    - **Viewer Request**: AFTER CloudFront receives the response FROM a VIEWER
    - **Origin Request**: BEFORE CloudFront forwards the response TO the ORIGIN
    - **Origin Response**: AFTER CloudFront receives the response FROM the ORIGIN
    - **Viewer Response**: BEFORE CloudFront forwards the response TO the VIEWER
- Author your functions in one AWS Region, then CloudFront replicates to its locations

### CloudFront Functions vs Lambda@Edge - Use Cases

- **CloudFront Functions**:
    - Cache key normalization:
        - Transform request attributes (headers, cookies, query strings, URL) to create an optimal cache key
    - Header manipulation:
    - URL rewrites or redirects
    - Request Auth

- **Lambda@Edge**:
    - Longer execution time 
    - Adjustable CPU or memory
    - Your code depends on 3rd party libraries (eg. AWS SDK to access other services)
    - Network access to use external services for processing
    - File system access or access to the body of HTTP requests

# Lambda in VPC

### Lambda by Default

- by default, your Lambda function is launched outside your own VPC (in AWS-owned VPC)
- Therefor it cannot access resources in your VPC (RDS, ElastiCache, internal ELB...)

- you must define the VPC ID, the Subnets and the SGs
- Lambda will create an ENI (Elastic Network Intervace) in your subnets
- **AWSLambdaVPCAccessExecutionRole**

### Lambda in VPC - Internet Access

- A Lambda in your VPC does not have internet access
- **Deploying a Lambda function in a public subnet does NOT give it internet access or a public IP**
- Deploying a Lambda function in a private subnet gives it internet access if you have a **NAT Gateway / Instance**
- You can use **VPC endpoints** to privately access AWS services without a NAT

# Lambda Function Performance

- **RAM**: 
    - From 128 MB to 10 GB in 1 MB increments
    - The more RAM you add, the more vCPU credits you get
    - At 1,792 MB, a function has the equivalent of 1 vCPU
    - After 1,792 MB, you get more than 1 CPU, and need to use multi-threading in your code to benefit from it
- **If your application is CPU-bound (computation heavy), increase RAM**

- **Timeout**: default 3 seconds, maximum is 900 seconds (15 minutes)

### Lambda Execution Context

- The execution context is a temporary runtime env that inits any external dependencies of your code
- Great for DB connectoins, HTTP clients, SDK clients..
- The execution context is maintained for some time in anticipation of another Lambda function invocation
- The next function invocation can "re-use" the context to execution time and save time in initializing connection objects
- The execution context includes the */tmp* directory

### Lambda Functions /tmp space

- if your function needs to download a big file to work..
- if your function needs disk space to perform operations..

- you can use the /tmp directory
- Max size is 10 GB
- The directory content remains when the execution context is frozen, providing transient cache that can be used for multiple invocations (helpful to checkpoint your work)
- For permanent persistence of object (non temp), use S3
- To encrypt content on /tmp, you must generate KMS Data Keys

# Lambda Layers

- Custom Runtimes
    - Ex:
        - C++
        - Rust
- Externalize Dependencies to re-use them

# Lambda - File Systems Mounting

- Lambda functions can access EFS file systems if they are running in a VPC
- Configure Lambda to mount EFS file systems to a local directory during initialization
- Must leverage EFS Access Points
- Limitations: watch out for the EFS connection limits (one function instance = one connection) and connection burst limits

# Lambda Concurrency

- Concurrency limit: up to 1000 concurrent executions
- Can set a "**reserved concurrency**" at the function level (=limit)
- Each invocation over the concurrency limit will trigger a "Throttle"
- Throttle behavior:
    - if synchronous invocation -> return ThrottleError - 429
    - if asynchronous invocation -> retry automatically and then go to DLQ
- If you need a higher limit, open a support ticket

### Concurrency Issue

- If you dont reserve concurrency, the following can happen:
    - if one Lambda application maxes out concurrency, it can choke other Lambda applications

- If the function doesn't have enough concurrency available to process all events, additional requests are throttled
- For throttling errors (429) and system errors (5xx), Lambda returns the event to the queue and attempts to run the function again for up to 6 hours
- The retry interval increases exponentially from 1 sec after the 1st attempt to a maximum of 5 minutes

### Cold Starts & Provisioned Concurrency

- **Cold Start**:
    - New instance -> code is loaded and code outside the handler run (init)
    - If the init is large (code, dependencies, SDK...) this process can take some time
    - First request served by new instances has higher latency than the rest

- **Provisioned Concurrency**:
    - Concurrency is allocated before the function is invoked
    - So the cold start never happens and all invocations have low latency
    - Application Auto Scaling can manage concurrency (schedule or target utilization)

- **Note**:
    - Cold starts in VPC have been dramatically reduced in Oct & Nov 2019

# Lambda External Dependencies

- If you Lambda depends on external libraries: (AWS SDKs, Database Clients, etc...)
- **You need to install the packages alongside your code and zip it together**:
    - For Node.js, use npm & "node_modules" directory
    - For Python, use pip --target options
    - For Java, included the relevant .jar files
- Upload the **zip** straight to Lambda if less than 50 MB, else to S3 first
- Native libs work: they need to be compiled on Amazon Linux
- AWS SDK comes by default with every Lambda function

# Lambda & CloudFormation

- **Inline**
    - inline functions are very simple
    - Use the **Code.ZipFile** property
    - You cannot include function dependencies with inline functions

- **Through S3**
    - You must store the Lambda zip in S3
    - You must refer the S3 zip location in the CloudFormation code
        - S3Bucket
        - S3Key: full path to zip
        - S3ObjectVersion: if versioned bucket
    - **If you update the code in S3, but don't update S3Bucket, S3Key or S3ObjectVersion, CloudFormation won't update your function**

- **Through S3 (Multiple Accounts)**
    - Add bucket policy on S3 bucket in Account_1 to give Account_2 CloudFormation access to the bucket
    - Add Execution Role on CloudFormation in Account_2 to get & list bucket in Account_1

# Lambda Container Images

- Deploy Lambda as container images of up to 10 GB from ECR
- Pack complex dependencies, large dependencies in a container
- Base images are available for Python, Node.js, Java, .NET, Go, Ruby
- Can create your own images as long as it implements the **Lambda Runtime API**
- Test the containers locally using the Lambda Runtime Interface Emulator
- Unified workflow to build apps

### Best Practices

- Strategies for optimizing container images:
    - **Use AWS-provided Base Images**
        - Stable, Built on Amazon Linux 2, cached by Lambda service
    - **Use Multi-Stage Builds**
        - Build your code in larger preliminary images, copy only the artifacts you need in your final container image, discard the preliminary steps
    - **Build from Stable to Frequently Changing**
        - Make your most frequently occurring changes as late in your ***Dockerfile*** as possible
    - **Use a Single Repository for Functions with Large Layers**
        - ERC compares each layer of container image when it is pushed to avoid uploading and storing duplicates

- Use them to upload large Lambda Functions (up to 10GB)

# Lambda Versions & Aliases

### Lambda Versions

- When you work on a Lambda function, we work on **$LATEST**
- When we're ready to publish a Lambda, we create a version
- Versions are immutable
- Versions have increasing version numbers
- Versions get their own ARN
- Version = code + config
- Each version of the lambda can be accessed

### Lambda Aliases

- "pointers" to Lambda versions
- We can define "prod/test/dev" aliases and have them point at different lambdas
- Aliases are mutable
- Aliases enable Canary deployment by assigning weights to lambda functions
- Aliases enable stable configuration of our event triggers / destinations
- Aliases have their own ARNs
- **Aliases cannot reference aliases**

# Lambda & CodeDeploy

- **CodeDeploy** can help you automate traffic shift for Lambda aliases
- Feature is integrated within the SAM framework
- **Linear** grow traffic every N minutes until 100%
- **Canary**: try X percent then 100%
- **AllAtOnce**: immediate
- Can create Pre & Post Traffic hooks to check the health of the Lambda and Rollback

### Lambda & CodeDeploy: AppSpec.yml

- **Name (required)** - name of the Lambda
- **Alias (required)** - nam eof the Lambda Alias
- **CurrentVersion (required)** - name of the version the Lambda currently points to (in case of rollback)
- **TargetVersion (required)** - the version of the Lambda traffic will be shifted to

# Lambda Function URL

- Dedicated HTTP(s) endpoint for your Lambda
- A unique URL endpoint is generated for you (never changes)

- Invoke via web browser, curl, Postman, or any HTTP client
- Access your function URL through the public internet only
    - Doesn't support PrivateLink
- Supports Resource-based Policies & CORS configs
- Can be applied to any function alias or to $LATEST (can't be applied to other function versions)
- Create and configure using AWS Console or AWS API
- Throttle your function by using Reserved Concurrency

### Lambda - Function URL Security

- **Resource-based Policy**
    - Auth other accounts / specific CIDR / IAM principals

- **CORS**
    - if you call your Lambda URL from a different domain

- **AuthType NONE** - Allow public and unauthenticated access
    - Resource-based Policy is always in effect (must grant public access)

- **AuthType AWS_IAM** - IAM is used to auth requests
    - Both Principal's Identity-based Policy & Resource-based Policy evaluated
    - Principal must have **lambda:InvokeFunctionUrl** permissions
    - **Same Account** - Identity-based **OR** Resource-based Policy as ALLOW
    - **Cross Account** - Identity-based **OR** Resource-based Policy as ALLOW


# Lambda - CodeGuru Integration

- Gain insights into runtime performance of your Lambda using CodeGuru Profiler
- CodeGuru creates a Profiler Group for your functions
- Supported for Java and Python runtimes
- Activate from AWS Lambda Console
- When activated, Lambda adds:
    - CodeGuru Profiler layer to your function
    - Env variables to your function
    - **AmazonCodeGuruProfilerAgentAccess** policy to your function

# Lambda Limits

### Limits to Know - *per Region*

- **Execution**:
    - Memory allocation: 128 MB - 10 GB (1 MB increments)
    - Maximum execution time: 15 minutes
    - Env Variables (4 KB)
    - Disk capacity in the "function container" (in /tmp): 512 MB to 10 GB
    - Concurrency executions: 1000 (can be increased)

- **Deployment**:
    - Lambda function deployment size (compressed .zip): 50 MB
    - size of uncompresses deployment (code + deps): 250 MB
    - Can use the /tmp directory to load other files at startup
    - Size of environment variables: 4 KB

# Lambda Best Practices

- **Perform heavy-duty work outside of your function handler**
    - Connect DBs outside of your function handler
    - Initialize the AWS SDK outside of your function handler
    - Pull in deps or datasets outside of your function handler
- **Use env variables for**:
    - DB Connection Strings, S3 bucket, etc...
    - Passwords, sensitive values... can be encrypted with KMS
- **Minimize your deployment package size to its runtime necessities**
    - Break down the function if need be
    - Remember the AWS Lambda limits
    - Use Layers where necessary
- **Avoid using recursive code, never have a Lambda call itself**

