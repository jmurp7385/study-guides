# AWS Security and Encryption: KMS, SSM, Parameter Store, CloudHSM, Shield WAF

## Encryption 101

- Encryption in flight (SSL)
  - data is encrypted before sending and decrypted after receiving
  - SSL certificates help with encryption (HTTPS)
  - encryption in flight ensures no MITM (man in the middle attack) can happen
- Server side encryption at rest
  - data is encrypted after being received by the server
  - data is dercrypted before being sent
  - it is stored in an encrypted form thanks to a key (usually a data key)
  - the encryption/decryption keys must be manmaged somewhere and the server must have access to it
- Client Side encryption
  - data is encrypted by the client and never decrypted by the server
  - data will be decrypted by a receiving client
  - the server should not be able to decrypt the data
  - could leverage **Envelope Encryption**

## Key Management Service (KMS)

- Anytime you hear "encryption" for an AWS service, it is most likely KMS
- AWS manages encryption keys for us
- fully integrated with IAM for authorization
- able to audit KMS Key usage using CloudTrail
- seamlessly integrated into moost AWS services (EBS, S3, RDS, SSM,...)
- **never store your secrets in plaintext, especially in your code**
  - KMS Key Encryption also available through API calls (SDK/CLI)
  - encrypted secrets can be stored in the code/environment variables
- KMS Keys Type
  - **KMS Keys is the new name of KMS Customer Master Key**
  - Symmetric (AES-256 keys)
    - single encryption key that is used to Encrypt and Decrypt
    - AWS services that are integrated with KMS use Symmetric CMKs
    - you never get access tot eh KMS Key unencrypted (must call KMS API to use)
  - Asymmetric (RSA & ECC key pairs)
    - Public (Encrypt) and Private (Decrypt) Key pair
    - used for Encrypt/Decrypt, or Sign/Verify operations
    - the public key is downloadable, but you can't access the Private Key unencrypted
    - use case: encryption outside of AWS by users who cany call the KMS API
- KMS Key Types
  - AWS Managed Key: free (aws/service-name, example;: aws/rds or aws/ebs)
  - Customer Manged Keys (CMK) created in KMS: $1/month
  - Customer Manged Keys imported (must be 256bit symmetric key): $1/month
- pay for API calls to KMS ($0.03/10000 calls)
- Automatic key rotation
  - AWS-managed KMS Key: automatic every 1 year
  - Customer-managed KMS Key: (must be enabled), automatic every 1 year
  - Imnported KMS Key: only manual roation possible using alias
- copying snapshots across regions
  - snapshot and re-encrypt with new KMS key in new region and restore

### KMS Key Policies

- control access to KMS keys "similar" to S3 bucket policies
- difference: you cannot control access without them
- **Default KMS Key Policy**
  - created if you dont provide a specific KMS Key Policy
  - complete access to teh key to the root user = entire AWS account
- **Custom KMS Key Policy**
  - define users, roles that can access the KMS key
  - define who can administer the key
  - useful for cross-account access of your KMS key

### KMS Example: Copying Snapshots Across Accounts

1. create a snapshot, encrypted with your own KMS Key (customer managed key)
2. **Attach a KMS Key policy to authorize cross-account access**
3. share the encrypted snapshot
4. (in target) create a copy of the snapshot, encrypt it with a CMK in your account
5. Create a volume from the snapshot

### KM Multi-Region Keys

- identical KMS keys in different AWS regions that can be used interchangeably
- Multi-Region keys have the same key ID, key material, automatic rotation,...
- encrypt in one region and decrypt in other regions
- no need to re-encrypt or making cross-region API calls
- KMS Multi-Region are NOT global (Primary + Replicas)
- each Multi-Region key is managed **independently**
- use case: global client-side encryption, encryption on Global DynamoDB, Global Aurora

### DynamoDB Global Tables and KMS Multi-Region Keys Client-Side encryption

- we can encrypt specific attributes client-side in our DynamoDB table using **Amazon DynamoDB Encryption Client**
- combined with Global Tables, the clien-side encrypted data is replicated to other regions
- if we use a multi-region key, replicated in the same region as the DynamoDB Global Table, then clients in these regions can use low-latency API calls to KMS in their region to decrypt the data client-side
- using client-side encryption we can protect specific fields and guarantee only decryption if the client hass access to an API Key
- **we can protect specific fields even from databse admins**

### Global Aurora and KMS Multi-Region Keys Client-Side encryption

- we can encrypt specific attributes client-side in our Aurora table using **Amazon Encryption Client**
- combined with Global Tables, the clien-side encrypted data is replicated to other regions
- if we use a multi-region key, replicated in the same region as the Global Aurora Tables, then clients in these regions can use low-latency API calls to KMS in their region to decrypt the data client-side
- using client-side encryption we can protect specific fields and guarantee only decryption if the client hass access to an API Key
- **we can protect specific fields even from databse admins**

## S3 Replication with Encryption

- **unencrypted objects and objects encryped with SSE-S3 are replicated by default**
- objects encrypted withi SSE-C (customer provided key) are never replicated
- For objects encrypted with SSE-KMS, you need to enable to option
  - specify which KMS Key to encrypt the objects within the target bucket
  - adapt the KMS Key Policy for the target key
  - an IAM role with kms:Decrypt for the source KMS Key and the kms:Encrypt for the target KMS Key
  - you might get KMS throttling errors, in which case you can ask for a Service Quotas Increase
- **you can use multi-region AWS KMS Keys, but they are currently treated as independant keys by Amazon S3 (the object will stil be decrypted and then encrypted)**

## Encypted AMI Sharing Process via KMS

1. AMI in Source Account is encrypted with KMS Key from Source Account
2. Must modify the image attribute to add a **Launch Permission** which corresponds to the specified AWS account
3. must share the KMS Keys used to encrypt the snapshot the AMI references with the Target Account / IAM Role
4. The IAM Role/User in the Target Account must have the permissions to DescribeKey, ReEncrypted, CreateGrant, Decrypt
5. when launching an EC2 instance from the AMI, optionally the target account can specify a new KMS key in its own account to re-encrypt the volumes

## SSM Parameter Store

- secure storage for configuration and secrets
- optional seamless encryption using KMS
- serverless, scalable, durable, easy SDK
- version tracking of configurations/secrets
- security through IAM
- notifications with Amazon EventBridge
- integration with CloudFormation
- Parameter Store Hierarchy - can store in a tree/"folder" hierarchy
- Parameter Policies
  - allow to assign a TTL to a parameter (expiration date) to force updating or deleting sensitive data, such as passwords
  - can assign multiple policies at a time

|                                                            |       Standard       |                Advanced                |
| :--------------------------------------------------------: | :------------------: | :------------------------------------: |
| Total # of parameters allowed (per AWS Account and Region) |        10,000        |                100,000                 |
|              Maximum size of parameter value               |         4 KB         |                  * KB                  |
|                Parameter policies available                |          No          |                  Yes                   |
|                            Cost                            | no additional charge |             Charges apply              |
|                      Storage Pricing                       |         Free         | $0.05 per advanced parameter per month |

## AWS Secrets Manager

- newer service, meant for storing secrets
- capability to force **rotation of secrets** every X days
- automate generation of secrets on rotation (uses Lambda)
- integration with Amazon RDS (MySQL, PostgreSQL, Aurora)
- secrets are encrypted using KMS
- mostly meant for RDS integration

### Multi-Region Secrets

- replicate secrets across multiple AWS Regions
- Secrets Manager keeps read replicas in sync with primary secret
- ability to promote a read replkicate Secret to a standalone Secret
- use cases: multi-region apps, disaster recovery strategies, multi-region DB

## AWS Certificate Manager (ACM)

- easily provision, manage, and deploy **TLS Certificates**
- provide in-flight encryption for websites (HTTPS)
- supports both public and private TLS certificates
- free of charge for public TLS certificates
- automatic TLS certificate renewal
- integrations with (load TLS certificates on)
  - Elastic Load Balancers (CLB, ALB, NLB)
  - CloudFront Distributions
  - APIs on API Gateway
- cannot use ACM with EC2 (can't be extracted)

### Requesting Public Certificates

1. list domains to be included in the certificate

   - Fully Qualified Domain Name (FQDN): corp.example.com
   - Wildcard Domain: *.example.com

2. select validation method: DNS Validation or Email Validation

   - DNS Validation is preffered for automation purposes
   - Email Validation will send emails to contact addresss in the WHOIS database
   - DNS Validation will leverage a CNAME record to DNS config (ex: Route 53)

3. It will take a few hours to get verified
4. The Public Certificate will be enrolled in automatic renewal

   - ACM automaticall renews ACM-generated certificates 60 days before expiry

### Importing Public Certificates

- option to generate the certificate outside of ACM and them import it
- **no automatic renewal**, must import new certificate before expiry
- **ACM sends daily expiration events** starting 45 days prior to expiration
  - the \# of days can be configured
  - events are appearing in EventBridge
- **AWS Config** has managed rule named *acm-certificate-expiration-check* to check for expiring certificates (configurable \# of days)


### API Gateway and ACM

- create a Custom Domain Name in API Gateway
- API Gateway Endpoint Types
  - Edge-Optimized (default)
    - requests are routed throught the CloudFront Edge locations (improves latency)
    - the API gateway still lives in only one region
    - **the TLS Certificate must be in the same region as CloudFront, in use-east-1**
    - then setup CNAME or (better) A-Alias record in Route 53
  - Regional
    - for clients within the same region
    - could manually combine with CloudFront (more control over the caching strategies and the distribution)
    - **the TLS Certificate must be imported on API Gateway, in the same region as the API Stage**
    - then setup CNAME or (better) A-Alias record in Route 53
  - Private
    - can only be accessed from your VPC using an interface VPC endpoint (ENI)
    - use a resource policy to define access

## Web Application Fire Wall (WAF)

- protects you web applications from common web exploits (Layer 7)
- **Layer 7 is HTTP** (vs Layer 4 is TCP/UDP)
- Deploy on
  - Application Load Balancer
  - API Gateway
  - CloudFront
  - AppSync GraphQL API
  - Cognito User Pool
- define Web ACL (Web Access Control List) Rules
  - **IP Set: up to 10,000 IP addresses** - use multiple Rules for more IPs
  - HTTP Headers, HTTP Body, or URI strings Protects from common attack - **SQL injection** and **Cross-Site Scripting (XSS)**
  - size constraints, **geo-matching (block countries)**
  - **Rate-based rules** (to count occurences of events) - **for DDoS protection**
- Web ACL are Regional except for CloudFront
- a rule group is **a reusable set of rules that you can add to a web ACL**
- WAF - Fixed IP while using WAF with a Load Balancer
  - WAF does not supoort the Network Load Balancer (Layer 4)
  - We can use Global Accelerator for fixed IP and WAF onm the ALB

## Shield - DDoS Attacks

- **DDoS: Distributed Denial of Service - many requests at the same time**
- AWS Shield Standard
  - free service that is activated for every AWS customers
  - provides protection from attacks such as SYN/UDB Floods, Reflection Attacks and other layer 3/4 attackss
- AWS Shield Advanced
  - optional DDoS mitigation service ($3,000/month/organization)
  - protect against more sophisticated attack on Amazon EC2, Elastic Load Balancing (ELB), Amazon CloudFront, AWS Global Accelerator, and Route 53
  - 24/7 access to AWS DDoS response team (DRP)
  - protect against higher fees during usage spikes due to DDoS
  - Shield Advanced automatic application layer DDoS mitigation automaticall creates, evaluates, and deploys AWS WAF rules to mitigate layer 7 attacks

## Firewall Manager

- **manage rules in all accounts of an AWS Organization**
- Security Policy: common set of security rules
  - WAF rules (Application Load Balance, API Gateway, CloudFront)
  - AWS Shield Advanced (ALB, CLB, NLB, Elastic IP, CloudFront)
  - Security Groups for EC2, Application Load Balancer, and ENI resources in VPC
  - AWS Network Firewall (VPC Levels)
  - Amazon Route 53 Resolver DNS Firewall
  - Policies created at the region level
- **rules are applied to new resources as they are created (good for compliance) across all and future accounts in your Organization**
- WAF vs Shield vs Firewall Manger
  - **WAF, Shield, and Firewall Manger are used together for comprehensive protection**
  - Define your Web ACL rules in WAF
  - for granular protection of your resources, WAF alone is the correct choice
  - if you want to use AWS WAF across accounts, accelerate WAF Configurations, automate the protection of new resources, use Firewall Manager with AWS WAF
  - Shield Advanced adds additional features on top of AWS WAF, such as dedicated support from the Shield Response Team (SRT) and advanced reporting
  - If you are prone to frequent DDoS attacks, consider purchasing Shield Advanced

## DDoS Protection Best Practices

- AWS Best Practices for DDoS Resiliency - Edge Location Mitigation (BP1, BP3)
  - **BP1 - CloudFront**
    - web application delivery at the edge
    - protect from DDoS Common Attacks (SYN floods, UDP reflection)
  - **BP1 - Global Accelerator**
    - access your application from the edge
    - integration with Shield for DDoS protection
    - helpful if your backend is not compatible with CloudFront
  - **BP3 - Route 53**
    - Domain Name Resolution at the edge
    - DDoS Protection mechanism
- AWS Best Practices for DDoS Resiliency - Best practices for DDoS Mitigation
  - **Infrastructure Layer Defense (BP1, BP3, BP6)**
    - protect Amazon EC2 against high traffic
    - includes using Global Accelerator, Route 53, CloudFront, Elastic Load Balancing
  - **Amazon EC2 with Auto Scaling (BP7)**
    - helps scale in case of sudden traffic surges including a flash crowd or a DDoS attack
  - **Elastic Load Balancing (BP6)**
    - Elastic Load Balancing scales with traffic increases and will distribute to many EC2 isntances
- AWS Best Practices for DDoS Resiliency - Application Layer Defense
  - **detect and filter malicious web requests (BP1, BP2)**
    - CloudFront cache static content and serve it from edge locations, protecting your backend
    - AWS WAF is used on top of CloudFront and Application Load Balancer to filter and block requests based on request signature
    - WAF rate-based rules can automatically block the IPs of bad actors
    - use managed rules on WAF to block attacks based on IP reputation, or block anonymous IPs
    - CloudFront can block specific geographies
  - **Shield Advanced (BP1, BP2, BP6)**
    - Shield Advanced automatic application layer DDoS mitigation automatically creates, evaluates, and deploys AWS WAF rules to mitigate layer 7 attacks
- **AWS Best Practices for DDoS Resiliency - Attack Surface Reduction**
  - **Obfuscating AWS Resourcess (BP1, BP4, BP6**)
    - using CloudFront, API Gateway, Elastic Load Balancing to hide your backend resources (Lambda Functions, EC2 Instances)
  - **Security groups and Network ACLs (BP5)**
    - use security groups and NACLs to filter traffic based on specific IP at subnet or ENI-level
    - Elastic IP are protected by AWS Shield Advanced
  - **Protecting API endpoints (BP4)**
    - hide EC2, Lambda, elsewhere
    - Edge-optimized mode or CloudFront + regional mode (more control for DDoS)
    - WAF + API Gateway: burst limits, headers filtering, use API Keys

## Amazon GuardDuty

- Intelligent Threat discovery to Protect AWS Account
- uses machine learning algorithms, anomaly detection, 3rd party data
- one click to enable (30 day trial), no need to install software
- Input Data includes
  - **CloudTrail Events Logs** = unusual API calls, unauthorized deployments
    - **CloudTrail Management Events** - create VPC subnet, create trail,...
    - **CloudTrail S3 Data Events** - get object, list objects, delete object,...
  - **VPC Flow Logs** - unusual internal traffic , unusual IP address
  - **DNS Logs** - compromised EC2 instances sending encoded data within DNS queries
  - **Kubernetes Audit Logs** - suspicious activities and potential EKS cluster compromises
- can setup **CloudWatch Event Rules** to be notified in case of findings
- CloudWatch Events rules can target AWS Lambda or SNS
- **can protect against CryptoCurrency attacks (has dedicated "finding" for it)**

## Amazon Inspector

- **Automated Security Assessments**
- **For EC2 Instances**
  - leveraging the **AWS System Manager** agent
  - analyze against **unintended network accessiblity**
  - analyze the **running OS** against **known vulnerabilities**
- **For Container Images push to Amazon ECR**
  - assessment of Container images as they are pushed
- **For Lambda Functions**
  - identifies software vulnerabilities in function code and package dependencies
  - assessment of functions as they are deployed
- Reporting and integration with AWS Security Hub
- send findings to Amazon EventBridge
- What does Inspector Evaluate
  - **only EC2 instances, container images, lambda functions**
  - continuous scanning of the infrastructure, only when needed
  - package vulnerabilities (EC2, ECR, & Lambda) - database of CVE
  - network reachability (EC2)
  - a risk score is associated with all vulnerabilities for prioritization

## Amazon Macie

- fully managed data security and data privacy service that uses **machine learning and pattern matching to discover and protect your sensitive data in AWS**
- helps identify and alert you to **sensitive data, such as personally identifiable information (PII)**
