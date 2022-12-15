# Networking

## CIDR, Private, vs Public IP

### **Classless Inter-Domain Routing (CIDR)** - a method for allocating IP addressess

- **Base IP**
  - represents an IP contained in the range (XX.XX.XX.XX)
  - example: 10.0.0.0, 192.168.0.0
  - made of 4 octets
- **Subnet Mask**
  - defines how many bits can change in the IP
  - example: /0, /24, /32
  - can take two forms:
    - /0 <-> 0.0.0.0 (all octets can change)
    - /8 <-> 255.0.0.0 (last 3 octets can change)
    - /16 <-> 255.255.0.0 (last 2 octets can change)
    - /24 <-> 255.255.255.0 (last octet can change)
    - /32 <-> 255.255.255.255 (no octets can change)
    - **formula: 2^(32-/X)**
- used in **Security Groups** ruls and AWS networking in general
- they help to define an IP address range:
  - WW.XX.YY.ZZ/32 -> one IP
  - 0.0.0.0/0 -> All IPs
  - we can define 192.16.0.0/26 -> 192.168.0.0 - 192.162.0.63 (64 IP addresses)

### Public vs Private IP (IPv4)

- the Internet Assigned Numbers Authority (IANA) established certain blocks of IPv4 addresses for the use of private (LAN) and public (internet) addresses
- **Private IP** can only allow certain values
  - 10.0.0.0 - 10.255.255.255 (10.0.0.0/8) - Big networks
  - 172.16.0.0 - 172.31.255.255 (10.0.0.0/12) - AWS default VPC
  - 192.168.0.0 - 192.168.255.255 (10.0.0.0/16) - e.g., home networks
- All the rest of the IP addresses on the Internet are Public

## Default VPC

- All new AWS accounts have a defualt VPC
- new EC2 instances are launching into the default VPC if no subnet is specified
- Default VPC has Internet connectivity and all EC2 instances inside it have IPv4 addressses
- we also get a public and private IPv4 DNS name

## VPC

- **VPC = Virtual Private Cloud**
- you can have multiple VPCs in an AWS Region
  - 5 per region - soft limit
- Max CIDR per VPC is 5, for each CIDR
  - min size is /28 (16 IP addresses)
  - max size is /16 (65536 IP addresses)
- because VPC is private only the Private IPv4 ranges are allowed
  - 10.0.0.0 – 10.255.255.255 (10.0.0.0/8)
  - 172.16.0.0 - 172.16.255.255 (172.16.0.0/12)
  - 192.168.0.0 - 192.168.255.255 (192.168.0.0/16)

## Subnet

- subrange of IPv4 addresses within VPC
- AWS reserves 5 IP addresses (first 4 and last 1) in each subnet
- these 5 IP addresses are not available for use and can't be assigned to an EC2 instance
- Example: if CIDR block 10.0.0.0/24, then reserved IP addresses are:
  - 10.0.0.0 - Network Address
  - 10.0.0.1 - reserved by AWS for the VPC router
  - 10.0.0.2 - reserved by AWS for mapping Amazon-provided DNS
  - 10.0.0.3 - reserved by AWS for future use
  - 10.0.0.255 - NEtwork Broadcast Address. AWS does not support broadcast in a VPC, therefore the address is reserved
- **Exam Tip**, if you need 29 IP addresses for EC2 instances:
  - You cannot chose a subnet of size /27 (32 IP addresses, 32-5 = 27 < 29)
  - You need chose a subnet of size /26 (64 IP addresses, 64-5 = 59 > 29)

## Internet Gateways & Route Tables

- allows resources (e.g., EC2 instances) in a VPC to connect to the Internet
- it scales horizonatally and is highly available and redundant
- must be created seperately from a VPC
- one VPC can only be attached to on IGW and vice versa
- Internet Gateways on their own dont allow Internet access
- route tables must also be edited

## Bastion Hosts

- we can use a Bastion Host to SSH into our private EC2 instances
- the bastion is in the public subnet which is then connected to all other private subnets
- **Bastion Host security group must allow** inbound from the internet on port 22 from restricted CIDR, for example the **public CIDR** of your corporation
- Security Group of the EC2 instances must allow the Security Group of the Bastion Host ort the **private IP** of the Bastion Host

## NAT Instances (outdated, NAT Gateway is newer)

- **Network Address Translation = NAT**
- allows EC2 instances in privte subnets to connect to the internet
- must be launched in a public subnet
- must disable EC2 setting: **Source/Destination Check**
- must have an Elastic IP attached to it
- Route Tables must be configured to route traffic from private subnets to the NAT Instance
- pre-configured Amazon Linux AMI is available
  - reached end of standard support on December 31, 2020
- not highly available/resislient setup out of the box
  - you need to create an ASG in multi-AZ + resilient data user script
- internet traffic bandwidth depends on EC2 instance type
- you must manage security groups and rules
  - Inbound
    - Allow HTTP/HTTPS traffic coming from Private Subnets
    - Allow SSH from your home network (access is provided through Internet Gateway)
  - Outbound
    - Allow HTTP/HTTPS traffic to the Internet

## NAT Gateway

- **Network Address Translation = NAT**
- AWS-managed NAT, higher bandwidth, high availability, no administration
- pay per hour for usage and bandwidth
- NATGW is created in a specific Availability Zone, uses and Elastic IP
- cant be used by EC2 instance in the same subnet (only from other subnets)
- requires an IGW (Private Subnet -> NATGW -> IGW)
- 5 Gbps of bandwidth with automatic scaling up to 45 Gbps
- no security groups to manage/required
- High Availability
  - **NAT Gateway is resilient within a single AZ**
  - must create **multiple NAT Gateways** in **multiple AZs** for fault tolerance
  - there is no cross-AZ failovert needed because if an AZ goes down, it doesnt need a NAT

|                     |                   NAT Gateway                    |                   NAT Instance                    |
| :-----------------: | :----------------------------------------------: | :-----------------------------------------------: |
|    Availability     | Highly Available within AZ (Creat in another AZ) | Use a script to manage failover between instances |
|      Bandwidth      |                  up to 45 Gbps                   |           Depends on EC2 instance type            |
|     Maintenance     |                  Managed by AWS                  |       Managed by You (software, OS patches)       |
|        Cost         |      per hour & amount of data transferred       |  per hour, EC2 instance type & size, + netowrk $  |
|     Public IPv4     |                     &check;                      |                      &check;                      |
|    Private IPv4     |                     &check;                      |                      &check;                      |
|   Security Groups   |                     &cross;                      |                      &check;                      |
| Use as Bastion Host |                     &cross;                      |                      &check;                      |

## Network Access Control List (NACL) & Security Groups

- NACL -> Security Groups
- NACL is stateless, evalutated inboud and outbound
- SG is stateful, if allowed in then allowed out

### Network Access Control List (NACL)

- NACL are like a firewall which control traffic from and to subnets
- **One NACL per subnet**, new subnets are assigned the **Default NACL**
- you define **NACL Rules**
  - rules hav a number (1-32766), lower number = higher precedence
  - first rule match with drive the decision
  - example: if you define `#100 ALLOW 10.0.0.10/32` and `#200 DENY 10.0.0.10/32`, the IP address will be allowed because 100 has a higher precedence over 200
  - the last rule is an asterisk (*) and denies a request in case of no rule match
  - AWS recommends adding rules by increments of 100
- newly created NACLs will deny everything
- NACL are a great way of blocking specific IP addresses at the subnet level

### Default NACL

- accepts everying inbound/outbound with the subnets it is associated with it
- do **NOT** modify the Defaul NACL, instead create custom NACLs

#### Inbound Rules

| Rule # |       Type       | Protocol | Port Range |  Source   | Allow/Deny |
| :----: | :--------------: | :------: | :--------: | :-------: | :--------: |
|  100   | All IPv4 Traffic |   All    |    All     | 0.0.0.0/0 |   Allow    |
|   *    | All IPv4 Traffic |   All    |    All     | 0.0.0.0/0 |    Deny    |

#### Outbound Rules

| Rule # |       Type       | Protocol | Port Range |  Source   | Allow/Deny |
| :----: | :--------------: | :------: | :--------: | :-------: | :--------: |
|  100   | All IPv4 Traffic |   All    |    All     | 0.0.0.0/0 |   Allow    |
|   *    | All IPv4 Traffic |   All    |    All     | 0.0.0.0/0 |    Deny    |

### Ephemeral Ports

- for any two endpoints to establish a connection, they must use ports
- clients connect to a **defined port**, and expect a response on an **ephemeral port**
- different operating sytems use different port ranges, examples
  - IANA & MS Windows 10 = 49512 -> 65535
  - Many Linux Kernals = 32768 -> 60999
- **use port ranges to cover ephemeral ports in NACL rules**

|                               Security Groups                               |                                                        NACL                                                        |
| :-------------------------------------------------------------------------: | :----------------------------------------------------------------------------------------------------------------: |
|                       Operates at the instance level                        |                                            operates at the subnet level                                            |
|                              Allow rules only                               |                                                 Allow & Deny rules                                                 |
| Statefule: return traffic is automatically allowed, regardless of any rules |               Stateless: return traffic must be explicitly allowed buy rules (think ephemeral ports)               |
|      all rules are evaluated before dedciding whether to allow traffic      | rules are evaluated in order (lowest to highest) when deciding to allow whether to allow traffic, first match wins |
|            applies to an EC2 instance when specified by someone             |                 automatically applies to all EC2 isntances in the subnet that it's associated with                 |

## VPC Peering

- privately connect two VPCs using AWS network
- make them behave as if they are the same network
- must not have overlapping CIDRs
- VPC Peering connection is **NOT transitive**
  - must be established for each VPC that need to communicate with each other
- **you must update route tables in *each VPC's subnet* to ensure EC2 instances can communicate with each other**
- you can create VPC Peering between VPCs in **different AWS accounts/regions**
- you can reference Security Groups in a peered VPC (works across acount, same region)

## VPC Endpoints (AWS Private Link)

- every AWS service is publicly exposed (public URL)
- VPC endpoints (powered by AWS Private Link) allows you to connect to AWS services using a **private network** instead of the public internet
- they are redundant and scale horizontally
- they remove the need of IGW, NATGW, to access AWS Services
- in case of issues
  - Check DNS Setting Resolution in your VPC
  - Check Route Tables
- Types of Endpoints
  - Interface Endpoints (powered by Private Link)
    - provisions an ENI (private IP address) as an entry
    - supports most AWS services
    - $ per hour + $ per GB of data processesed
  - Gateway Endpoints
    - provisions a gateway and must be used **as a target in a route table (does not use security groups)**
    - supports both S3 and DynamoDB
    - Free
- Gateway or Interface Endpoint for S3
  - **Gateway is most likely going to be preferred all the time at the exam**
  - Cost: free for gateway, $ for interface endpoint
  - interface endpoint is preferred if access is required from on-premise (site to site VPN or Direct Connect), a different VPC or a different Region

## VPC Flow Logs

- capture information about IP traffic going into your interfaces
  - VPC Flow Logs
  - Subnet Flow Logs
  - Elastic Network Interface (ENI) Flow Logs
- helps monitor & troubleshoot connectivity issues
- Flow logs data can go into S3 / CloudWatch logs
- captures network information from AWS Managed interfaces too
  - ELB, RDS, ElastiCache, Redshift, WorkSpaces, NATGW, Transit Gateway
- Format
  - **srcaddr & dstaddr** - help identify problematic IP
  - **srcport & dstport** - help identify problematic ports
  - **Action** - success or failure of the request due to Security Groups / NACL
    - Incoming Requests
      - Inbound REJECT = NACL or SG
      - Inbound ACCEPT, Outbound REJECT = NACL
    - Outgoing Requests
      - Outbound REJECT = NACL or SG
      - Outbound ACCEPT, Inbound REJECT = NACL
  - can be used for analytics on usage patterns, or malicious behavior
  - **Query VPC Flow Logs using Athena on S3 or Cloud Watch Log Insights**
- Architectures
  - VPC Flow Logs -> CloudWatch Logs -> CloudWatch Contributor Insights -> Top 10 IP addresses
  - VPC Flow Logs -> CloudWatch Logs -> CloudWatch Alarm -> Amazon SNS
  - VPC Flow Logs -> S3 Bucket -> Amazon Athena -> Amazon QuickSight

## Site to Site VPN, Virtual Private Gateway, and Customer Gateway

- Site-To-Site VPN
  - **create customer gateway, create Virtual Private Gateway on AWS, connect the two using Site-To-Site VPN**
- Virtual Private Gateway (VGW)
  - VPN concentrator on the AWS side of the VPN connection
  - VGW is created and attached to the VPC from which you want to create the Site-To-Site VPN connection
  - possibility to customize the ASN (Autonomous System Number)
- Customer Gateway (CGW)
  - software application or physical device on customer side of the VPN connection
  - What IP address to use?
    - Public Internet-routable IP addressm for your Customer Gateway Device
    - if it is behind a NAT device that is enabled for NAT traversal (NAT-T), use the public IP address for that NAT device
  - **Important Step:** enable **Route Propogation** for the Virtual Private Gateway in the Route Table that is associated with your subnets
  - if you need to ping your EC2 instances from on-premise, make sure you add the ICMP protocol on the inbound of your security groups
- AWS VPN CloudHub
  - provide secure comminication between multiple sites, if you have multiple VPN connection
  - low-cost hub-and-spoke model for primary or secondary network connectivity between different locations (VPN only)
  - it is a VPN connection so it goes over the public internet
  - to set up, connect multiple VPN connections on the same VGW, setup dynamic routing and configure route tables

## Direct Connect and Direct Connect Gateway

### Direct Connect

- provides a dedicated **private** connection from a remote network to your VPC
- dedicated connection must be setup between your Data Center and AWS Direct Connect Locations
- you need to setup a Virtual Private Gateway on your VPC
- access public resources (S3) and Privatee (EC2) on same network
  - uses Private and Public VIF (Virtual Interface )
- Use Cases:
  - increase bandwidth throughput - working with large datasets - lower cost
  - more consistent network experience - applications using real-time data feeds
  - Hybrid Environments (on-premise + cloud)
- supports both IPv4 and IPv6
- Direct Connect Gateway
  - **if you want to setup a Direct Connect to one or more VPC in many different regions (Same Account), you must use Direct Connect Gateway**
- Connection Types
  - Dedicated Connections: 1 Gbps, 10Gbps, and 100 Gbps
    - physicle ethernet port dedicated to a customer
    - request made to AWS first, then complete by AWS Direct Connect Partners
  - Hosted Connections: 50 Mbps, 500 Mbps, to 10Gbps
    - connection requests are made via AWS Direct Connect Partners
    - capacity can be added or removed on demand
    - 1,2,5,10 Gbps available at select AWS Direct Connect Partnets
  - **lead times are often longer than 1 month to establisha new connection**
- Encryption
  - data in transit is not encrypted, but it is private
  - AWS Direct Connect + VPN provides an IPsec-encrypted Private Connection
  - Good for extra level of security, but slightly more complex to put in place
- Resiliency
  - High Resiliency for Critical Workloads
    - One connection at multiple workloads
  - Maximum Resiliency for Critical Workloads
    - separate connections terminating on seperate devices in more than one location

### Direct Connect and Site to Site VPN

- in case Direct Connect Fails, you can setup a backup Direct Connect connection (expensive), or a Site-to-Site VPN connection

## Transit Gateway

- **for having transitive peering between thousands of VPC and on-premises, hub-and-spoke (star) connection**
- regional resource, can work cross-region
- share cross-account using Resource Access Manager (RAM)
- you can peer Transit Gateways across regions
- Route Tables: limit which VPCs can talk with other VPCs
- works with Direct Connect Gateway, VPN Connections
- *supports **IP Multicast** (not supported by any other AWS services)*
- Site-to-Site VPN ECMP
  - **Equal-Cost Multi-Path = ECMP**
  - routing strategy to allow packet forwarding over multiple best path
  - use case: create multiple Site-To-Site VPN conections to increase bandwidth of your connection to AWS
  - Throuput Examples
    - VPN to VPG = 1.25 Gbps
      - uses 2 tunnels
    - VPN to Transit Gateway
      - 1x = 2.5 Gbps (ECMP) - 2 tunnels used
      - 2x = 5.0 Gbps (ECMP)
      - 3x = 7.5 Gbps (ECMP)
      - a lot more money per GB of TGW procressed data
- Share Direct Connect between multiple accounts
  - Direct Connect -> Direct Connect Gateway -> Transit Gateway into both VPCS in different accounts

## VPC Traffic Mirroring

- allows you to capture and inspcet network traffic in your VPC non-intrusively
- route the traffic to security appliances that you manage
- Capture the traffic
  - From (Source) - ENIs
  - To (Targets) - an ENI or a Network Load Balancer
- capture all packets or capture the packets of your interest (optionally truncate packets)
- Source and Target can be in the same VPC or different VPCs (VPC Peering)
- use cases: content inspection, threat monitoring, troublshooting,...

## IPv6 for VPC

- **IPv4 cannot be disabled for your VPC and subnets**
- you can enable IPv6 (they're public IP addresses) to operate in dual-stack mode
- your ec2 instances will get at least a private internal IPv4 and a public IPv6
- they can communicate using either IPv4 and IPv6 to the internet through an Internet Gateway
- Troubleshooting
  - **IPv4 cannot be disabled for your VPC and subnets**
  - cant launch EC2 instance in your subnet
    - not because it cannot aqquire an IPv6 (space is very large)
    - is because there are no available IPv4 in your subnet
    - SOLUTION: create a new IPv4 CIDR in your subnet

### IPv6

- IPv4 was designed to provide 4.3 Billion addresses (they will be exhausted soon)
- IPv6 is the successor of IPv4
- IPv6 is designed to provide 3.4 x 10^38 unique IP addresses
- every IPv6 address is public and internet-routable (no private range)
- format x.x.x.x.x.x.x.x (x is hexadecimal, range can be from 0000 to ffff)
  - examples
    - 2001:db8:3333:4444:5555:6666:7777:8888
    - 2001:db8:3333:4444:cccc:dddd:eeee:ffff
    - :: -> all segments are 0
    - 2001:db8:: -> last 6 segments are 0
    - ::7777:8888 -> first 6 segments are 0
    - 2001:db8::7777:8888 -> middle 4 segments are 0

## Egress Only Internet Gateway

- **used only for IPv6**
- similar to NAT Gateway but for IPv6
- allows instances in your VPC outbound connections over IPv6 while preventing the internet initiating an IPv6 conection to your instances
- **you must update the Ro ute Tables**
- private VPC -> Egress-only Internet Gatewat

## VPC Section Summary

- **CIDR** - IP Range
- **VPC** - Virtual Private Cloud -> we define a list of IPv4 and IPv6 CIDR
- **Subnets** - tied to an AZ, we define a CIDR
- **Internet Gateway** - at the VPC level, provide IPv4 & IPv6 Internet Access
- **Route Tables** - must be edited to add routes from subnets to the IGW, VPC Peering Connections, VPC Endpoints
- **Bastion Host** - public EC2 instance to SSH into tha has SSH connectivity to EC2 instances in private subnets
- **NAT Instances** - give internet access to EC2 instances in private subnets, **OLD**, must be setup in public subnet, disable Source/Destination Check Flag
- **NAT Gateway** - managed by AWS, provides scalable internet access to private EC2 instances, IPv4 only
- **Private DNS + Route 53** - enable DNS resoultion + DNS Hostnames (VPC)
- **Network Access Control Lists (NACL)** - stateless, subnet rules for inbound and outbound, dont forget Ephemeral Ports
- **Security Groups** - statful, operate at the EC2 instance level
- **Reachability Analyzer** - perform network connectivity testing between AWS resources
- **VPC Peering** - connect 2 VPCs with non-overlapping CIDR, non-transitive (peer both to eachother)
- **VPC Endpoints** - provide private access to AWS services (S3, DynamoDB, CloudFormation, SSM) within a VPC
- **VPC Flow Logs** - can be setup at the VPC/Subnet/ENI level, for ACCEPT and REJECT traffic, helps identifying attacks, analyze using Athena or CloudWatch Logs Insight
- **Site-To-Site VPN** - setuip a Customer Gateway on DC, a Virtual Private Gateway on VPC, and site-to-site VPN over public internet
- **AWS VPN CloudHub** - hub-and-spoke VPN model to connect your sites
- **Direct Connect (DX)** - setup a Virtual Private Gateway on VPC and establish a direct private connection to an AWS Direct Connect Location
- **Direct Connect Gateway** - setup a Direct Connect to many VPCs in different AWS regions
- **AWS PrivateLink / VPC Endpoint Services**
  - connect services privately from your service VPC to customers VPC
  - doesnt need VPC Peering, public internet, NAt Gateway, Route Tables
  - must be used with Network Load Balancer & ENI
- **ClassicLink** - connect EC2-Classic EC2 instances privately to your VPC
- **Transit Gateway** - transitive peering connections for VPC, VPN, & DX
- **Traffic Mirroring** - copy network traffic from ENIs for further analysis
- **Egresss-only Internet Gateway**  - like NAT Gateway, but for IPv6

## Networking Costs in AWS

- Instance -> Instance within AZ = Free per GB
- Instance -> Instance and AZ -> AZ and Private IP = $0.01 per GB
- Instance -> Instance and AZ -> AZ and Public/Elastic IP = $0.02 per GB
- Region -> Region = $0.02 per GB
- use private IP instead of Public IP for good savings and better network performance
- use same AZ for maximum savings (at the cost of high availability)

### Minimizing egress traffic network cost

- Egress Traffic - outbound traffic (from AWS to outside)
- Ingress Traffic - inbound traffic - from outside AWS (typically free)
- try to keep as much internet traffic withiin AWS to minimze costs
- **Direct Connect location that are co-located in the same AWS region results in lower cost for egress network**

### S3 Data Transfer Pricing Analysius (USA)

- S3 ingress: free
- S3 to Internet: $0.09 per GB
- S3 Transfer Acceleration
  - faster transfer times (50-500% better)
  - additional cost on top of Data Transfer Pricing: +$0.04 to $0.08 per GB
- S3 to CloudFront: $0.00 per GB
- CloudFront to Internet: $0.085 per GB (slightly cheaper than S3)
  - caching capability (low latency)
  - reduce costs associated with S3 Requests Pricing (7x cheaper with CloudFront)
- S2 Cross-Region Replication: $0.02 per GB

### NAT Gateway vs Gateway VPC Endpoint

- VPC Endpoint is cheaper, no cost for using and $0.01 Data transfer in/out in same region
- NAT Gateway $0.045 NAT Gateway / hour, $0.045 NAT Gateway Data Processed / GB, $0.09 Data Transfer out to S3 (same/cross region)

## AWS Network Firewall

- protect entire AWS VPC
- Layer 3 to Layer 7
- Any direction, you can inspect:
  - VPC to VPC traffic
  - Outbound to internet
  - Inbound from internet
  - to/from Direct Connect & Site-to-site VPN
- internally, the AWS network firewall uses the AWS Gateway Load Balancer
- rules can be centrally mannaged cross-account buy AWS Firewall Manager to apply to many VPCs
- Fine Grained Controls
  - supports 1000s of rules
  - IP & Port - ex: 10000s of IPs filtering
  - Protocol - ex: block SMB protocol for outbound communications
  - Stateful domain list rule groups: only allow outbound traffic to *.mycorp.com or 3rd party software repo
  - General pattern matching with Regex
- **Traffic Filtering: Allow, drop, or alert for the traffic that matches the rules**
- **Active Flow Inspection** to protect against network threas with intrusion-prevention capabilities (like Gateway Load Balancer, but all managed by AWS)
- send logs of rule matches to S3, CloudWatch Logs, Kinesis Data Firehose