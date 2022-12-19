# Disaster Recovery and Migrations

- A **Disaster** is any event that has a negative impact on a company's business continuity or finances is a disaster
- **Disaster Recovery (DR)** is about preparing for and recovering from a disaster
- What kind of disaster recovery?
  - On-premise -> On-premise: traditional DR, very expensive
  - On-premise -> AWS Cloud: hybrid discover
  - AWS Cloud Region A -> AWS Cloud Region B
- Recovery Point Objective (RPO)
  - how much of a disaster can you handle?
  - how much of a data loss can you handle if a disaster happens?
- Recovery Time Objective (RTO)
  - how long is the downtime on your application

## Disaster Recovery Strategies

### Backup and Restore

- Slowest RTO
- High RPO
- Cheapest
- on-premise with storage gateway or snowball device (snow ball could be up to a week of RPO)
- AWS Cloud with regular snapshots
  - Higher RTO and RPO

### Pilot Light

- 2nd Slowest RTO
- 2nd Cheapest
- a small version of the app is always running in the cloud
- useful for the critical core (pilot light)
- very similar to Backup and Restore
- faster than Backup and Restore since critical systems are already up and running

### Warm Standby

- 2nd Fastest RTO
- 2nd Most Expensive
- full system up and running, but at minimum size, and upon distaster, we scale to production load

### Hot Site / Multi Site Approach

- Fastest RTO (minutes or seconds)
- Most Expensive
- full production scale is running AWS or On-Premise
- Multi Region
  - full production scale in multiple regions

## Disaster Recovery Tips

- Backup
  - EBS Snapshots, RDS Automated Backups / Snapshots, etc
  - regular pushes to S3 / S3 IA / Glacier Lifecycle Policy, Corss Region Replication
  - From On-Premise: Snowball or Storage Gateway
- High Availability
  - Use Route 53 to migrate DNS over from Region to Region
  - RDS Mulit-AZ, ElastiCache Multi-AZ, EFS, S3
  - Site to Site VPN as a recovery from direct connect
- Replication
  - RDS Replication (Cross Region), AWS Aurora + Global Databases
  - Database Replication from On-Premise to RDS
  - Storage Gateway
- Automation
  - CloudFormation / Elastic Beanstalk to re-create a whole new environment
  - Recover / Reboot EC2 instances with Cloud Watch if alarms fail
  - AWS Lambda Functions for customized automations
- Chaos
  - Netflix has a "simian-army" randomly terminating EC2 to make sure they can recover

## Database Migration Serivce (DMS)

- Quickly and securely migrate databases to AWS, resilient, self healing
- the source database remains available during the migration
- Supports
  - Homogeneous migrations: ex Oracle to Oracle
  - Heterogeneous migrations: ex Microsoft SQL Server to AWS Aurora
- continuous data replication using CDC
- you must create an EC2 instance to perform the replication task
- source (on-premise) to target (aws) any database
- Sources
  - On-premise and EC2 instances
  - databases: Oracle, MS SQL Sever, MySQL, MariaDB, PostgreSQL, MongoDB, SAP, DB2
  - Azure: Azure SQL Databases
  - Amazon RDS: All inluding Aurora
  - Amazon S3
- Targets
  - On-premise and EC2 instances
  - databases: Oracle, MS SQL Sever, MySQL, MariaDB, PostgreSQL, SAP
  - Amazon RDS
  - Amazon Redshift
  - Amazon DynamoDB
  - Amazon S3
  - ElasticSearch Service
  - Kinesis Data Streams
  - DocumentDB

### AWS Schema Conversion Tool

- Covert your Database's Schema from one Engine to another
- Example OLTP: (SQL Server or Oracle) to MySQLq, PostgreSQL, Aurora
- Example OLAP: (Teradata or Oracle) to Amazon Redshift
- **you do not need to use SCT if you are migrating the same DB engine**
  - Example OLAP: PostgreSQL to RDS PostgreSQL
  - the DB engine is still PostgreSQL (RDS is the platform)
- can do **Continuous Replication** after schema is converted and imported

## RDS and Aurora Migrations

- RDS MySQL to Aurora MySQL
  - Option 1: DB snapshots from RDS MySQL restored as MySQL Aurora DB
  - Option 2: Create an Aurora Read Replica from your RDS MySQL and when replication lag is 0, promote it as its own DB cluster (can take time and cost $)
- External MySQL to Aurora MySQL
  - Option 1:
    - use Percona XtraBackup to create a file backup in Amazon S3
    - create an Aurora MySQL DB from Amazon S3
  - Option 2:
    - create an Aurora MySQL DB
    - use the mysqldump utility to migrate MySQL into Aurora (slowm than S3 method)
- **Use DMS if both databases are up and running**
- RDS PostgreSQL to Aurora PostgreSQL
  - Option 1: DB Snapshots from RDS PostgreSQL restored as PostgreSQL Aurora DB
  - Option 2: Create an Aurora Read Replica from your RDS PostgreSQL, and when the replication lag has 0, propote it to its own DB cluster (can take time and $)
- External PostgreSQL to Aurora PostgreSQL
  - Create a backup and put it in Amazon S3
  - import the backup using the aws_s3 Aurora Extension

## On-premise Strategies with AWS

- Ability to download Amazon Linux 2 AMI as a VM (.iso format)
  - VMWare, KVM, VirtualBox (Oracle VM), Microsoft Hyper-V
- VM Import/Export
  - migrate exisiting applications into EC2
  - create a DR repository strategy for your on-premise VMs
  - can export back the VMs from EC2 to on-premise
- AWS Application Discovery Service
  - gather information about your on-premise servers to plan a migration
  - server utilization and dependancy mapping
  - track with AWS Migration Hub
- AWS Database Migration Service
  - replciate on-premise -> AWS, AWS -> AWS, AWS -> on-premise
  - works with various database technologies (Oracle, MySQL, DynamoDB, etc...)
- AWS Server Migration Service
  - Incremental replication of on-premise live servers to AWS

## AWS Backup

- Fully managed service
- centrally mange and automate backups across AWS services
- no need to create custom scripts and manual processes
- Supported Services
  - Amazon EC2 / EBS
  - Amazon S3
  - Amazon RDS (all DB engines) / Aurora / DynamoDB
  - Amazon Document DB / Amazon Neptune
  - Amazon EFS / FSx (Lustre & Windows File Server)
  - AWS Storage Gateway (Volume Gatway)
- Supports cross-region backups
- supports cross-account backups
- supports PIIT for supported services
- on-demand andn scheduled backups
- tag-based backup policies
- you create Backup policies known as Backup Plans
  - Backup Frequency (every 12 hours, daily, weekly, monthly, cron expression)
  - Backup Window
  - Transition to Cold Storage (never, days, weeks, months, years)
  - Retention Period (always, days, weeks, months, years)
- backed up to internally specific S3 bucket for AWS Backup
- AWS Backup Vault Lock
  - enforce a WORM (Write Once Read Many) state for all backups that you store in your AWS Backup Vault
  - Additional layer of defense to protect your backups against
    - inadvertent or malicious delete operations
    - updates that shorten or alter retention periods
  - even the root user cannot delete backups when enabled

## Application Migration Service (MGN)

- plan migration projects by gathering information about on-premise data centers
- server utilization data and dependancy mapping are important for migrations
- Agentless Discovery (AWS Agentless Discover Connector)
  - VM inventory, configuration, and performance history such as CPU, memory, and disk usage
- Agent-based Discovery
  - system configuration, system performance, running processes, and details of the network connections between systems
- resulting data can be viewed within AWS Migration Hub
- *The "AWS Evolution" of CloudEndure Migration, replacing AWS Server Migration Service (SMS)*
- life-and-shift (rehost) solution which simplify **migrating** applications to AWS
- converts your physical, virtual, and cloud-based servers to run natively on AWS
- supports wide range of platforms, Operating systems, and databases
- minimal downtime and reduced costs

## Transferring Large Datasets into AWS

- Example: transfer 200 TB of dat in cloud and we have 100 Mbps internet connection
  - Over the Internet / Site to Site VPN
    - immediate to setup
    - will take 200(TB)\*1000(GB)\*1000(MB)\*8(Mb)\*100Mbps = 16,000,000s = 185 days
  - Over direct connect 1Gbps
    - long for the one-time setup (over a month)
    - will take 200(TB)\*1000(GB)\*8(Gb)/1Gbps = 1,600,000s = 18.5 days
  - Over Snowball
    - will take 2 to 3 snowballs in parallel
    - takes about a 1 week for the end-to-end transfer
    - can be combined with DMS
  - For on-going replication/transfers: Site-to-Site VPC or DX with DMS or DataSync

## VMware Coud on AWS

- some customers use VMware Cloud to manage their on-premise Data Center
- they want to extend the Data Center capacity to AWS, but keep using the VMware Cloud Software
- use cases
  - migrate VMware vSphere-based workloads to AWS
  - run your production workloads across VMware, vSphere-based private, public, and hybrid cloud envrionments
  - have a disaster recovery strategy
