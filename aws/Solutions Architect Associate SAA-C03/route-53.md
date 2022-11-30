# Route 53

- highly available, scalable, fullt managed, and **Authoritative** DNS
  - authoritative = the customer cann update the DNS records
- also a Domain Registrar
- ability to check the health of resources
- only AWS service with 100% availability SLA
- 53 is a reference to the traditional DNS port
- Records
  - define how you want to route traffic for a domain
  - each contains
    - Domain/subdomain Name: example.com
    - Record Type: A or AAAA
    - Value: 12.34.56.78
    - Routing Policy: how Route 53 responds to queries
    - TTL (Time To Live): amount of time the record cached at DNS Resolvers
  - supports
    - must know: A / AAAA / CNAME / NS
    - advanced: CAA / DS / MX / NAPTR / PTR / SOA / TXT / SPF / SRV
- Record Types
  - **A**: maps a hostname to IPv4
  - **AAAA**: maps a hostname to IPv6
  - **CNAME**: maps a hostname to another hostname
    - target is a domain name which must have an A or AAAA record
    - cant create a CNAME record for the top node of a DNS namespcae (Zone Apex)
      - not for example.com but for www.example.com
  - **NS**: Name Servers for the hosted zone
    - control how traffic is routed for a domain
- Hosted Zones
  - a container for records that define how to route the traffic to a domain and its subdomains
  - Public Hosted Zones - contains records that specify how to route traffic on the internet (public domain names)
  - Private Hosted Zones - contains records that specify how to route traffic within one or more VPCs (private domain names)
  - cost is $0.50 per month per hosted zone

## What is DNS?

- Domain Name System which translates the human friendly hostnames into the machine IP address
- www.google.com => 172.217.18.36
- DNS is backbone of the internet
- DNS uses hierarchical naming structure
- DNS Terminologies
  - Domain Registrar: Amazon Route 53, GoDaddy
  - DNS Records: A, AAAA, CNAME, NS, ...
  - Zone File: contains DNS records
  - Name Server: resolves DNS queries (Authoritative or Non-Authoritative)
  - Top Level Domain (TLD): .com, .us, .in, .gov
  - Second Level Domain (SLD): amazon.com, google.com
  - Sub Domain: www.example.com
  - Domain Name: api.www.example.com
  - Protocol: http, https
- How it works
  - Web Browser -> Local DNS Server -> Root DNS Server -> TLD DNS Server-> SLD DNS Server -> Cached on Local DNS Server -> Cached in Browser

## TTL

- High TTL ~ 24hrs
  - less traffic on Route 53
  - possibly outdated records
- Load TTL ~ 60sec
  - more traffic on Route 53
  - less outdated records
  - easy to change records
- Strategy - update TTL to lower value then update record and increase TTL again

## CNAME vs Alias

- AWS Resources (Load balancer/CloudFront) expose and AWS hostname
- CNAME Records
  - point a hostname to any other hostname (app.mydomain.com => blabla.anything.com)
  - only for non **ROOT** domain (aka something.mydomain.com)
- Alias Records
  - points a hostname to an AWS Resource (app.domain.com => blabla.amazon.aws.com)
  - works for **ROOT** and **NON-ROOT** domains (aka mydomain.com)
  - free of change
  - native health check
  - Records
    - extension of DNS functionality
    - automatically reconizes changes in the resource's IP addresses
    - unlike CNAME, it can be used fort the top node of a DNS namespace (Zone Apex) (example.com)
    - always of type A/AAAA for AWS resources (IPv4/IPv6)
    - you can't set the TTL
  - Targets
    - Elastic Load Balancer, CloudFront Distributions, API Gateway, Elastic Beanstalk env, S3 Websites, VPC Interface Endpoints, Global Accelerator accelerator, Route 53 record in the same hosted zone
    - CANNOT set an ALIAS for an EC2 DNS name

## Routing Policy

- define how Route 53 responds to DNS queries
- dont get confused by the word "Routing"
  - not the same as load balancer routing which routes traffice
  - DNS does not route any traffic, it only responds to DNS queries
- Policy Types
  - simple
  - weighted
  - failover
  - latency based
  - geolocation
  - multi-value answer
  - geoproximity (using Route 53 Traffic Flow feature)

### Simple Policy

- typically route traffic to a single resource
- can specify multiple values in the same record
  - random choice is chosen by client
- when Alias enabled, specify only one AWS resource
- cannot be associated with health checks

### Weighted Policy

- control the % of the requests that go to each resource
- weights between 0 and 255
- assign each record a relative weight
  - traffic % = weight for record / sum of all record weights
- DNS records must have the same name and type
- can be associated with Health Checks
- use cases: load balancing between regions, testing new application versions
- Assign a weight of 0 to a record to stop sending traffic to a resource
- if all records have 0, then all records will be returned equally

### Latency-based Policy

- redirect to the resource that has the lowest latency
- super helpful when latency is a priority
- Latency is based on traffic between users and AWS Regions
- Germany users may be redirected to the US (if lowest latency)
- can be associated with Health Checks (has failover capability)

### Failover (Active-Passive) Policy

- 2 EC2 instances (Primary, Secondary)
- if Health Check fails on Primary, Route 53 autokmatically routes to Secondary

### Geolocation Policy

- different from latenct based
- based on user location
- specify location by Continent, Country, or by US State
  - most precise location used
- should create a default record in case there are no matches
- Use Cases:
  - website localization, restrict content distribution, load balancing
- can be associated with Health Checks

### Geoproximity

- route traffice to resources based on geographic location of users and resources
- ability to shift more trasffic to resource based on defined bias
- to change size of geographic region, specify bias values:
  - to expand (1 to 99) - more traffic to resource
  - to shrink (-1 to -99) - less traffic to resource
- Resources can be
  - AWS resources (specify AWS region)
  - Non-AWS resources (specify Latitude and Longitude)
- Must use Route 53 Traffic Flow (advanced) to use this feauture

### Multi Value

- use when routing traffic to multiple resources
- Route 53 return multiple values/resources
- can be associated with Health Checks (return only values for healthy resources)
- up to 8 healthy records are returned for each Multi-Value query
- **Multi-Value is not a substitute for having and ELB**

## Health Checks

- HTTP Health Checks are only for **public resources**
- Health Check => Automated DNS Failover
  - Health check that monitors an endpoint (application, server, other AWS resource)
  - Health check that monitors other health checks (calculated health checks)
  - Health check that monitor CloudWatch Alarms (full control)
    - throttles of DynamoDB, alarms on RDS, custom metrics
    - helpful for private resources
- have their own metrics integrated with CloudWatch

### Endpoint Health Check

- about 15 global health checkers willm check the endpoint health
- Healthy/Unhealthy Threshold - 3
- Interval - 30 sec ( can set to 10 sec - higher cost)
- Supported Protocol: HTTP, HTTPS, TCP
- ability to choose locations you want Route 53 to use
- only pass with 2xx or 3xx status codes
- setup to pass/fail based on the text in the first **5120 bytes** of the response
- configure your router/firewall to allow incoming requests from Route 53 Health Checkers

### Calculated Health Checks

- combine the results of multiple Health Checks into a single Health Check
- can use **AND**, **OR**, or **NOT** to combine checks
- can monitor up to 256 Child Health Checks
- specify how many of the health checs need to pass to make the parent pass
- usage: perform maintenance to your website without causing all health checks to fail

### Private Hosted Zones

- Route 53 health checkers are outside the VPC
  - they cannot access private endpoints (private VPC or on-premise resources)
- Create a CloudWatch Metric and associate a CloudWatch Alarm, then create a Health Check that checks the alarm itself

## 3rd Party Domains

- if you buy a domain on a 3rd party registrar, you can still use Route 53 as the DNS Service Provider
  - Create a Hosted Zone in Route 53
  - Update NS Records on 3rd party site to use Route 53 **Name Servers**
- Domain Registrar != DNS Service
  - most Domain Registars include some DNS features
