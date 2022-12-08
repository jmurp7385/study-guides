# Containers on AWS

## Docker

- Docker is a software development platform to deploy apps
- apps are packaged in **containers** that can be run by any OS
- Apps runs the same regardless of where they are run
  - any machines
  - no compatibility issues
  - predictable behavior
  - less work
  - easier to maintain and deploy
  - works with any language, any OS, any technology
- Use case: microservices architecture, lift-and-shift apps from on-premise to the AWS Cloud,...
- where are docker images stored?
  - Docker images are stored in Docker Repositories
  - [Docker Hub](https://hub.docker.com)
    - public repository
    - find base images for many technologies or OS (e.g., Ubuntu, MySQL, ...)
  - Amazon ECR (Amazon Elastic Container Registry)
    - Private repository
    - [Public repository (Amazon ECR Public Gallery)](https://gallery.ecr.aws)
- Docker vs Virtual Machine
  - Docker is "sort of" a virtualization technology, but not exactly
  - resources are shared with the host -> many containers on one server
- Getting Started
  - Dockerfile
  - Build an Image
  - Push Image to repository
  - Pull an Image
  - Run an image
- Docker Container Management on AWS
  - Amazon Elastic Container Service (Amazon ECS)
    - Amazon's own container platform
  - Amazon Elastic Kubernetes Service (Amazon EKS)
    - Amazon's managed Kubernetes (open source)
  - AWS Fargate
    - Amazon's own Serverless container platform
    - works with ECS and EKS
  - Amazon ECR
    - Store Container Images

## ECS

- ECS = Elastic Container Service
- EC2 Launch Type: you must provision & maintain the infrastructure (the EC2 instances)
  - launch Docker container on AWS = Launch ECS Tasks on ECS Clusters
  - Each EC2 Instance must run the ECS Agent to register in the ECS Cluster
  - AWS takes care of starting/stopping containers
- Fargate Launch Type
  - launch docker containers on AWS
  - **you do not provision the infrastructure (no EC2 instances to manage)**
  - **it is all Serverless**
  - just create task definitions
  - AWS just runs ECS Tasks for you based on the CPU/RAM you need
  - to scale, just increase the number of tasks, simple - no more EC2 instances
- IAM Roles for ECS
  - EC2 Instance Profile (EC2 Launch Type only):
    - use by the ECS Agent
    - Makes API calls to ECS service
    - send container logs to CloudWatch Logs
    - pull Docker image from ECR
    - Reference sensitive data in Secrets Manager or SSM Parameter Store
  - ECS Task Role
    - Allows each task to have a specific role
    - use different roles for the diferenrt ECS services you run
    - Task Role is defined in the **Task Definition**
- Load Balancer Integrations
  - *Application Load Balancer* supported and worksk for most use cases
  - *Network Load Balancer* recommended only for high throughput/perfomance use cases, or to pair it with AWS Private Link
  - **Elastic Load Balancer** supported but not recommended (no advanced features, no Fargate)
- Data Volumes (EFS)
  - mount EFS file systems onto ECS tasks
  - works for both **EC2** and **Fargate** launch types
  - tasks running in any AZ will share the same data in the EFS file system
  - **Fargate + EFS = serverless**
  - use cases: peristent multi-AZ shared storage for your containers
  - **note: Amazon S3 cannot be mounted as a file system**

### ECS Auto Scaling

- automatically increase/decrease the desired number of ECS tasks
- Amazon ECS Auto Scaling uses **AWS Application Auto Scaling**
  - ECS Service Average CPU Utilization
  - ECS Service Average Memory Utilization - Scale on RAM
  - ALB Request Count Per Target - metric coming from ALB
- **Target Tracking** - scale based on the target value for a specific CloudWatch metric
- **Step Scaling** - scaled based on specific CloudWatch Alarm
- **Scheduled Scaling** - scale based on a specified date/time (predictable changes)
- ECS Service Auto Scaling (task level) != EC2 Auto Scaling (EC2 isntance level)
- Fargate Auto Scaling is must easier to setup (becuase **Serverless**)
- Auto Scaling EC2 Instances
  - accommodate ECS Services Scaling by adding underlying EC2 Instances
  - Auto Scaling Group Scaling
    - scale your ASG based on CPU Utilization
    - Add EC2 instances over time
  - **ECS Cluster Capacity Provider (preferred)**
    - used to automatically provision and scale the infrastrucure for your ECS Tasks
    - Capacity Provider paired with an Auto Scaling Group
    - add EC2 instances when you're missing capacity (CPU, RAM, ...)

### ECS Solutions Architecture

- ECS Tasks invoked by Event Bridge
  - upload object to S3
  - S3 event to Event Bridge
  - Run ECS task based on event from Event Bridge
  - AWS Fargate runs ECS task with ECS Task role access to S3 and DynamoDB
  - Save result in DynamoDB
- ECS Tasks invoked by Event Bridge Schedule
  - Event Bridge triggers task every 1 hour
  - Run ECS task based on event from Event Bridge
  - AWS Fargate runs ECS task with ECS Task role access to S3
  - batch process and save results in S3
- SQS Queue Example
  - SQS Queue
  - ECS Service Auto Scaling tat poll for messages and scale based on queue

## ECR

- ECR = Elastic Container Registry
- store and manage Docker images on AWS
- **Private** and **Public** repository ([Amazon ECR Public Gallery](https://gallery.ecr.aws))
- fully integrated with ECS backed by Amazon S3
- Access is controlled through IAM (permission errors => policy)
- Supports image vulnerability scanning, versioning, image tags, image lifecycle

## EKS

- Amazon EKS = Amazon Elastic **Kubernetes** Service
- it is a way to launch **managed Kubernetes clusters on AWS**
- Kubernetes is an open-source system for automatic deployment, scaling, and management of containerized (usually Docker) applications
- it is an alternative to ECS. similar goal but different API
- EKS supports EC2 if you want to deploy worker nodes or Fargate to deploy serverless containers
- Use case: if you company is already using Kubernetes on-premise or in another cloud, aws want to migrate to AWS using Kubernetes
- **Kubernetes is cloud-agnostic** (Can be used in any cloud - Azure, GCP, ...)
- Node Types
  - Managed Node Groups
    - create and manage Nodes (EC2 instances) for you
    - nodes are part of an ASG managed by EKS
    - supports On-Demand or Spot Instances
  - Self-Managed Nodes
    - nodes created by you and registered to the EKS cluster and managed by and ASG
    - you can use prebuilt AMI - Amazon EKS Optimized AMI
    - supports On-Demand or Spot Instances
  - AWS Fargate
    - no maintainance required, no nodes managed
- Data Volumes
  - need to specify StorageClass manifest in EKS Cluster
  - leverages a **Container Storage Interface (CSI)** compliant driver
  - support for: EBS, EFS (with Fargate), FSx for Lustre, FSx for NetApp ONTAP

## App Runner

- fully managed service that makes it easy to deploy web applications and APIs at scale
- no infratstructure experience required
- start with source code or container image
- automatically builds and deploy the web app
- automatic scaling, highly available, load balancer, encryption
- VPC access support
- connect to database, cache, and message queue services
- Use Case: web apps, APIs, microservices, rapid production deployments
