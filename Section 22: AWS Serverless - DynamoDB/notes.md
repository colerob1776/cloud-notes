# DynamoDB Overview

- NoSQL serverless database
    - non-relational and **distributed**
    - do not support query joins (or just limited support)
    - All the data that is needed for a query is present in one document
    - NoSQL DBs don't perform aggregations such as "SUM", "AVG", ...
    - **Scale horizontally**

- Fully Managed, highly available, with replication across multiple AZs
- Scales to massive workloads, distributed database
- Millions of requests per second, trillions of rows, 100s of TBs of storage
- Fast and consistent in performance (low latency on retrieval)
- Integrated with IAM for security, auth, and administration
- Enables event driven programming with DynamoDB Streams
- Low cost and auto-scaling capabilities
- Standard & Infrequent Access Table Class

### DynamoDB - Basics

- DynamoDB is made of **Tables**
- Each table has a **Primary Key** (must be decided at creation time)
- Each table can have an infinited number of items (=rows)
- Each item has **attributes** (can be added over time - can be null)
- Maximum size of an item is 400 KB
- Data types supported are:
    - **Scalar Types** - String, Number, Binary, Boolean, Null
    - **Document Types** - List, Map
    - **Set Types** - String Set, Number Set, Binary Set

### DynamoDB - Primary Keys

- **Option 1: Partition Key (HASH)**:
    - Partition key must be unique for each item
    - Partition key must be "diverse" so that the data is distributed
    - Ex: "User_ID" for a users table

- **Option 2: Partition Key + Sort Key (HASH + RANGE)**:
    - The combination must be unique for each item
    - Data is grouped by partition key
    - Ex: users-games table, "User_ID" for partition key and "Game_ID" for Sort Key

- ***Exam Question***:
    - **Q**:
        - You are building a movie database. What is the best Partition Key to maximize data distribution?
            - movie_id
            - producer_name
            - lead_actor_name
            - movie_language
    - **A**:
        - "movie_id" has hte highest cardinality so it's a good candidate
        

# DynamoDB WCU & RCU - Throughput

### Read/Write Capacity Modes

- Control how you manage your table's capacity (read/write throughput)

- **Provisioned Mode (default)**:
    - You specify the number of reads/write per second
    - You need to plan capacity beforehand
    - Pay for *provisioned* read & write capacity units

- **On-Demand Mode**:
    - Read/writes automatically scale up/down with your workloads
    - No capacity planning needed
    - Pay for what you use, more expensive ($$$)

- You can switch between different modes once every 24 hours

### Write Capacity Units (WCU)

- One *Write Capacity Unit (WCU)* represents one write per second for an item up to **1 KB** in size

- **Example 1**: we write 10 items per second, with item size 2 KB
    - **20 WCUs**

- **Example 2**: we write 6 items per second, with item size of 4.5 KB
    - **30 WCUs*

### READ Capacity Units (RCU)

- One *Read Capacity Unit (RCU)* represents **one Strongly Consistent Read** per second or **two Eventually Consistent Reads** per second, for an item up to **4 KB** in size
- if the items are larger than 4 KB, more RCUs are consumed

- **Strongly Consistent Read Mode**
    - if we read just after write, we get the correct data
    - Set "**ConsistentRead**" parameter to **True** in API calls (GetItem, BatchGetItem, Query, Scan)
    - **Consumes Twice the RCU**

- **Eventually Consistent Read Mode (default)**
    - if we read just after a write, it's possible we'll get some stale data because of replication

### Partitions Internal

- Data is stored in partitions
- Partitions Keys go through a hashing algorithm to know which partition to go to
- **WCUs and RCUs are spread evently across partitions**

### Throttling

- if we exceed provisioned RCUs or WCUs, we get "**ProvisionedThroughputExceededExceptions**"
- Reasons:
    - **Hot Keys** - one partition key is being read too many times
    - **Hot Partitions** 
    - **Very large items**
- Solutions:
    - **Exponential backoff** when exception is encountered (already in SDK)
    - **Distribute partition keys** as much as possible
    - if RCU issue, we can **use DynamoDB Accelerator (DAX)**

### R/W Capacity Modes - On-Demand

- Read/writes automatically scale up/down with your workloads
- no capacity planning needed (RCU/WCU)
- Unlimited WCU & RCU, no throttle, more expensive
- You're charged for reads/writes that you use in terms of **RRU** and **WRU**
- **Read Request Units (RRU)** - throughput for reads (same as RCU)
- **Write Request Units (WRU)** - throughput for writes (same as WCU)
- 2.5x more expensive than provisioned capacity (use with care)
- Use cases: unknown workloads, unpredicable application traffic, ...

# DynamoDB Basic Operations

### Writing Data

- **PutItem** 
    - Creates a new item or fully replaces an old item (same PK)
    - Consumes WCUs

- **UpdateItem**
    - Edits an existing item's attributes or adds a new item if it doesn't exist
    - Can be used to implement **Atomic Counters** - a numeric attribute that's unconditionally incremented

- **Conditional Writes**
    - Accept a write/update/delete only if the conditions are met, otherwise returns an error
    - Helps with concurrent access to items
    - No performance impact

### Reading Data

- **GetItem**
    - Read based on Primary Key
    - Primary Key can be **HASH** or **HASH+RANGE**
    - Eventually Consistent Read (default)
    - Option to use Strongly Consistent Reads (more RCU - might take longer)
    - **ProjectionExpression** can be specified to retrieve only certain attributes

- **Query**
    - **KeyConditionExpression**
        - Partition Key value (**must be = operator**) - required
        - Sort Key value (=, <, <=, >, >=, Between, Begins with) - optional
    - **FilterExpression**
        - Additional filtering after the Query operation (before data returned to you)
        - Use only with non-key attributes (does not allow HASH or RANGE attributes)
    - Returns:
        - The number of items specified in **Limit**
        - Or up to 1 MB of data
    - Ability to do pagination on the results
    - Can query table, a Local Secondary Index, or a Global Secondary Index

- **Scan**
    - The entire table and then filter out data (inefficient)
    - Returns up to 1 MB of data - use pagination to keep on reading
    - Consumes a lot of RCU
    - Limit impact using **Limit** or reduce the size of the result and pause
    - For faster performance, use **Parallel Scan**
        - Multiple workers scan multiple data segments at the same time
        - Increases the throughput and RCU consumed
        - Limit the impact of parallel scans just like you would for Scans
    - Can use **ProjectionExpression** & **FilterExpression** (no changes to RCU)

### Deleting Data

- **DeleteItem**
    - Delete an individual item
    - Ability to perform a conditional delete

- **DeleteTable**
    - Delete whole table and all its items
    - Much quicker deleting than calling **DeleteItem** on all items

### Batch Operations

- Allows you to save in latency by reducing the number of API calls
- Operations are done in parallel for better efficiency
- Part of a batch can fail; in which case we need to try again for the failed items

- **BatchWriteItem**
    - Up to 25 **PutItem** and/or **DeleteItem** in one call
    - Up to 16 MB of data written, up to 400 KB of data per item
    - Can't update items
    - ***UnprocessedItems*** for failed write operations (exponential backoff or add WCU)

- **BatchGetItem**
    - Return items from one or more talbes
    - up to 100 items, up to 16 MB of data
    - Items are retrieved in parallel to minimize latency
    - ***UnprocessedKeys*** for failed read operations (exponential backoff or add RCU)

### PartiQL

- SQL-compatible query language for DynamoDB
- Allows you to select, insert, update, and delete data in DynamoDB using SQL
- Run queries across multiple DynamoDB tables
- Run PartiQL queries from:
    - Management Console
    - NoSQL Workbench for DynamoDB
    - DynamoDB APIs
    - AWS CLI
    - AWS SDK

# DynamoDB - Conditional Writes

- For **PutItem, UpdateItem, DeleteItem** and **BatchWriteItem**
- You can specify a Condition expression to determine which items should be modified:
    - **attribute_exists**
    - **attribute_not_exists**
    - **attribute_type**
    - **contains**
    - **begins_with**
    - ProductCategory **IN** (:cat1, :cat2) **and** Price **between** :low **and** :high
    - **size** (string length)
    
- **Note: Filter Expressions filter the results of read queries, while Condition Expressions are for write operations**

# DynamoDB - Indexes (GSI + LSI)

### Local Secondary Index (LSI)

- **Alternative Sort Key** for your table
- The Sort Key consists of one scalar attribute
- Up to 5 Local Secondary Indexes per table
- **Must be defined at table creation time**
- **Attribute Projections** - can contain some or all the attributes of the base table (**KEYS_ONLY, INCLUDE, ALL**)

### Global Secondary Indexes (GSI)

- **Alternative Primary Key (HASH or HASH+RANGE)** from the base table
- Speed up queries on non-key attributes
- The Index Key consists of scalar attributes 
- **Attribute Projections** - some or all the attributes of the base table (**KEYS_ONLY, INCLUDE, ALL**)
- Must provision RCUs & WCUs for the index
- **Can be added/modified after table creation**

### Indexes and Throttling (EXAM)

- Global Secondary Index:
    - **If the writes are throttled on the GSI, then the main table will be throttled**
    - Even if the WCU on the main tables are fine
    - Choose your GSI partition key carefully
    - Assign your WCU capacity carefully 

- Local Secondary Index:
    - Uses the WCUs and RCUs of the main table
    - No special throttling considerations

# DynamoDB - PartiQL

- Use a SQL-like syntax to manipulate DynamoDB tables
- Supports:
    - INSERT
    - UPDATE
    - SELECT
    - DELETE
- supports batch ops

# DynamoDB - Optimistic Locking

- Snapshot Isolation
- strategy to ensure an item hasnt changed before you update/delete it
- Each item has an attribute that acts as a *version number*

# DynamoDB Accelerator (DAX)

- Fully-managed, HA, seamless in-memory cache for DynamoDB
- Microseconds latency for cached reads & queries
- Doesnt require application logic modification
- Solves the "Hot Key" problem (too many reads)
- 5 minutes TTL for cache (default)
- up to 10 nodes in the Cluster
- Multi-AZ
- Secure (Encryption at rest, VPC, IAM, CloudTrail)

# DynamoDB Streams

- Ordered stream of item-level mods (CUD) in a table
- Stream records can be:
    - Sent to **Kinesis**
    - Read by **Lambda**
    - Read by **Kinesis**
- Data retention for up to 24 hours
- Use cases:
    - react to changes in real-time
    - Analytics
    - Insert derivative tables
    - Insert in to OpenSearch Service
    - Implement cross-region replication

- Ability to choose the info that will be written to the stream:
    - **KEYS_ONLY** - only the key attrs of the modified item
    - **NEW_IMAGE** - the entire item, as it appears after it was modified
    - **OLD_IMAGE** - the entire item, as it appeared before it was modified
    - **NEW_AND_OLD_IMAGES** - both new and old images of the item
- DynamoDB Streams are made of shards, just like Kinesis
- You don't provision shards, it is automated by AWS

- **Records are not retroactively populated in a stream after enableing it**

### DynamoDB Streams & Lambda

- You need to define an **Event Source Mapping** to read from a DynamoDB Stream
- You need to ensure the Lambda function has the **appropriate perms**
- **Your Lambda is invoked synchronously**

# DynamoDB TTL

- Auto delete items after an expiration timestamp
- Doesn't consume any WCUs
- The TTL attr must be a **Number** data type with a **Unix Epoch Timestamp** value
- Expired items deleted within 48 hours of expiration
- Expired items, that haven't been deleted, appears in reads/queries/scans (if you want to filter them out)
- Expired items are deleted from both LSIs and GSIs
- A deleted op for each expired item enters the DynamoDB Streams (can help recover expired items)
- Use cases:
    - reduce stored data
    - adhere to regulatory obligations


# DynamoDB CLI

- **--projection-expression**: one or more attrs to retrieve
- **--filter-expression**: filter items before returned to you

- General AWS CLI Pagination options
    - **--page-size**: specify that AWS CLI retrieves the full list of items but with a larger number of API calls instead of one API call (default: 1000 items)
    - **--max-items**: max number of items to show in the CLI (returns **NextToken**)
    - **--starting-token**: specify the last **NextToken** to retrieve the next set of items

# DynamoDB Transactions

- Coordinated, all-or-nothing ops to multiple items across one or more tables
- ACID
- **Read Modes** - Eventual Consistency, Strong Consistency, Transactional
- **Write Modes** - Standard, Transactional
- **Consumes 2x the WCUs & RCUs**
    - perform 2 ops for every item (prepare & commit)
- 2 operations:
    - **TransactGetItems** - one or more **GetItem** ops
    - **TransactWriteItems** - one or more **PutItem**, **UpdateItem**, and **DeleteItem** ops
- Use cases:
    - financial transactions
    - managing orders
    - multiplayer games

### Transactions - Capacity Computations (just multiply normal by 2)

- 3 TX writes per second with item size 5 KB (1 KB/WCU)
    - 30 WCUs

- 5 TX reads per second, with item size 8 KB (4 KB/RCU)
    - 20 RCUs

# DynamoDB - Session State

- it's common to use DynamoDB to store session state

- **vs ElastiCache**:
    - ElastiCache is in-memory, but DynamoDB is serverless
    - Both are key/value stores
- **vs EFS**:
    - EFS must be attached to EC2 instances as network drive
- **vs EBS & Instance Store**:
    - EBS & Instance Store can only be used for local caching, not shared caching
- **vs S3**:
    - S3 is higher latency, and not meant for small objects

# DynamoDB - Partitioning Strategies

### Write Sharding

- Imagine we have a voting app with two candidates, A and B
- If **Partition Key** is **"CandidateID"**, this results in two partitions, which will generate issues (eg Hot Partition)

- A strategy that allows better distribution of items evenly across partitions
- **Add a suffix to Partition Key value**
- Two Methods:
    - Sharding using Random Suffix
    - Sharding using Calculated Suffix (hashing)

# DynamoDB Conditional Writes, Concurrent Writes & Atomic Writes

### Write Types

- **Concurrent Writes** - subsequent writes overwrite the previous
- **Conditional Writes** - overwrite ONLY IF value equals a provided value (Snapshot Isolation)
- **Atomic Writes** - Values are Increased/Decreased (not overwritten)
- **Batch Writes** - Many writes at the same time

# DynamoDB Patterns with S3

### Large Objects Pattern

- DynamoDB contains fields for S3 Metadata that points to the S3 Object

### Indexing S3 Objects Metadata

1. Object gets uploaded to S3
2. CreateObject invokes Lambda
3. Lambda stores Object metadata to DynamoDB Table

# DynamoDB Operations

- **Table Cleanup**
    - **Option 1** - Scan + Delete Item (expensive and slow)
    - **Option 2** - Drop Table + Recreate Table (quick and cheap)

- **Copying a DynamoDB Table**
    - **Option 1** - Using AWS Data Pipeline
        - Amazon ECR Cluster that writes/reads to/from an S3 Bucket to replicate changes
    - **Option 2** - Backup and restore into a new table
        - takes some time but is more efficient and doesnt require other services
    - **Option 3** - Scan + PutItem or BatchWriteItem
        - Not recommended

# DynamoDB - Security & Other

- **Security**
    - VPC Endpoints available to access DynamoDB without using the Internet
    - Access fully controlled by IAM
    - Encryption at rest using KMS and in-transit using SSL/TLS

- **Backup and Restore feature available**
    - Point-in-time Recovery (PITR) like RDS
    - No performance impact

- **GlobalTables**
    - Multi-region, multi-active, fully replicated, high performance

- **DynamoDB Local**
    - Develop and test apps locally without accessing the DynamoDB web service (without Internet)

- AWS Database Migration Service (AWS DMS) can be used to migrate to DynamoDB (from MongoDB, Oracle, MySQL, S3, ...)

### Users Interact with DynamoDB Directly

- Use Identity Provider (Cognito, Google, OpenID Connect) for Web/Mobile clients to access DynamoDB records (only records they own)

- Using **Web Identity Federation** or **Cognito Identity Pools**, each user gets AWS creds
- You can assign an IAM Role to these users with a **Condition** to limit their API access to DynamoDB
- **LeadingKeys** - limit row-level access for users on the Primary Key
- **Attributes** - limit specific attrs the user can see

