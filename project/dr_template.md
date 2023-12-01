# Infrastructure

## AWS Zones
Zone 1 = us-east-2
Zone 2 = us-west-1

## Servers and Clusters

### Table 1.1 Summary
| Asset            | Purpose                                                                                      | Size      | Qty | DR                                                                                                                                                                                                                        |
|------------------|----------------------------------------------------------------------------------------------|-----------|-----|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| VPC              | AWS local area network                                                                       | -         | 2   | Asset is deployed in two regions (one in us-east-2 and another one in us-west-1) for cross-region redundancy                                                                                                              |
| EKS              | Container orchestration (Kubernetes cluster)                                                 | -         | 2   | There is one EKS cluster (K8S control plane) per region to ensure region redundancy. Each control plane is configured to use all availability zones in the region (3 in us-east-2 and 2 in us-west-1) for AZ redundancy.  |
| EKS Worker Nodes | Virtual machines (EC2) configured in a nodegroup) used to host containers in the K8S cluster | t3.medium | 4   | There are two worker nodes per cluster for redundancy. Both worker nodes are deployed in different AZs.                                                                                                                   |
| EC2              | AWS virtual machine                                                                          | t3.micro  | 6   | There are 3 EC2 instances in each region. Each VM is deployed in a different AZ.                                                                                                                                          |
| ALB              | Application Load Balancer                                                                    | -         | 2   | One ALB in each region for region redundancy.                                                                                                                                                                             |
| RDS instances    | Instances that comprise database clusters                                                    | t2.small  | 4   | There are two DB clusters, one in each region. Replication is enabled on a primary-secondary mechanism. Each cluster has two DB instances in different AZs                                                                |


### Descriptions
VPC - Works as a Local Area Network in AWS and it represents the root of the networking infrastructure. Only after a VPC is created we can create subnets, gateways, security groups and other networking resources.
EKS Cluster - Kubernetes AWS managed service that enables the orchestration of containerized applications (Prometheus/Grafana stack in our case). It represents only the control plane of a Kubernetes cluster, the worker plane is managed separately.
EKS Worker Nodes - EC2 instances grouped in a nodegroup that host the containerized applications (Prometheus/Grafana). 
EC2 - Virtual machines (compute resources) that host applications.
ALB - Application Load Balancers are Layer7 Load Balancers that are used to distribute traffic accross multiple EC2s.
RDS instances - RDS is an AWS managed service to deploy databases. There can be one or more instances that can comprise a database cluster.


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