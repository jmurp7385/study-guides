# RDS, Aurora, Elasticache

## RDS - Relational Database Service

- managed DB services for DB that use SQL as a query language
- create Databases in the cloud that are managed by AWS
  - Postgres
  - MySQL
  - MariaDB
  - Oracle
  - Microsoft SQL Server
  - Aurora (AWS Propietary Database)
- Advantage of RDS over deploying DB on EC2
  - automated provisioning, OS patching
  - continuous backups and restore to specific timestamp (Point in Time Restore)
  - read replicas for improved read performance
  - Multi AZ setup for Distaster Recovery (DR)
  - Maintainance windows for upgrades
  - scaling capability (vertical and horizontal)
  - storage backed by EBS (gp2 or io1)
  - CANNOT SSH into isntance
- Storage Auto Scaling
  - helps increase storage on RDS DB instance dynamically
  - when RDS detects you are running out of free database storage, it automatically scales
  - avoid scaling your database automatically
  - you have to set a Maximum Storage Threshhold (maximum limit for DB storage)
  - Automatically modify storage if
    - free storage is less than 10% of allocated storage
    - low-storage lasts at least five minutes
    - 6 have passed since lasty modification
  - Supports all RDS database engines (MariaDB, MySQL, PostgreSQL, SQL Server, Oracle)

### Read Replicas vs Multi AZ

- Read Replicas
  - up to 5 replicas
  - Within AZ, Cross AZ, or Cross Region
  - Replication is ASYNC, so reads are eventually consistent
  - replicas can be promoted to their own DB
  - applications must update the connection string to leverage read replicas
  - Use cases
    - prod db with normal load and another team wanting to run some analytics
    - run read replica and run the new workload on the replica
    - prod application is unaffected
    - read replicas are used for SELECT (=read) statements only
  - Network Costs
    - dont pay fee for cross AZ in same Region
    - Fees for Cross Region
- Multi AZ
  - sync replication
  - one DNS name - automatic app failover to standby
  - increase availablity
  - failover in case of loss of AZ, loss of network, instance or storage failure
  - no manual intervention in apps
  - not used for scaling
  - **Read Replicas can be setup as Multi AZ for Disaster Recovery (DR)**
    - common exam question
  - Single to Multi AZ
    - zero downtime operation
    - click modify and enable multi-az
    - internal operations
      - snapshot taken
      - new DB is restored from snapshot in new AZ
      - synchronization is established between the two databases

### RDS Custom for Oracle and Microsoft SQL Sever

- RDS automates setup, operation, and scaling of database in AWS
- custom - full admin access to the underlying database and OS so you can
  - configure settings
  - install patches
  - enable native features
  - access the underlying EC2 instance using **SSH** or **SSM Session Manager**
  - for Oracle and Microsoft SQL Sever

## Aurora

- Aurora is a proprietary technology from AWS (not open sourced)
- Postgres and MySQL are both supported as Aurora DB
  - means drivers will work as is Aurora was a Postgres or MySQL database
- Aurora is "AWS Cloud Optimzed"
  - claims 5x performance improvemnt over MySQL on RDS
  - claims 3x performance of Postgres on RDS
- storage automaticall grows in increments of 10GB, up to 128 TB
- can have 15 replicas while MySQL has 5 and the replication proceess is faster (sub 10ms replica lag)
- failover in Aurora is instananeous. It is HA native
- costs 20% more than RDS but is more efficient
- High Availability & Read Scaling
  - 6 copies of data across 3 AZ
    - 4/6 copies needed for writes
    - 3/6 copies needed for reads
    - self-healing with peer-to-peer replication
    - storage is striped across 100s of volumes
  - one instance takes writes (master)
  - automated failover for master is less than 30 seconds
  - Master + up to 15 Aurora Read Replicas serve reads
  - support for Cross Region Replication
- DB Cluster
  - shared storage volume (auto exapnding from 10GB to 128 TB)
  - only master writes
    - writer endpoint pointing to master
    - reader endpoint
      - connection load balancing for read replicas (auto-scaling)
- Features
  - Automatic Fail-Over
  - Backup & Recovery
  - Isolation & Security
  - Industry Compliance
  - Push-button scaling
  - Automated patching with Zero Downtime
  - Advanced Monitoring
  - Routine Maintenance
  - Backtrack
    - restore data at any point in time without using backups

### Aurora Advanced Concepts

- Auto Scaling
  - Replicas Auto Scaling
    - automaticallu extends reader endpoints to handle more requests
- Custom Endpoints
  - define a subset of Aurora Instances as a Custom Endpoint
  - run analytical queries on specific replicas
  - reader endpoint is generally not used after defining Custom Endpoints
- Aurora Serverless
  - automated database instantiation and auto-scaling based on actual usage
  - good for infrequent, intermittent, or unpredictable workloads
  - no capacity planning needed
  - pay per second, can be more cost effective
- Aurora Multi-master
  - immediate failover for write node (HA)
    - every node does Read/Write - vs promoting Read Replica as new master
- Global Aurora
  - Aurora Cross Region Read Only Replicas
    - useful for disaster recovery
    - simple to put in place
  - Aurora Global Database (recommended)
    - 1 Primary Region (read/write)
    - Up to 5 secondary (read-only) regions
      - replication lag is less than 1 second
    - Up to 16 Read Replicas per secondary region
    - helps decrese latency
    - promoting from another region (for disaster recovery) has an RTO of < 1 minute
    - **Typical cross-region replication takes less than 1 second**
- Aurora Machine Learning
  - enables you to add ML-based predictions to your application via SQL
  - simple, optimized, and secure integration between Aurora and AWS ML Services
    - Amazon SageMaker (use with any ML model)
    - Amazon Comprehend (for sentiment analysis)
  - dont need ML experience
  - fraud detectoin, ads targeting, sentiment analysis, product recommendations

## RDS and Aurora Backup and Monitoring

- RDS
  - Automated backups
    - daily full backups of the database (during the maintainance window)
    - transacton logs are backed-up by RDS every 5 minutes
    - ability to restore to any point in time (from oldest backup to 5 minutes ago)
    - 1 tom 25 days of retention, set 0 to disable automated backups
  - Manual DB Snapshots
    - manually triggered by the user
    - retention of backup for as long as you want
  - In a stopped RDS Database, you will still pay for storage. If you plan on stopping it for a long time, you shouldn snapshot & restore istead
- Aurora
  - Automated backups
    - 1 to 35 days (cannot be disabled)
    - point-in-time recovery in that timeframe
  - Manual DB Snapshots
    - manually triggered by the user
    - retention of backup for as long as you want
- Restore Options
  - restoring a RDS/Aurora backup or snapshot creates a new database
  - restoring MySQL RDS database from S3
    - create a backup of your on-premise database
    - store it on Amazon S3 (object storage)
    - restore the backup file onto a new RDS instance running MySQL
  - restoring MySQL Aurora Cluster from S3
    - create a backup of your on-premise database useing Percone XtraBackup
    - store backup file on Amazon S3
    - restore the backup file onto a new Aurora cluster running MySql
- Aurora Cloning
  - create a new Aurora DB Cluster from an exisiting one
  - faster than snapshot & restore
  - new DB cluster uses the same cluster volume and data as the original but will change when data updates are made
  - very fast and cost-effective
  - useful to create a "staging" database from a "production" database without impacting production database

### RDS & Aurora Security

- At-rest encryption
  - database master and replicas encryption use AWS KMS - must be defined at launch time
  - if the master is not encrypted, the read replicas cannot be encrypted
  - to encrypt an unencrypted database, go through a DB snapshot and restore as encrypted
- In-flight encryption
  - TLS-ready by default
  - use the AWS TLS root certificates client-side
- IAM Authentication
  - IAM roles to connect to your database (instead of username/password)
- Security Groups
  - control network access to your RDS/Aurora DB
- **No SSH available** except on RDS Custom
- **Audit Logs can be enabled** and sent to CloudWatch Logs for longer retention

### RDS Proxy

- fully managed proxy for RDS
- allows apps to pool and share DB connections established with the DB
- **improving database efficiency by reducing the stress on database resources (eg CPU, RAM) and minimize open connections (and timeouts)**
- serverless, autoscaling, highly available (multi-AZ)
- Reduced RDS & Aurora failover time by up to 66%
- Supports RDS (MySQL, PostgreSQL, MariaDB) and Aurora (MySQL and PostgreSQL)
- no code changes needed for most apps
- **enforce IAM Authentication for DB and securly store credentials in AWS Secrets Manager**
- **RDS Proxy is never publicaly accessible (must be accessed from VPC)**

## ElastiCache

- managed Redis or Memcached
- caches are in-memory databases with really high performance, low latency
- helps reduce load off of databases for read instensive workloads
- helps make applciation stateless
- AWS takes care of OS maintenance/patching, optimizations, setup, coinfiguration, monitoring, failure recovery, and backups
- using ElastiCache requires heavy application code changes
- Solution Architecture
  - DB Cache
    - application queries ElastiCache, if not available from RDS and store in ElastiCache
    - help relieve load in RDS
    - cache must have an invalidation strategy to make sure only the most current data is used in there
  - User Session Store
    - user logs into any of the applications
    - application writes session store into ElastiCache
    - the user hits another instance of application
    - instance retrieves the data and is already logged in
- Redis (high availablity backup)
  - multi AZ with Auto-failover
  - read replicas to scale reads and have high availibility
  - data durability using AOF persistence
  - backup and restore features
- Memcached (pure distributed cache)
  - multi-node for partitioning of data (sharding)
  - no high availability (replication)
  - non-persistent
  - no backup and restore
  - multi-threaded architecture
- Cache Security
  - All
    - do not support IAM Authentication
    - IAM policies on Elasticache are only used for AWS API-level security (create/delete cache)
  - Redis AUTH
    - can set a "password/token" when you create a Redis Cluster
    - extra level of security for your cache (on top of security groups)
    - supports SSL in flight encryption
  - Memcached
    - supports SASL-based authentication (advanced)
- Patterns
  - lazy loading
    - all the read data is cached, data can become stale in cache
  - write through
    - adds or updates data in cache when written to a DB
    - no stale data
  - session store
    - store temporary session data ina cache (using TTL features)
- Redis Use Case
  - gaming leaderboard are computationally complex
  - redis sorted sets guarantee both uniqueness and element ordering
  - each time a new element added it is ranked in realtime then added in correct order

## Ports

Here's a list of **standard** ports you should see at least once. You shouldn't remember them (the exam will not test you on that), but **you should be able to differentiate between an Important (HTTPS - port 443) and a database port (PostgreSQL - port 5432)**

- Important ports:
  - FTP: 21
  - SSH: 22
  - SFTP: 22 (same as SSH)
  - HTTP: 80
  - HTTPS: 443
- RDS Databases ports:
  - PostgreSQL: 5432
  - MySQL: 3306
  - Oracle RDS: 1521
  - MSSQL Server: 1433
  - MariaDB: 3306 (same as MySQL)
  - Aurora: 5432 (if PostgreSQL compatible) or 3306 (if MySQL compatible)