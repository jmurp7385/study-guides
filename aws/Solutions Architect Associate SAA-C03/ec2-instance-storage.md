# EC2 Instance Storage

## EBS Volume - Elastic Block Store

- network drive attached to instances while running
  - might be some latency
  - can be detached from one isntance and reattached to another
- allows persisting of data even after termination
- only mounted to one instance at a time (at the CCP level)
- bound to a specific AZ
  - can move AZ with a snapshot
- Free Tier - 30 GB of EBS General Purpose SSD or Magnetic
- have to provision capacity (size GB and IO Operations/second (IOPS))
- delete on termination attribute
  - default root enabled
  - default other disabled

### Volume Types

- **gp2**/**gp3** (SSD) - general performance SSD with balanced price and performance
  - boot volumes, virtual desktops, dev/test envs
  - 1GiB - 16 TiB
  - **gp3**
    - baseline 3k IOPS and throughput of 125 MiB/s
    - can increase to 16k IOPS and throughput of 1000 MiB/s
  - **gp2**
    - small **gp2** volumes can burst IOPS to 3k
    - size of volume and IOPS are linked, max IOPS 16k
    - gain 3 IOPS per GB (5334 GB max IOPS)
- **io1**/**io2** (SSD) - highest performance SSD for mission critical low-latency or high-throughput workloads
  - provisioned IOPS SSD
  - for apps that nmeed more that 16k IOPS
  - great for database workloads
  - **io1**/**io2**
    - Max PIOPS - 64k for Nitro EC2 instances & 32k for other
    - can increase PIOPS independantly from storage size
    - **io2** have more durability and IOPS per GiB than **io1** at same price
  - **io2** Block Express
    - sub-ms latency
    - Max PIOPS - 256k with an IOPS:GiB ratio of 1000:1
    - Supports EBS multi-attach
- **st1** (HDD) - low cost HDD for frequently accessed, throughput-intensive workloads
  - 125 MiB to 16 TiB
  - max throughput 500 MiB/s - max IOPS 500
- **sc1** (HDD) - lowest cost HDD for less frequently accessed workloads
  - 125 MiB to 16 TiB
  - max throughput 250 MiB/s - max IOPS 250
  - when lowest cost is important
- characterized in Size | Throughput | IOPS (I/O Ops Per Sec)
- only **gp2**/**gp3** and **io1**/**io2** can be used as boot volumes

### General Purpose SSD volumes

|                           | General Purpose SSD                           | General Purpose SSD |
| :------------------------ | :-------------------------------------------- | :------------------ |
| Volume Type               | gp3                                           | gp2                 |
| Durability                | 99.8%-99.9%                                   | 99.8%-99.9%         |
| Use Cases                 | low-latency interactive apps<br>dev/test envs | ...                 |
| Volume Size               | 1 GiB - 16 TiB                                | ...                 |
| Max IOPS per volume       | 16000                                         | ...                 |
| Max throughput per volume | 1000 MiB/s                                    | 250 MiB/s           |
| EBS Multi-attach          | Not supported                                 | ...                 |
| Boot Volume               | Supported                                     | ...                 |

### Provisioned IOPS SSD volumes

|                           | Provisioned IOPS SSD                                                                     | Provisioned IOPS SSD                                                    | Provisioned IOPS SSD |
| :------------------------ | :--------------------------------------------------------------------------------------- | :---------------------------------------------------------------------- | :------------------- |
| Volume Type               | io2 Block Express                                                                        | io2                                                                     | io1                  |
| Durability                | 99.999%                                                                                  | 99.999%                                                                 | 99.8%-99.9%          |
| Use Cases                 | sub-ms latency<br>sustained IOPS performance<br>over 64000 IOPS<br>1000 MiB/s throughput | sustained IOPS<br>over 16000 IOPS<br> I/O instensive database workloads | ...                  |
| Volume Size               | 4 GiB - 64 TiB                                                                           | 4 GiB - 16 TiB                                                          | ...                  |
| Max IOPS per volume       | 256000                                                                                   | 64000                                                                   | ...                  |
| Max throughput per volume | 4000 MiB/s                                                                               | 1000 MiB/s                                                              | ...                  |
| EBS Multi-attach          | Supported                                                                                | ...                                                                     | ...                  |
| Boot Volume               | Supported                                                                                | ...                                                                     | ...                  |

### Throughput Optimized HDD and Cold HDD volumes

|                           | Throughput Optimized HDD                      | Cold HDD                                                                                            |
| :------------------------ | :-------------------------------------------- | :-------------------------------------------------------------------------------------------------- |
| Volume Type               | st1                                           | sc1                                                                                                 |
| Durability                | 99.8%-99.9%                                   | 99.8%-99.9%                                                                                         |
| Use Cases                 | big data<br>data warehouses<br>log processing | throughput oriented storage for data that is infrequently accessed<br>lowest cost is most important |
| Volume Size               | 125 GiB- 16 TiB                               | ...                                                                                                 |
| Max IOPS per volume       | 500                                           | 250                                                                                                 |
| Max throughput per volume | 500 MiB/s                                     | 250 MiB/s                                                                                           |
| EBS Multi-attach          | Not supported                                 | ...                                                                                                 |
| Boot Volume               | Not supported                                 | ...                                                                                                 |

### EBS Snapshots

- backup at any point in time of EBS volume
- recommended to detach volume before snapshotting, but not required
- can copy snapshots accross AZ or Region
- EBS Snapshot Archive
  - move snapshot to archive tier (75% cheaper)
  - takes 24 to 72 hrs for restoring
- Recycle Bin for EBS Snapshots
  - setup rules to retain deleted snapshots so they can be recovered
  - specify retention (1 day to 1 year)
- Fast Snapshot Restore
  - for full initialization of snapshort to hjave not latency on first use
    - costs more money

### EBS Multi-Attach

- **only io1/io2 volume families**
- attach the same EBS Volumes to multiple EC2 instances in the same AZ
- each instance will have full read & write permissions to the high-performance volume
- Use cases
  - achieve higher application availability in clustered linux applications
  - applications must manage concurrent write operations
- **up to 16 EC2 instances at a time**
- must use a file system that is cluster-aware (not XFS, EX4, etc)

### EBS Encryption

- encrypted volumes have
  - data at rest is encrypted inside volume
  - all data in flight between instance and volume is encrypted
  - all snapshots are encrypted
  - all volumes created from snapshot are encrypted
- encryption/decryption are handled transparently (you have nothing to do)
- minmal impace on latency
- EBS Encryption leverages kets from KMS (AES-256)
- copying an unencrypted snapshot allows encryption
- encrypt an unencrypted EBS Volume
  1. Create EBS snapshot of volume
  2. Encrypt the EBS snapshot using copy
  3. Create new EBS volume from the snapshot (volume will also be encrypted)
  4. Attach encrypted volume to the original instance

## AMI - Amazon Machine Image

- customization of an EC2 instance
  - add your own software, configuration, OS, monitoring
  - faster boot / config time bc all software is pre-packaged
- built for a specific region (can be copied across regions)
- Types
  - Public AMI - AWS provided
  - Your own AMI - you make and maintain on your own
  - AWS Marketplace AMI - AMI someone else made and potentially sells
- Process
  - Start EC2 instance and customize it
  - Stop instance (for data integrity)
  - Build an AMI - this will create ebs snapshots
  - Launch new instances from new AMI in new AZs

## Instance Store

- high performance hardware disk
- better I/O performance
- lose storage is stopped (ephemeral)
- good for buffer/cache/scratch data/temporary content
- risk of data loss if hardware fails

## EFS - Elastic File System

- Manged NFS (network file system) that can be mounted on many EC2
- EFS works with EC2 instances in multiple AZ
- highly available, scalable, expensive, (3x gp2), pay per use
- use cases
  - content management
  - web serving
  - data sharing
  - Wordpress
- uses NFSv4.1 protocol
- use security groups to control access
- compatible with Linux based AMI (not Windows)
- encryption at rest using KMS
- POSIX file system (~Linux) that has standard file API
- file systems scales automatically, pay-per-use each GB, no capacity planning

### Performance

- EFS Scale
  - 1000s of concurrent NFS clients, 10 GB+ /s throughput
  - grow to petabyte scale network file system automatically
- Performance Mode
  - general purpose (defualt): latency-sensitive use cases (web server, CMS, etc)
  - max I/O - higher latency, throughput, highly parallel (big data, media processing)
- Throughput Mode
  - bursting (1TB = 50 MiB/s + burst up to 100 Mib/s)
  - Provisioned: set your throughput regardless of storage size, ex 1 GiB for 1 TB storage

### Storage Classes

- Storge Tiers (lifecycle management feature - move files after N days)
  - standard - frequently accessed files
  - Infrequent access (EFS-IA)
    - cost to retrieve files
    - lower price to store
    - can be enabled with a lifecycle policy
- Availablity and Durability
  - standard: Multi-AZ, great for prod
  - One Zone: One AZ, great for dev, backup enabled by default, compatible with IA (EFS One Zone-IA)
    - over 90% cost savings
- Example Questions: When to use EFS and what options to set to ensure requirments are met?

## EBS vs EFS

- EBS (Elastic Block Storage)
  - attached to one instance at a time
  - locked at the Availability Zone level
  - gp2: IO increases if the disk size increases
  - io1: can increase IO independantly
  - migrate EBS across AZ
    - take a snapshot
    - restore snapshot to another AZ
    - EBS backups use IO and you shouldnt run them while your application is handling a lot of traffic
  - Root EBS Volumes of instances get terminated by default if the EC2 instance gets terminated
    - can be disabled
- EFS (Elastic File System)
  - mounting 100s of instances across AZ
  - EFS share website files (WordPress)
  - only for Linux Instances (POSIX)
  - higher price point than EBS
  - can leverage EFS-IA for cost savings
  - pay for what you use
- Instance Store
  - max I/O
  - ephemeral drive
