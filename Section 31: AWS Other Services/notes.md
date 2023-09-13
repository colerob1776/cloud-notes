# AWS SES - Simple Email Service

- Send emails to people using:
    - SMTP interface
    - Or AWS SDK

- Ability to receive email. Integrates with:
    - S3
    - SNS
    - Lambda

- Integrated with IAM for allowing to send emails

# AWS OpenSearch Service - Overview

- **Amazon OpenSearch is successor to Amazon ElasticSearch**
- In DynamoDB, queries only exist by primary key or indeces...
- **With OpenSearch, you can search any field, even partial matches**
- It's common to use OpenSearch as a complement to another DB
- 2 modes: managed cluster or serverless cluster
- Does *not* natively support SQL (can be enable via plugin)
- Ingestion from Kinesis, IoT and CW Logs
- Security through Cognito & IAM, KMS encryption, TLS
- Comes with OpenSearch Dashboards (visualizations)

# AWS Athena

- **Serverless** query service to analyze data stored in S3
- Uses standard SQL to query files (built on Presto)
- Supports CSV, JSON, ORC, Avro & Parquet
- Pricing: $5.00 / TB of data scanned
- Commonly used with Amazon Quicksight for reporting / dashboards

- **use cases**:
    - Business intel
    - analytics
    - reporting
    - analyze & query VPC flow logs
    - ELB logs
    - **CloudTrail trails**
    - etc...
- ***Exam Tip***: analyze data in S3 using serverless SQL, use Athena

### Athena - Performance Improvements

- Use **columnar data** for cost-savings (less scans)
    - Apache Parquet or ORC is recommended
    - Huge performance improvement
    - Use Glue to convert your data to Parquet or ORC
- **Compress data** for smaller retrievals (bzip2, gzip, iz4, snappy, zlip, zstd...)
- **Partition** datasets in S3 for easy querying on virtual columns
- **Use larger files** (> 128MB) to minimize overhead

### Athena - Federated Query

- Allows you to run SQL queries across data stored in relational, non-relational, object, and custom data sources (AWS or on-premise)
- Uses Data Source Connectors that run on AWS Lambda to run Federated Queries (eg. CW Logs, DynamoDB, RDS...)
- Store results back in S3 

# Amazon MSK - Overview

- Managed Streaming for Apache Kafka
- Alternative to Kinesis
- Fully managed Kafka cluster on AWS
    - Allow you to Create/Update/Delete clusters
    - MSK creates & managed Kafka broker nodes & Zookeeper nodes for you
    - Deploy the MSK cluster in your VPC, multi-AZ (up to 3 for HA)
    - Automatic recovery from common Apache Kafka failures
    - Data is stored on EBS volumes **for as long as you want**

- **MSK Serverless**
    - Run Apache Kafka on MSK without managing the capacity
    - MSK automatically provisions resources and scales compute & storage

### Kinesis Data Streams vs Amazon MSK

- **Kinesis**
    - 1 MB message size limit
    - Data Streams with Shards
    - Shard Splitting & Merging
    - TLS in-flight encryption
    - KMS at-rest encryption

- **Amazon MSK**:
    - 1 MB default, configure for higher (ex. 10 MB)
    - Kafka Topics with Partitions
    - Can only add partitions to a topic
    - PLAINTEXT or TLS in-flight Encryption
    - KMS at-rest encryption

# Amazon Certificate Manager (ACM)

- Let's you provision, manage, and deploy **SSL/TLS Certificates**
- Used to provide in-flight encryption for websites (HTTPS)
- Supports both public and private TLS certs
- Free for public TLS certs
- Automatic TLS cert renewal
- Integrations with (load TLS certs on):
    - ELBs
    - CloudFront Distributions
    - APIs on API Gateway

# ACM Private Certificate Authority - Overview

- Managed service allows you to create private CAs, including root and subordinate CAs
- Can issue and deploy end-entity X.509 certs
- Certs are trusted only by your Organization (not public internet)
- Works for AWS services that are integrated with ACM
- Use case:
    - Encrypted TLS communicatio, Cryptographically signing code
    - Auth users, computers, API endpoints, and IoT devices
    - Enterprise customers building a Public Key Infrastructure (PKI)

# Amazon Macie

- fully managed data security and data privacy service that uses **ML and pattern matching to discover and protect your sensitive data in AWS**
- helps identify and alert you to **sensitive data, such as personally identifiable info (PII)**
- one-click to activate

# AWS AppConfig - Overview

- Configure, validate and deploy dynamic configurations
- Deploy dynamic configuration changes to your apps independently of any code deployments
    - you dont need to restart the app
- Feature flags, application tuning, allow/block listing...
- use with apps on EC2, Lambda, ECS, EKS...
- Gradually deploy the config changes and rollback if issues occur
- Validate config changes before deployment using:
    - **JSON Schema**
    - **Lambda Function** - run code to perform validation

# CloudWatch Evidently

- Safely validate new features by serving them to a specified % of users
    - reduce risk and identify unintended consequences
    - Collect experiment data, analyze using stats, monitor performance
- **Launches** (= feature flags): enable / disable features for a subset of users
- **Experiments** (= A/B testing): compare multiple versions of the same feature
- **Overrides**: pre-define a variation for a specific 
- Store evaluation events in CW Logs or S3