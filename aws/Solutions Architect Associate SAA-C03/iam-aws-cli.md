# IAM & AWS CLI

## IAM - Identity and Access Management

- Global Service
- Root account created by default and shouldnt be used or shared
- **Users** are people in an organization and can be grouped
  - users can belong to multiple or no groups

### IAM Permissions

- Users or Groups can be assigned a JSON document call policies
  - defines a user/group permissions
- Least privilege principle - dont give more permissions than needed
- can create tags (key-value pairs) to organize, track, and apply perssions to users

### IAM Policies Inheritance

- policies applied to a group will apply to all members of that group
  - can inherit from multiple groups
- Inline policy is a policy only attached to a single users
- groups can only container users not other groups

### IAM Policy Structure

- version number - usually a date
- id - how to identify
- statement - one or more
  - Sid - statement identifier
  - effect - allow/deny access
  - principal - account/user/role the policy applies to
  - action - list of actions the policy allows/denies
  - resource - list of resources the actions apply to
  - condition (optional) - conditions for wqhen this policy is in effect

### IAM Password Policy

- strong password = higher security for you account
- can set min password length
- require specific character types
  - upppercase, lowercase, numbers, non-alphanumeric characters
- allow IAM users to change their own passwords
- password expiration
- prevent password re-use

### IAM Roles for Servies

- IAM roles are policies that can be applied to services
- common cases are EC2 and Lambda roles

### IAM Security Tools

- IAM Credentials Report
  - report that lists all your account's users and the status of their various credentials
- IAM Access Advisor
  - shows the service permissions granted to a user and when they were last accessed

### Best Practices

- dont use the root account except for AWS account setup
- one physical user = one AWS user
- **Asign users to groups** and assign permissions to groups
- create a strong password policy
- use and enforce use of MFA
- create and use roles for AWS service permissions
- use access keys for CLI/SDK
- Audit permissions of your account with IAM Credentials Report
- Never share IAM users & access keys

### Multi Factor Authentication - MFA

- password you know + security device you own
- Device Options
  - vitural MFA
    - Google Authenticator (phone only)
    - Authy (muti-device)
  - Universal 2nd Factor Security Key
    - YubiKey by Yubico (3rd party)
      - support for multiple root and IAM users with single security key
  - Hardware Key Fob MFA Device by Gemalto (3rd party)
  - Hardware Key Fob MFA Device for AWS GovCloud (US) by SurePassID (3rd party)

## AWS Access Methods

### AWS Management Console

- protected by password & MFA

### AWS Command Line Interface (CLI)

- protected by access keys
- enables interaction with AWS services using commands in your command-line shell
- direct accest to the public APIs of AWS services
- develop scripts to manage resources
- [open-source](https://github.com/aws/aws-cli)
- alternative to AWS Management Console
- built on AWS SDK for Python
- AWS Cloudshell
  - AWS cloud terminal to use AWS CLI from
  - default region is the region chosen in Management Console
  - can download/upload files
  - files persist

### AWS Software Developer Kit (SDK)

- protected by access keys
- language specific APIs (set of libaries)
- enables you to access and manages AWS services programmatically
- embedded in application
- Supports
  - SDK (JavaScript, Python, PHP, .NET, Ruby, Java, Go, Node.js, C++ , ...)
  - Mobile SDK (Android, iOS, ...)
  - IoT Device SDK (Embedded C, Arduino, ...)

### Access keys

- are generated in AWS Console
- Users manage their own keys
- secret - dont share
- access key id = usrname
- secret access key = password