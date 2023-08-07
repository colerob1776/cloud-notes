# VPC

## Crash Course

- Virtual Private Cloud
- something you should know in depth for hte AWS Certified Solutions Architect & AWS SysOps Administrator

- **At the AWS Certified Developer Level, you should know about**:
    - VPC, Subnets, Internet Gateways & NAT Gateways
    - Security Groups, Network ACL (NACL), VPC Flow Logs
    - VPC Peering, VPC Endpoints
    - Site to Site VPN & Direct Connect

- Just 1 or 2 questions on the exam

## VPC & Subnets Primer

- **VPC**: private network to deploy your resources (regional resource)
- **Subnets**: allow you to partition your network inside your VPC (Availability Zone resource)
- A **public subnet** is a subnet that is accessible from the internet
- A **private subnet** is a subnet that is not accessible from the internet
- To define access to the internet and between subnets, we use **Route Tables**

## Internet Gateway & NAT Gateways

- **Internet Gateway** helps our VPC instances to connect with the internet
- Public Subnets have a route to the internet gateway

- **NAT Gateway** (AWS-managed) & **NAT Instances** (self-managed) allow your instances in your **Private Subnets** to access the internet while remaining private

## NACL, SG, & VPC Flow Logs

### Network ACL & Security Groups

- **NACL** (Network ACL)
    - A firewall which controls traffic from and to subnet
    - Can have ALLOW and DENY rules
    - Are attached at the **Subnet** level
    - Rules only include IP addresses

- **Security Groups**
    - A firewall that controls traffic to and from **an ENI / an EC2 Instance**
    - Can only have ALLOW rules
    - Rules include IP addresses and other SGs

### VPC Flow Logs

- Capture info about IP traffic to/from your interfaces:
    - VPC Flow Logs
    - Subnet Flow Logs
    - Elastic Network Interface (ENI) Flow Logs
- Helps to monitor & troubleshoot connectivity issues. Ex:
    - Subnets to internet
    - Subnets to Subnets
    - Internet to Subnets
- Captures network info from AWS managed interfaces too: LBs, ElastiCache, RDS, Aurora, etc...
- VPC Flow Logs data can go to S3, CloudWatch, Logs, and Kinesis Data Firehose

## VPC Peering, Endpoints, VPN, DX

### VPC Peering

- Connect 2 VPC, privately using AWS' network
- Make them behave as if they were in the same network
- MUST NOT have overlapping CIDR (IP address range)
- VPC Peering connection is **NOT Transitive** (must be established for each VPC that need to communicate with one another)

### VPC Endpoints (VERY IMPORTANT FOR EXAM)

- **Helpful anytime you need PRIVATE access within your VPC to an AWS Service**
- **If the exam asks how you need to privately connect to an AWS service, VPC Endpoints are the way**
- Endpoints allow you to connect to AWS Services **using a private network** instead of the public www network
- This gives you enhanced security and lower latency to access AWS services
- VPC Endpoint Gateway: S3 & DynamoDB
- VPC Endpoint Interface: the rest

- ONLY used within your VPC

### Site to Site VPN & Direct Connect

- Site to Site VPN:
    - Connect an on-premise VPN to AWS
    - The connection is auto encrypted
    - Goes over the public internet

- Direct Connect (DX):
    - Establish a physical connection between on-premises and AWS
    - The connection is private, secure and fast
    - Goes over a private network
    - Takes at least a month to establish

## VPC Closing Comments

- **VPC**: Virtual Private Cloud
- **Subnets**: tied to an AZ, network partition of the VPC
- **Internet Gateway**: at the VPC level, provide Internet Access
- **NAT Gateway / Instances**: give internet access to private subnets
- **NACL**: Stateless, subnet rules for inbound and outbound
- **Security Groups**: Stateful, operate at the EC2 instance level
- **VPC Peering**: Connect 2 VPC with non overlapping IP ranges, non transitive
- **VPC Endpoints**: Provide private access to AWS services within VPC
- **VPC Flow Logs**: network traffic logs
- **Site to Site VPN**: VPN over PUBLIC internet between on-premises DC and AWS
- **Direct Connect**: direct PRIVATE connection to AWS
