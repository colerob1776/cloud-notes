
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
    1. Classic Load Balancer (v1) - HTTP, HTTPS, TCP, SSL
    2. Application Load Balancer (v2) - HTTP, HTTPS, WebSocket
    3. Network Load Balancer(v2) - TCP, TLS, UDP
    4. Gateway Load Balancer(v2) - Operates at layer 3 (Network Layer) IP Protocol
- Overall, it is recommended to use the newer gen (v2) LBs as they provide more features
- Some LBs can be setup as **internal** (private) or **external** (public) ELBs

### Load Balancer Security Groups

- Security Group on LB to allow all 80/443 traffic to LB
- Application Security Group (source is the SG on the LB) on Service to only allow traffic from LB