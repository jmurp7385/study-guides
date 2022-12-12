# Data and Analyitics

## Athena

- **severless** query service to analyize data stored in Amazon S3
- uses standard SQL language to query the files (build on Presto)
- supports CSV, JSON, ORC, Avro, and Parquet
- pricing: $5.00 per TB of data scanned
- commonly used with Amazon Quicksight for reporting/dashboard
- Use Cases: business intelligence/analytics/reportings, analyze & query VPC Flow Logs, ELB Logs, **CloudTrail trails**, etc
- **EXAM TIP: analyze data in S3 using serverless SQL, use Athena**
- Performance Improvment
  - user **columnar data** for cost-savings (less scan)
    - Apache Parquet or ORC is recommended
    - huge performance improvment
    - use Glue to conver your data to Parquet or ORC
  - **Compress Data** for smaller retrievals (bzip2, gzip, lz4, snappy, zlip, zstd)
  - **Partition** datasets in S3 fdor easy querying on virtual columns
    - `s3://yourBucket/pathToTable/<PARTITION_COLUMN_NAME> = <VALUE>/<PARTITION_COLUMN_NAME> = <VALUE>/<PARTITION_COLUMN_NAME> = <VALUE>/etc...`
    - example: s3://athena-examples/flight/parquet/year=1991/month=1/day=1
  - **use larger files** (>128MB) to minimize overhead)
- Federated Query
  - allows you to run SQL queries across data stored in relational, non-relational, object, and custom data sources (AWS or on-premise)
  - use Data Source Connectors that run on AWS Lambda to run Federated Queries (e.g., CloudWatch Logs, DynamoDB, RDS, ...)
  - store results back in Amazon S3

## Redshift

- Redshift is based on PostgreSQL, but **it is not used for OTLP**
- **OLAP: online analytical processing (analytics and data warehousing)**
- 10x better performance thatn other data warehouses, scale to PBs of data
- **Columnar** storage of data (instead of row based) & parallel query engine
- pay as you go based on the instances provided
- has a SQL interface for performing the queries
- BI toolsa such as Amazon Quicksight ro Tableau integrate with it
- **vs Athena**: faster queries / joins / aggregations thanks to indexes
- Redshift Cluster
  - Leader Node: for query planning, results, aggregation
  - Computer Node: for performing the queries, send results to leader
  - you provision the node size in advance
  - you can use Reserved Instances for cost savings
- Snapshots & DR
  - **Redshift has no "Multi-AZ" mode**
  - snapshots are point-in-time backups of a cluster stored internally in S3
  - snapshots are incremental (only what has changed is saved)
  - you can resotre a snapshot to a **new cluster**
  - Automated: every 8 hours, every 5 GB, or on a schedule. set retention
  - Manual: snapshot is retained until you delete it
  - **you can configure Amazon Redshift to automatically copy snapshots (Automated or manual) of a cluster to another AWS Region**
- Loading Data into Redshift (**large inserts are much better**)
  - Amazon Kinesis Data Firehose using S3 copy
  - S3 using COPY command
    - Public Internet or through VPC with Enhanced VPC routing
  - EC2 instance
    - JDBC driver
    - better to write data in large batches
- Redshift Spectrum
  - query data taht is already in S3 without loading it
  - **must have Redshift cluster available to start the query**
  - query is then submitted to thousands of Redshift Spectrum nodes

## OpenSearch

- **Amazon OpenSearch is successor to Amazon ElasticSearch**
- can query on any field, even partial matches
- common to use OpenSearch to compliment other databases
- OpenSearch requires a cluster of instances (not serverless)
- doest *not* support SQL (it has its own query language)
- security through Cognito & IAM, KMS encryption, TLS
- comes with OpenSearch Dashboards (visualizations)
- OpenSearch Patterns
  - CRUD to DynamoDB -> DynamoDB Stream -> Lambda -> OpenSearch
    - API to search items (Open Search), API to retrieve items (DynamoDb)
  - CloudWatch Logs -> Subscription Filter -> Lambda Function (realtime) -> Amazon Open Search
  - CloudWatch Logs -> Subscription Filter -> Kinesis Data Firehose (near realtime) -> Amazon Open Search
  - Kinesis Data Streams -> Kinesis Data Firehose (near real time) -> Amazon OpenSearch
  - Kinesis Data Streams -> Lambda (real time) -> Amazon OpenSearch

## Elastic MapReduce (EMR)

- Elastic MapReduce
- helps creat Hadoop Clusters (Big Data) to analyze and process vast amounts of data
- clusters can be made of hundreds of EC2 instances
- EMR comes bundled with Apache Spark, HBase, Presto, Flink, ...
- EMR takes care of all the provisioning and configuration
- Auto-scaling and integrated with Spot instances
- Use cases: data processing, machine learning, web indexing, big data,...
- Node Types
  - Master Node: manage cluster, coordinate, manage health - long running
  - Core Node: runt tasks and store data - long running
  - Task Node (optional): just to run tasks - usally Spot Instances
  - Purchasing Options:
    - On-demand: reliable, predictable, wont be terminated
    - Reserved (min 1 year): cost savings, EMR will automatically use if available
    - Spot Instances: cheaper can be terminated, less reliable
  - can have long-running cluster or transient (temporary) cluster

## QuickSight

- Serverless machine learning-powered business intelligence service to create interactive dashboards
- fast, automatically scalable, embeddable, with per-session pricing
- use cases:
  - business analytics
  - building visualizations
  - perform ad-hoc analysis
  - get business insights using data
- integrated with RDS, Aurora, Athena, Redshift, S3, ...
- **in-memory computation using SPICE** engine if data is imported into QuickSight
- Enterprise Edition: able to setup **Column Level Security (CLS)**
- Integrations
  - RDS, Aurora, Redshift, Athena, S3, OpenSearch, Timestream
  - Salesforce, Jira
  - teradata, on-premise
  - XLSX, CSV, JSON, TSV, ELF & CLF (log format)
- Dashboard & Analysis
  - Define users (standard version) and Groups (enterprise version)
    - ***only exist in QuickSight, not IAM***
  - A Dashboard
    - is a read-only snapshot of an analysis you can share
    - preserves the configuration of the analysis (filtering, parameters, controls, sort)
    - have to publish a dashboard to share it
  - **you can share ther analysis or dashboard with Users or Groups**
  - users who see the dashboard can also see the underlying data

## Glue

- managed extract, transform, and load (ETL) service
- useful to prepare and transform data for analytics
- fully **serverless** service
- Patterns
  - S3 / RDS -Extract-> Glue ETL (Transform) -Load-> Redshift Data Warehouse
  - Convert to Parquet Format
    - S3 Put -import CSV-> Glue -Parquet-> S3 Output -Analyze-> Athena
- Glue Data Catalog: catalog of datasets
  - crawls databases (S3, RDS, DynamoDB, JDBC)
  - write metadata to Glue Data Catalog
  - Athena, Redshift Spectrum, EMR can discover that data
- **Glue Job Bookmarks**: prevent re-processing old data
- **Glue Elastic Views**:
  - combine and replicated datg across multiple data stores using SQL
  - no custom code, Glue monitors for changes in source data, serverless
  - leverages a "virtual table" (materialized views)
- **Glue DataBrew**: clean and normalize data using pre-built transformation
- **Glue Studio**: new GUI to create, run, and monitor ETL jobs in Glue
- **Glue Streaming ETL**:
  - built on Apache Spark Structured Streaming
  - compatible with Kinesis Data Streaming, Kafka, MSK (managed Kafka)

## Lake Formation

- data lake = central place to have all data for analytical purposes
- fully managed service that makes it easy to setup a **data lake** in days
- discover, cleanse, transform, and ingest data into your Data Lake
- it automates many complex manual steps (collecting, cleansing, moving, cataloging data,... ) and de-duplicate (ML-transforms)
- combine structured and unstructured data in the data lake
- **out-of-the-box** source blueprints: S3, RDS, Relational & NoSQL DB, ...
- **fine-grained access control for your applications (row & column level)**
- built on top of AWS Glue
- Easy to do **centralized permissions** in data lake instead of trying to do security in each data source

## Kinesis Data Analytics

- SQL Applications
  - Pattern
    - Kinesis Data Streams or Firehose
    - Kinesis Data Analytics for SQL Applications (enrich data with S3) (apply SQL for real-time analytics)
    - Kinesis Data Streams or Firehose
  - Real-time analytics on Kinesis Data Streams & Firehose using SQL
  - add reference data from S3 to enrich streaming data
  - fully managed, no servers to provision
  - automatic scaling
  - pay for actual consumption rate
  - Output
    - Kinesis Data Streams: create streams out of the real-time analytics queries
    - Kinesis Data Firhose: send analytics query results to destinations
  - Use cases:
    - time-series analytics
    - real-time dashboards
    - real-time metrics
- Flink
  - use Flink (Java, Scala, or SQL) to process and analyze streaming data
  - run any Flink application on a managed cluster on AWS
    - provisioning of compute resources, parallel computation, automatic scaling
    - application backups (implemented as checkpoints and snapshots)
    - use any Apache Flink programming features
    - **Flink does not read from Firehose (use Kinesis Analytics for SQL instead)**

## MSK - Managed Streaming for Apache Kafka

- alternative to Amazone Kinesis
- Fully managed Apache Kafka on AWS
  - allow you to create, update, delete clusters
  - MSK creates & manages Kafka brokers nodes & Zookeeper nodes for you
  - deploy the MSK cluster in your VPC, multi-AZ (up to 3 for high availability)
  - automatic recovery from common apache kafka failures
  - data is stored on EBS volumes **for as long as you want**
- MSK Serverless
  - run Apache Kafka on MSK without managing the capacity
  - MSK automatically provisions resources and scales compute & storage
- MSK Consumers
  - Kinesis Data Analytics for Apache Flink
  - AWS Glue
    - streaming ETL jobs
    - powered by Apache Spark Streaming
  - Lambda
  - Applications on EC2, ECS, EKS

|   Kinesis Data Streams    |                  Amazon MSK                   |
| :-----------------------: | :-------------------------------------------: |
|  1 MB message size limit  | 1 MB default, configure for higher (ex: 10MB) |
| Data Streams with Shards  |         Kafka Topics with Partitions          |
| Shard Splitting & Merging |      can only add partitions to a topic       |
| TLS In-flight encryption  |     PLAINTEXT or TLS In-Flight Encryption     |
|  KMS at-rest encryption   |            KMS at-rest encryption             |

## Big Data Ingestion Pipeline

- Request
  - fully serverless
  - collect data in realtime
  - transform the data
  - query the transformed data using SQL
  - created reports using queries are stored in S3
  - load data into a warehouse and create dashboards
- Solution
  - IoT Core allows you to harvest data from IoT devices
  - Kinesis is great for real-time data collection
  - Firehose helps with data deliveryr to S3 in near real-time (1 minute)
  - lambda can help Firehose with data transformations
  - S3 can trigger notifications to SQS
  - Lambda can subscribe to SQS (we could have connector S3 to Lambda)
  - Athena is a serverless SQL service and results are stored in S3
  - the reporting bucket contains analyzed data and can be used by reporting tools such as AWS QuickSight, Redshift, etc