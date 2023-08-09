
# S3 - Lifecycle Rules

- **Transition Actions** - configure objects to transition to another storage class:
    - Ex. Move objects to Standard IA after 60 days after creation

- **Expiration Actions** - configure objects to expire (delete) after some time
    - Access log files can be set to delete after 365 days
    - **Can be used to delete old versions of files**
    - Can be used to delete incomplete Multi-Part uploads

- Rules can be created for a certain prefix
- Rules can be created for certain object Tags

### S3 Analytics - Storage Class Analysis

- Help you decide when to transition objects to the right storage class
- Recommendations for **Standard** and **Standard IA** 
    - Does NOT work for One-Zone IA or Glacier
- Report is updated daily
- 24 to 48 hours to start seeing data analysis

# S3 Event Notifications

- S3:ObjectCreated, S3:ObjectRemoved, ...
- Object name filtering possible (*.jpg)
- Use Case: generate thumbnails of images uploaded to S3
- **Can create as many "S3 events" as desired

- S3 event notifications typically deliver events in seconds but can sometimes take a minute or longer

### Event Notifications - IAM Permissions

- Can send events to SNS, SQS, Lambda, and EventBridge

- attach SNS:Publish Policy to S3 Bucket (S3 is Principal)
- attach SQS:SendMessage Policy to S3 Bucket (S3 is Principal)
- attach Lambda:InvokeFunction Policy to S3 Bucket (S3 is Principal)

### Event Notifications with AWS EventBridge

- S3 -> EventBridge -> other AWS Services

- **Advanced filtering** options with JSON rules (metadata, object size, name...)
- **Multiple Destinations** - ex Step Functions, Kinesis Streams / Firehose
- **EventBridge Capabilities** - Archive, Replay Events, Reliable delivery

## S3 Baseline Performance

- S3 auto scales to high request rates, latency 100-200ms
- Your application can achieve at least **3,500 PUT/COPY/DELETE or 5,500 GET/HEAD requests per second per prefix in a bucket**
- There are no limits to the number of prefixes in a bucket

## S3 Performance

- **Multi-Part upload**:
    - recommended for files > 100MB
    - MUST use for files > 5GB

- **S3 Transfer Acceleration**:
    - Increase transfer speed by transferring file to a local AWS edge location which will forward the data to the S3 bucket in the target region (Edge Location -> S3 bucket is on AWS network so extremely fast)
    - Compatible with multi-part upload

- **Byte-Range Fetches**:
    - Parallelize GET by requesting specific byte ranges
    - Better resilienc in case of failures
    - **Can be used to speed up downloads**
    - **Can be used to retrieve only partial data (for ex. a head of a file)**

- **Select & Glacier Select**:
    - Retrieve less data using SQL by performing **server-side filtering**
    - Can filter by rows & columns (simple SQL statements)
    - Less network transfer, less CPU cost client-side

# S3 User-Defined Object Metadata & S3 Object Tags

- **S3 User-Defined Object Metadata**
    - When uploading an object, you can also assign metadata
    - Name-value pairs
    - must begin with "x-amz-meta-"
    - S3 stores user-defined metadata keys in lowercase
    - Metadata can be retrieved while retrieving the object

- **S3 Object Tags**:
    - Key-value pairs for objects in S3
    - Useful for fine-grained permissions (only access specific objects with specific tags)
    - Useful for analytics purposes (using S3 Analytics to group by tags)

- **You cannot search the object metadata or object tags**
- (EXAM) Instead, you must use an external DB as a search index such as DynamoDB





