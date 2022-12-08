# Decoupling Applications

- when we start deploying multiple applications, they will inevitably need to communicate with each other
- two patterns of communication
  - Synchronous Based
    - application -> application
    - can be problamatic if there are sudden spikes in traffic
  - Asynchronous / Event Based
    - application -> queue -> application
    - dont talk directly to each other
- better to decouple applications so services can scale independantly
  - using SQS = queue model
  - using SNS = pub/sub model
  - using SQS = real-time streaming model

## Standard Queue Service (SQS)

- buffer to decouple services
- Producer - services that sends messages into a queue
- Consumer - services that polls/reads messages from a queue
- Standard Queue
  - oldest offering (over 10 years old)
  - fully managed service used to decouple service
  - Attributes
    - unlimited throughput, unlimited number of messages in queue
    - default retention of messages: 4 days, max 14 days
    - low latency (<10ms on publish and recieve)
    - limitation of 256KB per message send
    - at least once delivery (can have duplicate messages occasionaly)
    - best effort ordering (can be out of order)
- Producing Applications
  - produced to SQS using the SDK (SendMessage API)
  - message is persisted in SQS until consumer deletes it
  - default retention of messages: 4 days, minimum: 1 minute, maximum 14 days
  - example: send an order to be processed
    - order id
    - customer id
    - other attributes
- Consuming Applications
  - can run on EC2, on-premise servers, Lambdas, etc
  - Poll SQS for messages (receive up to 10 messages at a time)
  - process the message (e.g. insert message into RDS database)
  - delete messages from queue using DeleteMessage API
  - Multiple EC2 Instances Consumers
    - consumers recieve and process messages in parallel
    - at least once delivery
    - best-effort message ordering
    - consumers delete after processings
    - can scale horizontally to increase consumer throughut
    - **can be behind an Auto Scaling Group (ASG)**
      - CloudWatch Alarm for queue length to trigger scaling of ASG
- Security
  - Encryption
    - in-flight encryption using HTTPS API
    - at-rest encryption using KMS keys
    - client-side encryption if the client wants to perform encryption/decryption itself
  - Access Controls
    - IAM policies to regulate access to the SQS API
  - SQS Access Policies (similar to bucket policies)
    - useful for cross-account access to SQS queues
    - useful for allowing other services (SNS, S3,...) to write to an SQS Queue

### Message Visibility Timeout

- after a message is polled by a consumer, it becomes **invisible** to other consumers
- by default, the "message visibilty timeout" is **30 seconds**
- that means message has 30 seconds to be processed
- after the message visibility timeout is over, the message is "visible" in SQS
- if not processed within the visibility timeout, it will be processed twice
- consumer could call the ChangeMessageVisibility API to get more time
- if visibility time is high (hours), an consumer crashes, re-processing will take time
- if visibility time is low (seconds), we may get duplicates

### Long Polling

- when a consumer requests messages from the queue, it can optionally "wait" for messages to arrive if there are none in the queue
- **Long Polling decreases the number of API calls made to SQS while increasing the efficiency and latency of your application**
- the wait tieme can be betweeen 1 second and 20 seconds (20 sec preferable)
- Long Polling is preferable to Short Polling
- Long Polling can be enabled at the queue level or at the API Level using **WaitTimeSeconds**

### First In First Out (FIFO) Queues

- exact ordering of messages
- limited throughput: 300 messages/second without batching, 3000 messages/second with
- exactly-once send capability (removing duplicates)
- messages are processed in order by consumers
- Queue name must end in ".fifo"

### SQS & Auto Scaling Groups

- scale consumers with queue size
  - use CloudWatch Metric for Queue length to trigger scaling
- SQS as a buffer to database writes (common exam pattern)
- request -> ASG Enqueue Message -> SQS Queue -> ASG Dequeue Message -> insert
  - only works if client doesnt need confirmation of database write
- decouple between appplication tiers
- keywords in exam: **sudden spike, decoupling, timeout**

## Simple Notification Service (SNS)

- one message to many recievers
- Pub/Sub
  - service -> SNS Topic -> each subscriber recieves the message to process
- "event producer" (publishers) only sends message to one SNS Topic
- as many "event recievers" (subscriptions) as we want to listen to the SNS Topic notifications
- each subscriber to the topic will get all the messages
  - new feature to filter messages
- up to 12,500,000 subscriptions per topic
- 100,000 Topic Limits
- integrates with a lot of AWS services
- Publishing
  - Topic Publish (using SDK)
    - Create Topic
    - create Subscription
    - Publish to Topic
  - Direct Publish (mobile SDK)
    - create a platform application
    - createe a platform endpoint
    - publish to the platform endpoint
    - works with Google GCM, Apple APNS, Amazon ADS
- **Subcribers**
  - **Amazon Kinesis Data Firehose**
  - Amazon SQS
  - AWS Lambda
  - Email
  - Email-JSON
  - HTTP
  - HTTPS
  - SMS
- Security
  - Encryption
    - in-flight encryption using HTTPS API
    - at-rest encryption using KMS keys
    - client-side encryption if the client wants to perform encryption/decryption itself
  - Access Controls
    - IAM policies to regulate access to the SNS API
  - SNS Access Policies (similar to bucket policies)
    - useful for cross-account access to SQS queues
    - useful for allowing other services (S3,...) to write to an SNS Queue

### SNS & SQS Fan Out Pattern

- push once in SNS, revieve in all SQS queues that are subscribers
- fully decoupled, no data loss
- SQS allows for data persistence, delayed processing and retries of work
- ability to add more SQS subscribers over time
- make sure your SQS queue **access policy** allows for SNS to write
- Cross-Region Delivery: works with SQS Queues in other regions
- Applications
  - S3 events to multiple queues
    - for the same combination of: **event type** (e.g. object create) and **prefix** (e.g. /images/) you can only have **one S3 Event Rule**
    - to send same S3 event to many SQS queues, use fan-out
  - SNS to Amazone S3 throughtn Kinesis Data Firehose
    - SNS -> Kinesis -> S3 or any other KDF Destination (really extensible to persist messages from SNS topic)
- SNS FIFO
  - **Ordering** my message group ID (all message in same group are ordered)
  - **Deduplication** using a Deduplication ID or Content Based Deduplication
  - **can only have SQS FIFO Queues as subscribers**
  - limited throughput: 300 messages/second without batching, 3000 messages/second with
    - same as SQS FIFO
- SNS FIFO + SQS FIFO: Fan Out
  - for cases when you need fan out + ordering + deduplication
- SNS Message Filtering
  - JSON policy used to filter messages sent to SNS topic's subscriptions
  - if subscription doens't havee a filter policy, it recieves every message

## Kinesis

- make it easy to **collect**, **process**, and **analyze** streaming data in real-time
- ingest real-time data such as: Application Logs, Metrics, Website clickstreams, IoT Telemetry
- **Kinesis Data Streams**: capture, process, and store data streams
- **Kinesis Data Firehose**: load data streams into AWS data stores
- **Kinesis Data Analytics**: analyze data streams with SQL or Apache Flink
- **Kinesis Video Streams**: capture, process, and store video streams

### Kinesis Data Streams

- capture, process, and store data streams
- made of multiple shards
  - have to provision ahead of time
  - can scale # of shards
- producers
  - Applications
  - Client
  - SDK, KPL (Kinesis Producer Library)
  - Kinesis Agent
  - 1 MB/second or 1000 message/second per shard
  - Record
    - Partition Key
    - Data Blob (up to 1MB)
- consumers
  - Applications (KCL (Kinesis Consumer Library), SDK)
  - Lambda
  - Kinesis Data Firehose
  - Kinesis Data Analytics
  - 2 MB/second (shared) per shard all consumers **OR** 2 MB/second (enhances) per shard all consumers
  - Record
    - Partition Key
    - Sequence #
    - Data Blob
- retention between 1 day to 365 days
- ability to reprocess (replay) data
- once data is inserted in Kinesis, it cannot be deleted (immutability)
- data that shares the same partition goes to the same shard (ordering)
- Producers: AWS SDK, Kinesis Producer Library (KPL), Kinesis Agent
- Consumers :
  - Write your own: Kinesis Client Library (KCL), AWS SDK
  - Managed: AWS Lambda, Kinesis Data Firehose, Kinesis Data Analytics
- Capacity Modes
  - Provisioned Mode
    - you choose the number of shards provisioned, scale manually or using API
    - each shard gets 1 MB/second in (or 1000 records/second)
    - each shard gets 2 MB/second out (classic or enhanced fan-out consumer)
    - pay per shard per hour
  - On-Demand Mode
    - no need to provision or manage capacity
    - default capacity provisioned (4MB/s in or 4000 records/second)
    - scaled automatically based on observed throughput peak during last 30 days
    - pay per stream per hour & data in/out per GB
- Security
  - deployed within a region
  - Control Access / authorization using IAM policies
  - encryption in-flight using HTTPS endpoints
  - encryption at rest using KMS
  - you can implement encryption/decryption of data on client side (harder)
  - VPC endpoints available for Kinesis to access withing VPC
  - Monitor API calls using CloudTrail

### Kinesis Data Firehose

- fully managed service
- pay for data going through Firehose
- new Realtime
  - 60 seconds latency minimum for non full batches
  - minimum of 1 MB of data at a time
- supports many data formats, conversions, transformations, compression
- supports custom data transformations using AWS Lambda
- can send all or failed data to a backup S3 buckets
- Producers
  - Applications
  - Client
  - SDK, KPL (Kinesis Producer Library)
  - Kinesis Agent
  - Kinesis Data Streams
  - Amazon Cloud Watch
  - AWS IoT
  - 1 MB/second or 1000 message/second per shard
- Batch Writes to consumers
- Consumers
  - ***AWS Destinations***
    - Amazon S3
    - Amazon Redshift (COPY thru S3)
    - Amazon Elastic Search
    - 3rd Party Destinations
      - Datadog
      - splunk
      - New Relic
      - mongoDB
    - Custom destinations
      - API: HTTP endpoint

#### Data Streams vs Firehouse

|               Data Streams               |                                Firehose                                 |
| :--------------------------------------: | :---------------------------------------------------------------------: |
|  streaming service for ingest at scale   | load streaming data into S3/Redhift/ElasticSearch/3rd Party/Custom HTTP |
|   write custom code (producer/consumer   |                              fully managed                              |
|            realtime (~200ms)             |              near realtime (buffer time minimum 60 seconds              |
| manage scaling (shard splitting/merging) |                            automatic scaling                            |
|      data storage for 1 to 365 days      |                             no data storage                             |
|        supports replay capability        |                    doesn't support replay capability                    |


### Kinesis Data Analytics

- see [Data & Analytics](data-analytics.md)

## Data Ordering for Kinesis vs SQS FIFO

- Kinesis
  - imagine you have 100 trucks (truck_1, truck_2, ..., truck_100) sending GPS regularly to AWS
  - you want to consume the data in order for each truck so that you can accurately track their movement
  - how should you send that data to Kinesis?
    - send using "Partition Key" value of "truck_id"
    - the same key will always go to the same shard
- SQS
  - no ordring for standard
  - FIFO, if you dont use a Group ID, messages are consumed in order they are send, **with only one consumer**
  - if you want to scale the number of consumers, but you want messages to be "grouped" when they are related to each other
  - then you use a Group ID (similar to Partition Key in Kinesis)
- Kinesis vs SQS
  - **assume 100 trucks, 4 kinesis shards, 1 SQS FIFO**
  - Kinesis Data Streams
    - on average there will be 20 trucks per shard
    - trucks wil have their data ordered within each shard
    - maximum amount of consumers in parallel possible is 5
    - can receive up to 5 MB/s of data
  - SQS FIFO
    - only have one
    - have 100 Group ID
    - can have up to 100 consumers (due to the 100 Group ID)
    - have up to 300 messages per second (or 3000 if using batching)

## SQS vs SNS vs Kinesis

|                      SQS                       |                      SNS                       |                         Kinesis                          |
| :--------------------------------------------: | :--------------------------------------------: | :------------------------------------------------------: |
|              consumer "pull data"              |          push data to many subscibers          |           standard: pull data (2MB per shard)            |
| can have as many workers (consumers) as needed |          up to 12,500,000 subscibers           | enhanced-fan out: push data (2MB per shard per consumer) |
|        no need to provision throughput         | data is not persisited (lost if not delivered) |                possiblity to replay data                 |
|       ordering guaranteed only with FIFO       |                    Pub/Sub                     |       mean for realtime big data analytics and ETL       |
|      individual message delay capability       |              up to 100,000 topics              |               ordering at the shard level                |
|                                                |        no need to provision throughput         |                data expires after X days                 |
|                                                |          FIFO capability for SQS FIFO          |          provisioned or on-demand capacity mode          |

## Amazon MQ

- SQS and SNS are "cloud-native" services: proprietary protocols from AWS
- traditional applications running on-premis may use open protocols such as MQTT, AMQP, STOMP, Openwire, WSS
- **when migrating to cloud** , instead of re-engineering the application to use SQS and SNS we can use Amazon MQ
- **Amazon MQ is a managed message broker service for RabbitMQ and ActiveMQ**
- doesnt scale as much and SQS/SNS
- runs on servers, can run in Multi-AZ with failover
- has both queue feature (~SQS) and topic feature (~SNS)
- High Availability
  - 2 brokers in differnet AZs with Amazon EFS for storage
