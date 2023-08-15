# Docker Introduction

### Where are Docker images stored?

- Docker Repositories
    - Docker Hub - public repo
    - **Amazon ECR (Elastic Container Registry)** - private and public repo

### Docker Containers Management on AWS

- **ECS (Elastic Container Service)**
    - Amazon's own container platform

- **EKS ( Elastic Kubernetes Service)**
    - Amazon's managed Kubernetes (open source)

- **Fargate**
    - Amazon's own Serverless container platform
    - Works with ECS and with EKS

- **Amazon ECR**
    - Store container images

# Amazon ECS

## EC2 Launch Type

- Launch Docker containers on AWS = Launch **ECS Tasks** on ECS Clusters
- **EC2 Launch Type: you must provision & maintain the infra (the EC2 instance)**
- Each EC2 Instance must run the ECS Agent to register in the ECS Cluster
- AWS takes care of starting / stopping containers

## Fargate Launch Type

- Launch Docker containers on AWS
- **You do not provision the infra (no EC2 instances to manage)**
- it's all **Serverless**
- You just create task definitions
- AWS just runs EC2 Tasks for you based on the CPU/RAM you need
- To scale, just increase the number of tasks. Simple - no more EC2 Instances

## IAM Roles for ECS

- **EC2 Instance Profile (EC2 Launch Type only)**
    - Used by the *ECS Agent*
    - Makes API calls to ECS service
    - Send container logs to CloudWatch Logs
    - Pull Docker image from ECR
    - Reference sensitive data in Secrets Manager or SSM Parameter Store

- **ECS Task Role**
    - Allows each task to have specific role
    - Use different roles for the different ECS Services you run
    - Task Role is defined in the **task definition**

## Load Balancer Integrations

- **ALB** supported and works for most use cases
- **NLB** recommended only for high throughput / performance use cases, or to pair with AWS Private Link

## Data Volumes (EFS)

- Mount EFS file systems onto ECS tasks
- Works for both **EC2** and **Fargate** launch types
- Tasks running in any AZ will share the same data in the EFS file system
- **Fargate + EFS = Serverless**

- Use case: persistent multi-AZ shared storage for your containers

- Note: S3 cannot be mounted as a file system

## ECS Service Auto Scaling

- Auto increase/decrease the desired number of ECS tasks
- ECS Auto Scaling uses **AWS Application Auto Scaling**
    - ECS Service Average CPU Utilization
    - ECS Service Average Memory Utilization - Scale on RAM
    - ALB Request Count Per Target - metric coming from ALB
- **Target Tracking** - scale based on target value for a specific CloudWatch metric
- **Step Scaling** - scale based on a specific CloudWatch Alarm
- **Scheduled Scaling** - scale based on a specific date/time (predictable changes)

- ECS Service Auto Scaling (task level) != EC2 Auto Scaling (EC2 instance level)
- Fargate Auto Scaling is much easier to setup (because **Serverless**)

### EC2 Launch Type - Auto Scaling EC2 Instances

- Accommodate ECS Service Scaling by adding underlying EC2 Instances

- **Auto Scaling Group Scaling** 
    - Scale your ASG based on CPU Utilization
    - Add EC2 instances over time

- **ECS Cluster Capacity Provider**
    - Used to auto provision and scale the infra for your ECS Tasks
    - Capacity Provider paired with an ASG
    - Add EC2 Instances when you're missing capacity (CPU, RAM, ...)

## ECS Rolling Updates

- When updating from v1 to v2, we can control how many tasks can be started/stopped, and in which order
- **min** - (0-100%) specify the mininmum task capacity to run when updating
- **max** - (0-200%) specify the max task capacity to run when adding a new version of tasks

## ECS - Task Definitions

- Task definitions are metadata in **JSON** form to tell ECS how to run a Docker container
- It contains crucial info (EXAM):
    - Image Name
    - Port Binding for Container and Host
    - Memory and CPU required
    - Environment Variables
    - Networking Information
    - IAM Role
    - Logging configuration (ex CloudWatch)
- Can define up to 10 containers in a Task Definition

### ECS - Load Balancing (EC2 Launch Type)

- We get a **Dynamic Host Port Mapping** if you define only the container port in the task definition
- The ALB find the righ port on your EC2 Instances
- **you must allow on the EC2 instance's SG *any port* from the ALB's SG**

### ECS - Load Balancing (Fargate Launch Type)

- Each task has a **unique private IP**
- **Only define the container port** (host port is not applicable)

- Example:
    - **ECS ENI Security Group**
        - Allow port 80 from the ALB
    - **ALB Security Group**
        - Allow port 80/443 from web

## ECS - IAM Roles (EXAM)

- **ONE IAM ROLE PER TASK DEFINITION**
- Q: Where do you define IAM Role for ECS Tasks
- A: On your Task Definition

## ECS - Environment Variables

- Environment Variable:
    - **Hardcoded** - e.g. URLs
    - **SSM Parameter Store** - sensitive variables (e.g. API keys, shared configs)
    - **Secrets Manager** - sensitive variables (e.g. DB passwords)
- Environment Files (bulk) - Amazon S3

## ECS - Data Volumes (Bind Mounts)

- Share data between multiple containers in hte same Task Definition
- Works for both **EC2** and **Fargate** tasks
- **EC2 Tasks** - using EC2 instance storage
    - Data is tied to the lifecycle of the EC2 instance
- **Fargate Tasks** - using ephemeral storage
    - Data is tied to the containers using them
    - 20 GB - 200 GB (default 20 GB)

- Use Cases:
    - Share ephemeral data between multiple containers
    - "Sidecar" container pattern, where the "sidecar" container used to send metrics/logs to other destinations (separation of concerns)

# ECS Task Placements

- When a task of type EC2 is launched, ECS must determine where to place it, with the constraints of **CPU**, **memory**, and **available port**
- Similarly, when a service scales, ECS needs to determine which task to terminate
- To assist with this, you can define a **task placement strategy** and **task placement constraints**
- Note: this is only for EC2 Launch Type, not Fargate

## Task Placement Process

- Task placement strategies are a best effort
- When ECS places tasks, it uses the following process to select container instances
    1. Identify the instances that satisfy CPU, RAM, and port requirements in the task definition
    2. Identify the instances that satisfy the task placement constraints
    3. Identify the instances that satisfy the task placement strategies
    4. Select the intances for task placement

## Task Placement Strategies

- **Binpack**
    - Place tasks based on the least available amount of CPU or memory
    - This minimizes the number of instances in use (cost savings)

- **Random**
    - Place the task randomly

- **Spread**
    - Place the task evenly based on the specified value
    - Example: *InstancId*, *attribute:ecs.availability-zone*

- you can mix strategies together

# Task Placement Constraints

- **distinctInstance** - place each task on a different container instance
- **memberOf** - places task on instances that satisfy an expression
    - Uses the Cluster Query Language (advanced)

# Amazon ECR

- Elastic Container Registry
- Used to store and manage docker containers on ECS
- **Private** and **Public** repos (**Amazon ECR Public Gallery**)
- Fully integrated with ECS, backed by Amazon S3
- Access is controlled through IAM (permission errors => policy)
- Supports image vulnerability scanning, versioning, image tags, image lifecycle, ...

# AWS Copilot

- CLI tool to build, release, and operate production-ready containerized apps
- Run your apps on **AppRunner**, **ECS**, and **Fargate**
- Helps you focus on building apps rather than setting up infra
- provisions all required infra for containerized apps (ECS, VPC, ELB, ECR, ...)
- Automated deployments with one command using CodePipeline
- Deploy to multipl environments
- Troubleshooting, logs, health, status, ...

# Amazon EKS Overview

- Elastic **Kubernetes** Service
- It is a way to launch **managed Kubernetes clusters on AWS**
- Kubernetes is an open-source system for automatic deployment, scaling, and management of containerized apps
- alternative to ECS, similar goal but different API
- EKS supports **EC2** if you want to deploy worker nodes or **Fargate** to deploy serverless containers
- **Use case**: if your company is already using kubernetes on-premises or in another cloud, and wants to migrate to AWS using Kubernetes
- **Kubernetes is cloud agnostic** (can be used in any cloud - Azure, GCP, ...)

### Node Types

- **Managed Node Groups**
    - Creates and manages Nodes (EC2 instances) for you
    - Nodes are part of an ASG managed by EKS
    - Supports On-Demand or Spot Instances

- **Self-Managed Nodes**
    - Nodes created by you and registered to the EKS cluster and managed by an ASG
    - you can use prebuilt AMI - Amazon EKS Optimized AMI
    - Supports On-Demand or Spot Instances

- **AWS Fargate**
    - No maintenance required; no nodes managed

### Data Volumes

- Need to specify **StorageClass** on your EKS cluster
- Leverages a **Container Storage Interface (CSI)** compliant driver

- Support for:
    - EBS
    - EFS (works with Fargate)
    - Amazon FSx for Lustre
    - Amazon FSx for NetApp ONTAP





        