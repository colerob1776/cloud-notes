# Amazon RDS Overview

- Relational Database Service
- Managed DB service for SQL DBs
- **Supported Databases** (Exam):
    - Postgres
    - MySQL
    - MariaDB
    - Oracle
    - SQL Server
    - Aurora (AWS Proprietary DB)

### Advantages over useing RDS vs deploying DB on EC2

- RDS is a managed service:
    - Automated provisioning, OS patching
    - Continuous backups and restore to specific timestamp (Point in Time Restore!)
    - Montoring dashboards
    - Read Replicas for improved read performance
    - Multi AZ setup for DR (disaster recovery)
    - Maintenance windows for updgrades
    - Scaling capability (vertical and horizonal)
    - Storage is backed by EBS (gp2 or io1)
- BUT you cant SSH into your instances

### RDS - Storage Auto Scaling (Exam)

- Helps you increase storage on your RDS DB instance dynamically
- When RDS detects you are running out of free storage, it scales automatically
- You need to set a **Maximum Storage Threshold** 
- Auto modify storage if:
    - Free storage is less than 10% of allocated storage
    - Low-storage lasts at least 5 minutes
    - 6 hours have passed since last modification
- Useful for apps with **unpredictable workloads**
- Supports all RDS database engines (MariaDB, MySQL, Postgres, Oracle, SQL Server)

# RDS Read Replicas for Read Scalability

- Up to 15 read replicas
- Within AZ, Cross AZ, or Cross Region
- Replication is **ASYNC**, so reads are eventually consistent
- Replicas can be promoted to their own DB (w/ writes)
- Applications must update the connection string to leverage read replicas

### Read Replica - Use Cases:

- You have a prod DB that is taking on normal load
- You want to run a reporting app to run analytics
- You create a RR to run the new workload there
- the prod app is unaffected
- RRs are used for SELECT only kind of statements (not INSERT, UPDATE, DELETE)

### Read Replica - Network Costs

- in AWS, there's a network cost when data goes from one AZ to another AZ
- For RDS RRs within the same region, you don't pay that fee

### RDS Multi AZ (Disaster Recovery)

- **SYNC** replication
- One DNS name - automatic app failover to standby DB
- Increase **availability**
- Failover in case of loss of AZ, loss of network, instance or storage failure
- No manual intervention in apps
- Not used for scaling

- **Note (Exam)**: The RRs can be setup as Multi AZ Failover for Disaster Recovery

### From Single-AZ to Multi-AZ (Exam)

- Zero downtime operation
- Just click on "modify" for the DB
- The following happens internally:
    - A new snapshot is taken
    - A new DB is restored from the snapshot in a new AZ
    - Synchronization is established between the 2 DBs

# Amazon Aurora

- proprietary DB from AWS (not open source)
- Postgres and MySQL are both supported as Aurora DB (that means your drivers will work as if Aurora was a Postgres or MySQL DB)
- Aurora is "AWS cloud optimized" and claims 5x performance improvement over MySQL on RDS, over 3x performance of Postgres on RDS
- Aurora storage automatically grows in increments of 10 GB, up to 128 TB.
- Aurora can have up to 15 replicas and the replication process is faster than MySQL (sub 10ms replica lag)
- Failover in Aurora is instantaneous. Its HA native
- Aurora costs more than RDS (20% more) - but it's much more efficient so makes more sense for savings at scale

### Aurora High Availability and Read Scaling
- 6 copies of your data across 3 AZs:
    - 4 copies out of 6 needed for writes
    - 3 copies out of 6 needed for reads
    - Self healing with peer-to-peer replication
    - Storage is striped across 100s of volumes
- One Aurora instance takes writes (master)
- Auto failover for master in less than 30 seconds
- Master + up to 15 Aurora Read Replicas serve reads (any RR can be converted to Master on failover)
- Support Cross Region Replication

### Aurora DB Cluster

- shared storage Volume (auto expanding 10GB to 128TB)
- **Connecting to Cluster (EXAM)**:
    - **Writer endpoint** for pointing to master
    - **Reader endpoint** for reads:
        - Note: The read endpoint, points to an underlying LB that distributes reads to the RRs

### Features of Aurora

- Auto fail-over
- backup and recovery
- isolation and security
- industry compliance
- Push-button scaling
- Auto Patching w/ Zero Downtime
- Advanced Monitoring
- Routine Maintenance
- Backtrack: restore data at any point in time without using backups

# RDS & Aurora Security

- **At-rest encryption**:
    - storage is encrypted
    - Database master & replicas encryption using AWS KMS - must be defined as launch time
    - if the master is not encrypted, the read replicas cannot be encrypted
    - To encrypt an un-encrypted database, go through the DB snapshot & restore as encrypted
- **In-flight encryption**: TLS-ready by default, use the AWS TLS root certificates client-side
- **IAM Authentication**: IAM roles to connect to your database (instead of username/pwd)
- **Security Groups**: Control Network access to your RDS/Aurora DB
- **No SSH Available**: exept on RDS Custom
- **Audit Logs can be enabled** and sent to CloudWatch Logs for longer retention

# RDS Proxy

- fully managed DB proxy for RDS
- **Allows apps to pool and share DB connections established with the DB**, good when there are many connections on a database such as Lambda functions
- **improves database efficiency by reducing stress on database resources (eg CPU, RAM) and minimize open connections (and timeouts)**
- Serverless, autoscaling, highly available (multi AZ)
- **Reduced RDS & Aurora failover time by up to 66%**
- Supports RDS (MySQL, Postgres, MariaDB, SQL Server), and Aurora
- No code changes required for most apps
- **Enforce IAM Auth for DB, and securely store creds in AWS Secret Manager
- **RDS Proxy is never publicly available (must be accessed from VPC)**
- Great for Lambda Functions!

# ElastiCache Overview

- managed Redis or Memcached
- in-memory databases with really high performance, low latency
- helps reduce load off of databases for read intensive workloads
- helps make your application stateless
- AWS takes care of OS maintenance/patching, optimizations, setup, configuration, monitoring, failure recovery and backups
- **Using ElastiCache involves heavy application code changes**

### Solution Architecture - DB Cache

- Application queries ElastiCache, if not available, get from RDS and store in ElastiCache
- Helps relieve load in RDS
- Cache must have an invalidation strategy to make sure the data is not outdated

### Solution Architecture - User Session Store

- User logs in to any of the application
- the app writes to the session data into ElastiCache
- The user hits another instance of our application
- The instance retrieves the data and the user is already logged in

### Redis vs Memcached

- **Redis**:
    - Multi AZ with Auto-Failover
    - RRs to scale reads and have high availability
    - Data Durability using AOF persistence
    - Backup and restore features
    - Supports Sets and Sorted Sets

- **Memcached**:
    - Multi-node for partitioning of data (sharding)
    - No High Availability (replication)
    - Non Persistent
    - No backup and restore
    - Multi-threaded architecture

# ElastiCache Strategies

### Caching Implementation Considerations

- Is it safe to cache data? Data may be out of date, eventually consistent
- Is caching effective for that data?
    - Pattern: data changing slowly, few keys are frequently needed
    - Anti Pattern: data changing rapidly, all large key space frequently
- Is data structured well for caching?
    - ex: key value caching, or caching of aggregation results

### Lazy Loading/Cache-Aside/Lazy Population (all same thing)

- check for cache hit (you find what you are looking for in the cache)
- if cache hit and it is not outdated, use it
- otherwise you have a cache miss
    - on cache miss, fetch from RDS
    - before using data, make sure to update cache

Pros: 
    - Only requestd data is cached (this cache isnt filled with unused data)
    - Node failures are not fatal (jsut increased latency to warm the cache)
Cons:
    - Cache miss penalty that results in 3 round trips, noticeable delay for that request
    - Stale data: data can be updated in the DB but outdated in the cache

### Write Through

- Add or Update cache when database is updated

Pros: 
    - data in cache is never stale, **reads are quick**
    - **Write Penalty** vs Read penalty (each write requires 2 calls)

Cons:
    - Missing Data until DB is added/updated. Mitigation is to implement Lazy Loading stratgy in tandem
    - Cache churn - a lot of the data will never be read (wasted RAM)

### Cache Evictions and Time-to-live (TTL)

- Cache eviction can occur in three ways:
    - you delete the item explicitely in the cache
    - item is evicted bc the memory is full and it is not recently used (LRU)
    - You set an item **time-to-live** (TTL)
- TTL are helpful for any kind of data:
    - Leaderboards
    - Comments
    - Activity Streams
- TTL can range from few seconds to hours or days

- if too many evictions happen due to memory, consider scaling (vertical or horizontal)

### Final words of wisdom

- Lazy Loading / Cache Aside is easy to implement and works for many situations as a foundation, especially on the read side
- Write-through is usually combined with Lazy Loading as targeted for the queries or workloads that benefit from this optimization
- Setting a TTL is usually not a bad idea, except when you're useing Write-through. Set it to a sensible value for your application
- Only cache the data that makes sense (user profiles, blogs, etc)

- *Quote: There are only two hard things in Computer Science: cache invalidation and naming things*

# Amazon MemoryDB for Redis

- Redis-compatible, **durable, in-memory database service**
- **Ultra-fast performance with over 160 million requests/second**
- Durable in-memory data storage with Multi-AZ transactional log
- Scale seamlessly from 10s GBs to 100s TBs of storage
- Use cases: web and mobile apps, online gaming, media streaming...
