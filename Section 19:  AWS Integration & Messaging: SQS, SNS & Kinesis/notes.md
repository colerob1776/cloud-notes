# Intro to Messaging

- There are 2 patterns for application communication:
    1. Synchronous Communications (service to service)
        - Synchronous between apps can be problematic if there are sudden spikes in traffic
    2. Asynchronous Communications (Event based / Queues)

- What if you suddenly need to encode 1000 videos but it's usually 10
- It's better to **decouple** your applications:
    - using **SQS**: queue model
    - using **SNS**: pub/sub model
    - using **Kinesis**: real-time streaming model
- These services can scale independently of our application

# SQS

## What is a Queue

- 3 Parties:
    - **Producers** - sends messages to the Queue
    - **Queue** - temporarily stores messages 
    - **Consumers** - polls Queue to retrieve the latest messages 

## SQS - Simple Queue Service

- Oldest offering (10 years)
- Fully managed service, use to **decouple applications**
- Attributes:
    - Unlimited throughput, unlimited number of messages in queue
    - Default retentions of messages: 4 days, maximim of 14 days
    - Low latency ( <10ms on pulish and receive> )
    - Limitation of 256KB per message sent
- Can have duplicate messages (at least once delivery, occasionally)
- Can have out of order messages (best effort ordering)

### SQS - Producing Messages

- Produced to SQS using the SDK (SendMessage API)
- The message is **persisted** in SQS until a consumer deletes it
- Message retention: default 4 days, up to 14 days
- SQS standard: unlimited throughput

### SQS - Consuming Messages

- Consumers (any server or process)
- Poll SQS for messages (receive up to 10 messages at a time)
- Processes the messages ( ex: insert message into DB )
- Delete the messages using the DeleteMessage API

- **Multiple Consumers**
    - Consumers receive and process messages in parallel
    - At least once delivery
    - Best-effort message ordering
    - Consumers delete messages after processing them
    - We can scale consumers horizontally to improve throughput of processing (with ASG)

- **Auto Scaling Consumers with ASG**
    - create CloudWatch Metric on *SQS QueueLength*
    - set ASG to reference CloudWatch Metric for scaling EC2 Consumers

### SQS - Security

- **Encryption**
    - in-flight using HTTPS API
    - at-rest using KMS keys
    - client-side if the client wants to perform encrypt/decrypt

- **Access Controls**: IAM policies to regulate access to the SQS API

- **SQS Access Policies** (similar to S3 bucket policies)
    - Useful for cross-account access to SQS queues
    - Useful for allowing other services (SNS, S3, ...) to write to an SQS queue

# SQS Queue Access Policy (EXAM)

- **Cross Account Access** - allow resource in one account to READ from SQS in another account
- **Publish S3 Event Notifications to SQS Queue** - allow ARN (policy condition) from S3 bucket to WRITE to SQS queue

# SQS - Message Visibility Timeout

- After a message is polled by a consumer, it becomse **invisible** to other consumers
- By default, the "message visibility timeout" is **30 seconds**
- That means the message has 30 seconds to be processed
- After the message visibility timeout is over, the message is "visible" in the SQS queue

- If a message is not processed within the visibility timeout, it will be processed **twice**
- A consumer could call the **ChangeMessageVisibility API** to get more time
- If visibility timeout is high (hours), and consumer crashes, re-processing will take time
- If visibility timeout is too low (seconds), we may get duplicates

# SQS - Dead Letter Queue

- If a consumer fails to process a message within the Visibility Timeout, the message goes back to the queue
- We can set a threshold of how many times a message can go back to the queue
- After a **MaximumReceives** threshold is exceeded, the message goes into a dead letter queue (DLQ)
- Useful for *debugging*
- **DLQ of a FIFO queue must also be a FIFO queue**
- **DLQ of a Standard queue must also be a Standard queue**
- Make sure to process the messages in th DLQ before they expire:
    - Good to set a retention of 14 days in the DLQ

### DLQ - Redrive to Source

- Feature to help consume messages in the DLQ to understand what is wrong with them
- When our code is fixed, we can redrive the messages from the SLQ back into the source queue (or any other queue) in batches without writing custom code

# SQS - Delay Queues

- Delay a message (consumers don't see it immediately) up to 15 minutes
- default is 0 seconds
- Can set a default at queue level
- Can override the default on send using the **DelaySeconds** parameter

# SQS - Long Polling

- When a consumer requests messages from the queue, it can optionally "wait" for messages to arrive if there are none in the queue
- This is called **Long Polling**
- **LongPolling decreases the number of API calls made to SQS while increasing the efficiency and latency of your application**
- The wait time can be between 1 sec to 20 sec (20 sec preferable)
- Long Polling is prefereable to Short Polling
- Long Polling can be enabled at the queue level or at the API level using **ReceiveMessageWaitTimeSeconds**

# SQS - Extended Client

- Message size limit is 256KB
- Using the SQS Extended Client library, we can send larger messages
    - the large message is stored in S3 and a small metadata message about the S3 file is piped through the SQS queue

# SQS - Must know API

- **CreateQueue (MessageRetentionPeriod)**
- **DeleteQueue**
- **PurgeQueue**: delete all messages
- **SendMessage (DelaySeconds)**
- **ReceiveMessage**
    - **MaxNumberOfMessages** (parameter): default 1, max 10 
- **DeleteMessage**
- **ReceiveMessageWaitTimeSeconds**: Long Polling
- **ChangeMessageVisibility**: change the message timeout

- Batch APIs for **SendMessage**, **DeleteMessage**, **ChangeMessageVisibility** helps decrease you cost

# SQS - FIFO Queues

- First-in-First-out (ordering of messages in the queue)
- Limited throughput: 300 msg/s without batching, 3000 msg/s with batching
- Exactly-once send capability (by removing duplicates)
- Messages are processed in order by the consumer

## SQS FIFO - Deduplication

- De-duplication interval is 5 minutes
- 2 de-duplication methods:
    - Content-based deduplication :will do a SHA-256 hash of the message body
    - Explicitly provide a Message Deduplication ID

## SQS FIFO - Message Grouping

- If you send the same value of **MessageGroupID** in an SQS FIFO queue, you can only have one consumer, and all the messages are in order
- To get ordering at the level of a subset of messages, specify different values for **MessageGroupID**:
    - Messages that share a common MessageGroupID will be in order within the group
    - Each Group ID can have a different consumer (parallel processing)
    - Ordering across groups is not guaranteed

# Amazon SNS

- What if you want to send one message to many receivers?
- Pub/Sub

- The "event producer" only sends messages to one SNS topic
- As many "event receivers" as we want to listen to the SNS topic notifications
- Each subscriber to the topic will get all the messages (note: new feature to filter messages)
- Up to 12,500,000 subscriptions per topic
- 100,000 topics limit

### How to Publish

- Topic Publish (using the SDK)
    - Create a topic
    - Create a subscription (or many)
    - Publish the topic

- Direct Publish (for mobile apps SDK)
    - Create a platform app
    - create a platform endpoint
    - publish to the platform endpoint
    - Works with Google GCM, Apple APNS, Amazon ADM...

### SNS - Security

- **Encryption**:
    - In-flight using HTTPS API
    - At-rest using KMS keys
    - Client-side

- **Access-Control**: IAM policies to regulate access to the SNS API

- **SNS Access Policies** (similar to S3 bucket policies):
    - Useful for cross-account access to SNS Topics
    - Useful for allowing other services (S3...) to write to an SNS topic

## Amazon SNS + SQS - Fan Out Pattern

- Push once in SNS, receive in all SQS queues that are subscribers
- Fully decoupled, no data loss
- SQS allows for: data persistence, delayed processing and retries of work
- Ability to add more SQS subscribers over time
- Make sure your SQS queue **access policy** allows for SNS to write
- Cross-Region Delivery: works with SQS queues in other regions

### Fan Out Applications - S3 Events to multiple queues

- For the same comination of: **event type** and **prefix**, you can only have *one S3 Event rule*
- If you want to send the same S3 event to multiple queues, use "Fan Out"

### Fan Out Applications - SNS to Amazon S3 through Kinesis Data Firehose

- SNS can send to Kinesis and therefore we can have the following solutions architecture

### Fan Out Applications - FIFO Topic

- First in, First out
- **In case you need fan out + ordering + deduplication**
- Similar features as SQS FIFO:
    - **Ordering** by Message Group ID 
    - **Deduplication** using a Deduplication ID or Content Based Deduplication

- *Can only have SQS FIFO qeueus as subscribers*
- Limited throughput (same throughput as SQS FIFO)

### Fan Out Applications - Message Filtering

- JSON policy used to filter messages sent to the SNS topic's subscriptions
- If a subscription doesn't have a filter policy, it receives every message

# Amazon Kinesis

- Make it easy to **collect**, **process**, and **analyze** streaming data in real-time
- Ingest real-time data such as: Application logs, Metrics, Website clickstreams, IoT telemetry data

- **Kinesis Data Streams**: capture, process, and store data streams
- **Kinesis Firehose**: load data streams into AWS data stores
- **Kinesis Data Analytics**: analyze data streams with SQL or Apache Flink
- **Kinesis Video Streams**: capture, proces, and store video streams

## Kinesis Data Streams

- retention between 1 day to 365 days
- Ability to reprocess (replay) data
- Once data is inserted in Kinesis, it can't be deleted (immutability)
- Data that shares the same partition goes to the same shared (ordering)
- Producers: AWS SDK, Kinesis Producer Library (KPL), Kinesis Agent 
- Consumers: 
    - Write your own: Kinesis Client Library (KCL), AWS SDK
    - Managed: AWS Lambda, Kinesis Data Firehose, Kinesis Data Analytics

### Kinesis Data Streams - Capacity Modes

- **Provisioned Mode**:
    - you choose the number of shards provisioned, scale manually or using API
    - Each shard gets 1MB/s in (or 1000 records/s)
    - Each shard gets 2MB/s out (classic or enhanced fan-out consumer)
    - You pay per shard provisioned per hour

- **On-demand mode**:
    - No need to provision or manage the capacity
    - Default capacity provisioned(4MB/s in or 4000 records/s)
    - Scales automatically based on observed throughput peak during the last 30 days
    - Pay per stream per hour & data in/out per GB

### Kinesis Data Streams - Security

- Control access / authorization using IAM policies
- in-flight, at-rest and client site **encryption**
- VPC endpoints available for Kinesis to access within VPC
- Monitor API calls using CloudTrail

## Kinesis Producers

- Puts data records into data streams
- Data record consist of:
    - Sequence number (unique per partition-key)
    - partition key (must specify while put records into stream)
    - Data blob (up to 1 MB)
- Producers:
    - **AWS SDK**: simple
    - **Kinesis Producer Library (KPL)**: C++, Java, batch, compression, retries
    - **Kinesis Agent**: monitor log files
- PutRecord API
- Use batching with PutRecords API to reduce cost & increase throughput

### Kinesis Error - ProvisionedThroughputExceeded

- **Solution**:
    - Use highly distributed partition key
    - Retries with exponential backoff
    - Increase shards (scaling)

## Kinesis Consumers

- Get data from stream and process them

- AWS Lambda
- Kinesis Data Analytics
- Kinesis Data Firehose
- Custom Consumer (AWS SDK) -  Classic or Enhanced Fan-Out
- Kinesis Client Library (KCL): library to simplify reading from data stream

### Kinesis Consumers - Custom Consumer

- **Shared (Classic) Fan-out Consumer**:
    - 2 MB/s per shard across all consumers
    - GetRecords()
    - *Pull Model*
        - Low number of consuming applications
        - Read throughput: 2 MB/s per shard across all consumers
        - Max. 5 GetRecords API calls/s
        - Minimize Cost ($)
        - Consumer polls data from Kinesis using GetRecords API call
        - Returns up to 10 MB (then throttle for 5 seconds) or up to 10,000 records

**Enhanced Fan-out Consumer**:
    - 2 MB/sec per consumer per shard
    - SubscribeToShard()\
    - *Push Model*:
        - Multiple consuming applications for the same stream
        - 2 MB/s per consumer per shard
        - Latency ~70 ms
        - Kinesis pushes data to consumers over HTTP/2 (SubscribeToShard API)
        - Soft limit of 5 consumer applications (KCL) per data stream (default)

### Kinesis Consumers - AWS Lambda

- Supports Classic & Enhanced fan-out consumers
- Read records in batches
- Can configure **batch size** and **batch window**
- If error occurs, Lambda retries until succeeds or data expired
- Can process up to 10 batches per shard simultaneously

## Kinesis Client Library (KCL)

- A java library that helps read records from a Kinesis Data Stream with distributed applications sharing the read workload
- Each shard is to be read by only one KCL instance
    - 4 shards = max. 4 KCL instances
    - 6 shards = max. 6 KCL instances
- Progress is checkpointed into DynamoDB (needs IAM access)
- Track other workers and share the work amongst shards using DynamoDB
- KCL can run on EC2, Elastic Beanstalk, and on-premises
- Records are read in order at the shard level
- Versions:
    - KCL 1.x (supports shared consumer)
    - KCL 2.x (supports shared & enchanced fan-out consumer)

## Kinesis Operations

### Shard Splitting

- Used to increase the Stream capacity (1 MB/s in per shard)
- Used to divide a "hot shard"
- The old shard is closed and will be deleted once the data is expired
- No automatic scaling (manually increase/decrease capacity)
- Can't split into more than two shards in a single operation

### Merging Shards

- Decrease the Stream capacity and save costs
- Can be used to group two shards with low traffic (cold shards)
- Old shards ar closed and will be deleted once the data is expired
- Can't merge more than 2 shards in a single operation

# Kinesis Data Firehose

- Similar to Kinesis Data Streams but allows for data transformations with Lambda and Firehose handles writes (no Consumers)
- Fully Managed Service, no administration, automatic scaling, serverless
- Pay for data going through Firehose
- **Near Real Time**:
    - 60 seconds latency minimum for non full batches
    - Or minimum 1 MB of data at a time
- Supports many data formats, conversions, transformations, compression
- Supports custom data transformations using AWS Lambda
- Can send failed or all data to a backup S3 bucket

**AWS Destinations**:
    - S3
    - Redshift (COPY through S3)
    - OpenSearch
**3rd Party Destinations**:
    - Datadog
    - Splunk
    - New Relic
    - MongoDB
**CustomDestinations**:
    - HTTP Endpoints

### Kinesis Data Streams vs Firehose

**Kinesis Data Streams**:
    - Streaming service for ingest at scale
    - Write custom code (producer/consumer)
    - Real-time (~200ms)
    - Managed Scaling (shard splitting / merging)
    - Data storage for 1 to 365 days
    - Supports replay capability

**Kinesis Firehose**:
    - Load streaming data into other services
    - Fully managed
    - Near real-time (buffer time 60 secs)
    - Automatic scaling
    - No data storage
    - Doesn't support replay capability

# Kinesis Data Analytics

## For SQL Applications

- Real-time analytics on **Kinesis Data Streams & Firehose** using SQL
- Add reference data from S3 to enrich streaming data
- Fully managed, no servers to provision
- Auto scaling
- Pay for actual consumption rate
- Output: 
    - Kinesis Data Streams: create streams out of the real-time analytics queries
    - Kinesis Data Firehose: send analytics query results to destinations
- Use Cases:
    - Time-series analytics
    - Real-time dashboards
    - Real-time metrics

## For Apachy Flink

- Use Flink (Java, Scala or SQL) to process and analyze streaming data
- Run any Apache Flink application on a managed cluster on AWS
    - Provisioning compute resources, parallel computation, auto scaling
    - Application backupts ( implemented as checkpoints and snapshots )
    - Use any Apache Flink programming features
    - Flink does not read from Firehose ( use Kinesis Analytics for SQL instead )

# Data Ordering for Kinesis vs SQS FIFO

- Imagine you have 100 trucks on the road sending their GPS positions regularly into AWS
- You want to consume the data in order for each truck, so that you can track their movement accurately
- **Q**: How should you send that data into Kinesis?

**Kinesis**
    - **A**: send using a "Partition Key" value of the "truck_id". The same key will always go to the same shard

**SQS FIFO**
    - **A**: send using a "Group ID" value of the "truck_id". Messages of the same group persist ordering

# SQS vs SNS vs Kinesis

**SQS**:
    - consumer "pulls data"
    - Data is deleted after being consumed
    - Can have as many workers as we want
    - No need to provision throughput
    - Ordering guaranteed only with FIFO queues
    - Individually message delay capability

**SNS**
    - Push data to many subscribers (PubSub)
    - Up to 12.5 Million subscribers
    - Data is not persisted 
    - Pub/Sub
    - Up to 100k topics
    - No need to provision throughput
    - Integrates with SQS for fan-out architecture pattern
    - FIFO capability for SQS FIFO

**Kinesis**
    - Standard: pull data (2 MB per shard)
    - Enhanced-fan out: push data (2 MB per shard per consumer)
    - Possibility to replay data
    - Meant for real-time big data, analytics and ETL
    - Ordering at the shard level
    - Data expires after X days
    - Provisioned mode or on-demand capacity mode