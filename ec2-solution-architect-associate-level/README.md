<!--
 Copyright 2024 lesongvi

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 You may obtain a copy of the License at

     https://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
-->

# Private vs. Public vs. Elastic IP

## Fundamental Differences

- Public IP:
  - Public IP means the machine can be identified on the internet (WWW).
  - Must be unique across the whole web (not two machines can have the same public IP).
  - Can be geo-located easily.
- Private IP:
  - Private IP means the machine can only be identified on a private network only
  - The IP must be unique across the private network.
  - BUT two different private networks (two companies) can have the same IPs
  - Machines connect to WWW using an internet gateway (a proxy).
  - Only a specified range of IPs can be used as private IPs
- Elastic IP:
  - When you stop and then start an EC2 instance, it can change its public IP.
  - If you need to have a fixed public IP for your instance, you need to use an Elastic IP.
  - An Elastic IP is a public IPv4 address you own as long as you don't delete it
  - You can attach it to one instance at a time.
  - With an Elastic IP address, you can mask the failure of an instance or software by rapidly remapping the address to another instance in your account.
  - You can only have 5 Elastic IP in your account (by default, you can ask AWS to increase this limit).
  - Overall, **try to avoid using Elastic IP**:
    - They often reflect poor architectural decisions because they imply hard-coding public IP in your architecture.
    - Instead, use a random public IP and register a DNS name to it.
    - Or, as we'll see later, use a Load balancer and don't use a public IP

## Private vs. Public IP (IPv4) in AWS EC2 - Hands On

- By default, your EC2 machine comes with:
  - A private IP for the internal AWS Network
  - A public IP, for the WWW
- When we are doing SSH into our EC2 machines:
  - We can't use a private IP, because we are not in the same network
  - We can only use the public IP
- If your machine is stopped and then started, **the public IP can change**.

Counter increased by 1
On my way to achieve the Solution Architect Associate Level certification, I will be sharing my notes and summaries on this repository. I hope it will help you too. If you have any questions, please feel free to ask me. I will be happy to help you.

# EC2 Placement Groups

- Sometimes you want control over the EC2 instance placement strategy
- That strategy can be defined using placement groups
- When you create a placement group, you specify one of the following strategies for the group:
  - Cluster: clusters instances into a low-latency group in a single Availability Zone
  - Spread: spreads instances across underlying hardware (max 7 instances per group per AZ) - recommended for critical applications
  - Partition: spreads instances across many different partitions (which rely on different sets of racks with independent power and cooling) within an AZ. Scales to 100s of instances per group (recommended for large distributed and replicated workloads, such as Hadoop, Cassandra, etc.)

## EC2 Placement Groups - Cluster

![](/assets/placement-groups-cluster.png)

- Pros: Great network (10 Gbps bandwidth between instances with Enhanced Networking enabled - recommended)
- Cons: If the AZ fails, all instances fails at the same time
- Use case:
  - Big Data job that needs to complete fast
  - Application that needs extremely low network latency and high network throughput between instances

## EC2 Placement Groups - Spread

![](/assets/placement-groups-spread.png)

- Pros:
  - Can span across multiple AZ
  - Reduced risk is simultaneous failures
  - EC2 Instances are on different hardware
- Cons:
  - Limited to 7 instances per AZ per placement group
- Use case:
  - Application that needs to maximize high availability
  - Critical applications where each instance must be isolated from failure from each other

## EC2 Placement Groups - Partition

![](/assets/placement-groups-partition.png)

- Up to 7 partitions per AZ
- Can span across multiple AZs in the same region
- Up to 100s of EC2 instances
- The instances in a partition do not share racks with the instances in other partitions
- A partition failure can affect many EC2 instances but won't affect other partitions
- EC2 instances get access to the partition information as metadata
- Use case:
  - Large distributed and replicated workloads (Hadoop, Cassandra, etc.)

**Note: Each partition represents a rack with independent power and cooling**

# Elastic Network Interfaces (ENI)

![](/assets/elastic-network-interfaces-eni.png)

- Logical component in a VPC that represents a **virtual network card**
- The ENI can have the following attributes:
  - Primary private IPv4, one or more secondary IPv4
  - One Elastic IP (IPv4) per private IPv4
  - One public IPv4
  - One or more security groups
  - A MAC address
- You can create ENI independently and attach them on the fly (move them) on Ec2 instances for failover
- Bound to a specific AZ
