# Elastic Block Storage

An EC2 loses its root volume when it is manually terminated. Unexpected terminations happen from time to time.

Attached volumes are EBS volumes, which is a network drive that you can attach to your instance. 

Uses the network to communicate with the instance, it's not a physical drive.
- There can be latency as time is taken to communicate.
- Can be detached and another volume attached quickly

EBS volumes are LOCKED to Availability Zones
- An EBS volume in eu-west-2a cannot be used in eu-west-2b
- To move a volume, take a snapshot of it

EBS volumes have provisioned capacity (GBs and IOPs)
- You are billed for all provisioned capacity.
- You can increase capacity over time.


EBS volumes are characterised in:
- Size
- Throughput
- IOPs (Input/Output Operations per Second)

EBS Volume Types
- GP2 (SSD), balanced performance and cost. Can be used as a root volume.
- IO1 (SSD), high performance, mission critical, low latency or high-throughput loads. Can be used as a root volume.
- STI (HDD), Low cost HDD designed for frequent access, throughput intensive workloads
- SC1 (HDD), Lowest cost HDD designed for less frequently accessed workloads



## GP2 - General Purpose, Cheap

Recommended for most workloads, least expensive for an SSD. Can be used for System boot volumes, virtual desktops, low-latency interactive apps.

Size Range 1GB - 16 GB
Small GP2 volumes can burst IOPs to 3000
Max IOPs is 16000
You get a baseline of 3 IOPs per GB (Maximum ratio of provisioned IOPs to requested volume is 3:1)
Throughput cannot be set for GP2

## IO1 - Provisioned IOPs, Expensive 

Business critical applications that require sustained IOPs performance, or more than 16k IOPs per volume
Large database workloads such as MongoDB, MicrosoftSQL, MySQL etc

Size Range 4GB - 16GB
IOPs is provisioned (PIOPS), Min 100 - Max 64,000 (Nitro instances, not in scope of this course) otherwise it's max 32,000.
Maximum ratio of provisioned IOPs to requested volume is 50:1. You must scale storage if you want to increase IOPS
Size of volume and IOPs are independent.

## ST1 - Throughput Optimised HDD

Streaming workloads that require consistent, fast throughput at a low price
Big data, data warehouse, log processing, Apache Kafka
Cannot be a boot volume

Size Range 500GB - 16TB
Max IOPs is 500
Max throughput of 500MB/s - this can burst as well
Throughput increases as size increases


## SC1, Cold HDD - Infrequently Accessed

Throughput oriented storage for large volumes of data that is infrequently accessed.
Scenarios where the lowest storage cost is important
Cannot be a boot volume

Size Range 500GB - 16TB
Max IOPs is 250
Max throughput is 250MB/s - this can burst as well

## EBS vs Instance Store

Some instances do not come with Root EBS volumes. Instead, they come with 'instance store' - this is *ephemeral* storage. Instance stores are physically attached to the machine, where EBS is a network drive.

Pros:
- Better I/O performance
- Good for buffer / cache / scratch data / temporary content
- Data survives reboot

Cons:
- On 'stop', or 'termination', instance store data is lost
- You can't resize the instance store
- Backups must be operated by the user

REMEMBER: The Instance Store is a *physical* disk attached to the instance. Very high IOPs because it's **physically** connected.

High IOPS = Local Instance Store.

Disks up to 7.5 TB, can be stripped (?) to reach 30 TB
Block storage, just like EBS, but PHYSICAL.

Risk of data loss if hardware fails (e.g. 'stop' instance). Ensure you replicate data storage across a number of block instances to ensure data redundancy.

# Elastic File System

Managed NFS (network file system) that can be mounted on many EC2 across many AZ, therefore EFS is not locked to an AZ.
Highly available, scalable, pay per use (approx 3x GP2), expensive.

Use cases: content management, web serving, data sharing, Wordpress
Uses NFSv4.1 Protocol
Uses security groups to control access to EFS
ONLY compatible with Linux Based AMIs (not Windows)
Encryption at rest using KMS

POSIX file system (Linux) that has a standard file API
File system scales automatically, only pay for what you use. If you are using 5MB of storage, you will be charged for 5MB.
No capacity planning required.

## EFS Performance and Storage Classes - Exam Tip

EFS Scale
- 1000s of concurrent NFS clients, 10GB + /s throughput
- Grow to Petabyte-scale network file system, automatically

Performance Mode (set at EFS creation time)
- General purpose (default), latency-sensitive use cases (web server, CMS)
- Higher processing power required - max I/O - higher latency, throughput, highly parallel (big data, media processing)

Storage Tiers (lifecycle management feature, moves files between tiers after N days)
- Standard: frequently accessed files
- Infrequent Access: **(EFS-IA)**, this can come up in the exam. Costs to retrieve files, lower price to store.

Lifecycle management feature allows the movement of files between storage tiers, i.e. if a file is not accessed for X days, move it to 'infrequent access' to reduce storage costs

You can mount EFS across a number of AZs. For each AZ you should assign a security group

# EBS vs EFS

## EBS Volumes

- Can be attached to a single instance at a time
- are locked to a single availability zone
- gp2: IO increases as disk size increase
- io1: IO can be increased independently of volume size
- Pay for what you provision (100gb volume = you pay for 100gb storage)

To migrate an EBS volume
- Take a snapshot
- Restore the snapshot in to a volume in another AZ
- EBS backups use IO and you shouldn't run whilst your application is handling a lot of traffic

Root EBS volumes are terminated by default if the EC2 instance is terminated

## EFS

- Mounting 100s or 1000s of instances across AZ
- Can only be ran on Linux (POSIX) instances
- Share website files such as WordPress
- More expensive than EBS (~ 3x more expensive)
- Can use Infrequent Access storage tier to save costs, you will be billed for retrieval.
- Pay for what you use (10gb of data = you pay for 10gb storage)




