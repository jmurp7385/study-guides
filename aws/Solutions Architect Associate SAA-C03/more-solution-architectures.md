# More Solution Architectures

## High Performance Computing (HPC)

### Enhanced Networking

- EC2 Enhanced Networking (SR-IOV)
  - Option 1: **Elastic Network Adapter (ENA)** - up to 100Gbs
  - Option 2: Intel 82599VF up to 10Gbps - legacy
- Elastic Fabric Adapter (EFA)
  - improved **ENA** for HPC, only works for Linux
  - great for inter-node communications, tightly coupled workloads
  - leverages Message Passing Interface (MPI) standard
  - bypassed the underlying Linux OS to provide low-latency, reliable transport

### Automation and Orchestration

- AWS Batch
  - **AWS Batch** supports multi-node parallel jobs, which enables you to run single jobs that span multiple EC2 instances
  - easily schedule jobs and launch EC2 instances accordingly
- AWS ParallelCluster
  - open-source cluster management tool to deploy HPC on AWS
  - configure with text files
  - automate creation of VPC, Subnet, Cluster Type, and instance type
  - **Ability to enable EFA on the cluster (improve network performance)**
