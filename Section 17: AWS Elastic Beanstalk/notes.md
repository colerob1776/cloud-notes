# Elastic Beanstalk Overview

- **Developer Problems in AWS**
    - Managing Infra
    - Deploying Code
    - Configuring all the databases, load balancers, etc
    - Scaling concerns

- Most web apps have the same architecture (ALB + ASG)
- All the developers want is for their code to run
- Possibly, consistently across different applications and environments

- Elastic Beanstalk is a developer centric view of deploying an application on AWS
- it uses all the components we've seen before: EC2, ASG, ELB, RDS ...
- Managed Service:
    - Auto handles capacity provisioning, load balancing, scaling, application health monitoring, instance configuration, ...
    - Just the application code is the responsibility of the developer
- We still have full control over the configuration
- Beanstalk is free but you pay for the underlying instances

## Components

- **Application** - collection of Beanstalk components (environments, versions, configurations, ..)
- **Application Version** - an iteration of your code
- **Environment**
    - Collection of AWS resources running an application version (only one application version at a time)
    - **Tiers** - Web Server Environment Tier & Worker Environment Tier
    - You can create multiple environments (dev, test, prod, ...)

## Supported Platforms

    - Go
    - Java
    - .NET
    - Node.js
    - PHP
    - Python
    - Ruby
    - Packer Builder
    - Single Container Docker
    - Multi-container Docker
    - Preconfigured Docker

    - If not supported you can write your custom platform (advanced)

## Server Tier vs. Worker Tier

- Server Tier - ELB + ASG to scale EC2 instances

- Worker Tier - event based architecture to run EC2 workers

## Deployment Modes

- Single Instance
- High Availability with Load Balancer

### Beanstalk Deployment Modes - All at Once

- Stop running applications to deploy new version

- Fastest deployment
- App has downtime
- Great for quick iterations in dev environment
- No Additional Costs

### Beanstalk Deployment Modes - Rolling

- Application runs below capacity, and deploys new versions to open slots and keeps rolling until all instances are updated

- Application runs both versions simultaneously
- No Additional Costs
- Long Deployment

### Beanstalk Deployment Modes - Rolling w/ Additional Batch

- Same as Rolling but deploys additional updated instances before ramping down the old instances

- Small Additional Cost
- Longer for deployment
- Good for Prod

### Beanstalk Deployment Modes - Immutable

- New Code is deployed to new instances on a temporary ASG

- Zero Downtime
- High Cost, double capacity
- Longest Deployment
- Quick rollback in case of failures (just terminate ASG)
- Great for Prod

### Beanstalk Deployment Modes - Blue Green

- Create a new "stage" environment and deploy v2 there

- Not a "direct feature" of Elastic Beanstalk
- Zero downtime and release facility
- The new environment (green) can be validated independently and roll back if issues
- Route 53 can be setup using weighted policies to redirect a little bit of traffic to the stage env
- Using Beanstalk "swap URLs" when done with the environment test

### Beanstalk Deployment Modes - Traffic Splitting

- **Canary Testing**
- New application version is deployed to a temporary ASG with the same capacity

- A small % of traffic is sent to the temporary ASG for a configurable amount of time
- Deployment health is monitored
- If there's a deployment failure, this triggers an **automated rollback** (very quick)
- no application downtime
- New instances are migrated from the temp to the original ASG
- Old application version is then terminated

# Elastic Beanstalk CLI

- We can install additional CLI called the "EB cli" which makes working with Beanstalk from the CLI easier
- Basic Commands are:
    - eb create
    - eb status
    - eb health
    - eb events
    - eb logs
    - eb open
    - eb deploy
    - eb config
    - eb terminate

- helpful for deployment pipelines

## Elastic Beanstalk Deployment Process

- Describe dependencies (requirements.txt, package.json, etc)
- package code as zip
- **Console**: upload zip (creates new app version), and then deploy
- **CLI**: create new app version using CLI (uploads zip), and then deploy

- Elastic Beanstalk will deploy the zip on each EC2 instance, resolve dependencies, and start the application

# Beanstalk Lifecycle Policy

- Beanstalk can store at most 1000 app versions
- If you don't remove old versions, you won't be able to deploy anymore
- To phase out old app verions, use a **lifecycle policy**
    - based on time
    - based on space
- Versions that are currently used won't be deleted
- Option not to delete the source bundle in S3 to prevent data loss

# Elastic Beanstalk Extensions

- A zip file containing our code must be deployed to Beanstalk
- All the parameters set in the UI can be configured with code using files
- Requirements:
    - in the .ebextensions/ directory in the root of source code
    - YAML / JSON format
    - **.config** extensions (example: logging.config)
    - able to modify some default settings using: options_settings
    - Ability to add resources such as RDS, ElastiCache, DynamoDB, etc

- Resources managed by .ebextensions get deleted if the environment goes away

# Beanstalk & CloudFormation

- Under the hood, Beanstalk relies on CloudFormation
- CloudFormation is used to provision other AWS services 
- Use Case: you can define CloudFormation resources in you **.ebextensions** to provision ElastiCache, an S3 bucket, anything you want

# Beanstalk Cloning

- Clone an environment with the same configuration
- Useful for deploying a "test" version of your app

- All resources and configuration are preserved:
    - Load Balancer type and config
    - RDS database type (but data is not preserved)
    - Environment variables

- After cloning, you can change settings

# Beanstalk Migrations

### Load Balancer

- After creating an Elastic Beanstalk environment, **you cannot change the Elastic Load Balancer type** (only the config)
- To migrate:
    1. create a new environment with the same config except LB (can't clone)
    2. deploy our app onto the new environment
    3. perform a CNAME swap or Route 53 update

### RDS 

- RDS can be provisioned with Beanstalk, which is great for dev/test
- This is not great for prod as the database lifecycle is tied to the Beanstalk env

- The best for prod is to separately create an RDS database and provide our EB application with the connection string

- **How to Decouple RDS**:
    1. Create a snapshot of RDS (as a safeguard)
    2. Go to the RDS console and protect the RDS database from deletion
    3. Create a new EB env, without RDS, point your application to existing RDS (using env variables)
    4. perform a CNAME swap (blue/green) or Route 53 update, confirm working
    5. Terminate the old environment (RDS won't be deleted)
    6. Delete CloudFormation stack (in DELETE_FAILED state)