## EBS

* It's a network drive
  * It uses network to communicate with the instance, which means there might be latency
  * It can be detached from an EC2 instance and attached to another one quickly
* It's locked to an Availability Zone (AZ)
  * An EBS Volume in us-east-1a cannot be attached to us-east-1b
  * To move a volume across, you first need to snapshot it
* Have a provisioned capacity (size in GBs, and IOPS)
  * You get billed for the provisioned capacity
  * You can increase the capacity over time
* Only GP2 and IO1 can be used as boot volumes

### GP2 volumes I/O burst

* If your gp2 volume is less than 1000 GiB (means IOPS less than 3000) it can "burst" to 3000 IOPS performance
* This is similar to t2 instances with their CPU
* You accumulate "burst credit over time", which allows your volume to have good performance when needed
* The bigger the volume the faster you fill up your "burst credit balance"
* **What happens if I empty my I/O credit balance?**
  * The maximum I/O you get becomes the baseline you paid for
  * If you see the balance at 0 all the time, consider increasing the GP2 volume or switch to io1, as it is under-performing
  * Use CloudWatch to monitor the I/O credit balance
* Burst concept also applies to st1 or sc1 (for increase in throughput)

### Computing MB/s based on IOPS

* **gp2:**
  * **Throughput in MiB/s = (Volume size in GiB) * (IOPS per GiB) * (I/O size in KiB)**
    * e.g.: 300 I/O operations per second * 256 KiB per I/O operation = 75 MiB/s
  * Limit to a max of 250 MiB/s (means volume >= 334 GiB won't increase throughput)
* **io1:** 
  * **Throughput in MiB/s = (Provisioned IOPS) * (I/O size in KiB)**
    * The throughput limit of io1 volumes is 256 KiB/s for each IOPS provisioned
    * Limit to a max of 500 MiB/s (at 32000 IOPS) and 1000 MiB/s (at 64 IOPS)

### EBS Volume Resizing

* You can only increase the EBS volumes:
  * Size (any volume type) 
  * IOPS (only in IO1)
* After resizing an EBS volume, you need to repartition your drive
* After increasing the size, it's possible fo rthe volume to be in a long time in the "optimisation" phase. The volume is still usable

### EBS Snapshots

* Incremental - only backup changed blocks
* EBS backups use IO and you shouldn't run them while your application is handling a lot of traffic
* Snapshots will be stored in S3 (but you won't directly see them)
* Not necessary to detach volume to do a snapshot, but it is recommended
* Max 100000 snapshots
* Can copy snapshots across AZ or Region
* Can make Image (AMI) from Snapshot
* EBS volumes restored by snapshots need to be pre-warmed (using fio or dd command to read the entire volume)
* Snapshots can be automated using Amazon Data Lifecycle Manager

### EBS Migration

* EBS volumes are locked to a specific AZ
* To migrate it to a different AZ (or region):
  * Snapshot the volume
  * (Optional) Copy the volume to a different region
  * Create a volume from the snapshot in the AZ of your choice

### EBS Encryption

* When you create an encrypted EBS volume, you get the following:
  * Data at rest is encrypted inside the volume
  * All the data in flight moving between the instance and the volume is encrypted
  * All snapshots are encrypted
  * All volumes created from the snapshot are encrypted
* Encryption and decryption are handled transparently (no manual interaction needed)
* Ebcryption has a minimal impact on latency
* EBS Encryption leverages keys from KMS (AES-256)
* Copying an unencrypted snapshot allows encryption
* Snapshots of encrypted volumes are encrypted

#### Encryption: encrypt an unencrypted EBS volume

* Create an EBS snapshot of the volume
* Encrypt the EBS snapshot (using copy)
* Create new ebs volume from snapshot (the volume will also be encryṕted)
* Now you can attach the encrypted volume to the original instance

### EBS VS Instance Store

* Some instances do not come with Root EBS volumes, but instead with "Instance Store" (ephemeral storage)
* Instance store is physically attached to the machine (EBS is a network drive)
* Pros:
  * Better I/O performance
  * Good for buffer / cache / scratch data / temporary content
  * Data survives reboots
* Cons:
  * On stop or termination, the instance store is lost
  * You can't resize the instance store
  * Backups must be operated by the user

#### Local EC2 Instance Store

* Physical disk attached to the physical server where your EC2 is
* Very high IOPS
* Disks up to 7.5 TiB (can change over time), stripped to reach 30 TiB (can change over time)
* Block Storage (like EBS)
* Cannot be increased in size
* Risk of data loss if hardware fails

### EBS for SysOps

* If you pan to use the root volume of an instance after it's termination, set the Delete On Termination to "No".
* If you use EBS for high performance, use EBS-optimized instance types
* If an EBS volume is unused, you still pay for it
* For cost saving over a long period, it can be cheaper to snapshot a volume and restore it later if unused

#### EBS Troubleshooting

* **High wait time or slow response for SSD => increase IOPS**
* EC2 won't start with EBS volume as root: make sure volume names are properly mapped (/dev/xvdb instead of /dev/xvda for example)
* After increasing a volume size, you still need to repartition to use the incremental storage (xfs_growfs for example)

### EBS RAID Options

* EBS is already redundant storage (replicated within an AZ)
* But what if you want to increase IOPS to say 100000 IOPS?
* What if you want to mirror your EBS volumes?
* You would mount volumes in parallel in RAID settings
* RAID is possible as long as your OS supports it
* Some RAID options are:
  * RAID 0
  * RAID 1
  * RAID 5 (not recommended for EBS - see documentation)
  * RAID 6 (not recommended for EBS - see documentation)

#### RAID 0 (increase performance)

* Combining 2 or more volumes and getting trhe total disk space and I/O
* But one disk fails, all the data is failed
* Use cases would be:
  * An application that needs a ot of IOPS and doesn't need fault-tolerance
  * A database that has replication already built-in
* Using this, we can have a very big disk with a lot of IOPS
* For example:
  * Two 500 GiB Amazon EBS io1 volumes with 4000 provisioned IOPS each will create a 1000 GiB RAID 0 array with an available bandwidth of 8000 IOPS and 1000 MB/s of throughput

#### RAID 1 (increase fault tolerance)

* RAID 1 = Mirroring a volume to another
* If one disk fails, our logical volume is still working
* We have to send the data to two EBS volumes at the same time (2x network)
* Use case:
  * Application that need increase volume fault tolerance
  * Applicaiton wher eyou need to service disks
* For example:
  * Two 5000 GiB Amazon EBS io1 volumes with 4000 provisioned IOPS each will create a 500 GiB RAID 1 array with an available bandwidth of 4000 IOPS and 500 MB/s of throughput

### CloudWatch and EBS

* Important EBS Volume CloudWatch metrics:
  * **VolumeIdleTime**: number of seconds when no read / write is submitted
  * **VolumeQueueLength:** number of operations waiting to be executed. High number means probably an IOPS or application issue
  * **BurstBalance:** if it becomes 0, may need a volume with more IOPS
* gp2 volume types: 5 minutes interval
* io1 volume types: 1 minute interval
* EBS Volumes have status check:
  * Ok - The volume is performing well
  * Impaired - Stalled, performance severely degraded
  * Insufficient-data - metric data collection in progress

## EFS - Elastic File System

* Managed NFS (network file system) that can be mounted on many EC2 at once
* EFS works with EC2 instances in multi-AZ
* Highly available, scalable, expensive (3x gp2), pay only for what is used
* Use cases:
  * Content management
  * Web serving
  * Data sharing 
  * Wordpress
* Uses NFSv4.1 protocol
* Uses security group to contorl access to EFS
* Compatible only with Linux based AMI
* Encryption at rest using KMS
* POSIX file system (~Linux) that has a standard file API
* File system scales automatically, pay-per-use, no capacity planning!

### EFS - Performance & Storage Classes

* **EFS Scale**
  * Thousands of concurrent NFS clients, 10 GB+ /s throughput
  * Grow to Petabyte-scale network file system, automatically
* **Performance mode (set at EFS creation time)**
  * General purpose (default): latency-sensitive use cases (web server, CMS, etc...)
  * Max I/O - higher latency, throughput, highly parallel (big data, media processing)
* **Storage Tiers (lifecycle management feature - move file after N days)**
  * Standard: for frequently accessed files
  * Infrequent access (EFS-IA): cost to retrieve files, lower price to store

