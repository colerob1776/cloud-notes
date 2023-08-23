# Monitoring in AWS Overview

**Why is monitoring important**
    - We know how to deploy apps:
        - Safely
        - Automatically
        - Using Infra as Code
        - Leveraging the best AWS Components
    - Our apps are deployed, and our users don't care how we did it..
    - Our users only care that the app is working
        - Application latency: will it increase over time?
        - Application outages: customer experience should not degrade
        - Users contacting IT department or complaining is not a good outcome
        - Troubleshooting and remediation
    - Internal Monitoring
        - Can we prevent issues before they happen?
        - Performance and Cost
        - Trends (scaling patterns)
        - Learning and Improvement

**Monitoring in AWS**

- **AWS CloudWatch**
    - Metrics: Collect and track key metrics
    - Logs: Collect, monitor, analyze, and store log files
    - Events: Send notifications when certain events happen in your AWS
    - Alarms: React in real-time to metrics / events
- **AWS X-Ray**:
    - Troubleshooting application performance and errors
    - Distributed tracing of microservices
- **AWS CloudTrail**:
    - Internal monitoring of API calls being made
    - Audit changes to AWS Resources by your users

# CloudWatch Metrics

- CloudWatch provides metrics for every service in AWS
- **Metric** is a variable to monitor (CPUUtilization, NetworkIn,...)
- Metrics belong to **namespaces**
- **Dimension** is an attribute of a metric (instance id, environment, etc...)
- Up to 30 dimensions per metric
- Metrics have **timestamps**
- Can create CloudWatch dashboards of metrics

### EC2 Detailed Monitoring

- EC2 instance metrics have metrics "every 5 minutes"
- With detailed monitoring (for a cost), you get data "every 1 minute"
- Use detailed monitoring if you want to scale faster for your ASG

- The AWS Free Tier allows us to have 10 detailed monitoring metrics
- Note: EC2 Memory usage is by default NOT pushed (must be pushed from inside the instance as a custom metric)

# CloudWatch Custom Metrics

- possibility to define and send your own custom metrics to CloudWatch
- Ex: memory (RAM) usage, disk space, number of logged in users ...
- Use API call **PutMetricData**
- Ability to use dimensions to segment metrics
    - Instance.id
    - Environment.name
- Metric resolution (**StorageResolution** API parameter - 2 possible values)
    - Standard: 1 minute
    - High Resolution: 1/5/10/30 second(s) - Higher cost
- **Important**: Accepts metric data points 2 weeks in the past and 2 hours in the future (make sure to configure your EC2 instance time correctly)

# CloudWatch Logs

- **Log Groups**: arbitrary name, usually representing an app
- **Log Stream**: instances within apps / log files / containers
- Can define log expiration policies ( never expire, 1 day to 10 years)
- **CloudWatch Logs can send logs to**:
    - S3 (S3 Export)
    - Kinesis Data Streams
    - Kinesis Firehose
    - Lambda
    - OpenSearch
- Logs are encrypted by default
- Can setup KMS with your own keys

### Logs - Sources

- SDK, CloudWatch Logs Agent, CloudWatch Unified Agent
- Elastic Beanstalk: collection of logs from app
- ECS: collection from containers
- AWS Lambda: collection from function logs
- VPC Flow Logs: Network specific logs
- API Gatway
- CloudTrail based on filter
- Route53: Log DNS queries

### CloudWatch Logs Insights

- Used to query logs
- Example: find a specific IP inside a log, count occurrences of "ERROR" in your logs
- Provides a purpose-built query language
    - auto discovers fields from AWS services and JSON log events
    - Fetch desired event fields, filter based on conditions, calc aggregate statistics, sort events, limit number of events...
    - Can save queries and add them to CloudWatch Dashboards
- Can query multiple Log Groups in different AWS Accounts
- It's a query engine, not a real-time engine

### Logs - S3 Export

- Log data can take **up to 12 hours** to become available for export
- The API call is **CreateExportTask**

- Not near-real time or real-time... use Logs Subscriptions instead

### CloudWatch Log Subscriptions

- Get a real-time log events from CloudWatch Logs for processing and analysis
- Send to Kinesis Data Streams, Firehose, or Lambda
- **Subscription Filter** - filter logs and events, and deliver to a destination

- **Cross-Account Subscription** - send log events to resources in different AWS account (KDS, KDF)

# CloudWatch Agent & CloudWatch Logs Agent

## CloudWatch Logs for EC2

- By default, no logs from your EC2 machine will go to CloudWatch
- You need to run a CloudWatch agent on EC2 to push the logs files you want
- Make sure IAM permissions are correct
- The CloudWatch log agent can be setup on-premises too

## CloudWatch Logs Agent & Unified Agent

- For virtual servers 
- **CloudWatch Logs Agent**:
    - Old version of the agent
    - Can only send the CloudWatch logs

- **CloudWatch Unified Agent**:
    - Collection additional system-level metrics such as RAM, processes, etc..
    - Collect logs to send to CloudWatch Logs
    - Centralized configuration using SSM Parameter Store

### CloudWatch Unified Agent - Metrics

- Collected directly on your Linux server / EC2 Instance ( individual metrics NOT on exam):
    - **CPU** - (active, guest, idle, system, user, steal)
    - **Disk Metrics / DiskIO** - (free, used, total) / (writes, reads, bytes, iops),
    - **RAM** - (free, inactive, used, total, cached)
    - **Netstat** - (number of TCP and UDP connections, net packets, bytes)
    - **Processes** - (total, dead bloqued, idle, running, sleep)
    - **Swap Space** - (free used, used %)

# CloudWatch Logs - Metric Filter

- CloudWatch Logs can use filter expressions
    - For ex, find a specific IP inside of a log
    - Or count occurrences of "ERROR" in your logs
    - Metric filters can be used to trigger alarms

- **Filters do not retoactively filter data. Filters only publish the metric data points for events that happen after the filter was created**
- Ability to specify up to 3 Dimensions for the Metric Filter (optional)

# CloudWatch Alarms

- used to trigger notifications for any metric
- Various options (sampling, %, min, max, etc...)
- Alarm States:
    - OK
    - INSUFFICIENT_DATA
    - ALARM
- Period: 
    - Length of time in seconds to evaluate metric
    - High resolution custom metrics: 10 sec, 30 sec, or multiples of 60 sec

### Alarms - Targets

- Stop, Terminate, Reboot, or Recover an EC2 Instance
- Trigger Auto Scaling Action
- Send notifications to SNS (from which you can do pretty much anything)

### Alarms - Composite Alarms

- CloudWatch Alarms are on a single metric
- **Composite Alarms are monitoring the states of multiple other alarms**
- AND and OR conditions
- Helpful to reduce "alarm noise" by creating complex composite alarms

### EC2 Instance Recovery

- *Status Check*:
    - Instance Status = check the EC2 VM
    - System Status = check the underlying hardware
- **Recovery** Same Private, Public, Elastic IP, metadata, placement group

### Alarm - Good to Know

- Alarms can be created based on CloudWatch Logs Metrics filters

- To test alarms and notifications, set the alarm state to Alarm the CLI
- **aws cloudwatch set-alarm-state --alarm-name "myalarm" --state-value ALARM --state-reason "testing purpose"**

# CloudWatch Synthetics

### CloudWatch Synthetics Canary

- Configurable script that monitor your APIs, URLs, Website...
- Reproduce what your customers do programmatically to find issues before customers are impacted
- Checks the availability and latency of your endpoints and can store load time data and screenshots of the UI
- Integration with CloudWatch Alarms
- Scripts written in Node.js or Python
- Programmatic access to a headless Google Chrome browser
- Can run once or on a regular schedule

***Canary Blueprints**
    - **Headbeat Monitor** - load URL, store screenshot and an HTTP archive file
    - **API Canary** - test basic basic read and write functions of REST APIs
    - **Broken Link Checker** - check all links inside the URL that you are testing
    - **Visual Monitoring** - compare a screenshot taken during a canary run with a baseline screenshot
    - **Canary Recorder** - used with CloudWatch Synthetics Recorder (record your actions on a website and automatically generates a script for that)
    - **GUI Workflow Builder** - verifies that actions can be taken on your webpage (e.g. test a webpage with a login form)

# Amazon EventBridge 

- Schedule: Cron jobs (scheduled scripts)
- Event Pattern: Event rules to react to a service doing something
- Trigger Lambda Functions, send SQS/SNS messages...

### EventBridge - Event Bus

- Amazon EventBridge IS the default "Event Bus", but you can use 3rd Party or Custom Event Busses (Zendesk, Datadog, etc..)
- Event buses can be accessed by other AWS accounts using Resource-based Policies
- You can **archive events** (all/filter) sent to an event bus (indefinitely or set period)
- Ability to **replay archived events**

### EventBridge - Schema Registry

- EventBridge can analyze the events in your bus and infer the **schema**
- The **SchemaRegistry** allows you to generate code for your application, that will know in advance how data is structured in the event bus
- Schema can be versioned

### EventBridge - Resource-based Policies

- Managed permissions for a specific Event Bus
- Example: allow/deny events from another AWS account or AWS region
- Use Case: aggregate all events from your AWS Organization in a single AWS account or AWS region

### EventBridge - Multi-account Aggregation

- Event Rule Targets can be Event Buses in other AWS Accounts
- To do multi-account aggregation, set up Event Rules in all secondary accounts to point to the Event Bus in the aggregating account

# X-Ray

### Problem

- Debugging in Production, the good old way:
    - test locally
    - add log statements everywhere
    - Re-deploy in production

- Log Formats differ across apps using CloudWatch and analytics is hard
- Debugging: monolith "easy", distributed services "hard"
- No common views of of your entire architecture

- Enter... AWS X-Ray

### Overview

- Visual analysis of our applications

- **advantages**
    - Troubleshooting performance (bottlenecks)
    - Understand dependencies in a microservice architecture
    - Pinpoint service issues
    - Review request behaviors
    - Find errors and exceptions
    - Are we meeting time SLA?
    - Identify users that are impacted

- **compatibility**
    - Lambda
    - Beanstalk
    - ECS
    - ELB
    - API Gateway
    - EC2 instances

### Tracing

- Tracing is an end to end way to follow a "request"
- Each component dealing with the request adds its own "trace"
- Tracing is made of segments (+ sub segments)
- Annotations can be added to traces to provide extra-info
- Ability to trace:
    - Every request
    - Sample request (ror example, as a % or a rate per minute)
- X-Ray Security:
    - IAM for auth
    - KMS for encryption at-rest

### How to Enable it? (EXAM)

- **Supported Languages**:
    - Java
    - Python
    - Go
    - Node.js
    - .NET

1. **MUST import the X-Ray SDK**
    - Very little code needed
    - the app SDK will then capture:
        - calls to AWS services
        - HTTP/HTTPS requests
        - Database Calls (RDS, DynamoDB)
        - Queue Calls (SQS)

2. **Install the X-Ray daemon or enable X-Ray AWS Integration**:
    - X-Ray daemon works as a low level UDP packet interceptor (Linux / Windows / Mac...)
    - AWS Lambda / other AWS services already run the X-Ray daemon for you
    - Each application must have the IAM rights to write data to X-Ray

### How X-Ray Works

- X-Ray service collects data from all the different services
- Service map is computed from all the segments and traces
- X-Ray is graphical, so even non-technical people can help troubleshoot

### Troubleshooting

- If X-Ray is not working in EC2:
    - Ensure the EC2 IAM Role has the proper permissions
    - Ensure the EC2 isntance is running the X-Ray Daemon

- To enable on AWS Lambda:
    - Ensure it has an IAM execution role with proper policy (AWSX-RayWriteOnlyAccess)
    - Ensure that X-Ray is imported in the code
    - Enable **Lambda X-Ray *Active Tracing***

## X-Ray: Instrumentation and Concepts

- **Instrumentation** means the measure of product's performance, diagnose errors, and to write trace info
- To instrument your app code, you use the **X-Ray SDK**
- Many SDK require only config changes
- You can modify your app code to customize and annotate the data that the SDK sends to X-Ray, using **interceptors, filters, handlers, middleware...**

### X-Ray - Concepts

- *Segments*: each app / service will send them
- *Subsegments*: if you need more details in your segment
- *Trace*: segments collected together to form an end-to-end trace
- *Sampling*: decrease the amount of requests sent to X-Ray, reduce cost
- *Annotations*: KeyValue pairs used to **index** traces and use with **filters**
- *Metadata*: Key Value pairs, **NOT indexed**, not used for searching

- The X-Ray daemon / agent has a config to send traces cross account:
    - make sure IAM permissions are correct - the agent will assume the role
    - This allows to have a central account for all your application tracing

### X-Ray - Sampling Rules

- With sampling rules, you control the amount of data that you record
- You can modify sampling rules without changing your code

- By default, the X-Ray SDK records the first request **each second (reservoir)**, and **five percent (rate)** of any additional requests
    - *reservoir* - ensures that at least one trace is recorded each second as long as the service is serving requests
    - *rate* = rate at which additional requests beyond the reservoir size are sampled

 **Custom Sampling Rules** - You can create your own rules with the *reservoir* and *rate*

# X-Ray APIs (used by X-Ray daemon)

### Write API

- **PutTraceSegments**: Uploads documents to X-Ray
- **PutTelemetryRecords**: Used by X-Ray daemon to upload telemetry
    - SegmentsReceivedCount
    - SegmentsRejectedCounts
    - BackendConnectionErrors
    - etc
- **GetSamplingRules**: Retrieve all sampling rules (to know what/when to send)
- GetSamplingTargets & GetSamplingStatisticSummaries: advanced
- **The X-Ray daemon needs to have an IAM policy authorizing the correct API calls**

### Read API

- **GetServiceGraph**: main graph
- **BatchGetTraces**: Retrieves a list of traces specified by an ID. 
- **GetTraceSummaries**: Retrieves IDs and annotations for traces available for a specified time frame using an optional filter. For Full Traces, pass the trace IDs to BatchGetTraces
- **GetTraceGraph**: Retrieves a service graph for one or more specific trace ID(s)

# X-Ray with Beanstalk

- AWS Beanstalk platforms include the X-Ray daemon 
- you can run the daemon by setting an option in the Beanstalk console or with a config file (in .ebextensions/xray-daemon.config)

- Make sure to give you instance profile the correct IAM permissions so that the daemon can function correctly
- Then make sure your app code is instrumented with the X-Ray SDK
- Note: the daemon is not provided for Multicontainer Docker

# X-Ray with ECS

- X-Ray Container as Daemon:
    - Run an X-Ray Daemon Container on each instance (X-Ray does not have to run inside each container)

- X-Ray Container as a "Side Car":
    - ECS Cluster
        - Run an X-Ray Container Daemon alongside each Container
    - Fargate Cluster
        - Run an X-Ray Container Daemon alongside each Container by specifying X-Ray configs in the Task Definition

# AWS Distro for OpenTelemetry

- Secure, production-ready AWS-supported distribution of the open-source project OpenTelemetry project
- Provides a single set of APIs, libraries, agents, and collector services
- Collects distributed traces and metrics from your apps
- Collects metadata from your AWS resources and services
- **Auto-instrumentation Agents** to collect traces without changing your code
- Send traces and metrics to multiple AWS services and partner solutions
    - X-Ray, CloudWatch, Prometheus...
- Instrument your apps running on AWS as well as on-premises
- **Migrate from X-Ray to AWS Distro for OpenTelemetry if you want to standardize with open-source APIs from Telemetry or send traces to multiple destinations simultaneously**

- AWS CloudTrail

- **Provides governance, compliance and audit for your Account**
- CloudTrail is enabled by default
- Get a **history of events / API calls made within your Account** by:
    - Console
    - SDK
    - CLI
    - AWS Services
- Can put logs from CloudTrail into CloudWatch Logs or S3
- **A trail can be applied to All Regions (default) or a single Region**
- If a resource is deleted in AWS, investigate CloudTrail first!

### CloudTrail - Events

- **Management Events:**
    - Operations that are performed on resources in your Account
    - Ex:
        - Configuring Security 
        - Configuring rules for routing data
        - Setting up logging
    - **By default, trails are configured to log management events**
    - Can separate **Read Events** from **Write Events**

- **Data Events:**
    - **By default, data events are not logged (because high volume operations)**
    - S3 object-level activity: can separate Read / Writes
    - Lambda function execution activity

- **CloudTrail Insights Events:**
    - **Detects unusual activity** in your account
        - inaccurate resource provisioning
        - hitting service limits
        - Bursts of IAM actions
        - Gaps in periodic maintenance activity
    - Insights analyzes normal management events to create a baseline
    - And then **continuously analyze *write* events to detect unusual patterns**
        - Anomalies appear in CloudTrail Console
        - Event is sent to S3
        - EventBridge event is generated (for automation needs)

#### Events Retention

- Events are stored for 90 days
- To keep events beyond this period, log them to S3 and use Athena

# CloudTrail - EventBridge Integraion

- CloudTrail sends events to EventBridge and you create an EventBridge Rule to trigger automations

# CloudTrail vs CloudWatch vs X-Ray

- **CloudTrail**
    - Audit API calls made by users / services / AWS console
    - Useful to detect / audit unauthorized calls or root cause of changes

- **CloudWatch**:
    - Metrics over time for monitoring
    - Logs for storing application logs
    - Alarms to send notifications in case of unexpected metrics

- **X-Ray**:
    - Automated Trace Analysis & Central Service map Visualization
    - Latency, Errors, and Fault Analysis
    - Requests tracking across distributed systems