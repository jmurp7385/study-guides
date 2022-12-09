# Serverless

- serverless is a new paradigm in which the developers dont have to manage servers anymore
- just deploy code/functions
- initially serverless == Faas (Function as a Service)
- serverless was pioneered by AWS Lambda but now also includes anything that's managed
  - databases, messaging, storage, etc
- **serverless does not mean there are no servers, it means you don't manage/provison them**
- Serverless in AWS
  - AWS Lambda
  - DynamoDB
  - AWS Cognito
  - AWS API Gateway
  - Amazon S3
  - AWS SNS & SQS
  - AWS Kinesis Data Firehose
  - Aurora Serverless
  - Step Functions
  - Fargate

## Lambda

- virtual functions - no servers to manage
- limited by time - **short executions**
- run on-demand
- **scaling is automated**
- Benefits
  - easy pricing
    - pay per request and compute time
    - free tier of 1,000,000 AWS Lambda requests and 400,000 GBs of compute time
  - integrated with the whole AWS suite of services
  - integrated with many programming languages
  - easy monitoring through AWS CloudWatch
  - easy to get more resources per function (up to 10GB of RAM)
  - increasing RAM also improves CPU and Network
- Language Support
  - Node.js
  - Python
  - Java
  - C# (.NET Core)
  - Golang
  - C# / Powershell
  - Ruby
  - Custom Runtime API (community supported, example Rust)
  - Lambda Container Image
    - the container must implement the Lambda Runtime API
    - ECS/Fargate is preferred for running arbitrary Docker images
- Service Integration Example: Serverless Thumbnail creation
  - image upload to S3 -> lambda function creates thumbnail to S3 -> lambda functon save metadata to DynamoDB
- Service Integration Example: Serverless CronJob
  - CloudWatch Events Event Bridge -> every hour -> AWS lambda performs task
- Pricing
  - pay per call
    - first 1,000,000 are freee
  - pay per duration (incremements of 1ms)
    - 400,000 GB-seconds of compute time per month if FREE
    - == 400,000 seconds if function is 1GB RAM
    - == 3,200,000 seconds if function is 128MB RAM
    - after that, $1 for 600,000 GB-seconds
  - usually very cheap so it is very popular

### Lambda Limits

- per region limits
- Execution
  - Memory Allocation: 128MB - 10GB (1MB increments)
  - Maximum execution time: 900 seconds (15 minutes)
  - Environment Variables: 4KB
  - Disk Capacity is the "function container" (in /tmp): 512MB - 10GB
  - Concurrency executions: 1000 (can be increased)
- Deployment
  - Lambda functions deployment size (compressed .zip): 50MB
  - Size of uncompressed deployment (code + dependencies): 250MB
  - Can use the /tmp directory to load other files at startup
  - Size of environment variables: 4KB

### Lambd@Edge & Cloudfront Functions

- Many modern applications execute some form of logic at the edge
- Edge Function
  - code written and attached to CloudFront distributions
  - runs close to users to minimize latency
- CloudFront provides two types: **CloudFront Functions & Lambda@Edge**
- pay for what you use
- fully serverless
- Use Cases:
  - customize CDN content
  - website security and privacy
  - dynamic web applications at the edge
  - Search Engine Optimization
  - itelligently Route Across Origins and Data Centers
  - Bot mitigation at the Edge
  - Real-time image transformation
  - A/B testing
  - User Authentication & Authorization
  - User Prioritization
  - User tracking and Analytics
- CloudFront Functions
  - lighweight functions written in JavaScript
  - for high-scale, latency-sensitive CDN customizations
  - sub-ms startup timesa, **millions of requests/second**
  - user to change viewer requests and responses
    - Viewer Request: after CloudFront receives a request from a view
    - Viewer Response: before CloudFront forwards a response to a viewer
  - Native feature of CloudFront (manage code entirely within CloudFront)
- Lambda@Edge
  - Lambda functions written in Node.js or Python
  - scales to **1000s of requests/second**
  - Used to change CloudFront requests & responses
    - Viewer Request: after CloudFront receives a request from a view
    - Origin Request: before CloudFront forwards a request to the origin
    - Origin Response: after CloudFront recieves a response from the origin
    - Viewer Response: before CloudFront forwards a response to a viewer
  - author functions in one AWS Region (us-east-1), then CloudFront replicates to its locations
- Use Cases
  - CloudFront Functions
    - Cache key normalization
      - transform request attributes (headers, cookies, query strings, URL) to create an optimal cache key
    - header manipulation
      - insert/modify/delete HTTP headers in the request/response
    - URL reqwrites or redirects
    - Request authentication/authorization
      - create and validate user-generated tokens (JWT) to allow/deny requests
  - Lambda@Edge
    - longer execution time (several ms)
    - Adjustable CPU memory
    - code depends on 3rd party libraries (AWS SDK to access other services)
    - network access to use external services for processing
    - file system access or access to the body of HTTP requests

|                                    |       CloudFront Functions        |                    Lambda@Edge                    |
| :--------------------------------: | :-------------------------------: | :-----------------------------------------------: |
|          Runtime Support           |            JavaScript             |                 Node.js & Python                  |
|           # of Requests            |     Millions requests/second      |             Thousands requests/second             |
|        CloudFront Triggers         |      Viewer Request/Response      | Viewer Request/Response & Origin Request/Response |
|        Max. Execution Time         |               < 1ms               |                   5-10 seconds                    |
|             Max Memory             |                2MB                |                   128 MB - 10GB                   |
|         Total Package Size         |               10KB                |                     1MB-50MB                      |
| Network Access, File System Access |                NO                 |                        YES                        |
|     Access to the request body     |                NO                 |                        YES                        |
|              Pricing               | free tier, 1/6th pricing of @Edge |   no free tier, charged per request & duration    |

### Lambda in VPC

- by default, Lambda function is launched outside your own VPC (in an AWS-owned VPC)
  - cannot access resources in you VPC (RDS, ElastiCache, internal ELB)
- must define VPC ID, subnets, and security groups
- Lambda will create an ENI (Elastic Network Interface) in your subnets
- Lambda with RDS Proxy
  - if lambda functions directly access your database, they may open to many applications under high load
  - RSD Proxy
    - improves scalability by pooling and sharing DB connections
    - improve availability by reducing failover time by 66% and preserving connections
    - improve security by enforcing IAM authentication and storing of credentials in Secrets Manager
  - lambda function MUST be deployed in your VPC
    - **RDS Proxy is never publicly accessible**

## DynamoDB

- fully managed, highly available with replication across multiple AZs
- NoSQL database - not a relational database - with transaction support
- scals to massive workloads, distributed database
- millions of requests per seconds, trillions of rowqs, 100s of TB of storage
- fast and consitent in performance (single digit ms)
- integrated with IAM for security, authorization, and administration
- low cost and auto-scaling capabilities
- no mainenance or patching, always available
- Standard & Infrequent Access (IA) Table Class
- Basics
  - made of tables
  - each table has a Primary Key (decided at creation time)
  - each table can have infinite number of items (rows)
  - each item has attributes (can be added over time, can be null)
  - Maximum size of an item is 400KB
  - Data types supported are
    - Scalar Types - String, Number, Binary, Boolean, Null
    - Document Types - List, Map
    - Set Types - String Set, Number Set, Binary Set
  - **can rapidly evolve schemas in DynamoDB**
- Read/Write Capacity Modes
  - control how you manage your tables capacity (read/write throughput)
  - Provisioned Mode (defualt)
    - specify number of read/writes per second
    - need to plan capacity beforehand
    - pay for provisioned Read Capacity Units (RCU) and Write Capacity Units (WCU)
    - possibility to add **auto-scaling** mode fo RCU & WCU
  - On-Demand Mode
    - read/writes automatically scale up/down with your workloads
    - no capacity planning needed
    - pay for what you use, more expensive
    - great for **unpredictable** workloads, **steep sudden spikes**

### Advanced Features

- DynamoDB Acclerator (DAX)
  - fully-managed, highly available, seamles, in-memory cache for DynamoDB
  - **help solve read congestion by caching**
  - ***Microseconds* latency for cached data**
  - doesn't require application logic modification (compatible with existing DynamoDB APIs)
  - 5 minutes TTL for cache (default)
- DAX vs ElastiCache
  - ElastiCache
    - store aggregation resultrs
  - DAX
    - individual objects, query/scan cache
- Stream Processing
  - ordered stream of item-level modifications (create/update/delete) in a table
  - use cases
    - react to changes in real-time (welcome email to new users)
    - real-time usage analytics
    - insert into derivative tables
    - implement cross-origin replication
    - invoke AWS Lambda on changes to your DynamoDB table
  - 24 hours of retention
  - limited # of consumers
  - process using AWS Lambda Triggers or DynamoDB Steam Kinesis Adapter
  - Kinesis Data Streams (newer)
    - 1 year retention
    - high # of consumers
    - process using AWS Lambda, Kinesis Data Analytics, Kinesis Data Firehose, AWS Glue Steaming ETL, ...
- Global Table
  - table replicated across multiple regions
  - make accessible with **low latency** in multiple regions
  - active-active replication
  - Applications can Read and Write to the table in any region
  - must enable DynamoDB Streams as a pre-requisite
- Time To Live (TTL)
  - automatically delete items after expiry timestamp
  - use cases
    - reduce stored data by only keeping current items
    - adhere to regulatory obligations
    - web session handling
- Backups of Disaster Recovery
  - continuous backups using point-in-time recovery (PITR)
  - Point-in-time recovery to any time within the backup window
  - recovery process creates a new table
- On-demand backups
  - full backups for long-term retention, until explicitly deleted
  - doesn't affect performance or latency
  - can be configured and managed in AWS Backup (enables cross-region copy)
  - recovery process creates a new table
- Integration with S3
  - can export to S3 (must enable PITR)
    - works for any point in time for the last 35 days
    - doesn't effect read capacity of table
    - perform data analysis on top of DynamoDB
    - retain snapshots for auditing
    - ETL on top of S3 data before importing it back into DynamoDB
    - exported in JSON or ION format
  - Import from S3
    - import CSV, DynamoDB JSON, or ION format
    - doesn't consume any write capacity
    - creats a new table
    - import errors are logged in CloudWatch Logs

## API Gateway

- AWS Lambda & API Gateway: no infrastructure to manage
- support for WebSocket Protocol
- handle API versioning
- handle different environments
- handle security (authentication/authorization)
- create API keys, handle request throttling
- Swagger/Open API import to quickly define APIs
- transform and validate requests and responses
- generate SDK and API specifications
- cache API responses
- Integrations
  - Lambda
    - invoke Lambda function
    - easy way to expose REST API backed by AWS Lambda
  - HTTP
    - expose HTTP endpoints in the backend
    - example: internal HTTP API on-premise, Application Load Balancer
    - why? add rate limiting, caching, user authentications, API Keys, etc
  - AWS Service
    - expose any AWS API thru the API Gateway
    - example: start and AWS Step Function workflow, post message to SQS
    - why? add authentication, deploy publicly, rate control
- Endpoint Types
  - Edge-Optimized (default): For Global Clients
    - requests are routed through the CloudFront Edge locations (improves latency)
    - API Gateway still only lives in one region
  - Regional
    - for clients within the same region
    - could manually combine with CloudFront (more control over the caching strategies and distrobution)
  - Private
    - can only be accessed thru your VPC using an interface VPC endpoint (ENI)
    - use a resource policy to define access
- Security
  - User Authentication through
    - IAM Roles (useful for internal applications)
    - Cognito (itendity for external users, e.g., mobile users)
    - Custom Authorizer (your own logic)
  - Custom Domain Name HTTPS security through integration with AWS Certificate Manager (ACM)
    - if using Edge-Optimized, the certificate must be in **us-east-1**
    - if using Regional, the certificate must be in the API Gateway Region
    - must setup CNAME or A-alias record in Route 53

## Step Functions

- build serverless visual workflow to orchestrate your Lambda Functions
- Features: sequence, parallel, conditions, timeouts, error handling
- con integraete with EC2, ECS, on-premise servers, API Gateway, SQS Queues, etc
- possible to implement a human approval feature
- use cases: order fulfilment, data processing, web applications, any workflow
