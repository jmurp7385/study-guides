### Classic Solutions Architecture Discussions

### rogression of Solutions Architect mindset thru several case studies

## 5 Pillars for a well architected application

- Costs
- Performance
- Reliability
- Security
- Operational Excellence

## WhatsTheTime.com

- stateless web app
- allows people to know what time it is
- dont need database
- start small and can accept downtime
- want to fully scale vertically and horizontally with 0 downtime

### Solution 1

- EC2
  - t2
  - elastic IP Address

### Solution 2

- EC2
  - Vertical Scaling (downtime while upgrading)
  - change instance to m5

### Solution 3

- EC2
  - Horizontal Scaling
  - add more ec2 instances with elastic IPs

### Solution 4

- EC2
  - m5
- Route 53
  - DNS query for api.whatisthetime.com
  - A Record
  - TTL 1 Hour
  - in front of ec2 intances
  - still has downtime if instance fails

### Solution 5

- elb + health checks => private EC2 instances
  - security groups to only allow traffic to EC2 from ELB
- Route 53
  - alias record to ELB
  - TTL 1 Hour

### Solution 6

- ELB + Health checks
- EC2
  - M5
  - Auto scaling group
- Route 53
  - alias record to ELB
  - TTL 1 Hour

### Solution 7

- ...solution 5
- multi AZ

### Solution 8

- ...solution 6
- minimum 2 AZ lets reserve capacity to save costs

## MyClothes.com

- stateful web app
- allows users to buy clothes online
- shopping cart
- need to scale, maintain horizontal scaling
- keep web application as stateless as possible
- should not lose their shopping cart
- should have their personal details in a database

### Solution 1 (initial)

- Route 53
- Multi AZ ELB
  - stickiness (session affinity)
- EC2
  - auto scaling group

### Solution 2 (shopping cart)

- send shopping cart in cookies
  - stateless
  - http requests are heavier
  - security risk (cookies can be altered)
  - cookies must be validated
  - cookies must be less than 4KB
- Route 53
- ELB
  - Multi AZ
- EC2
  - auto scaling group
    - Multi AZ

### Solution 3 (shopping cart)

- send session_id in cookies
- Route 53
- ELB
  - Multi AZ
- EC2
  - auto scaling group
    - Multi AZ
- ElastiCache
  - store/retrieve session data from session_id

### Solution 4 (storing user data)

- ...solution 3
- RDS
  - DynamoDB
  - store/retrieve user data (address, name, etc)

### Solution 5 (scaling reads)

- ...solution 4
- RDS
  - type one (read replicas)
    - Master writes
    - read replicas
    - store/retrieve user data (address, name, etc)
  - type two (write thru)
    - checks cache
    - read rds
    - cache again

### Solution 6 (multi az)

- send session_id in cookies
- Route 53
- ELB
  - Multi AZ
- EC2
  - auto scaling group
    - Multi AZ
- ElastiCache
  - multi AZ
  - store/retrieve session data from session_id
- RDS
  - multi AZ
  - checks cache
  - read rds
  - cache again

### Solution 6 (security groups)

- send session_id in cookies
- Route 53
  - open 0.0.0.0/0
- ELB
  - Multi AZ
  - restrict traffic to EC2 Security Group from the LB
- EC2
  - auto scaling group
    - Multi AZ
- ElastiCache
  - restrict traffic to ElastiCache Security Group from the EC2 Security Group
  - multi AZ
  - store/retrieve session data from session_id
- RDS
  - restrict traffic to RDS Security Group from the EC2 Security Group
  - multi AZ
  - checks cache
  - read rds
  - cache again

## MyWordPress.com

- fully scalable wordpress website
- access and correctly display picture uploads
- user data and blog content should be stored in a MySQL database

### Solution 1 (start)

- Route 53
- ELB
  - Multi AZ
  - EC2 (m5)
    - auto scaling group
    - Multi AZ
- RDS Aurora
  - multi AZ
  - Read Replicas

### Solution 2 (storing images)

- EBS Volume (single instance)
  - image is stored on volume
  - with scaling each EC2 isntance has its own EBS volume
- EFS (distributed app)
  - ENI connected to all EC2 isntances to access EFS drive
  - all storage shared with isntances
  - more expensive (need to determine trade offs)

## Instantiating Applcations quickly

- When launching a full stack it can take time to
  - install applications
  - insert initial (or recovery) data
  - configure everything
  - launch the application
- EC2
  - Golden AMI: install applications, os deps, etc beofre and launch EC2 from the Golden AMI
  - Bootstrap using User Data: for dynamic config
  - Hybrid: mix of user data and golden AMI (Elastic Beanstalk)
- RDS Databases
  - restore from snapshot: database will have schemas and data ready
    - better than big insert statment
- EBS Volumes
  - restore from snapshot: the disk will be formatted and have the data

## Beanstalk Overview

- Developer problems on AWS
  - managing infrastructure
  - deploying code
  - configuring all the databases, load balancers, etc
  - scaling concerns
- most web apps have the same architecture (ALB + ASG)
- all developers want is the code to run
  - possibly, consistently acrosds different applications and environments
- Beanstalk
  - developer centric view of deploying an application on AWS
  - managed service
    - automatically handles capacity provisioning, load balancing, scaling, application health monitoring, isntance configuration, etc
    - just application code is responsiblity of developer
  - still have full control over configuration
  - free, pay for underlying instances
  - Components
    - Application: collection of Elastic Beanstalk components (environments,  versions, configs)
    - Application Version: an interation of application code
    - Environment
      - collection of AWS resources running an application version (one version at a time)
      - Tiers: Web Server Environment Tier & Worker Environment Tier
        - web
          - web architecture ALB + ASG
        - worker
          - scale based on sqs message queue
            - can push to sqs queue from other Web Tier
          - ec2 isntances are workers
      - multiple environments (dev,qa,prod)