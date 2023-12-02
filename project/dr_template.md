**I have modified the Prometheus queries in the prometheus_queries.md file.
I have added the correct information related to AZs.
I have added public subnets, private subnets, internet gateway, nat gateway, NLB and security groups as IT assets.
Can you clarify what exactly in the IAC does not match the requirements? I have used multiple AZs, there are 3 Ubuntu Web VMs in each zone, 2 EKS worker nodes for each EKS cluster, multiple AZs for RDS, backup retention of 5 days. What have I missed?**

# Infrastructure

## AWS Zones
Zone 1
Region: us-east-2
AZs: us-east-2a, us-east-2b, us-east-2c
Zone 2
Region: us-west-1
AZs: us-west-1a, us-west-1b

## Servers and Clusters

### Table 1.1 Summary
| Asset              | Purpose                                                                                      | Size      | Qty | DR                                                                                                                                                                                                                        |
|--------------------|----------------------------------------------------------------------------------------------|-----------|-----|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| VPC                | AWS local area network                                                                       | -         | 2   | Asset is deployed in two regions (one in us-east-2 and another one in us-west-1) for cross-region redundancy                                                                                                              |
| EKS                | Container orchestration (Kubernetes cluster)                                                 | -         | 2   | There is one EKS cluster (K8S control plane) per region to ensure region redundancy. Each control plane is configured to use all availability zones in the region (3 in us-east-2 and 2 in us-west-1) for AZ redundancy.  |
| EKS Worker Nodes   | Virtual machines (EC2) configured in a nodegroup) used to host containers in the K8S cluster | t3.medium | 4   | There are two worker nodes per cluster for redundancy. Both worker nodes are deployed in different AZs.                                                                                                                   |
| EC2                | AWS virtual machine                                                                          | t3.micro  | 6   | There are 3 EC2 instances in each region. Each VM is deployed in a different AZ.                                                                                                                                          |
| ALB                | Application Load Balancer                                                                    | -         | 2   | One ALB in each region for region redundancy.                                                                                                                                                                             |
| RDS instances      | Instances that comprise database clusters                                                    | t2.small  | 4   | There are two DB clusters, one in each region. Replication is enabled on a primary-secondary mechanism. Each cluster has two DB instances in different AZs                                                                |
| Public Subnet      | Subnet where resources can be accessed from the Internet                                     | -         | 5   | There are 5 public subnets (1 for each AZ - 3 in us-east-2, 2 in us-west-1) that enables AZ redundancy                                                                                                                    |
| Private Subnet     | Subnet where resources can't be accessed from the Internet                                   | -         | 5   | There are 5 private subnets (1 for each AZ - 3 in us-east-2, 2 in us-west-1) that enables AZ redundancy                                                                                                                   |
| NAT Gateway        | NAT conversion for resources in private subnets                                              | -         | 5   | There are 5 NAT Gateways, 1 for each private subnet enabling AZ redundancy.                                                                                                                                               |
| Internet Gateway   | Access to internet                                                                           | -         | 2   | There are 2 Internet Gateways, 1 for each VPC in each region for cross-region redundancy.                                                                                                                                 |
| NLB                | Network Load Balancer to distribute traffic evenly to EKS nodes                              | -         | 2   | There are 2 NLBs, one in each region/VPC for each EKS cluster and Grafana deployment                                                                                                                                      |
| Security Group Web | Set of rules controlling traffic to/from Web app EC2s.                                       | -         | 2   | There are 2 SGs, one in each region.                                                                                                                                                                                      |
| Security Group EKS | Set of rules controlling traffic to/from Web app EC2s Worker Nodes.                          | -         | 2   | There are 2 SGs, one in each region.                                                                                                                                                                                      |


### Descriptions
VPC - Works as a Local Area Network in AWS and it represents the root of the networking infrastructure. Only after a VPC is created we can create subnets, gateways, security groups and other networking resources.
EKS Cluster - Kubernetes AWS managed service that enables the orchestration of containerized applications (Prometheus/Grafana stack in our case). It represents only the control plane of a Kubernetes cluster, the worker plane is managed separately.
EKS Worker Nodes - EC2 instances grouped in a nodegroup that host the containerized applications (Prometheus/Grafana). 
EC2 - Virtual machines (compute resources) that host applications.
ALB - Application Load Balancers are Layer7 Load Balancers that are used to distribute traffic accross multiple EC2s.
RDS instances - RDS is an AWS managed service to deploy databases. There can be one or more instances that can comprise a database cluster.
Public Subnets - A subnet in a Virtual Private Cloud (VPC) that is configured to route its traffic to the internet via an Internet Gateway (IGW). 
Private Subnets -  A subnet within a Virtual Private Cloud (VPC) that does not have a direct route to the internet. Resources in these subnets can't be accessed from the Internet.
NAT Gateway - A Network Address Translation (NAT) gateway is a managed service that allows instances in a private subnet to initiate outbound traffic to the internet while preventing inbound traffic from reaching those instances.
Internet Gateways - A component that allows communication between instances in the VPC and the internet. It essentially serves as a gateway that enables traffic to flow in and out of the VPC.
NLB - Network Load Balancers are Layer4 load balancers that are used to distribute traffic accross multiple EC2s.
Security Groups - A security group is a component of the networking infrastructure that acts as a virtual firewall for instances that controls inbound and outbound traffic. It acts as a set of rules that govern the traffic to and from instances (virtual machines) within a Virtual Private Cloud (VPC) network.


## DR Plan
### Pre-Steps:
1. Create VPC 
2. Create one subnet in each availability zone.
3. Create a launch template for the Web application.
4. Create an AutoScalingGroup that uses the Launch Template from the previous point. The autoscaling group has to be configured for all AZs.
5. Create an Application Load Balancer for the Web Application.
6. Create EKS cluster
7. Create Nodegroup
8. Deploy Prometheus/Grafana
9. Create an Application Load Balancer for Grafana.
10. Create and RDS cluster and configure the DB replication.



## Steps:
1. Shut down Ubuntu-Web instances in a region one by one and make sure the app is still reachable/accessible. 
2. Once all instances in a region have been stopped make sure that the app is reachable now through the ALB of the 2nd region.
3. Stop instances in the 2nd region until only one is running and make sure the app is still reachable.
4. Perform a DB instance failover.
5. Perform a DB cluster failover.
