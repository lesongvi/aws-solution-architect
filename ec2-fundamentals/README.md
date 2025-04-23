<!--
 Copyright 2025 lesongvi

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

# EC2 Fundamentals

## EC2 Basics

### Amazon EC2

- EC2 is one of the most popular of AWS' offerings
- EC2 = Elastic Compute Cloud = Infrastructure as a Service (IaaS)
- It mainly consists in the capability of:
  - Renting virtual machines (EC2)
  - Storing data on virtual drives (EBS)
  - Distributing load across machines (ELB)
  - Scaling the services using an auto-scaling group (ASG)
- Knowing EC2 is fundamental to understand how the Cloud works

### EC2 sizing & configuration options

- Operating System (OS): Linux, Windows or MacOS
- How much compute power & cores (CPU)
- How much random-access memory (RAM)
- How much storage space:
  - Network-attached (EBS & EFS)
  - Hardware (EC2 instance store)
- Network card: speed of the card, Public IP address
- Firewall rules: Security Groups
- Bootstrap script (configure at first launch): EC2 User Data

### EC2 User Data

- It is possible to bootstrap our instances using an **EC2 User data** script
- **bootstrapping** means launching commands when a machine starts
- That script is **only run once** at the instance **first start**
- EC2 user data is used to automate boot tasks such as:
  - Installing updates
  - Installing software
  - Download common files from the internet
  - Anything you can think of
- The EC2 User Data Script runs with the root user

### EC2 instance types - Example

![](/assets/ec2_instance_types-example.png)

- _t2.micro_ is part of the AWS Free Tier (up to 750 hours per month)

## EC2 Instance types Basics

### EC2 Instance types - Overview

- You can use different types of EC2 instances that are optimised for different use cases ([https://aws.amazon.com/ec2/instance-types/](https://aws.amazon.com/ec2/instance-types/))

![](/assets/ec2_instance_type-overview.png)

- AWS has the following naming convention:
  - m5.2xlarge
    - m = instance class
    - 5 = generation (AWS improves them over time)
    - 2 = size within the instance class

### EC2 Instance types - General purpose

- Great for a diversity of workloads such as web servers or code repositories
- Balance between:
  - Compute
  - Memory
  - Networking

![](/assets/ec2_instance_type-general-purpose.png)

### EC2 Instance types - Compute Optimized

- Great for compute-bound applications that benefit from high-performance processors:
  - Batch processing workloads
  - Media transcoding
  - High-performance web servers
  - High-performance computing (HPC)
  - Scientific modeling & machine learning
  - Dedicated gaming servers

![](/assets/ec2_instance_type-compute-optimized.png)

### EC2 Instance types - Memory Optimized

- Fast performance for workloads that process large data sets in memory
- Use cases:
  - High-performance, relational/non-relational databases
  - Distributed web scale cache stores
  - In-memory databases optimized for BI (business intelligence) applications
  - Applications performing real-time processing of big unstructured data

![](/assets/ec2_instance_type-memory-optimized.png)

### EC2 Instance types - Storage Optimized

- Great for storage-intensive tasks that require high, sequential read and write access to very large data sets on local storage
- Use cases:
  - High-frequency online transaction processing (OLTP) systems
  - Relational & NoSQL databases
  - Cache for in-memory databases (for example, Redis)
  - Data warehousing applications
  - Distributed file systems

![](/assets/ec2_instance_type-storage-optimized.png)

### EC2Instance.info

- [https://www.ec2instance.info/](https://www.ec2instance.info/)
- A website that provides a comprehensive overview of all EC2 instance types

## Security Groups

### Introduction to Security Groups

![](/assets/security_groups-introduction.png)

### Security Groups - Deeper dive

![](/assets/security_groups-deeper_dive.png)

### Security Groups - Diagram

![](/assets/security_groups-diagram.png)

### Security Groups - Good to know

- Can be attached to multiple instances
- Locked down to a region/VPC combination
- Does live "outside" the EC2 - if traffic is blocked the EC2 instance won't see it
- _It's good to maintain one separate security group for SSH access_
- If your application is not accessible (timeout), then it's a security group issue
- If your application gives a "connection refused" error, then it's an application error or it's not launched
- All inbound traffic is **blocked** by default
- All outbound traffic is **authorised** by default

### Referencing other security groups diagram

![](/assets/security_groups-referencing_other_security_groups.png)

### Classic ports to know

- 22 = SSH (Secure Shell) - Log into a Linux instance
- 21 = FTP (File Transfer Protocol) - Transfer files into a file share
- 22 = SFTP (Secure File Transfer Protocol) - Transfer files using SSH
- 80 = HTTP (Hypertext Transfer Protocol) - Access unsecured websites
- 443 = HTTPS (Hypertext Transfer Protocol Secure) - Access secured websites
- 3389 = RDP (Remote Desktop Protocol) - Log into a Windows instance

## SSH Overview

### SSH Summary Table

![SSH Summary Table](/assets/ssh-summary_table.png)

## EC2 Instances Purchasing Options

- **On-Demand Instances** - short workload, predictable pricing, pay by second
- **Reserved Instances** (1 & 3 years)
  - **Reserved Instances** - long workloads
  - **Convertible Reserved Instances** - long workloads with flexible instance types
- **Saving Plans** (1 & 3 years) - commitment to an amount of usage, long workloads
- **Spot Instances** - short workloads, cheap, can lose instances (less reliable)
- **Dedicated Hosts** - book an entire physical server, control instance placement
- **Dedicated Instances** - no other customers will share your hardware
- **Capacity Reservations** - reserve capacity in a specific Availability Zone for any duration

### EC2 On-Demand Instances

- Pay for what you use
  - Linux or Windows - billing per second, after the first minute
  - All other operating systems - billing per hour
- Has the highest cost but no upfront payment
- No long-term commitment
- Recommended for **short-term** and **un-interrupted workloads**, where you can't predict how the application will behave.

### EC2 Reserved Instances

- Up to **72%** discount compared to On-Demand pricing
- You reserve a specific instance attributes (Instance Type, Region, Tenancy, OS)
- **Reservation Period** - 1 year (+discount) or 3 years (+++discount)
- **Payment Options** - No Upfront (+), Partial Upfront (++), All Upfront (+++)
- **Reserved Instance's Scope** - Regional or Zonal (reserve capacity in a specific AZ)
- Recommended for steady-state usage applications (think database)
- You can buy and sell in the Reserved Instance Marketplace

- **Convertible Reserved Instances**
  - Can change the EC2 instance type, instance family, OS, scope and tenancy
  - Up to **66%** discount compared to On-Demand pricing

### EC2 Saving Plans

- Get a discount based on long-term usage (up to 72% - same as Reserved Instances)
- Commit to a certain type of usage ($10/hour for 1 or 3 years)
- Usage beyond EC2 Saving Plans is billed at the On-Demand price

- Locked to a specific instance family & AWS region (e.g. M5 in us-east-1)
- Flexible across:
  - Instance size (e.g. m5.xlarge to m5.2xlarge)
  - OS (e.g. Linux to Windows)
  - Tenancy (Host, Dedicated, Shared)

### EC2 Spot Instances

- Can get a **discount of up to 90%** compared to On-Demand pricing
- Instances that you can "lose" at any point of time if your max price is less than the current spot price
- The **MOST cost-effective** instances in AWS

- **Useful for workloads that are resilient to failure**
  - Batch jobs
  - Data analysis
  - Image processing
  - Any **distributed** workloads
  - Workloads with a flexible start and end time
- **Not suitable for critical jobs or databases**

### EC2 Dedicated Hosts

- A physical server with EC2 instance capacity fully dedicated to your use
- Allows you address **compliance requirements** and **use your existing server-bound software licenses** (per-socket, per-core, or per-VM software licenses)
- Purchasing Options:
  - **On-Demand** - pay per second for active Dedicated Host
  - **Reservation** - reserve a Dedicated Host for a 1 or 3 year term (No Upfront, Partial Upfront, All Upfront)
- The most expensive option

- Useful for software that have complicated licensing model (BYOL - Bring Your Own License)
- Or for companies that have strong regulatory or compliance requirements

### EC2 Dedicated Instances

![EC2 Dedicated Instances](/assets/ec2-dedicated_instances.png)

### EC2 Capacity Reservations

- Reserve **On-Demand** instances capacity in a specific AZ for any duration
- You always have access to EC2 capacity when you need it
- **No time commitment** (create/cancel anytime), **no billing discount**
- Combine with Regional Reserved Instances and Savings Plans to benefit from billing discounts
- You're charged at On-Demand rate whether you run instances or not

- Suitable for short-term, uninterrupted workloads that needs to be in a specific AZ

## Which purchasing option is right for me?

![Which purchasing option is right for me?](/assets/which_purchasing_option_is_right_for_me.png)

## Price Comparison - Example - m4.large - us-east-1

![Price Comparison - Example - m4.large - us-east-1](/assets/price_comparison_example_m4_large_us_east_1.png)

## EC2 Spot Instance Request

- Can get a discount of up to 90% compared to On-Demand pricing

- Define **max spot price** and get the instance while **current spot price** is lower than your max price
  - The hourly spot price varies based on supply and demand
  - If the current spot price > your max price, you can choose to **stop** or **terminate** the instance with a 2 minutes grace period
- Spot Block - deprecated, not available for new customers

- **Used for batch jobs, data analysis, or workloads that are resilient to failure**
- **Not great for critical jobs or databases**

### Spot Requests

#### EC2 Spot Request Pricing

![](/assets/ec2-spot_instances_pricing.png)

#### How to terminate Spot Requests

![How to terminate Spot Requests](/assets/how_to_terminate_spot_requests.png)

### Spot Fleets

![Spot Fleets](/assets/spot_fleets.png)
