
<!-- Scalability & High Availability -->
# Scalability & High Availability

- Scalability means that an application system can handle greater loads by adapting
- there are 2 kinds of scalability:
    1. Vertical
    2. Horizontal (= elasticity)
- Scalability is linked but different to High Availability

### Vertical Scalability

- increasing size of instance
- upgrading t2.micro to t2.large
- Very common for non distributed systems
- RDS and ElastiCache are services that can scale vertically

## Horizontal Scalability

- increasing number of instances / services for the application
- implies distributed systems
- very common for web apps and other modern apps
- easy to horizontally scale due to cloud offerings like EC2

### High Availability

- High Availability usually goes hand in hand with horizontal scaling
- High availability means running your apps/system in at least 2 data centers (== AZ)
- The goal of high availability is to survive a data center loss
- high availability can be passive (for RDS Multi AZ)
- high availability can be active (for horizontal scaling)

### High Availability & Scalability for EC2

- vertical scaling: increase instance size
- horizontal: increase number of instances
    - Auto Scaling Group
    - Load Balancer
- High Availability: run instances for same application accross multi AZ
    - Auto Scaling Group multi AZ
    - Load Balancer multi AZ


<!-- Load Balancing -->
# Elastic Load Balancing (ELB) Overview

- servers that forward traffic to multiple servers downstream

### Why use a Load Balancer?

- spread load across multiple downstreams
- Expose single point of access (DNS) to your app
- Seamlessly handle failures of downsream instances
- Do regular health checks to your instances
- Provide SSL termination (HTTPS) for you sites
- Enforce stickiness with cookies
- High Availability across zones
- Separate Public traffic from private traffic

### Why use Elastic Load Balancer?

- **Managed Load Balancer**
    - AWS guarantees that is will be working
    - AWS takes care of upgrades, maintenance and HA
    - AWS provides only a few configuration knobs
- It costs less to setup your own load balancer but it will be a lot more effort on your end
- It is integrated with many AWS offerings/services
    - EC2, EC2 Auto Scaling Groups, ECS
    - AWS Certificate Manager (ACM), Cloudwatch
    - Route 53, WAF, Global Accelerator

### Health Checks

- crucial for load balancers
- enable the load balancer to know if instances it forwards traffic to are available to reply to requests
- done on a port and a route (/health is common)
- If the response is **not** 200 (OK), then the instance is unhealthy

### Types of Load Balancers

- **4 Kinds of Managed Load Balancers**:
    1. Classic Load Balancer (v1 - deprecated) - HTTP, HTTPS, TCP, SSL
    2. Application Load Balancer (v2) - HTTP, HTTPS, WebSocket (operates at layer 7 - Application Layer)
    3. Network Load Balancer(v2) - TCP, TLS, UDP ((operates at layer 4 - Transport Layer))
    4. Gateway Load Balancer(v2) - Operates at layer 3 (Network Layer) IP Protocol
- Overall, it is recommended to use the newer gen (v2) LBs as they provide more features
- Some LBs can be setup as **internal** (private) or **external** (public) ELBs

### Load Balancer Security Groups

- Security Group on LB to allow all 80/443 traffic to LB
- Application Security Group (source is the SG on the LB) on Service to only allow traffic from LB

# Application Load Balancer (ALB)

- ALBs are Layer & (HTTP)
- Load balancing to multiple HTTP apps across machines (target groups)
- Load balance to multiple applications on the same machine
- Support for HTTP/2 and WebSocket
- Support redirects (from HTTP to HTTPS for example)
- Routing tables to different target groups
    - routing based on path in URL
    - routing based on hostname in URL
    - routing based on query string and headers
- ALB are great for microservices & container-based apps
- Has port mapping feature to redirect to a dynamic port in ECS
- in comparison, we'd need multiple Classic Load Balancers per application

### Target Groups

- EC2 instances - HTTP
- ECS - HTTP
- Lambda - HTTP
- Ip Addresses - must be private IP

- ALB can route to multiple target groups
- Health checks are done at the target group level

### Good To Know

- Fixed hostname 
- application servers dont see the IP of hte client directly
    - The true IP of the client is inserted in the header **X-Forwarded-For**
    - We can also get Port (X-Forwarded-Port) and proto (X-Forwarded-Proto)

# Network Load Balancer (NLB)

- NLBs (layer 4) allow to:
    - forward TCP & UDP traffic to your instances
    - hadnle millions of requests/sec
    - less latency ~100ms (vs 400ms for ALB)

- NLB has **one static IP per AZ** and supports assigning Elastic IP (helpfull for whitelisting specific IPs)

- NLB are used for extreme performance TCP or UDP traffic
- Not included in the AWS free tier

### Target Groups

- EC2 instances
- IP Addresses - must be private IPs
- Application Load Balancer
- Health Checks support TCP, HTTP, and HTTPS

# Gateway Load Balancer (GLB)

- Deploy, scale, and manager a fleet of 3rd party network virtual appliances in AWS
- Examples:
    1. Firewalls
    2. Intrusion Detection & Prevention Systems
    3. Deep Packet Inspection Systems
    4. Payload Manipulation
- Routes traffic to 3rd party services (EC2 instances) that can decide if the request is permissible. If so, the service returns data back to the GLB
- Operates at Layer 3 (Network Layer) - IP
- Combines the following functions:
    - **Transparent Network Gateway** - single entry/exit for all traffic
    - **Load Balancer** - distributes traffic to your virtual appliances
- Uses the **GENEVE** protocol on port **6081**

### Target Groups (GLB)

- EC2 Instances
- IP Addresses - must be private IPs

# Sticky Sessions (Session Affinity)

- It is posible to implement stickiness so the same client is always redirected to the same instances behind a LB
- **supported by ALB and NLB**
- The "cookie" used for stickiness has an expiration date you control
- Use Case: make sure the user doesnt lose his session data
- Enabling stickiness may bring imbalance to the load over the backend EC2 instances
- client is responsible for sending the Cookie in requests so the LB knows about the session

### Cookie Names

- Application-based Cookies
    - Custom cookie:
        - Generated by target
        - Can include any custom attributes required by the app
        - Cookie name must be specified individually for each target group
        - Dont use **AWSALB, AWSALBAPP**, or ** AWSALBTG** (reserved for use by the ELB)
    - Application cookie:
        - Generated by the load balancer
        - Cookie name is **AWSALBAPP**
- Duration-based Cookies:
    - Cookie generated by the LB
    - Cookie name is **AWSALB** for ALB, **AWSELB** for Classic Load Balancer

# Cross-Zone Load Balancing

- With Cross-Zone Load Balancing, each LB distributes evenly accross all registered instances in all AZs
- Without Cross Zone Load Balancing, traffic only gets distributed evenly within a given AZ

- **Application Load Balancer**:
    - Enabled by default (can be disabled at the Target Group level)
    - No charges for inter AZ data

- **NLB and GLB**:
    - Disabled by default
    - You pay charges ($) for inter AZ data if enabled

- **Classic Load Balancer**
    - Disabled by default
    - No charges for inter AZ data if enabled

# Elastic Load Balancer

### SSL/TLS - Basics

- An SSL certificate allows traffic between your clients and your LB to be encrypted in transit (in-flight encryption)

- **SSL** refers to Secure Sockets Layer, used to encrypt connections
- **TLS** refers to Transport Layer Security, which is a newer version
- Nowadays, **TLS certificates are mainly used**, but people still refer as SSL

- Public SSL certificates are issued by Certificate Authorities (CA)
- Comodo, Symantec, GoDaddy, GlobalSign, Digicert, Letsencrypt, etc..

- SSL certificates have an expiration date (you set) and must be renewed

### Load Balancer - SSL Certificates

- only use HTTPS (encrypted) from Client -> LB
- LB -> applications services are HTTP (over private VPC)

- The LB uses an X.509 certificate (SSL/TLS server certificate)
- You can manager certificates useing ACM (AWS Certificate Manager)
- You can create upload your own own certificates alternatively
- HTTPS Listener:
    - You must specify a default certificate
    - You can add an optional list of certs to support multiple domains
    - **Clients can use SNI (Server Name Indication) to specify the hostname they reach**
    - Ability to specify a security policy to support older version of SSL/TLS

### Server Name Indication (SNI)

- SNI solves the problem of loading **multiple SSL certificates onto one web server** (to serve multiple websites)
- It's a "newer" protocol, and requires the client to **indicate** the hostname of the target server in the initial SSL handshake
- The server will then find the corret certificate, or return the default one

- Note:
    - Only works for ALB & NLB, and CloudFront
    - Does not work for CLB (older generation)

### Elastic Load Balancers - SSL Certificates

- **ALB** and **NLB** (v2):
    - Supports multiple listeners with multiple SSL certs
    - Users SNI to make it work

# Connection Draining (Exam)

- **Feature Naming**:
    - Connection Draining - for CLB
    - Deregistration Delay - for ALB & NLB

- Time to complete "in-flight requests" while the instance is de-registerting or unhealthy
- Stops sending new rquests to the EC2 instance which is de-registering
- Between 1 to 3600 seconds (default: 300 secs)
- Can be disabled (set to 0 seconds)
- Set to a low value if your requests are short

# Auto Scaling Group (ASG)
 - in real-life, the load on your websites and apps can change
 - in the cloud, you can create and get rid of servers very quickly

 - The goal of an ASG is to:
    - Scale out to match an increased load
    - Scale in to match a decreased load
    - Ensure we have a minimum and maximum number of EC2 instances running
    - Automatically register new instances to a LB
    - Re-create an EC2 instance in case a previous one is terminated (ex: if unhealthy)

- ASG are free (you only pay for the underlying EC2 instances)

### ASG in AWS

- set min capacity 
- set desired capacity
- set max capacity
- ASG will decide how many instances are needed

- a **Launch Template** is a template that details an EC2 config
- For ASGs behind a LB, the LB will be selected in the **ASG Launch Template**
- ASGs need **ASG Launch Templates** to know how to spin up Instances

### Auto Scaling - CloudWatch & Scaling

- It is possible to scale an ASG based on CloudWatch alarms
- An alarm monitors a metric (such as **Average CPU**)
- **Metrics such as Average CPU are computed for the overall ASG instances**
- Based on the Alarm:
    - we can create scale-out policies
    - we can create scale-in policies

### Scaling Policies

- **Target Tracking Scaling**:
    - Most simple and easy to set up
    - Ex: I want the average ASG CPU to stay around 40%
- **Simple/Step Scaling**:
    - When a CloudWatch alarm is triggered (Ex: CPU > 70%), then add 2 instances
    - When a CloudWatch alarm is triggered (Ex: CPU < 30%), then remove 1 instance

- **Scheduled Actions**:
    - Anticipate a scaling based on known usage patterns
    - Ex: increase the min capacity to 10 at 5PM on Fridays

- **Predictive Scaling**:
    - continuously forecast load and schedule scaling ahead

### Good Metrics to Scale on

- **CPUUtilization**: Avg CPU utilization across your instances
- **RequestCountPerTarget**: to make sure the number of requests per EC2 instances is stable (best with round robin LBs)
- **Average Network In/Out**: if your application is network bound
- **Any Custom Metric**: based on custom metrics pushed to CloudWatch

### Scaling Cooldown

- After a scaling activity happens, you are in the **cooldown period (default 300 seconds)**
- During the cooldown period, the ASG will not launch or terminate additional instances (to allow for metrics to stabilize)

- **Advice**: use a ready-to-use AMI to reduce configuration time in order to be service requests faster and reduce cooldown period

# Instance Refresh

- Goal: update launch template and then re-creating all EC2 instances
- For this we can use the native features of Instance Refresh
- Setting of minimum healthy percentage
- Specify warm-up time (how long until the instance is ready to use)