# SDK IAM Roles & Policies

- managed policies
  - polcies managed and updated overt time
- custom policies
  - can be made with a visual or JSON editor
  - can import a managed policy
  - can make more specific to your use cases
- inline policy
  - added to a role, but it can only be used by that role
- [Policy Generator](awspolicygen.s3.amazonaws.com/policygen.html)
  - used to make custom policies
- [Policy Simulator](policysim.aws.amazon.com)
  - used to test policies
    - choose service, action, and run simulkation with a given polcy to test if the correct permissions are granted
    - shows statements that match each action

## EC2 Instance Metadata

- powerful but least known to developers
- allows EC2 instances to "learn about themselves" without using an IAM Role for that purpose
- URL is <http://169.254.169.254/latest/meta-data/>
  - only accessible from an EC2 instance
- can retrieve the IAM Role name from the metadata, but you CANNOT retrieve the IAM Policy
- Metadata = info about the EC2 instance
- Userdata = launch script of the EC2 instance

## AWS SDK

- exam expects you know when yo use an SDK
- us-east-1 will be chosen as the region in the SDK if none is specified
- Languages
  - Java
  - .NET
  - Node.js
  - PHP
  - Python
  - GO
  - Ruby
  - C++
