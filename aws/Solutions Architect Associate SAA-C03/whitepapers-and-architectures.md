# Whitepapers and Architectures

## AWS Well-Archictected Framework and Well-Architected Tool

- [Well Architected Framework](https://aws.amazon.com/architecture/well-architected)
- stop guessing at capacity needs, use Auto Scaling Groups
- test systems at production scale
- automate to make your architectural experimentation easier
  - design based on changing requirments
- driver architectures using data
- improve though game days
  - simulate applications for flash sale days

### 6 Pillars of a Well-Architected Framework

1. Operation Excellence
2. Security
3. Reliability
4. Performance Efficiency
5. Cost Optimization
6. Sustainability

- **they are not something to balance, or trade-offs, they're a synergy**

### Well-Architected Tool

- [Well-Architected Tool](https://console.aws.amazon.com/wellarchitected/)
- [Well-Architected Tool Docs](https://docs.aws.amazon.com/wellarchitected/latest/userguide/intro.html)
- free tool to **review you architectures** against the 6 pillars of a Well-Architected Framework and **adopt architectural best practices**
- how does it work?
  - select your workload and answer questions
  - review your answers against the 6 Pillars
  - obtain advice: get videos and documentation, generate a report, see the results in a dashboard

## AWS Trusted Advisor Overview

- no need to install anything - high level AWS account assesment
- **Analyze your AWS accounts and provide recommendations**
  - cost optimization
    - low utilization EC2 instances, idle load balancers, under-utilized EBS Volumes
    - reserved instances & savings planß optimizations
  - performance
    - high utilization EC2 instances, CloudFront CDN optimizations
    - EC2 to EBS throughput optimizations, Alias records recommendations
  - security
    - MFA enabled on Root Account, IAM key rotation, exposed Access Keys
    - S3 bucket permissions for public access, security groups with unrestricted ports
  - fault tolerance
    - EBS Snapshot age, Availability Zone Balance
    - ASG Multi-Zone, RDS Multi-AZ, ELB Configurations
  - service limits
- core checks and recommendations - all customers
- can enable weekly email notifications from the console
- **Full Trusted Advisor** - Available for **Business & Enterprise** supports plans
  - ability to set CloudWatch alarms when reaching limits
  - **Programmatic Access using AWS Support API**

## Example of Architectures

- **Classic Patterns**: EC2, ELB, RDS, ElastiCache, etc,...
- **Severless Patterns**: S3, Lambda, DynamoDB, CloudFront, API Gateway, etc, ...
- [Architecture](https://aws.amazon.com/architecture)
- [Solutions](https://aws.amazon.com/solutions)
