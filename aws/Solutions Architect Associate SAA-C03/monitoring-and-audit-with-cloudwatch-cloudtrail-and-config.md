# AWS Monitoring and Audit: CloudWatch, CloudTrail, Config

## Cloudwatch Metrics

- CloudWatch provides metrics for *every* service in AWS
- **Metric** is a variable to monitor (CPUUtilization, NetworkIn,...)
- metrics belong to **namespaces**
- **Dimension** is an attribute of a metric (instance id, environment, etc,...)
- up to 10 dimensions per metric
- metrics have **timestamps**
- can create CloudWatch dashboards of metrics
- can create **CloudWatch Custom Metrics** (for the RAM for example)

### CloudWatch Metric Streams

- continually stream CloudWatch metrics to a destination with **near-real-time delivery** and low latency
- use Amazon Kinesis Data Firehose to stream to destinations
- destinations could be a 3rd party
  - DataDog, Dynatrace, New Relic, Splunk, Sumo Logic
- option to filter metrics to only stream a subset of them

## CloudWatch Logs

- **Log Groups**: arbitrary name, usually representing an application
- **Log Steam**: instances within applications / log files / containers
- can define log expiration policies (never expire, 30 days, etc, ...)
- **CloudWatch Logs can send logs to**
  - Amazon S3
  - Kinesis Data Streams
  - Kinesis Data Firehose
  - AWS Lambda
  - ElasticSearch / OpenSearch
- **Sources**
  - SDK, CloudWatch Logs Agent, CloudWatch Unified Agent
  - Elastic Beanstalk: collection of logs from application
  - ECS: collection from containers
  - AWS Lambda: collection from function logs
  - VPC Flow Logs: VPC specifc logs
  - API Gateway
  - CloudTrail based on filter
  - Route 53: log DNS queries
- Metric Filter & Insights
  - CloudWatch Logs can use filter expressions
    - for example, find a specific IP inside of a log
    - or cont occurences of "ERROR" in your log
  - metric filters can be used to trigger CloudWatch Alarms
  - CloudWatch Logs Insights can be used to query logs and add queries to CloudWatch Dashboards
- S3 Export
  - log data can take up to 12 hours to become available for export
  - the API call is CreateExportTask
  - not near-real time or real-time, use Logs Subscriptions instead
- Logs Aggregation, Multi-Account, Multi-Region: use subscription filters to pipe logs to Kinesis Data Streams

## CloudWatch Agent & CloudWatch Logs Agent

- CloudWatch Logs for EC2
  - by default no logs from EC2 machine will go to CloudWatch
  - need to run a CloudWatch agent on EC2 to push the log files you want
  - make sure Iam permissions are correct
  - The CloudWatch Log agent can be setup on-premise too
- agents are for virtual servers (EC2 instances and on-premise servers)
- CloudWatch Logs Agent (old)
  - old version of the agent
  - can only send to CloudWatch Logs
- CloudWatch Unified Agent (new)
  - collect additional system-level metrics such as RAM, processes, etc, ...
  - collect logs to send to CloudWatch Logs
  - centralized configuration using SSM Parameter Store
  - collected directly on your linux server / ec2 instance
  - CPU: active, idle, guest, system, user, steal
  - Disk Metrics: free, used, total, Disk I/O (reads, writes, bytes, iops)
  - RAM: free, inactive, used, total, cached
  - Netstat: number of TCP and UDP connections, net packets, bytes
  - Processes: total, dead, bloqued, idle, running, sleep
  - Swap Space: free, used, used %
  - **Reminder: out-of-the-box metrics for EC2**

## CloudWatch Alarms

- alarms can trigger notifications for any metric
- Various options (sampling, %, max, min, etc, ...)
- Alarm States: OK, INUSUFFICIENT_DATA, ALARM
- Period: lenght of time in seconds to evaluate metric, high resolution custom metrics (10s,30s or multiples of 60s)
- Alarm Targets
  - stop, terminate, reboot, or recover an EC2 instance
  - trigger auto scaling action
  - send notification to SNS (from which you can pretty much do anything)
- Composite Alarms
  - monitor states of multiple other alarms
  - **AND** and **OR** conditions
  - reduce "alarm noise" by creating comlex composite alarms
- EC2 Instance Recovery
  - Status Check
    - instance status = check the EC2 VM
    - system status = check the underlying hardware
    - recovery: same private, public, Elastic IP, metadata, placement group
- Good to know
  - alarms can be created based on CloudWatch Logs Metrics Filters
  - to test alarms and notifications, set the alarm stat to Alarm using CLI
    - `aws cloudwatch set-alarm-state --alarm-name "myalarm" --state-value ALARM --state-reason "testing"`

## EventBridge (CloudWatch Events)

- schedule: Cron jobs (scheduled scripts)
- event pattern: event rules react to a service doing something
  - trigger lambda functions, send SQS/SNS messages
- Default Event Bus
  - default EventBridge location for AWS Services
- Partner Even Bus
  - 3rd party can send events to EventBridge
- Custom Event Bus
  - send events to EventBridge from custom applications
- Event buses can be accessed by other AWS accounts using Resource-based Policies
- can archive events (all/filter) sent to an Event Bus (indefinitely or set period)
- ability to **replay archived events**
- Schema Registry
  - EventBridge can analyze the events in your bus and infer the **schema**
  - allows you to generate code for youre application that will know in advance how data is structured in the event bus
  - schema can be versioned
- Resouce-based Policy
  - manage permissions for a specific Event Bus
    - allow/deny evnts from another AWS account or AWS region
  - use case:
    - aggregate all events from your AWS Org in a single AWS account or region

## CloudWatch Insights and Operational Visibility

- Container Insights
  - collect, aggregate, summarize metrics, and logs from containers
  - available for containers on
    - Amazon Elastic Container Service (ECS)
    - Amazon Elastic Kubernetes Service (EKS)
    - Kuberneteres Platforms on EC2
    - Fargate (both for ECS and EKS)
  - **Amazon EKS and Kubernetes, CloudWatch Insights is using a containerized version of the CloudWatch Agent to discover containers**
- Lambda Insights
  - monitoring and troublshooting solution for serverlss applications running on AWS Lambda
  - collects, aggregates, and summerizes system-level metrics including CPU time, memory, disk, and network
  - collects, aggregates, and summerizes diagnostic information such as cold starts and Lambda worker shutdowns
  - Lambda Insights is provided as a Lambda Layer
- Contributer Insights
  - analyze log data and create time series that display contributor data
    - **see metrics about the top-N contributors**
    - the total number of unique contributors and their usage
  - helps find top talkers and understand who or what is impacting system performance
  - works for any AWS-generated logs (VPC, DNS, etc,...)
  - example
    - can find bad hosts, **identify the heaviest network users**, or find the URLs that generate the most errors
  - can build rules from scratch, or you can also use sample rules that AWS has created - **leverages CloudWatch Logs**
  - CloudWatch also provides built-in rules that you can use to analyze metrics from other AWS services
- Application Insights
  - **provides automated dashboards that show potential problems with monitored applications, to help isolate ongoing issues**
  - applications run on Amazon EC2 instances with select technologies only (Java, .NET, Microsoft IIS Web Server, databases)
  - you can use other AWS resources such as EBS, RDS, ELB, ASG, Lambda, SQS, SNS, DynamoDB, S3, ECS, EKS, API Gateway
  - powered by SageMaker
  - enhanced visibility into your applicatiojn health to reduce the time it wil take you to troubleshoot and repair your application
  - findings and alerts are sent to Amazon EventBridge and SSM OpsCenter
- Summary
  - CloudWatch Container Insights
    - ECS, EKS, Kubernetes on EC2, Fargate, need agent for kubernetes
    - metrics and logs
  - CloudWatch Lambda Insights
    - detailed metrics to troubleshoot serverless applications
  - CloudWatch Contributors Insights
    - find "Top-N" contributors through CloudWatch Logs
  - CloudWatch Application Insights
    - automatic dashboard to troublshoot your application and related AWS services

## CloudTrail

- **provides governance, compliace, and audit for your AWS account**
- enabled by default
- get **a history of events / API calls made within your AWS Acount by**
  - console, SDK, CLI, AWS Services
- can put logs from CloudTrail into CloudWatch Logs or S3
- **a trail can be applied to All Regions (defualt) or a single Region**
- if a resource is deleted in AWS, investigate CloudTrail first

### CloudTrail Events

- management events
  - operations that are performed on resources in your AWS Account
  - examples:
    - configuring security (IAM **AttachRolePolicy**)
    - configuring rules for routing events (Amazon EC2 **CreateSubnet**)
    - setting up logging (AWS CloudTrail **CreateTrail**)
  - **by default trails are configured to log management events**
  - can seperate **Read Events** (that dont modify resources) from **Write Events** (that may modify resources)
- Data Events
  - **by default, data events are not logged (because high volume operations)**
  - Amazon S3 object-level activity (ex: **GetObject, DeleteObject, PutObject**): can seperate Read & Write Events
  - AWS Lambda function execution activity (Invoke API)
- Insights Events
  - enable CloudTrail Insights to detect unusual activity in your account
    - inaccurate resource provisioning
    - hitting service limits
    - bursts of AWS IAM actions
    - gaps in periodic maintenance activity
  - analyzes normal management events to create a baseline and **continuously analyze write events to detect unusal patterns**
    - anomalies appear in the CloudTrail Console
    - event is sent to S3
    - and EventBridge event is generated (for automation needs)
- Retention
  - stored for 90 days in CloudTrail
  - to keep beyond this period, use S3 to store and Athena to analyze

## AWS Config

- helps with auditing and recording **compliance** of AWS resources
- helps record configurations and changes over time
- Questions that can be solved by AWS Config
  - is there unrestricted SSH access to my security groups
  - do my buckets have ant public access
  - how has my ALB configuration changed over time
- you can recieve alerts (SNS notifications) for any changes
- AWS Config is a per-region service
- can be aggregate across regions and accounts
- possibility of storing the configuration data into S3 (analyzed by Athena)

### Config Rules

- AWS managed rules (over 75)
- can make custom rules (must be defined in AWS Lambda)
  - ex: evaluate if each EBS disk is of type gp2
  - ex: evaluate if each EC2 isntance is t2.micro
- rules can be evaluated/triggered
  - for each config change and/or regular time intervals
- **AWS Config Rules do not prevent actions from happening (no deny)**
- Pricing: no free tier, #0.003 per configuration item recorded per region, $0.001 per config rule evaluation per region

### Config Resources

- view compliance of a resource over time
- view configuration of a resource over time
- view CloudTrail API calls of a resource over time

### Config Remidiations

- automate remediation of non-compliant resources using SSM Automation Documents
- use AWS-managed Automation Documents or create custom Automation Documents
  - can create a custom Automation Document that invokes a Lambda Function
- Can set **Remediation Retries** if the resource is still non-compliant after auto-remediation

### Config Notifications

- use EventBridge to trigger notifications when AWS resources are non-compliant
- ability to send configuration changes and compliance state notifications to SNS (all events - use SNS Filtering or filter at client-side)

## CloudTrail vs CloudWatch vs Config

- CloudWatch
  - performance monitoring (metrics, CPU, network, etc, ...) & dashboards
  - Events & Loggings
  - Log aggregation & analysis
- CloudTrail
  - record API calls made within your account by everyone
  - can define trails for specific resources
  - Global Service
- Config
  - record configuration changes
  - evaluate resources against compliance rules
  - get timeline of changes and compliance

### Example for an Elastic Load Balance

- CloudWatch
  - monitor incoming connections metric
  - visualize error codes as a % over time
  - make a dashboard to get an idea of your load balancer performance
- Config
  - track security group rules for the Load Balancer
  - track configuration changes for the Load Balancer
  - ensure an SSL certificate is always assigned to the load balancer (compliance)
- CloudTrail
  - track who made any changes to the Load Balancer with API calls