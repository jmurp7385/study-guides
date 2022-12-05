# CloudFront & AWS Global Accelerator

## Cloudfront

- Content Delivery Network (CDN)
- **Improves read performance, content is cached at the edge**
- improves user experience
- 216 Points of Presence globally (edge locations)
- **DDoS protection (because worldwide), integration with Shield, AWS Web Application Firewall**

### Origins

- S3 Bucket
  - for distributing files and caching them at the edge
  - enhanced security with cloudfront Origin Access Control (OAC)
  - OAC is replacing Origin Access Identity (OAI)
  - Cloudfront can be used as an ingress (to upload files to S3)
- Custom Origin (HTTP)
  - EC2 Instance
    - users -> edge lcoation -> allow Public IP of Edge Locations -> EC2 (must be public)
  - Application Load Balancer
    - users -> edge location -> allow Public IP of Edge Locations -> ALB -> allow security group of the load balancer -> EC2 (can be private)
  - S3 Website (must first enable bucekt as a static S3 Website)
  - Any HTTP backend you want

### Cloudfront vs S3 Cross Region Replication

- Cloudfront
  - Global Edge Network
  - Files are cached for a TTL (maybe a day)
  - **Great for static content that must be available everywhere**
- S3 Cross Region Replication
  - must be setup for each region you want replication
  - files are updated in near real-time
  - read only
  - **Great for dynamic content that needs to be available at low-latency in few regions**

### Geo Restriction

- can restrict access to distrobution
  - Allowlist: allow users access to your content only if they are in one of the approved countries
  - Blocklist: prevent users access to your content if they are in one the banned countries
- the "country" is determined using a 3rd Party Geo-IP database
- Use Case: Copyright Laws to control access to content

### Price Classes

- CloudFront Edge locations are all around the world
- cost of data out per edge location varies
- can reduce the number of edge locations for cost reduction
- 3 price classes
  - Price Class All: all regions - best performance
  - Price Class 200: most regions, excludes expensive regions
  - Price Class 100: only the least expensive regions

| Edge Locations Included Within | USA, Mexico, Canada | Europe, Israel | South America | Japan | Australia, New Zealand | Hong Kong, Philippines, Singapore, South Korea, Taiwan, Thailand | India |
| :----------------------------: | :-----------------: | :------------: | :-----------: | :---: | :--------------------: | :--------------------------------------------------------------: | :---: |
|        Price Class All         |         Yes         |      Yes       |      Yes      |  Yes  |          Yes           |                               Yes                                |  Yes  |
|        Price Class 200         |         Yes         |      Yes       |      Yes      |   X   |          Yes           |                                X                                 |  Yes  |
|        Price Class 100         |         Yes         |      Yes       |       X       |   X   |           X            |                                X                                 |   X   |

### Cache Invalidations

- in case you update the back-end origin, CloudFront doesn't know about it and will only get the refreshed content after the TTL has expires
- you can force an entire or partial cache refresh (bypassing the TTL) by performing a **CloudFront Invalidation**
- can invalidate all files (\*) or a special path (/images/\*)

## Global Accelerator

- you have deployed an application and have global users who want to access it directly, but it is only deployed in one region
- they go over public internet, which can add a lot of latency due to many hops
- we wish to go as fast as possible through AWS network to minimize latency
- Unicast IP vs Anycast IP
  - Unicast IP: one server holds one IP address
  - Anycast IP: all servers hold the same IP address and the client is routed to the nearest one
    - use by Global Accelerator
- leverage the AWS internal network to route to your application
- 2 Anycast IP are created for your applications
- the Anycast IP sends traffic directly to Edge Locations
- Edge Locations send the traffic to your application
- Works with Elastic IP, EC2 instances, ALB, NLB, public or private
- Consistent Performance
  - intelligent routing to lowest latency and fast regional failover
  - no issue with client cache (because IP doens't change)
  - internal AWS Network
- Health Checks
  - Global Accelerator performs a health check of your applications
  - helps make your application global (failover less than 1 minute for unhealthy)
  - great for disaster recovery (thanks to health checks)
- Security
  - only 2 external IP need to be whitelisted
  - DDoS protection thanks to AWS Shield

### Global Accelerator vs Cloudfront

- both use AWS Global Network and its edge locations around the world
- both integrate with AWS Shield for DDoS protection
- CloudFront
  - improves performance for both cacheable content (such as images and videos)
  - dynamic content (such as API acceleration and dynamic site delivery)
  - content is served at the edge
- Global Accelerator
  - improves performance for a wide range of applications over TCP or UDP
  - proxying packets at the edge to applications running in one or more AWS Regions
  - good fit for non-HTTP use cases, such as gaming (UDP), IoT (MQTT), or Voice over IP
  - good for HTTP use cases that require static IP addresses
  - good for HTTP use cases that require deterministic, fast regional failover
