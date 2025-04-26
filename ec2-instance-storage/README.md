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

# EC2 Instance Storage

## EBS Overview

### What's an EBS Volume?

- An **EBS (Elastic Block Store)** Volume is a **network** drive you can attach to your EC2 instance while they are running.
- It allows your instances to persist data, even after their termination.
- **They can only be mounted to one instance at a time** (at the CCP level). -> At SAA and DVA level, they can be mounted to multiple instances at the same time using multi-attach (we'll talk about this later).
- They are bound to **a specific availability zone**.

- Analogy: Think of them as a "network USB stick"
  ![USB stick](/assets/usb-stick.png)
- Free tier: 30 GB of free EBS storage of type General Purpose (SSD) or Magnetic (HDD) per month.

- It's a network drive (i.e. not a physical drive)

  - It uses the network to communicate the instance, which means there might be a bit of latency
  - It can be detached from an EC2 instance and attached to another one quickly

- It's locked to an Availability Zone (AZ)

  - An EBS volumn is us-east-1a cannot be attached to an instance in us-east-1b
  - To move a volume across, you first need to snapshot it, and then create a new volume in the other AZ from the snapshot

- Have a provisioned capacity (size in GBs, and IOPS)
  - You get billed for all the provisioned capacity, even if you don't use it

### EBS Volume - Example

![EBS Volume example](/assets/ebs-volume-example.png)

- Case 1: 1 EBS volume attached to 1 EC2 instance
- Case 2: 2 EBS volumes attached to 1 EC2 instance
- Case 3: If you want to attach EBS volumes to another AZ, you need to create a snapshot of the EBS volume and then create a new EBS volume from the snapshot in the other AZ. Then you can attach it to the EC2 instance in the other AZ.
- Case 4: You can just let the EBS volume unattached. It will still persist, but you still be billed for it.

### EBS - Delete on Termination attribute

![Delete on Termination attribute](/assets/delete-on-termination-attribute.png)

- Controls the EBS behavior when an EC2 instance is terminated
  - By default, the root EBS volume is deleted (attribute enabled)
  - By default, any other attached EBS volume is not deleted (attribute disabled)
- This can be controleld by the AWS console / AWS CLI
- **Use case: preserve root volume when instance is terminated**

## EBS Snapshots

- Make a backup (snapshot) of your EBS volume at a specific point in time
- Not necessary to detach the EBS volume from the EC2 instance, but it's recommended
- Can copy snapshots across AZ or Regions
  ![Snapshot example](/assets/snapshot-diagram.png)

### EBS Snapshots Features

![Snapshot features](/assets/snapshot-features.png)

## AMI Overview

- AMI = Amazon Machine Image
- AMI are a **customization** of an EC2 instance
  - You add your own software, configuration, operating system, monitoring, etc.
  - Faster boot / configuration time because all your software is pre-packaged
- AMI are built for a **specific region** (and can be copied across regions)
- You can launch EC2 instances from:
  - **A Public AMI**: AWS provided AMI
  - **Your own AMI**: You make and maintain them yourself
  - **An AWS Marketplace AMI**: An AMI someone else made (and potentially sells)

### AMI Process (from an EC2 instance)

![AMI process](/assets/ami-process.png)

- Start an EC2 instance and customize it
- Stop the EC2 instance (for data integrity)
- Build an AMI - this will also create a snapshot of the root EBS volume
- Launch instances from other AMIs

## EC2 Instance Store

- EBS volumes are **network drives** with good but "limited" performance
- **_If you need a high-performance hardware disk, use EC2 instance store_**

- Better I/O performance
- EC2 Instance Store lose their storage if they're stopped or terminated (ephemeral storage)
- Good for buffer / cache / scratch data / temporary content
- Risk of data loss if hardware fails
- Backups and Replication are your responsibility

### Local EC2 Instance Store

![EC2 instance store](/assets/ec2-instance-store.png)

## EBS Volume Types

- EBS Volumes come in 6 types

  - **gp2 / gp3**: General Purpose SSD volume that balances price and performance for a wide variety of workloads
  - **io1 / io2 Block Express (SSD)**: Highest-performance SSD volume for mission-critical low-latency or high-throughput workloads
  - **st1 (HDD\*)**: Low-cost HDD volume designed for frequently accessed, throughput-intensive workloads
  - **sc1 (HDD)**: Lowest-cost HDD volume designed for less frequently accessed workloads

- EBS Volumes are characterized in Size | Throughput | IOPS (Input/Output Operations Per Second)
- When in doubt, always consult the AWS documentation for the latest information on EBS volume types and their performance characteristics.
- **Only gp2/gp3 and io1/io2 Block Express can be used as boot volumes.**

### EBS Volume Type Use cases

#### General Purpose SSD

- Cost-effective storage, low-latency
- System boot volumes, virtual desktops, development and test environments
- 1 GiB - 16 TiB
- gp3:
  - Baseline of 3,000 IOPS and throughput of 125 MiB/s
  - Can increase IOPS up to 16,000 and throughput up to 1,000 MiB/s independently
- gp2:
  - Small gp2 volumes can burst IOPS to 3,000 IOPS
  - Size of the volume and IOPS are linked, max IOPS is 16,000
  - 3 IOPS per GB, means at 5,334 GB you get 16,000 IOPS

#### Provisioned IOPS (PIOPS) SSD

- Critical business applications with sustained IOPS performance
- Or applications that need more than 16,000 IOPS
- Great for **databases workloads** (sensitive to storage performance and consistency)
- io1 (4 GiB - 16 TiB):
  - Max PIOPS of 64,000 for Nitro EC2 instances & 32,000 for non-Nitro EC2 instances
  - Can increase PIOPS independently from storage size
- io2 (4 GiB - 64 TiB):
  - Sub-millisecond latency
  - Max PIOPS of 256,000 with an IOPS:GiB ratio of 1,000:1 (this means you can have 256,000 IOPS with a 256 GiB volume)
- Supports EBS Multi-Attach (attach to multiple instances at the same time)

#### Hard Disk Drive (HDD)

- Cannot be a boot volume
- 125 GiB to 16 TiB
- Throughput Optimized HDD (st1):
  - Big Data, Data Warehousing, Log Processing
  - **Max throughput** of 500 MiB/s - max IOPS of 500
- Cold HDD (sc1):
  - For data that is infrequently accessed
  - Scenarios where lowest cost is important
  - **Max throughput** of 250 MiB/s - max IOPS of 250

#### EBS - Volume Types Summary

![EBS volume types summary](/assets/ebs-volume-types-summary.png)

## EBS Multi-Attach

- Attach the same EBS volume to multiple EC2 instances at the same time within the same AZ
- Each instance has full read & write permissions to the high-performance EBS volume
- **Use case**:
  - Achieve **higher application availability** in clustered Linux applications (e.g. Oracle RAC, Teradata)
  - Applications must manage concurrent write operations
- **Up to 16 EC2 instances can be attached to the same EBS volume**
- Must use a file system that's cluster-aware (e.g. GFS2, OCFS2. Not XFS, EXT4, NTFS)
- **Only io1 and io2 Block Express volumes support Multi-Attach**

## EBS Encryption

- When you create an encrypted EBS volume, you get the following:
  - Data at rest encrypted inside the volume
  - All the data in flight between the volume and the instance is encrypted
  - All snapshots created from the volume are encrypted
  - All volumes created from the snapshot are encrypted
- Encryption and decryption are handled transparently by AWS (you have nothing to do)
- Encryption has a minimal impact on performance
- EBS Encryption leverages keys from KMS (Key Management Service, AES-256)
- Copying an unencrypted snapshot allows encryption of the snapshot
- Snapshots of encrypted volumes are encrypted

### Encryption: encrypt an unencrypted EBS volume

- Create a snapshot of the unencrypted volume
- Encrypt the snapshot (using copy snapshot)
- Create new EBS volume from the encrypted snapshot (The new volume will also be encrypted)
- Now you can attach the new encrypted volume to your original instance

## Amazon EFS

- Amazon EFS = Amazon Elastic File System
- Managed NFS (Network File System) that can be mounted to multiple EC2 instances at the same time
- EFS works with EC2 instances in multiple AZs
- Highly available, scalable, expensive (3x gp2), pay per use

![Amazon EFS](/assets/amazon-efs.png)

- **Use case**: content management, web serving, data sharing, Wordpress
- Uses NFSv4.1 protocol
- Uses security group to control access to EFS
- **Compatible with Linux based AMI (not Windows)**
- Encryption at rest using KMS

- POSIX file system (~Linux) that has a standard file API
- File system scales automatically, pay per use (for each GiB you use), no pre-provisioning!

### EFS - Performance & Storage Classes

- **EFS Scale**
  - 1000s of concurrent NFS clients, 10 GB+/s throughput
  - Grow to Petabyte-scale network file system, automatically
- **Performance Mode (set at EFS creation time)**
  - **General Purpose**: Latency-sensitive use cases (e.g. web serving, CMS, data sharing)
  - **Max I/O**: Higher latency, throughput, highly parallelized workloads (e.g. big data, media processing)
- **Throughput Mode**
  - **Bursting**: 1 TB = 50 MiB/s + burst of up to 100 MiB/s
  - **Provisioned**: Set your throughput regardless of storage size, ex: 1 GiB/s for 1 TB storage
  - **Elastic**: Automatically scales throughput up to down based on your workloads
    - Up to 3 GiB/s for reads and 1 GiB/s for writes
    - Used for unpredictable workloads

### EFS - Storage Classes

- **Storage Tiers (lifecycle management feature - move file after N days)**
  - **Standard**: For frequently accessed files
  - **Infrequent Access (IA)**: For infrequently accessed files (lower cost)
    - Files not accessed for 30 days are moved to IA
    - Files accessed again are moved back to Standard
  - **Archive**: rarely accessed files (few times a year), 50% cheaper than IA
    - Files not accessed for 90 days are moved to Archive
    - Files accessed again are moved back to Standard
  - Implement _lifecycle management policies_ to automatically move files between storage classes
- **Availability and durability**

  - Standard: Multi-AZ, great for prod
  - One Zone: One AZ, great for dev, backup enabled by default, compatible with IA (EFS One Zone-IA)

- Over 90% in cost savings
  ![Amazon EFS storage classes](/assets/amazon-efs-storage-classes.png)

## EFS vs. EBS

### EBS - Elastic Block Store

- EBS volumes...
  - One instance (except Multi-Attach io1/io2)
  - Are locked at the Availability Zone (AZ) level
  - gp2: IO increases if the disk size increases
  - gp3 & io1: can increase IOPS independently
- To migrate an EBS volume across AZs
  - Take a snapshot
  - Restore the snapshot to another AZ
  - EBS backups use IO and you shouldn't run them while your application is handling a lot of traffic
- Root EBS volumes are deleted by default when the instance is terminated (you can change this)

![EBS image](/assets/ebs-vs-efs.png)

### EFS - Elastic File System

- Mouting 100s of EC2 instances across multiple AZs
- EFS share website files (Wordpress, etc.)
- Only for Linux instances (POSIX)

- EFS has a higher price point than EBS
- Can leverage Storage Tiers for cost savings

- Remember: EFS vs. EBS vs. Instance Store
  - EBS: Block storage, one instance, persistent
  - EFS: File storage, multiple instances, persistent
  - Instance Store: Block storage, one instance, ephemeral

![Amazon EFS vs. EBS](/assets/amazon-efs-vs-ebs.png)
