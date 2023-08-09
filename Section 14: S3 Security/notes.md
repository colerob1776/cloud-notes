# S3 - Object Encryption

- You can encrypt objects in S3 buckets using one of 4 methods:
    - **Server-Side Encryption (SSE)**
        1. **Server-Side Encryption with S3-Managed Keys (SSE-S3) - Default**
            - encrypts S3 objects using keys handled, managed, and owned by AWS
        2. **Server-Side Encryption with KMS Keys sotred in AWS KMS (SSE-KMS)**
            - Leverage AWS Key Management Service (AWS KMS) to manage encryption keys
        3. **Server-Side Encryption with Customer-Provided Keys (SSE-C)**
            - When you want to manage your own encryption keys

    - **Client-Side Encryption**
        4. **Manage keys yourself**

### Encryption SSE-S3

- Encryption using keys handled, managed, and owned by AWS
- Object is encrypted server-side
- Encryption type is **AES-256**
- Must set header **"x-amz-server-side-encryption": "AES256"**
- **Enabled by default for new buckets & new objects**

### Encryption SSE-KMS

- Encryption using keys handled and managed by AWS KMS (Key Management Service)
- MKS advantages: user control + audit key usage using CloudTrail
- Object is encrypted server-side
- Must set header **"x-amz-server-side-encryption": "aws:kms"**

- **Limitations**:
    - If you use SSE-KMS, you may be impacted by the KMS limits
    - When you upload, it calls the **GenerateDataKey** KMS API
    - When you download, it calls the **Decrypt** KMS API
    - Count towards KMS quota per second (5500, 10000, 30000, req/s based on region)
    - (EXAM MAY TEST) you can request a quota increase using the Service Quota Console 

### Encryption SSE-C

- Encryption using keys handled and managed by the customer outside of AWS
- S3 does NOT store the encryption key you provide
- **HTTPS MUST be used**
- encryption key must be provided in the HTTP headers, for every HTTP request made

### Encyption - Client-Side

- Use client libraries such as **Amazon S3 Client-Side Encryption Library**
- Clients must encrypt data themselves before sending to S3
- Clients must decrypt data themselves when retrieving from S3
- Customer fully manages the keys and encryption cycle

## Encryption in transit (SSL/TLS)

- Encryption in flight is also called SSL/TLS

- S3 exposes 2 endpoints
    - **HTTP Endpoint** - non encrypted 
    - **HTTPS Endpoint** - encryption in flight (Recommended, but MANDATORY FOR SSE-C)

- **Force Encryption in Transit (aws:SecureTransport)**:
    - add **aws:SecureTransport** to the "Condition" section of the S3:GetObject Policy

## S3 - Default Encryption vs Bucket Policies

- SSE-S3 encryption is auto applied to new objects stored in S3
- Optionally, you can "force encryption" using a bucket policy and refuse and API call to PUT an S3 object without encryption headers (SSE-KMS or SSE-C)

- **Note: Bucket Policies are evaluated before "Default Encryption"**


# CORS

- **Cross-Origin Resource Sharing**
- **Origin = scheme (protocol) + host (domain) + port**
- **Web Browser** based mechanism to allow requests to other origins while visiting the main origin
- Same origin: http://example.com/app1 & http://example.com/app2
- Different origins: http://www.example.com & http://other.example.com
- The requests won't be fulfilled unless the other origin allows for the requests, using **CORS Headers** (example: **Access-Control-Allow-Origin**)

- (EXAM) if a client makes a cross-origin request on our S3 bucket, we need to enable the correct CORS headers
- you can allow for a specific origin or for * (all origins)

# S3 - MFA Delete

- force users to generate a code on a device before doing important operations on S3
- MFA will be required to:
    - permanent deletes
    - Suspend versioning on a bucket
- MFA won't be required to:
    - Enable Versioning
    - List deleted versions
- To use MFA Delete, **Versioning must be enabled** on the bucket
- **Only the bucket owner (root account) can enable/disable MFA delete**

# S3 - Access Logs

- For audit purposes, you may want to log all access to S3 buckets
- Any request made to S3, from any account, authorized or denied, will be logged into another S3 bucket
- The data can be analyzed using data analysis tools
- The target logging bucket must be in the same AWS region

- **Warning**:
    - Do not set your loggin bucket to be the monitored bucket
    - it will create an infinite logging loop and **your bucket will grow exponentially**

# S3 - Pre-Signed URLs

- **Used to give file access to a private bucket**
- Generate pre-signed URLs using hte **S3 Console, AWS CLI or SDK**
- **URL Expiration**
    - **S3 Console** - 1 min up to 720 mins (12 hours)
    - **AWS CLI** - configure expiration with *--expires-in* parameter in seconds (default 3600, max 604800)
- Users given a pre-signed URL inherit the permissions of the user that generated the URL for GET/PUT

- Examples:
    - Allow only logged-in users to download a premium video from your S3 bucket
    - Allow an ever-changing list of users to download files by generating URLs dynamically
    - Allow temporarily a user to upload a file to a precise location in your S3 bucket

# S3 - Access Points

- Access Points simplify security management for S3 Buckets
- Used to give an assortment of users/groups access to certain prefixes in a bucket without creating an assortment of policies to go with each user/group and prefix

1. Create access points to each/multiple prefix(es)
2. assign policies to the access points

- Each Access Point has:
    - its own DNS name (Internet Origin or VPC origin)
    - an access point policy (similar to bucket policy) - manage security at scale

### Access Points - VPC Origin

- We can define the access point to be accessible only from within the VPC
- You must create a VPC Endpoint to access the Access Point
- The VPC Endpoint Policy must allow access to the target bucket AND Access Point

# S3 Object Lambda

- Use Lambda Functions to change the object before it is retrieved by the caller
- Only one S3 bucket is needed, on top of which we create **S3 Access Point** and **S3 Object Lambda Access Points**
- Use Case:
    - redacting personally identifiable info for analytics or non-production environments
    - Converting across data formats, such as converting XML to JSON
    - Resizing and watermarking images on the fly using caller-specific details, such as the user who requested the object
