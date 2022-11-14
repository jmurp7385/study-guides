# EC2 - Elastic Compute Cloud

- infrastructure as a service
  - renting virtual machines (EC2)
  - storing data on virtual drives (EBS)
  - distributing load across machines (ELB)
  - auto-scaling group to scale services (ASG)

## Sizing and Config Options

- OS
  - Windows, Mac, Linux
- Compute Power & Cores (CPU)
- RAM
- Storage
  - Network-attached (EBS & EFS)
  - hardware (EC2 instance store)
- Network Card
  - speed of card
  - public IP address
- Firewall rules (security group)
- Bootstrap script using EC2 User Data Script (configure at first launch)
  - launch commmands on machine start
  - only run once on first start
  - can be used to
    - install updates/software
    - download common files from the internet
    - anything else you want
  - runs with root user

## EC2 Instance Types

- Types
  - general purpose
    - diverse uses (web servers, code repos)
    - compute, memory, networking balances
  - compute optimized
    - for tasks that require high performance processors
      - batch processing
      - media transcoding
      - high performance web servers
      - high performance computing (HPC)
      - scientific modeling & machine learning
      - dedicated game servers
  - memory optimized
    - for processing large data sets in memory (RAM)
      - high perfomance databases
      - distributed web scale cache stores
      - in memory DB for Business intelligence
      - real-time processing of unstructured data
  - storage optimized
    - things that require high, sequential read & write access to large data sets on local storage
      - high frequency online transaction processing (OLTP) systems
      - relational & NoSQL databases
      - cache for in-memory databases
      - data warehousing applications
      - distributed file systems
- Naming conventions
  - ex m5.2xlarge
    - "m" is the instance class
    - "5" is the generation/versions
    - "2xlarge" is the size withing the instance class

### SSH & Other connections methods

- SSH
  - Mac, Linux, Window > 10
  - allows control of remote machine with command line
  - need a .pem key file on local machine (use -i to specify path to file)
  - chmod 0400 on .pem file to protect file
- Putty
  - Windows
- EC2 Instance Connect
  - Mac, Linux, Windows
  - uses SSH behind the scenes

### Instance Roles

- attach role to EC2 instance instead of running aws configure so that the credentials are hidden

### EC2 Instances Purchasing Options

- On-Demand Instances - short workload, predictable pricing, pay by second
  - pay for what you use
    - Linux & Windows - per sec
    - Other OS - per hour
  - highest cost, but no upfront payment
  - no long-term commitment
  - recommended for **short-term** and **uninterrupted workloads**, where you can't predict how and application will behave
- Reserved (1 & 3 years)
  - Reserved Instances - long workloads
    - 72% (% subject to change)
    - resereve specific instance attributes (instance type, region, tenancy, OS)
    - Reservation Period - 1 year (+discount) or 3 years (+++discount)
    - Payment Options - No Upfrount (+discount). Partial Upfront (++discount), or  All Upfront (+++discount)
    - Reserved Instance's Scope - Regional or Zonal (reserved capacity in an AZ)
    - recommended for steady-state usage applications (database, etc)
    - can buy and sell in Reserved Instance Marketplace
  - Convertible Reserved Instances - long workloads with flexible instances
    - can change instance attributes (instance type, region, tenancy, OS)
    - up to 66% discount (% subject to change)
- Savings Plans (1 & 3 years) - commit to an amouint of useage, long workload
  - get discount based on long-term usage (up to 72% - same as RIs)(% subject to change)
  - commit to a certain type of usage ($10/hour for 1 or 3 years)
  - usage beyond EC2 Savings Plans is billed at the On-Demand price
  - Locked to a specific instance family & AWS region (M5 in use-east-1)
  - Flexible across instance
    - size (xlarge or 2xlarge)
    - OS (Windows or Linux)
    - Tenancy (Host, Dedicated, Default)
- Spot instances - short workloads, cheap, can lose instances (less reliable)
  - can get up to 90% discount
  - can lose at any time if max price is less than the current spot price
  - most cost-effecient instances
  - Useful for workloads resisilient to failure
    - batch jobs
    - data analysis
    - image processing
    - distributed workloads
    - workloads withe a flexible start and end
  - *not suitable for critical jobs or databases*
  - hourly spot price varies with offer & capacity
  - 2 min grace period if spot > max price to stop/terminate
  - Spot Block
    - block spot instance for specified time frame (1 to 6 hours) without interruptions
    - rare situations the instance may be reclaimed
  - Spot Requests
    - need to be in open, active, opr disabled state to cancel
      - cancelling does not termintate
    - cancel the spot request first to terminate the associated spot instances
  - Spot Fleets
    - set of spot instances + (optional) On-Demand Instances
    - try to mee the target capacity with price contraints
      - define launch pools - instance type, OS, Availability Zone
      - can have multiple launch pools
      - stops launching instances when reaching capacity/max const
    - allocation strategies
      - lowestPrice - from pool with lowest price (cost optimization, short workload)
      - diversified - distributed across all pools (great availability, long workloads)
      - capacityOptimized - pool with optimal capacity for the number of instances
- Dedicated Hosts - book and entire physical server, control instance placement
  - physical server with EC2 instance capacity fully dedicated to your use
  - allows you to addresss **compliance requirements** and **use your exisiting service bound software licenses** (per-socket, per-core, per-VM software licenses)
  - most expensive option
  - useful for software that have complicated licensing model (BYOL - Bring Your Own License)
  - companies with strong regulatory/compliance needs
- Dedicated Instances - no other customers will share hardware
  - dedicated instance runs on dedicated hardware
  - may share hardware with other instances in same account
  - no control over instance placement (can move hardware after Start/Stop)
- Capacity Reservations - reserve capacity in specific AZ for any duration
  - reserve On-Demand instances capacity in a specific AZ for any duration
  - you always have access to EC2 capacity when you need it
  - No time commitment (create/cancel anytime) no discounts
  - combine with Regional Reserved Instances and Savings Plans to benefit from discounts
  - You are charged at On-Demand rate whether you run instances or not
  - suitable for short-term, uninterrupted workloads that needs to be in a specific AZ

## Security Groups & Classic Ports Overview

### Security groups

- Security groups are fundamental of network security in AWS
- control traffic in/out of EC2 instances
- only contain "allow" rules
- rules can reference by IP or security group
- act as firewall on EC2 instances
  - access to ports
  - authorised IP ranges (IPv4 and IPv6)
  - control of inbound network (other to instance)
  - control of outbound network (instance to other)
- can be attached to multiple instances
- locked down to a region/VPC combo
- lives outside the EC2
- good to maintain one separate group for SSH access
- if application is not accessible (time out) then it is a security group issues
- if application gives "connection refused" then it is an application error or it is not launched
- by default
  - all inbound traffic is **blocked**
  - all outbound traffic is **allowed**

### Classic Ports

- 22 = SSH - log into a Linux instance
- 21 = FTP (File Transfer Protocol) - upload files into a file share
- 22 = SFTP (Secure File Transfer Protocol) - upload files using SSH
- 90 = HTTP - access unsecured websites
- 443 = HTTPS - access secured websites
- 3389 = RDP (Remote Desktop Protocol) - log into a Windows instance

### EC2 Ips

- AWS has support for
  - IPv4 - 1.160.10.240
    - allows 3.7 billion different addresses
  - IPv6 - 1900:4545:3:200:f8ff:fe21:67cf
    - newer and solves problems for IoT

- Private
  - only accessible within private network
  - IP unique only on private network
  - 2 different private networks and share IPS
  - machines connect to www using and internet gateway (a proxy)
  - only a specified range of IPs can be used as private
- Public
  - accessible over internet
  - IP unique across all web
  - can be geo-located easily
- Elastic
  - fixed IP for EC2 instance
  - public IPv4 IP for as long as you don't delete it
  - one instance at a time
  - can mask the failure of an instance by rapidly remapping the address to another instance on account
  - default of only 5 per account (can be increased)
  - often reflect poor architectural decisions
    - instead use random IP and register a DNS name to it
    - can use a Load Balancer and not public ip at all

### EC2 Placement Groups

- placement groups are strategies over EC2 instance placement
- Placement Strategies
  - Cluster
    - cluster instances into a low-latency group in a single Availablility Zone
    - same rack & same AZ
    - Pros
      - great network
    - Cons
      - if rack fails, all instances fail at the same time
    - Use Cases
      - big gata job that needs to complete fast
      - application that needs extremely low latency and high network throughput
  - Spread
    - spreads instances across underlying hardware (max 7 isntances per group per AZ for critical applications
    - Pros
      - can span across multiple AZ
      - reduce failure risk
    - Cons
      - limited to 7 instances per AZ per placement group
    - Use Cases
      - application that needs to maximize high availability
      - critical applications where each instance must be isolated from failure from each other
  - Partition
    - 7 partitions per AZ
    - can span across multiple AZ in same region
    - up to 100s of EC2 instances
    - each partition is isolated from failure
    - EC2 get access to partition information as metadata
    - Use Cases
      - HDFS, HBase, Cassandra, Kafka

### Elastic Network Interfaces

- logical component in a VPC that represents a virtual network card
- Eth0 - primiary ENI
- Primary private IPv4, 1+ secondary IPv4
- 1 elastic IP (IPv4) per privatre IPv4
- 1 public IPv4
- 1+ security groups
- can create ENI independently and attach them on a fly (move them) on EC2 instances for failover
- bound to specific AZ

### EC2 Hibernate

- Stop - data on disk is kept intact in the next start
- Terminate - any EBS volumes (root) also set-up to be destroyed is lost
- Instance Timeline
  - First Start - OS Boots and User Data script is run
  - Following starts - os boots
  - Then application starts, caches get warmed up
- Hibernate
  - in-memory (RAM) state is preserved
    - RAM state is writted into a file in the root EBS volume
    - root EBS volumne must be encrypted
  - instance boot is much faster (OS is not stopped/restarted)
  - long-running processes
  - saving the RAM state
  - services that take a long time to initialize
  - Restrictions
    - Supported Instance Families - C3, C4, C5, I3, M3, M4, R3, R4, T2, T3
    - Instance RAM Size - < 150 GB
    - Instance Size - not supported for bare metal instances
    - AMI - Amazon Linux 2, Linux AMI, Ubuntu, RHEL, CentOS, & Windows
    - Root Volume - must be EBS, encrypted, not instance store, and large
    - On-Demand, Reserved, and Spot instances
    - cannot hibernate for more than 60 days
