# Route 53

- A highly available, scalable, fully managed and *Authoritative* DNS
    - Authoritative - the customer (you) can update the DNS records
- Route 53 is also a Domain Registrar
- Ability to check the health of your resources
- The only AWS service which provides 100% SLA
- Why Route 53? 53 is a reference to the traditional DNS port

## What is DNS

- Domain Name System which translates human readable hostname into IP addresses
- DNS is the backbone of the internet
- DNS uses hierarchical naming structure

### DNS Terminologies

- **Domain Registrar**: Amazon Route 53, GoDaddy, etc.
- **DNS Records**: A, AAAA, CNAME, NS, etc.
- **Zone File**: contains DNS records
- **Name Server**: resolves DNS queries (Authoritative or Non-Authoritative)
- **Top Level Domain (TLD)**: .com, .us, .in, .io, .gov, .org, etc.
- **Second Level Domain (SLD)**: amazon.com, google.com, etc.

- **url** - http://api.www.example.com.
- **protocol** - http
- **Fully Qualified Domain Name (FQDN)** - api.www.example.com.
- **Sub Domain** - www.example.com.
- **SLD** - example.com.
- **TLD** - .com.
- **root** - .

### How DNS Works

- Parties:
    1. **Client** - Web Browser 
        1. routes request to **Local DNS Server**
        2. waits for IP response and makes request to the **Web Server** with the Local DNS response
    2. **Local DNS Server** - Assigned and managed by your company or assigned by ISP dynamically 
        1. asks **Root DNS Server** about *example.com*
        2. asks **TLD DNS Server** about *example.com*
        3. asks **SLD DNS Server** about *example.com*
        4. waits for all responses and responds to **Client** the most specific IP
    3. **Root DNS Server** - Managed by ICANN
        - responds to **Local DNS Server** *.com NS 1.2.3.4*
    4. **TLD DNS Server** - Managed by IANA (Branch of ICANN)
        - responds to **Local DNS Server** *example.com NS 5.6.7.8*
    5. **SLD DNS Server** - Managed by Domain Registrar (eg Amazon Registrar inc.)
        - responds to **Local DNS Server** *example.com IP 9.10.11.12*

## Route 53 - Records

- How you want to route traffic for a domain
- Each record contains:
    - **Domain/subdomain Name** - eg. example.com
    - **Record Type** - eg. A or AAAA
    - **Value** - eg. 12.34.56.78
    - **Routing Policy** - how Route 53 responds to queries
    - **TTL** - amount of time the record is cached at DNS Resolvers
- Route 53 supports the following DNS record types:
    - (exam) - A / AAAA / CNAME / NS
    - (advanced) - CAA / DS / MX / NAPTR / PTR / SOA / TXT / SPF / SRV

### Route 53 - Record Types

- **A** - maps a hostname to IPv4
- **AAAA** - maps a hostname to IPv6
- **CNAME** - maps a hostname to another hostname
    - The target is a domain name which must have an A or AAAA record
    - Cant create CNAME record for the top node of a DNS namespace (Zone Apex)
    - Example: you cant create for example.com, but you can create for www.example.com
- **NS** - Name Servers for the Hosted Zone
    - Control how traffic is routed for a domain

## Route 53 - Hosted Zones

- A container for records that define how to route traffic to a domain and its subdomains
- **Public Hosted Zones** - contains records that specify how to route traffic on the internet (public domain names)
- **Private Hosted Zones** - contain records that specify how you route traffic whichin one or more VPCs (private domain names)

- you pay $0.50 per month per hosted zone

### Route 53 - Public vs. Private Hosted Zones

- **Public** - allows anyone from internet to access
- **Private** - only accessible within VPC

## Route 53 - TTL (Time To Live)

- how long the client should cache the DNS result
- High TTL
    - less traffic on Route 53
    - Possibly outdated records
- Low TTL
    - More traffic on Route 53 ($$)
    - Records are outdated for less time
    - Easy to change records
- Except for Alias Records, TTL is mandatory for each DNS record

## CNAME vs Alias

- AWS Resources (Load Balancer, CloudFront, ...) expose an AWS hostname (AWS derived DNS)
- CNAME (Exam): 
    - Points a hostname to any other hostname (app.domain.com -> prefix.anything.com)
    - **ONLY FOR NON ROOT DOMAIN (eg. prefix.mydomain.com)**
- Alias (Exam):
    - Points a hostname to an AWS Resource (app.mydomain.com -> service.amazonaws.com)
    - **Works for ROOT DOMAIN and NON ROOT DOMAIN (eg. mydomain.com)**
    - Free of charge

## Route 53 - Alias Records

- Maps a hostname to an AWS resource
- An extension of DNS functionality
- Auto recognizes changes in the resources IP address
- Unlike CNAME, it can be used for the top node of a DNS namespace (Zone Apex), eg. example.com
- Alias Record is always of type A/AAAA for AWS resources (IPv4 or IPv6)
- Dont have to specify TTL, AWS handles it

### Route 53 - Alias Records Targets

- Elastic Load Balancers
- CloudFront Distributions
- API Gateway
- Elastic Beanstalk environments
- S3 Website
- VPC Interface Endpoints
- Global Accelerator
- Route 53 record in the same hosted zone

- **You cannot set an ALIAS record for an EC2 DNS name**

## Route 53 - Routing Policies

- Define how Route 53 responds to DNS queries
- Dont get confused by the word "Routing"
    - It's not the same as Load Balancer routing which routes the traffic
    - DNS does not route any traffic, it only responds to the DNS queries
- Route 53 supports the following Routing Policies
    - Simple
    - Weighted
    - Failover
    - Latency based
    - Geolocation
    - Multi-Value Answer
    - Geoproximity (using Route 53 Traffic Flow feature)

### Routing Policy - Simple

- Typically, route traffic to a single resource
- Can specify multiple values in the same record
- **If multiple values are returned, a random one is chosen by the client**
- When Alias enabled, specify only ONE resource
- Can't be associated with Health Checks

### Routing Policy - Weighted

- Control the % of the requests that go to each specific resource
- Assign each record relative weight:
    - weights dont have to sum to 100, but a percentage will still be derived
- DNS records must have the same name and type
- Can be associated with Health Checks
- Use cases: load balancing between regions, testing new application versions...
- **Assign a weight of 0 to a record to stop sending traffic to a resource**
- **If ALL records have a weight of 0, they will all have equal weights**

### Routing Policy - Latency

- Redirect to the resource with the least latency
- Super helpful when latency for users is a priority
- **Latency is based of traffic between users and AWS Regions**
- Germany users may be directed to the US (if that's the lowest latency)
- Can be associated with Health Checks (has failover capability)

## Routing Policy - Failover

- dependent on Health Checks
- if a Health Check returns Unhealthy, it fails over to a backup record

## Routing Policy - Geolocation

- Different that Latency-based!
- **This routing is based on user location**
- Specify location by Continent, Country, or State (if there's overlapping, most precise location selected)
- Should create a **"Default"** record (in case there's no match on location)
- Use cases: website localization, restrict content distribution, load balancing, ...
- Can be associated with Health Checks

## Routing Policy - Geoproximity

- Route traffic to your resources based on the geographic location of users and resources
- Ability **to shift more traffic to resources based** on the defined **bias**
- To change the size of the geographic region, specify **bias** values:
    - To expand (1 to 99) -  more traffic to the resource
    - To shrink (1 to 99) - less traffic to the resource
- Resources can be:
    - AWS resources (specify AWS region)
    - Non-AWS resources (specify Latitude and Longitude)
- **You must use Route 53 Traffic Flow (advanced) to use this feature**
- (exam) useful for shifting traffic to certain regions by shifting the bias to those regions


## Routing Policy - IP-based

- Routing is based on clients' IP addresses
- You provide a list of CIDRs for your clients adn the corresponding endpoints/locations (user-IP-to-endpoint mappings)
- Use cases: Optimize performance, reduce network costs...
- Example: route end users from a particular ISP to a specific endpoint

## Routing Policy - Multi-Value

- Use when routing traffic to multiple resources
- Route 53 return multiple values/resources
- Can be associated with Health Checks (return only values for healthy resources)
- Up to 8 healthy records are returned for each Multi-Value query
- **Multi-Value is NOT a substitute for having an ELB**


## Route 53 - Traffic Flow

- Simplify the process of creating and maintaining records in large and complex configs
- Visual editor to manage complex routing decision trees
- Configs can be saved as **Traffic Flow Policy**
    - Can be applied to different Route 53 Hosted Zones (different domain names)
    - Supports versioning


## Route 53 - Health Checks

- HTTP Health Checks are only for **public resources**
- Health Check -> Automated DNS Failover:
    1. Health checks that monitor and endpoint (application, server, other resource)
    2. Health checks that monitor other health checks (Calculated Health Checks)
    3. Health checks that monitor CloudWatch Alarms (full control !!) - eg. throttles of DynamoDB, alarms on RDS, custom metrics, ... (helpful for private resources)
- Health Checks are integrated with CW metrics

### Health Checks - Monitor an Endpoint

- **About 15 global heath checkers will check the endpoint health**
    - Health/Unhealthy Threshold - 3 (default)
    - Interval - 30 sec (can set to 10 sec - higher cost)
    - Supported Protocols: HTTP, HTTPS, and TCP
    - If > 18% of health checkers report the endpoint is healthy, Route 53 considers it **Healthy**. Otherwise, it's **Unhealthy**
    - Ability to choose which locations you want Route 53 to use
- Health Checks pass only when the endpoint response with 2xx and 3xx status codes
- Health Checks can be setup to pass / fail based on the text in the first **5120 bytes** of the response
- Configure you router/firewall to allow incoming requests from Route 53 Health Checkers IP Address (found in bottom right of screen)

### Route 53 - Calculated Health Checks
 
- Combine the results of multiple Health Checks into a single Health Check
- You can use **OR**, **AND**, or **NOT**
- Can monitor up to 256 Child Health Checks
- Specify how many of the health checks need to pass to make the parent pass
- Usage: perform maintenance to your website without causing all health checks to fail

### Health Checks - Private Hosted Zones

- Route 53 health checkers are outside the VPC
- They can't access **private** endpoints (private VPC or on-premises resource)

- You can create a **CloudWatch Metric** and associate a **CloudWatch Alarm**, then create a Health Check that checks the alarm itself

## Domain Registrar vs DNS Service

- You buy or register your domain name with a Domain Registrar typically by paying annual charges (eg GoDaddy)
- The Domain Registrar usually provides you with a DNS service to manage your DNS records
- But you can use another DNS service to manage your DNS records
- Example: purchase the domain from GoDaddy and use Route 53 to manage your DNS records

- Edit the **Name Servers** on the 3rd Party Registrar to equal the Route 53 Name Servers

### 3rd Party Registrar with Route 53

- **If you buy your domain on a 3rd pary registrar, you can still use Route 53 as the DNS Service provider**
    1. Create a Hosted Zone in Route 53
    2. Update the NS Records on 3rd party website to use Route 53 **Name Servers**

- **Domain Registrar != DNS Service**
- But every Domain Registrar usually comes with some DNS features