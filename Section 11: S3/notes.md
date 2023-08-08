# S3

## Section Introduction

- S3 is one of the main building blocks of AWS
- It's advertised as "infinitly scaling" storage

- many websites use S3 as a backbone
- Many AWS services use S3 as an integration as well

## Use Cases

- Backup and Storage
- Disaster Recovery
- Archive
- Hybrid Cloud Storage
- Application Hosting
- Media Hosting
- Data lakes & big data analytics
- Software delivery
- Static website

## Buckets

- S3 allows people to store objects (files) in "buckets" (directories)
- Buckets MUST have a **globally unique name (across all regions and accounts)**
- Buckets are defined at the region level
- S3 looks like a global service but buckets are created in a region
- Naming Convention:
    - No uppercase, No underscore
    - 3-63 characters long
    - Not an IP
    - Must start with lowercase letter or number
    - Must NOT start with the prefix **xn--**
    - Must nOT end with the suffix **-s3alias**

## Objects

- Objects (files) have a Key
- The **key** is the **FULL** path:
    - s3://my-bucket/**my_file.txt**
    - s3://my-bucket/**my-folder/another_folder/my_file.txt**
- The key is compose of *prefix* + **object name**
    - s3://my-bucket/*my-folder/another_folder/***my_file.txt**
- There's no concept of "directories" within buckets (although it seems like it)
- Just keys with very long names that contain slashes ("/")

- Object values are the content of the body:
    - Max. Object Size is 5TB
    - If uploading more than 5GB, must use "multi-part upload"

- Metadata (list of text key/value pairs - system of user metadata)
- Tags (Unicode key/value pair - up to 10) - useful for security / lifecycle
- Version ID (if versioning is enabled)

## S3 Securty - Bucket Policy

### Security

- **User-Based**:
    - **IAM Policies**: which API calls should be allowed for specific user from IAM

- **Resource Based**:
    - **Bucket Policies**: bucket wide rules from the S3 console - allows cross account
    - **Object Access Control List (ACL)**: finer grain (can be disabled)
    - **Bucket Access Control List (ACL)**: less common (can be disabled)

- **Note**: an IAM pricipal can access an S3 object if:
    - the use IAM permissions ALLOW it OR the resource policy ALLOWS it
    - AND there's no explicity DENY

- **Encryption**: encrypt objects in S3 using encryption keys

### Bucket Policies

- JSON based policies
    - Resources: buckets and objects
    - Effect: Allow/Deny
    - Actions: Set of API to Allow/Deny
    - Principal: account or user to apply the policy to

- Use a bucket policy to:
    - Grant public access to the bucket
    - Force objects to be encrypted at upload
    - Grand access to another account

- If Block Public Access is ON, public access bucket polices will not work. Block Public Access must be turned off First

## S3 Website Overview

- S3 can host static websites and have them accessible on the internet

- The website URL will be (depending on the region)
    - http://**bucket-name**.s3-website-**aws-region**.amazonaws.com
    OR
    - http://**bucket-name**.s3-website.**aws-region**.amazonaws.com

- If you get a **403 Forbidden** error, make sure the bucket policy allows PUBLIC READS

## S3 Versioning

- You can version your files in Amazon S3
- it is enabled at the **bucket level**
- Same key overwrite will change the "Version"
- it is best practice to version your buckets
    - Protect against unintended deletes
    - Easy roll back to previous version
- Notes:
    - Any file that is not versioned prior to enabling versioning will have version "null"
    - Suspending versioning does not delete the previous versions

## S3 - Replication (CRR & SRR)

- **Must enable Versioning** in source and destination buckets
- **Cross-Region Replication (CRR)**
- **Same-Region Replication (SRR)**
- Buckets can be in different AWS accounts
- Copying is async
- Must give proper IAM permissions to S3

- Use cases:
    - **CRR** - compliance, lower latency access, replication across acocunts
    - **SRR** - log aggregation, live replication between production and test accounts

### Replication Notes

- After you enable Replication, only new objects are replicated
- Optionally, you can replicate existing objects using **S3 Batch Replication**
    - Replicates existing objects and objects that failed replication

- For DELETE operations:
    - **Can replicate delete markers** from source to target (optional setting)
    - Deletions with a version ID are not replicated (to avoid malicious deletes)

- **There is no "chaining" of replication**:
    - If bucket 1 has replication into bucket 2, which has replication into bucket 3
    - Then objects created in bucket 1 are not replicated into bucket 3

## S3 Storage Classes

- **Standard - General Purpose**
- **Standard-Infrequent Access (IA)**
- **One Zone-Infrequent Access**
- **Glacier Instant Retrieval**
- **Glacier Flexible Retrieval**
- **Glacier Deep Archive**
- **Intelligent Tiering**

- Can move between classes manually or useing S3 Lifecycle configurations

### S3: Durability and Availability

- Durability:
    - High durability (99.999999999%) of objects across multiple AZs
    - If you store 10M objects with Amazon S3, you can on average expect to incur a loss of a single object once every 10k years
    - Same for all storage classes

- Availability:
    - Measures how readily available a service is
    - Varies depending on storage class
    - Ex: S3 standard has 99.99% availability = not available 53 minutes a year

### S3 Storage Classes: Standard - General Purpose

- 99.99% Availablity
- Used for fequently accessed data
- Low latency and high throughput
- Sustain 2 concurrent facility failures

- Use Cases: Big Data Analytics, mobile & gaming applications, content distribution

### S3 Storage Classes: Infrequent-Access

- For data that is less frequently accessed, but requires rapid access when needed
- Lower cost than S3 Standard

- **Amazon S3 Standard-Infrequent Access (S3 Standard-IA)**
    - 99.9% Available
    - Use Cases: Disaster Recovery, backups

- **Amazon S3 One Zone-Infrequent Access (S3 One Zone-IA)**
    - High Durability in single AZ; data lost when AZ is destroyed
    - 99.5% Availability

### S3 Storage Classes: Glacier Storage Classes

- Low-cost object storage meant for archiving/backup
- Pricing: price for storage + object retrieval cost

- **Glacier Instant Retrieval**
    - Millisecond retrieval, great for data accessed once a quarter
    - Minimum storage duration of 90 days

- **Glacier Flexible Retrieval**
    - Expedited (1 to 5 minutes), Standard (3 to 5 hours), Bulk (5 to 12 hours) - free
    - Minimum storage duration of 90 days

- **Glacier Deep Archive - for long term storage**
    - Standard (12 hours), Bulk (48 hours)
    - Minimum storage duration of 180 days

### S3 Intelligent Tiering

- Small monthly monitoring and auto-tiering fee
- Moves objects automatically between Access Tiers based on usage
- There are no retrieval charges in S3 Intelligent-Tiering

- *Frequent Access tier (automatic)*: default tier
- *Infrequent Access tier (automatic)*: objects not accessed for 30 days
- *Archive Instant Access tier (automatic)*: objects not accessed for 90 days
- *Archive Access tier (automatic)*: config from 90 days to 700+ days
- *Deep Archive Access tier (automatic)*: config from 180 days to 700+ days
