# Databases

- Choosing the right databases
- Questions to chose the right database based on your architecture
  - read-heavy, write-heavy, or balanced workload? throughput needs? will it change, does it need to scale or fluctuate during the day?
  - how much data to store and for how long? will it grow? average object size? how are they accessed?
  - data durability? source of truth for the data?
  - latency requirments? concurrent users?
  - data model? how will data be queried? joins? structured? semi-structured?
  - strong schema? more flexibility? reporting? search? RDBMS / NoSQL
  - license costs? switch to Cloud Native DB such as Aurora?
- Database types
  - RDBMS (=SQL/OLTP): RDS, Aurora - great for joins
  - NoSQL Database - no joins, no SQL : DynamoDB (~JSON), ElastiCache (key/value pairs), Neptune (graphs), DocumentDB (for MongoDB), Keyspaces (for Apache Cassandra)
  - Object Store: S3 (for big objects) / Glacier (backups/archives)
  - Data Warehouse (= SQL Analytics/BI): Redshift (OLAP), Athena,EMR
  - Search: OpenSearch (JSON) - free text, unstructured searches
  - Graphs: Amazon Neptune - displays relationships between data
  - Ledger: Amazon Quantum Ledger Database
  - Time Series: Amazon Timestream

## RDS

- managed PostgreSQL / MySQL / Oracle / SQL Server / MariaDB / Custom
- provisioned RDS Instance Sizes and EBS Volume type and size
- auto-scaling capability for Storage
- support for Read Replicas and Multi AZ
- security through IAM, Security Groups, KMS, SSL in transit
- Automated backup with **point in time restore** up to **35 days**
- manual DB snapshot for longer-term recovery
- support for IAM authentication, integration with Secrets Manager
- RDS custom for access to and customize underlying instance (Oracle & SQL Server)
- Use Cases
  - store relational datasets (RDBMS/OLTP)
  - perfirn SQL queries
  - transactions

## Aurora

- compatible for PostgrSQL/MySQL, seperation of storage and compute
- **Storage**: data is stored in 6 replicas across 3 AZ - highly available, self-healing, auto-scaling
- **Compute**: cluster of DB instances across multiple AZ, auto-scaling of read replicas
- **Cluster**: custome endpoints for writer & reader DB instances
- same security/monitoring/maintenance feaures of RDS
- [Back up & restore options for Aurora](rds-aurora-elasticache.md#rds-and-aurora-backup-and-monitoring)
- Aurora Serverless - for unpredictable/intermittent workloads, no capacity planning
- Aurora Multi-Master - for continuous write failovers (high write availability)
- Aurora Global - up to 16 DB Read instances in each region, < 1 second storage replication
- Aurora Machine Learning - perform ML using SageMaker & Comprehend on Aurora
- Aurora Database Cloning - new cluster from existing one, faster than restoring a snapshot
- Use case
  - same as RDS, but with less mainenance / more flexibility / more performance / more features

## ElastiCache

- managed Redis / Memcached (similar offerings as RDS, but for caches)
- in-memory data store, sub-ms latency
- must provision an EC2 instance type
- support for Clustering (Redis) and Multi AZ, Read Replicas (sharding)
- Security through IAM, Security Groups, KMS, Redis Auth
- backup/snapshot/point in time restore features
- managed/scheduled maintenance
- **requires some application code changes to be leveraged**
- Use case
  - Key/Value store, frequent reads, less writes, cache results for DB queries, store session data for websites, cannot use SQL

## DynamoDB

- AWS proprietary technology, managed serverless NoSQL database, millisecond latency
- Capacity Modes: provisioned capacity with optional auto-scaling or on-demain capacity
- can replace ElastiCache as a key/value store (storing session data for example, using TTL feature)
- highly available, multi AZ by default, Read and Writes are decoupled, transaction capability
- DAX cluster for read cache, **microsecond read latency**
- security, authentication and authorization is done though IAM
- Event Processing: DynamoDB streams to integrate with AWS Lambda, or Kinesis Data Streams
- Global Table feature: active-active setup
- Automated backups up to 35 days with PITR (restore to new table), or on-demand backups
- export to S3 without using RCU within the PITR window, import from S3 without using WCU
- **great to rapidly evolve schemas**
- use case
  - serverless application development (small documents 100KB), distributed serverless cache, doesnt have a SQL query language available

## S3

- key/value store for objects
- great for bigger objects, not so great for many small objects
- serverless, scales infinitely, max object size is 5 TB, versioning capability
- Tiers: S3 Standard, S3 Infrequent Access, S3 Intelligent, S3 Glacier + lifecycle policy
- Features: versioning, encryption, replication, MFA-Delete, Access Logs, ...
- Security: IAM, Bucket Policies, ACL, Access Points, Object Lambda, CORS, Object/Value Lock
- Encryption: SSE-S3, SSE-KMS, SSE-C, client-side, TLS in transit, default encryption
- Batch Operations: S3 Batch, listing with S3 inventory
- Performance: Multi-part upload, S3 transfer Acceleration, S3 Select
- Automation: S3 event notifications (SNS,SQS,Lambda,EventBridge)
- use cases: static files, key value store for big files, website hosting

## DocumentDB

- "AWS implementation" of MongoDB (NoSQL database)
- MongoDB is used to store, query, and index, JSON data
- similar "deployment concepts" as Aurora
- fully managed, highly available with replication across 3 AZ
- DocumentDB storage automatically grows in increments of 10GB, up to 64 TB
- automatically scales to workloads with millions of requests per second

## Neptune

- fully managed graph database
- popular graph dataset would be a social network
- highly available across 3 AZ, with up to 15 read replicas
- build and run applications working with highly connectewd datasets - optimized for these complex hard queries
- can store billions of relations and query the graph with ms latency
- Use Cases: great for knowledge graphs (Wikipedia), fraud detection, recommendation engines, social networking

## Keyspaces (for Apache Cassandra)

- Apache Cassandra is an open source NoSQL distributed database
- A managed version of Apache Cassandra-compatible database service
- serverless, scalable, highly available, fully managed by AWS
- automatically scales tables up/down based on applications traffic
- tables are replicated 3 times across multiple AZ
- Cassanda Query Language (CQL)
- single-digit ms latency at any scale, 1000s or requests per second
- Capacity: on-demand or provision with auto-scaling
- use cases: store IoT devices info, time-series data, ...

## QLDB

- Quantum Ledger Database
- a ledeger is a book **recording financial transationcs**
- fully managed, serverless, highly available, replication across 3 AZ
- used to **review history of all changes made to your application data** over time
- **Immutable** system: no entry can be removed or modified, cryptoigraphically verifiable
- 2-3x better performance than common ledger blockchain frameworks, manupulate data using SQL
- Difference with Amazon Managed Blockchain: no decentralization component, in accordance with financial regulation rules

## Timestream

- fully managed, fast, scalable, serverless **time series database**
- automatically scales up/down to adjust capacity
- store and analyze trillions of events per day
- 1000s times faster & 1/10th cost of relation databases
- scheduled queries, multi-measure records, SQL compatibility
- datra storage tiering: recent data kept in memory and historical data kept in cost-optimized storage
- built-in trime series analytics functions (helps identify patterns in your data in real time)
- encryption at rest and in transit
- Use cases: IoT apps, operation applications, real-time analytics
