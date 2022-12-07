# Storage Extras

## Snow Family

- highly-secure, portable devies to **collect and process data at the edge** and **migrate data into and out of AWS**
- offline devices to perform data migrations
  - takes more than a week, then use a snowball device

### Data Migration

- challenges with migration of network
  - limited connectivity
  - limited bandwidth
  - high network cost
  - shared bandwidth (can't maximize the line)
  - connection stability
- Snowcone
  - small, portable computing anywhere, rugged and secure, withstands harsh environments
  - light (4.5 pounds, 2.1 kg)
  - device used for edge computing, storage, and data transfer
  - **8TB of usable storage**
  - use Snowcone where Snowball does not fit (space-constrained environment)
  - must provide your own battery/cables
  - can be sent back to AWS offline or connect it to internet and use AWS DataSync to send data
- Snowball Edge
  - physical transport solution: move TBs or PBs of data in or out of AWS
  - alternative for moving data ovewr the network (and paying network fees)
  - pay per data transfer job
  - provide block storage and S3-compatible object storage
  - Snowball Edge Storage Optimized
    - 80 TB of HDD capacity for block volume and S3 compatible object storage
  - Snowball Edge Compute Optimized
    - 42 TB of HDD capacity for block volume and S3 compatible object storage
  - Use Cases: large data cloud migrations, Data Center decommission, disaster recovery
- Snowmobile
  - Transfer exabytes of data (1 EB = 1,000 PBs = 1,000,000 Tbs)
  - each Snowmobile has 100 PB of capacity (use multiple in parallel)
  - high security, tempurature controlled, GPS, 24/7 video surveillance
  - **better than Snowball if you transfer more than 10 PB**

|                    |           Snowcone            | Snowball Edge Storage Optimized |       Snowmobile        |
| :----------------: | :---------------------------: | :-----------------------------: | :---------------------: |
|  Storage Capacity  |          8 TB Usable          |          80 TB Usable           |        < 100 PB         |
|   Migration Size   | up to 24 TB, online & offline |    up to petabytes, offline     | up to exabytes, offline |
|   DataSync Agent   |         pre-installed         |                X                |            X            |
| Storage Clustering |               X               |         up to 15 nodes          |            X            |

#### Usage Process

1. Request Snowball device from AWS console for delivery
2. Install then snowball client / AWS OpsHub on your servers
3. Connect the snowball to your servers and copy files using the client
4. Ship the device backe to AWS when you're done (goes to right AWS facilty (e-ink marker))
5. Data will be loaded into an S3 bucket
6. Snowball is completely wiped using the highest security measures

### Edge Computing

- process data while it is being created on **an edge location**
  - a truck on the road, ship on the sea, underground mining station
- these locations may have
  - limited / no internet access
  - limited / no easy access to computing power
- setup a Snowball Edge / Snowcone device to do edge computing
- use cases: Preprocess data, maching learning at the edge, transcode media streams
- eventuall (if needed) can ship back to AWS (for transferring data, etc)
- Snowcone
  - 2 CPUs, 4 GB of memory, wired or wireless access
  - USB-C power using a cord or the optional battery
- Snowball Edge (Compute Optimized)
  - 52 vCPUs, 208 GiB of RAM
  - optional GPU (useful for video processing or machine learning)
  - 42 TB usable storage
- Snowball Edge (Storage Optimized)
  - up to 40 vCPUs, 80 GiB of RAM
  - object storage clustering available
- All: can run EC2 instances & AWS lambda functons (uses AWS IoT Greengress)
- long-term deployment options: 1 & 3 years discounted pricing

### OpsHub

- historically, to use Snow Family devices you needed a CLI
- currently, can use OpsHub installed on your computer to manage Snow Family Devices
  - unlocking and configuring single or clustered devices
  - transferring files
  - launching and managing instances running on Snow Family Devices
  - Monitor device metrics (storage capacity, active instances on your device)
  - launch compatible AWS services on your devices (EC2, DataSync, Network File System)

### Snowball into Glacier

- **Snowball cannot import to Glacier direclty**
- must use S3 first in combination with S3 Lifecycle Policy

## Amazon FSx

- **launch 3rd party high-performance file systems on AWS**
- fully managed service
- FSx for Lustre
  - type of parallel distributed file system, for large scale computing
  - name is derived from "Linux" and "Cluster"
  - machine learning, **High Performance Computing (HPC)**
  - video processing, financial modeling, electronic design automation
  - scales up to 100s GB/s, millions of IOPS, sub-ms latencies
  - Storage Options
    - SSD: low-latency, IOPS intensive workloads, small & random file operations
    - HDD: throughput-intensive workloads, large & sequential file operations
  - seamless integration with S3
    - can "read S3" as a file system (through FSx)
    - can write the output of the computations back to S3 (through FSx)
  - can be used from on-premise servers (VPN or Direct Connect)
- FSx for NetApp ONTAP
  - managed NetApp ONTAP on AWS
  - **compatible with NFS, SMB, iSCSI protocol**
  - move workloads running on ONTAP or NAS to AWS
  - works with
    - Linux
    - Windows
    - MacOS
    - VMWare Cloud on AWS
    - Amazon Workspaces & AppStream 2.0
    - Amazon EC2, ECS, and EKS
  - shrinks and grows automatically
  - snapshots, replication, low-cost, compression, and data de-duplication
  - **point-in-time instantaneous cloning (helpful for testing new workloads)**
- FSx for Windows File Server
  - fully managed Windows file system share drive
  - supports SMB protocol & Windows NTFS
  - Microsoft Active Directory integration, ACLs, user quotes
  - **Can be mounted on Linux EC2 instances**
  - supports  **Microsoft's Distributed File System (DFS) Namespaces** (group files across multiple FS)
  - scale up to 10s of GB/s, millions of IOPS, 100s PB of data
  - Storage Options
    - SSD: latency sensitive workloads (databases, media processing, data analytics,...)
    - HDD: broad spectrum workloads (home directory, CMS,...)
  - can be access from you on-premise infrastructure (VPN or Direct Connect)
  - can be configured to be Multi-AZ (high availability)
  - data is backed up daily to S3
- FSx for OpenZFS
  - managed OpenZFS file system on AWS
  - file system compatible with NFS (v3, v4, v4.1, v4.2)
  - move workloads running on ZFS to AWS
  - works with
    - Linux
    - Windows
    - MacOS
    - VMWare Cloud on AWS
    - Amazon Workspaces & AppStream 2.0
    - Amazon EC2, ECS, and EKS
  - up to 1,000,000 IOPS with < 0.5ms latency
  - snapshots, compression, low-cost
  - **Point-in-time instantaneous cloning (helpfull for testing new workloads)**
- File System Deployment Options
  - Scratch File System
    - temporary storage
    - data is not replicated (doesn't persist if server fails)
    - high burst (6x faster, 200MBps per TiB)
    - usage: short-term processing, optimize cost
  - Persisten File System
    - long-term storage
    - data is replicated in same AZ
    - replace failed files within minutes
    - usage: long-term processing, sensitive data

## Storage Gateway

- AWS is pushing for "hybrid cloud"
  - part of infrastructure is on the cloud and on-premise
- this can be due to
  - long cloud migrations
  - security requirments
  - compliance requirements
  - IT strategy
- S3 is a proprietary storage technology (unlike EFS/NFS), so we use the Storage Gateway to expose S3 data on-premise
- Cloud Native Options
  - Block: EBS, EC2 instance store
  - File: EFS, FSx
  - Object: S3, Glacier
- Storage Gateway
  - bridge between on-premise data and cloud data
  - use cases:
    - disaster recovery
    - backup & restore
    - tired storage
    - on-premise cache & low-latency file-access

### Types of Storage Gateway

- S3 File Gateway
  - gateway converts NFS/SMB -> https
  - configured S3 buckets are accessible using the NFS and SMB protocol
  - **most recently used data is cached in file gateway**
  - supports Standard, Standard-IA, One Zone-IA, Intelligent Tiering
  - **transition to S3 Glacier using a Lifecycle Policy**
  - bucket access using IAM roles for each File Gateway
  - SMB Protocol has integration for Active Directory (AD) for user authentication
- FSx File Gateway
  - native access to FSx for Windows File Server
  - **local cache for frequently accessed data** (reason to use over native access)
  - Windows native compatibility (SMB, NTFS, Active Directory)
  - useful for group file shares and hom directories
- Volume Gateway
  - Block storage using the iSCSI protocol backed by S3
  - backed by EBS snapshots which can help restore on-premise volumes
  - Cached Volumes: low latency access to most recent data
  - Stored Volumes: entire dataset is on premise, scheduled backups to S3
  - useful for backups
- Tape Gateway
  - some companies have a backup process using physical tapes (!)
  - with Tape Gateway, companies use the same process but in the cloud
  - Virtual Tape Library (VTL) backed by Amazon S3 and Glacier
  - back up data using exisiting tape-based processes (and iSCI interface)
  - works with leading backup software vendors
- Hardware appliance
  - using Storage Gateway means you need on-premise virtualization
  - otherwise you can use Storage Gateway Hardware Appliance
  - can be bought on <amazon.com>
  - works with File, Volume, and Tape Gateways
  - has the required CPU, memory, network, SSD cache resources
  - helpful for daily NFS backups in small data centers

## Transfer Family

- fully managed service for file transfers in and out of Amazon S3 or EFS using the FTP protocol
- supported protocols
  - AWS Transfer for FTP (File Transfer Protocol)
    - only within VPC
  - AWS Transfer for FTPS (File Transfer Protocol over SSL)
  - AWS Transfer for SFTP (Secure File Transfer Protocol)
- managed infrastructure, scalable, highly reliable (multi-AZ)
- pay per provisioned endpoint per hour + data transfers in GB
- store and manage users' credentials within the service
- integrate wiht exisiting authentication systems (Active Directory, LDAP, Okta, Amazon Congnito, custom)
- usage: sharing files, public datasets, CRM, ERP

## DataSync

- move a large amount of data to and from
  - on premise / other cloud to AWS (NFS, SMB, HDFS, S3 API,...) - **needs agent**
    - agent is installed on premise or in other cloud
  - AWS to AWS (different storage services) - no agent needed
- can synchronize to
  - S3 (any storage class, including Glacier)
  - EFS
  - FSx (Windows, Lustre, NetApp, OpenZFS)
- replication task can be scheduled hourly, daily, weekly
- **File permissions and metadata are preserved** (NFS POSIX, SMB)
- one agent task can use 10Gbps, can setup bandwidth limit
- **data sync without network capacity, use Snowcone bc datasync agent is pre-installed**

## Storage Comparison

|             Type             |                                 Use                                  |
| :--------------------------: | :------------------------------------------------------------------: |
|              S3              |                            object storage                            |
|          S3 Glacier          |                           object archival                            |
|         EBS Volumes          |            network storage for one EC2 istance at a time             |
|       Instance Storage       |                 physical storage for EC2 (high IOPS)                 |
|             EFS              |      Network File System for Linux instances, POSIX filesystem       |
|       FSx for Windows        |               Network File System for Windows servers                |
|        FSx for Lustre        |             High Performance Computing Linux file system             |
|     FSx for NetApp ONTAP     |                        High OS compatibility                         |
|       FSx for OpenZFS        |                       Managed ZFS file system                        |
|       Storage Gateway        | S3 & FSx File Gateway, Volume Gateway (cache & stored), Tape Gateway |
|       Transfer Family        |        FTP, FTPS, SFTP, interface on top of Amazon S3 or EFS         |
|           DataSync           |         scheduled data sync on-premise to AWS, or AWS to AWS         |
| Snowcone/Snowball/Snowmobile |          move large amounts of data to the cloud physically          |
|           Database           |      for specific workloads, usually with indexing and querying      |
