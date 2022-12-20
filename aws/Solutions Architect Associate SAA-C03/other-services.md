# Other Serices

## CloudFormation

- Cloud Formation is a declaritive way of outlining your AWS Infrastructure, for any resources (most of them are supported)
- for example
  - i want a security group
  - i want 2 EC2 instances using the above security group
  - i want an S3 bucket
  - i want a load balancer (ELB) in front of these machines
- then CloudFormation creates those for you, in the **right order**, with the **exact configuration** you specify
- benefits
  - Infrastructure as Code
    - no resources are manually created, which is excellent for control
    - changes to infrastructure are reviewed thru code
  - Cost
    - each resource within the stack is tagged with an identifier so can easy see how much a stack costs you
    - you can estimate the costs of your resources using the CloudFormation Template
    - saving strategy: in dev, you could have automatic deletion of templates at 5pm and recreated at 8am, safely
  - Productivity
    - ability to destroy and re-create an infrastructure on the cloud on the fly
    - automated generation of Diagram for your templates
      - shows all resources and relations between the components
    - declaritive programming (no need to figure out ordering and orchestration)
  - Don't reinvent the wheel
    - leverage exisiting templates on the web
    - leverage the documentation
  - **Supports (almost) all AWS resources**
    - everything we'll see in this course is supported
    - you can use the "custom resources" for resources taht are not supported

## Amazon Simple Email Service SES

- fully managed service to send emails securely, globally, and at scale
- allows inbound/outbound emails
- reputation dashboard, performance insights, anti-spam feedback
- provides statistics such as email deliveries, bounces, feedback loop results, email open
- supports DomainKeys Identified Mail (DKIM) and Sender Policy Framework (SPF)
- flexible IP deployment: shared, dedicated, and customer-owned IPs
- send emails through your application using AWS Console, APIs, or SMTP
- use cases: transactional, marketing, and bulk email communications

## Amazon Pinpoint

- scalable 2-way (outbound/inbound) marketing communications service
- supports email, SMS, push, voice, and in-app messaging
- ability to segment and personalize messages with the right content to customers
- possibility to recieve replies
- scales to billions of messages per day
- use cases: run campaigns by sending marketing bulk, transactional SMS messages
- versus SNS or SES
  - SNS & SES you manage each messages audience, content, and delivery schedule
  - Pinpoint you can create message templates, delivery schedules, highly-targeted segments, and full campaigns

## SSM Session Manager

- allows you to start a secure shell on your EC2 and on-premise servers
- no ssh access, bastion hosts, or ssh keys needed
- no port 22 needed (better security)
- supports MacOS, Linux, and Windows
- send session log data to S3 or CloudWatch logs

### SSM Other Services

#### Run Command

- execute a document (=script) or just run a command
- run command across multiple instances (using resource groups)
- no need for SSH
- command output can be show in the AWS Console, sent to S3 bucket, or CloudWatch Logs
- send notifications to SNS about command status (In Progress, Success, Failed,...)
- integrated with IAM & CloudTrail
- can be invoked using EventBridge

#### Patch Manager

- automatess the process of patching managed instances
- OS updates, aplications updates, security updates
- supports EC2 instances and on-premise servers
- supports MacOS, Linux, and Windows
- patch on-demand or on a schedule using **Maintenance Windows**
- scan instances and generate patch compliance report (missing patches)

#### Maintenance Windows

- defines a schedule for when to perform actions on your instances
- exmaple: OS patching, updating drivers, installing software,...
- maintenance window contains
  - schedule
  - duration
  - set of registered instances
  - set of regstered tasks

#### Automation

- simplifies common maintenance and deployment taskks of EC2 instances and other AWS resources
- examples: restart instances, create an AMI, EBS snapshot
- Automation Runbook: SSM Documents to define actions performed on your EC2 isntances or AWS resources (pre-defined or custom)
- can be triggered using
  - Manually using AWS Console, AWS CLI / SDK
  - Amazon EventBridge
  - On a schedule using Maintenance Windows
  - By AWS Config for rules remediations

## AWS Cost Explorer

- visualize, understand, and manage your AWS costs and usage overtime
- create custom reports that analyze cost and usage data
- analyze your data at a high level: total costs and useage across all accounts
  - monthly, hourly, resource level granularity
- choos an optimal **Savings Plan** (to lowers prices on your Bill)
- **Forecast up to 12 months based on previouis usage**

## Elastic Transcoder

- is used to **convert media files stored in S3 into media files in the formats required by consumer playback devices (phones, etc)**
- benefits
  - easy to use
  - highly scalable - can handle large volumes of media files and large file sizes
  - cost effective - duration-based pricing model
  - fully managed & secure, pay for what you use

## AWS Batch

- **fully managed batch processing at any scale**
- efficiently run 100,000s of computing batch jobs on AWS
- a "batch" job is a job with a start and an end (opposed to continous)
- batch with dynamically launch EC2 instances or Spot Instances
- AWS Batch provisions the right amount of compute / memory
- you submit or schedule the batch jobs and AWS batch does the rest
- batch jobs are defined as **Docker images** and run on **ECS**
- helpful for cost optimizations and focusing less on the infrastructure
- Batch vs Lambda
  - Lambda
    - time limit
    - limited runtimes
    - limited temporary disk space
    - severless
  - Batch
    - no time limit
    - any runtime as long as it is packaged as a Docker image
    - rely on EBS/instance store for disk space
    - relies on EC2 (can be managed by AWS)

## AWS AppFlow

- fully managed integration sevrice taht enables you yo securely transfew data between **Software-as-a-Sevice (SaaS) applications and AWS**
- **Sources**: Salesforce, SAP, Zendesk, Slack, and ServiceNow
- **Destinations**: AWS Services like AWS S3, Redshift, or non AWS such as SnowFlake and Salesforce
- **Frequency**: on a schedule, in response to events, or on demand
- **Data Transformation** capbilities like filtering and validation
- **Encrypted** over the public internet or privately over AWS PrivateLink
- dont spend time writing integrations and leverage the APIs immediately
