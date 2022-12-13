# Advanced IAM

## Organizations

- global service
- allows to manage multiple AWS accounts
- the main account is the management account
- other accounts are member accounts
- memeber accounts can only be part of one organization
- consolidated billing across all accounts - single payment method
- pricing benefits from aggregated usage (volume discount for EC2, S3)
- **shared reserved instances and Saving Plans disocunts across accounts**
- API is available to automate AWS account creation

### Organization Advantages

- Multi Account vs One Acount Multi VPC
- use tagging standards for billing purposes
- enable CloudTrail on all accounts, send logs to central S3 account
- send CloudWatch logs to central logging account
- establish Cross Account Roles for Admin purposes

### Organization Security - Service Control Policies (SCP)

- IAM policies applied to OU or accounts to restrict Users and Roles
- they do not apply to the management account (full admin power)
- must have an explicit allow (does not allow anything byu default - like IAM)

## Advanced Policies

### IAM Conditions

- aws:SourceIP - restrict the client IP **from** which the API calls are being made
- aws:RequestedRegion - restrict the region the API calls are made **to**
- ec2:ResourceTag - restrict based on tags
- aws:MultiFactorAuthPresent - to force MFA
- S3: arn vs arn/* is the difference between bucket level permissions and object level permissions
- aws:PrincipalOrgID - can be used in any resource policies restrict access to accounts that are member of an AWS Organization

## Resource-based Policies vs IAM Roles

- Cross Account
  - attaching a resource-based policy to a resource (ex: S3 bucket policy) or using a role as a proxy
- **when you assume a role (user, application, service), you givce up your original permissions and take the permissions of the assigned role**
- when using a resource-based policy, the principle doesnt have to give up his permissions
- example: user in Account A needs to scan a DynamoDB table in Account A and dump it in an S3 bucket in Account B
- supported by S3 Buckets, SNS topics, SQS queues, etc...
- EventBridge Security
  - when a rule runs it needs permission on the target
  - **Resource-Based Policy - Lambda, SNS, SQS, CloudWatch Logs, API Gateway**
  - **IAM Role: Kinesis stream, Systems Manger run Command, ECS task**

## [Policy Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)

- Permission Boundaries
  - supported for users and roles (not groups)
  - advanced feature to use a managed policy to set the maximum permissions and IAM entity can get
  - can be used in combinations of AWS Organizations SCP
  - explicit denys take precedence
  - use cases:
    - delegate responsibility to non administrators within their permission boundaries, for example creat new IAM users
    - allow developers to self-assign policies and manage their own permissions while making sure they cannot escalate their priviledges (make themselves admin)
    - useful to restrict one specific user (instead of a whole account using Organizations &  SCP)

## Amazon Cognito

- give users an identity to interact with our web or mobile application
- Cognito User pools:
  - sign in functionality for app users
  - integrate with API Gateway & Application Load Balancer
- Cognito Identity Pools (Federated Identity)
  - provide AWS credentials to users so they can access AWS resources directly
  - integrate with Cognito User Pools as an idenity provider
- Cognito vs IAM
  - "hundreds of users", "mobile users", "authenticate with SAML"

### Cognito User Pools (CUP)

- User Features
  - create a serverless database of users for your web/mobile applications
  - simple login: username or email and password
  - password reset
  - email & phone number verification
  - multi-factor authentication (MFA)
  - Federated Identities: users from Facebook, Google, SAML
- Integrations
  - CUP integrates with **API Gateway** and **Application Load Balancer**

### Cognito Identity Pools (Federated Identities)

- get identities for "users" so they can obtain temporary AWS credentials
- users source can be Cognito User Pools, 3rd Party Logins, etc,...
- **users can then access AWS services directly or through API Gateway**
- the IAM policies applied to the credentials are defined in Cognito
- they can be customized based on the user_id for fine grained control
- **Default IAM roels** for authenticated and guest users

## AWS IAM Identity Center (successor to AWS Single Sign-On)

- one login (single sign-on) for all you
  - **AWS accounts in AWS Organizations**
  - business cloud applications (e.g., Salesforce, Box, Microsoft 365, ...)
  - SAML2.0-enabled applications
  - EC2 Windows Instance
- Identity providers
  - build-in identity store in IAM Identity Center
  - 3rd Party: Active Directory (AD), OneLogin, Okta,...
- Multi-account Permissions
  - manage access across AWS accounts in your AWS Organizaton
  - permission sets - a collection of one or more IAM Policies assigned to usrs and groups to define AWS Access
- Application Assignments
  - SSO access to many SAML 2.0 business applications (Salesforce, Box, Microsoft 365)
  - provide requried URLs, certificates, and metadata
- Attribute-Based Access Control (ABAC)
  - fine-grained permissions based on users' attributes stored in IAM Identity Center Identity Stores
  - example: cost center, title, locale, ...
  - use case: define permissions once then modify AWS access by changing the attributes

## AWS Directory Services

### Microsoft AD

- found on any Windows Server with AD Domain Services
- database of objects: Users, Accounts, Computers, Printers, FIle Shares, Secutity Groups
- centralized security management, create account, assign permissions
- objects are organized in **trees**
- a groups of tress is a **forest**

### AWS Managed Microsoft AD

- create your own AD in AWS, manage users locally, supports MFA
- establish "trust" connections with your on-premise AD

### AD Connector

- directory gateway (proxy) to redirect to on-premise AD, supports MFA
- users are managed on the on-premise AD

### Simple AD

- AD-compatible managed directory on AWS
- cannot be joined with on-premise AD

### IAM Identity Center - Active Directory Setup

- **connect to an AWS Managed Microsoft AD (directory service)**
  - integration is out of the box
- **Connect to a Self-Managed Directory**
  - creat two-way trust relationship using AWS Managed Microsoft AD
  - create an AD Connector

## AWS Control Tower

- easy way to setup and govern a secure and compliant multi-account AWS environment based on best-practices
- AWS Control Tower uses AWS Organizations to create accounts
- Benefits
  - automate the set up of your environment in a few clicks
  - automate ongoing policy manngement using guardrails
  - detect policy violatioins and remediate them
  - monitor compliance through an interactive dashboard
- Guardrails
  - provides ongoing governance for your Control Tower environment (AWS Accounts)
  - Preventive Guardrail - using SCPs (e.g., restrict regions across all accounts)
  - Detective Guardrail - using AWS Config (e.g., identify untagged resources)
