# AWS Step Functions

- Model your **workflows** as **state machines (one per workflow)**
    - Order fulfillement, Data processing
    - Web apps, any workflow
- Written in JSON
- Visualization of the workflow and the execution of the workflow, as well as history
- Start workflow with SDK, API Gateway,  or Event Bridge

### Step Function - Task States

- *Do some work in your state machine*
- Invoke one AWS service
    - Lambda
    - AWS Batch Job
    - Run ECS task and wait for completion
    - Insert an item from DynamoDB
    - Publish message to SNS/SQS
    - Launch another Step Function workflow...
- Run an Activity
    - EC2, ECS, on-premises
    - Activities poll the Step functions for work
    - Activities send results back to Step Functions

### Step Function - States

- **Choice State** - Test for a condition to send to a branch 
- **Fail or Succeed State** - Stop execution with failure or success
- **Pass State** - pass its input to its output or inject some fixed data, without performing work
- **Wait State** - Provide a delay for a certain amount of time or until a specified time/date
- **Map State** - Dynamically iterate steps
- ***Parallel State*** (EXAM) - *Begin parallel branches of execution*

# Error Handling in Step Functions

- Any state can encounter runtime errors for various reasons:
    - State machine definition issues
    - Task failures
    - Transient issues (ex. network partition events)
- Use **Retry** and **Catch** in the State Machine to handle the errors instead of inside the Application Code
- Predefined error codes:
    - States.ALL: mathces any error name
    - States.Timeout: Task ran longer than TimeoutSeconds or no heartbeat received
    - States.TaskFailed: execution failure
    - States.Permissions: insufficient priviledges to execute code
- The state may report its own errors

### Step Functions - Retry (Task or Parallel State)

- Retry block evaluated from top to bottom
- **ErrorEquals**: match a specific kind of error
- **IntervalSeconds**: initial delay before retrying
- **BackoffRate**: multiply the delay after each retry
- **MaxAttempts**: default to 3, set to 0 for no retries
- When max attempts are reached, the **Catch** kicks in

### Step Functions - Catch (Task or Parallel State)

- Catch block evaluated from top to bottom
- **ErrorEquals**: match a specific kind of error
- **Next**: State to send to 
- **ResultPath** - A path that determines what input is sent to the state specified in the Next field (**pass the error to the next step**)

# Step Functions - Wait for Task Token

- Allows you to pause Step Functions during a Task until a Task Token is returned
- Task might wait for other AWS services, human approval, 3rd party integration, call legacy systems...
- Append **.waitForTaskToken** to the **Resource** field to tell Step Functions to wait for the Task Token to be returned

- Task will pause until it receives that Task Token back with a **SendTaskSuccess** or **SendTaskFailure** API call

# Step Functions - Activity Tasks

- Enables you to have the Task work performed by an ***Activity Worker***
- **Activity Worker** apps can be running on EC2, Lambda, mobile device...
- Activity Worker poll for a Task using **GetActivityTask** API
- After Activity Worker completes its work, it sends a response of its success/failure using **SendTaskSuccess** or **SendTaskFailure**
- To keep the Task active:
    - Configure how long a task can wait by setting **TimeoutSeconds**
    - Periodically send a heartbeat from your Activity Worker using **SendTaskHeartBeat** within the time you set in **HeartBeatSeconds**
    - Activity Tasks can wait 1 year if constantly receiving heartbeats

# Step Functions - Standard vs Express

- **Standard**:
    - Max Duration: up to 1 year
    - Execution Model: Exactly-once Execution
    - Execution Rate: over 2,000 / sec
    - Execution History: up to 90 days or using CloudWatch
    - Pricing: # of State Transitions
    - Use Cases: Non-idempotent actions (ex. Payment Processing)

- **Standard**:
    - Max Duration: up to 5 minutes
    - Execution Model (EXAM): 
        - **Synchronous**: At-Most once
        - **Asynchronous**: At-Least once
            - *Note: Doesnt wait for workflow to complete (get results from CW Logs)*
    - Execution Rate: over 100,000 / sec
    - Execution History: CloudWatch Logs
    - Pricing: # of executions, duration, and memory consumption
    - Use Cases: IoT data ingestion, streaming data, mobile app backends

# AppSync Overview

- **AppSync** is a managed service that uses **GraphQL**
- Retrieve data in **real-time with WebSocket or MQTT on WebSocket**
- For mobile apps: local data access & data synchronization
- It all starts with uploading one **GraphQL schema**

### AppSync - Security

- There are 4 ways you can authorize apps to interact with your AppSync GraphQL API:
    - **API_KEY**
    - **AWS_IAM**: IAM users / roles / cross-account access
    - **OPENID_CONNECT**: OpenId Connect Provider / JWT
    - **AMAZON_COGNITO_USER_POOLS**

- For custom domain & HTTPS, use CloudFront in front of AppSync

# AWS Amplify

- Set of tools to create mobile and web applications ("Beanstalk" of mobie and web apps)
- Must-have features such as **data storage, auth, storage, and ML**
- **FE Libraries** with ready to use components for React, Vue, JS, iOS, Android, Flutter, etc...
- Incorporates AWS best practices for reliability, security, scalability
- Build and deploy with the **Amplify CLI** or **Amplify Studio**

- **Amplify Studio** - Visually build a full-stack app, both FE and BE
- **Amplify CLI** - Configure an Amplify BE with a guided CLI workflow
- **Amplify Libraries** - Connect your app to existing AWS Services (Cognito, S3, and more)
- **Amplify Hosting** - Host secure, reliable, fast web apps or websites via the AWS CDN (CloudFront)

### AWS Amplify - Important Features

- **AUTH**:
    - Leverates **Cognito**
    - User registration, auth, account recovery & other operations
    - Support MFA, Social Sign-in, etc...
    - Pre-built UI components
    - Fine-grained authorization

- **DATASTORE**:
    - Leverates AppSync and DynamoDB
    - Work with local data and have **automatic synchronization to the cloud** without complex code
    - Powered by GraphQL
    - Offline and real-time capabilities
    - Visual data modeling w/ Amplify Studio

### Amplify - Hosting

- Build and Host modern web apps
- CI/CD
- PR Reviews
- Custom Domains
- Monitoring
- Redirect and Custom Headers
- Password Protection

### Amplify - E2E Testing

- run E2E test in the **test phase** in Amplify
- Catch regressions before pushing coce to production
- Use the test step to run any test commands at build time (**amplify.yml**)
- **Integrated with Cypress testing framework**
    - allows you to generate UI report for your tests