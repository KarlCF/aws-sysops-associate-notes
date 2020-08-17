## EC2

### Placement Groups

* When creating a placement group, you specify one of the three strategies:
  * **Cluster** -  cluster instances into a low latency group in a single AZ
    * **Pros: Great network (10Gbs bandwidth between instances)**
    * **Cons: If the rack fails, all instances fail at the same time**
    * **Use case**:
      * Big Data job that needs to complete fast
      * Application that needs extremely low latency and high network throughput
  * **Spread** - spreads instances across underlying hardware (max 7 instances per group per AZ) - critical applications
    * **Pros**: 
      * Can span across AZs
      * Reduced risk of simultaneous failure
      * EC2 Instances on different physical hardware
    * **Cons**: 
      * Limited to 7 instances per AZ per placement group
    * **Use case**:
      * Application that needs to maximize high availability
      * Crtiical Applications where each instance must be isolated from failure from each other
  * **Partition** - spreads instances across many different partitions (which rely on different sets of racks) within an AZ. Scales to 100s of EC2 instances per group (Hadoop, Cassandra, Kafka)
    * Up to 7 partitions per AZ, up to a total of 100s of EC2 instances
    * The instances in a partition do not share racks with the instances in other patitions
    * A partition failure can effect many EC2 but won't affect other partitions
    * EC2 instances get access to the partition information as **metadata**
    * **Use cases**: HDFS, HBase, Cassandra, Kafka

### Shutdown Behavior

* How should the instance react when shutdown is done using the OS?
  * Stopped: default
  * Terminated
* This is not applicable from AWS console or AWS API. CLI Attribute: **InstanceInitiatedShutdownBehavior**

#### Termination Protection

* **Enable termination protection**: To protect against accidental termination in AWS Console or CLI
* **Exam tip:** Does not override shutdown behavior = terminate when we shutdown from OS

### EC2 Launch Trouhbleshooting

* **# InstanceLimitExceeded error**: Max number of instances per region reached
  * Resolution: Launch in a different Region or request AWS to increase the account limit of the Region
  * Default Limit of Instances per Region: 20
* **# InsufficientInstanceCapacity**: AWS does not have that much On-Demand capacity in the particular AZ to which the instance is launched
  * Resolution: Wait a few minutes before sending another request. Break down the request for new instances to 1 at a time.
  * If urgent, submit a request for a different instance type, and resize it later
* **# Instance Terminates Immediately (goes from pending to terminated)**:
  * Reached EBS volume limit
  * The EBS Snapshot is corrupt
  * The root EBS volume is encrypted and you do not have permissions to access the KMS key for decryption
  * The instance store-backed AMI that was used to launch the instance is missing a required part (image.part.xx file)
  * **To find the reason**: Check on the EC2 console, instances, Description tab, note the reason next to the State transition reason label

### EC2 SSH - Troubleshooting

* Make sure the private key (pem file) on your linux machine has 400 permissions, else you will get "Unprotected Private Key File"
* Make sure the username for the OS is given correctly when logging via SSH, else you will get "Host key not found" error
* **Possible reasons for 'connection timeout' to EC2 instances via SSH**:
  * SG is not configured correctly
  * CPU load of the instance is high

### EC2 Instance Launch Types

* **On Demand**: **Short workload**, predictable pricing
* **Reserved (MINIMUM 1 year):**
  * **Reserved Instances**: **long workloads**
  * **Convert Reserved Instances**: long workloads with flexible instances
  * **Scheduled Reserved Instances**: **recurring workloads** on specified time-range (e.g. every thuersday between 3 to 6 pm)
* **Spot Instances**: short workload, for cheap, can lose instances (less reliable)
* **Dedicated Instances**: no other customers will share your hardware
* **Dedicated Hosts**: book an entire physical server, control instance placement 

##### Good combination: Reserved Instances for baseline + On-Demand & Spot for peaks

#### EC2 on Demand

* Pay for what you use (billing per second, after the first minute)
* Has the **highest cost** but no upfront payment
* No long term commitment
* Reccomended for short-term and un-interrupted workloads, where you can't predict the application's behavior

#### Reserved

* **Up to 72% discount** compared to On-demand
* Pay upfront for what you use with long term commitment
* Reservation period can be 1 or 3 years
* Reserve a specific instance type
* Recommended for steady state usage applications (think database)
* **Convertible Reserved Instance**
  * Can change the EC2 instance type
  * Up to 54%
* **Scheduled Reserved Instances**
  * Launch within time window you reserve
  * When you require a fraction of day / week / month

#### EC2 Spot Instances

* Can get a discount of up to 90% compared to On-demand
* Instances that you can "lose" at any point of time if your max price is less than the current spot price
* The MOST cost-efficient instances in AWS
* **Useful for workloads that are resilient to failure**
  * Batch jobs
  * Data analytics
  * Image processing
  * ...
* **Not good for critical jobs or databases**

#### EC2 Dedicated Hosts

* Physical dedicated EC2 server for use
* Full contorl of EC2 instance placement
* Visibility into the underlying sockets / physical cores of the hardware
* Allocated for your account for a 3 year period reservation
* More expensive
* Useful for software that have complicated licensing model (BYOL - Bring Your Own License), and companies with strong regulatory or compliance needs

#### EC2 Dedicated Instances 

* Instances running on hardware that's dedicated to you
* May share hardware with other instances in same account
* No control over instance placement (can move hardware after Stop / Start)

### EC2 Spot Instance & Spot Fleet

#### EC2 Spot Instance Requests

* Cang get a discount of up to 90% compared to On-Demand
* Define **max spot price** and get the instance while current spot price < max
  * The hourly spot price varies based on offer and capacity
  * If the current spot price > your max price you can choose to stop or terminate your instance within a 2 minutes grace period
* Other strategy: **Spot Block**
  * "Block" spot instance during a specified time frame (1 a 6 hours) without interruptions
  * In rare situations, the instance may be reclaimed
* Used for batch jobs, data analysis, or workloads that are resilient to failures
* Not great for critical jobs or databases

#### How to terminate Spot Instances?

* To properly terminate spot instances, you must first cancel a spot request, and then terminate the associated Spot Instances
* You can only cancel Spot Instance requests that are **open, active or disabled**
* Cancelling a Spot Request does not terminate instances

#### Spot Fleets

* Spot Fleets = set of Spot Instances + (optional) On-Demand Instances
* The Spot Fleet will try to meet the target capacity with price constraints
  * Define possible launch pools: instance type (e.g.: m5.large), OS, Availability Zone
  * Can have multiple launch pools, so that the fleet can choose
  * Spot Fleet stopts launching instances when reaching capacity or max cost
* Strategies to allocate Spot Instances:
  * **lowestPrice**: from the pool with the lowest price (cost optimization, short workload)
  * **diversified**: distributed across all pools (great for availability, long workloads)
  * **capacityOptimized**: pool with the optimal capacity for the number of instances
* **Spot Fleets allows the user to automatically request Spot Instances with the lowest price**

### EC2 Instance Types 

* R: applications that needs a lot of RAM - in-memory caches
* C: applications that need good CPU - compute / databases
* M: applications that are balanced (medium) - general / web app
* I: applications that need good local I/O (instance storage) - databases
* G: applications that need a GPU - video rendering / machine learning

#### Burstable Instances (T2 / T3)


* T2 / T3: burstable instances (up to a capacity)
* T2 / T3 - unlimited: unlimited burst

* Burst means that overral, the instance has OK CPU performance, and when there is a need to process something unexpected (spike in load), it can burst and the CPU becomes very good. 
* Machine bursts utilizes burst credits, and after the credits are gone, the CPU becomes weak
* Credits start to accumulate after the instance stops bursting


* Check: https://ec2instances.info/

#### T2 / T3 Unlimited

* Possible to have "unlimited burst credit balance"
* After the credit balance is gone, you pay extra for the consumption
* Use carefully, costs can go high

### AMI 

#### Why use a custom AMI?

* Using a custom built AMI can provide the following advantages:
  * Pre-installed packages needed
  * Faster boot time (no need for ec2 user data at boot time)
  * Machine comes configured with monitoring / enterprise software
  * Security concerns - control over the machines in the network
  * Control of maintenance and updates of AMIs over time
  * Active Directory Integration out of the box
  * Installing your app ahead of time (for faster deploys when auto-scaling)
  * Using someone else's AMI that is optimised for running an app, DB, etc...
* AMI's are built for a specific AWS region

#### Using Public AMIs

* You can leverage AMIs from other people
* You can also pay for other people's AMI by the hour
  * These people have optmised the software
  * The Machine is easy to run and configure
  * You basically rent "expertise" from the AMI creator
* AMI can be found and published on the Amazon Marketplace
* **Some AMIs might come with malware or may not be secure for your enterprise, be careful**

#### AMI Storage

* AMI takes space, and is stored in Amazon S3
* Amazon S3 is a durable, cheap and resilient storage where most of your backups live (but you won't see them in the S3 console)
* By default, AMis are private and locked for your account / region
* It is possible to make AMIs public and share with other AWS accounts or sell them on the AMI Marketplace

#### AMI Pricing

* S3 storage pricing
* Make sure to remove the AMIs you don't use

### Cross Account AMI Copy

* You can sahre an AMI with another AWS account
* Sharing an AMI does not affect the ownership of the AMI
* If you copy an AMI that has been shared with your account, you are the owner of the target AMI in your account
* To copy an AMI that was shared with you from another account, the owner of the resource AMI must grant you read permissions for the storage that backs the AMI, either the associated EBS snapshot (for an Amazon EBS-backed AMI) or an associated S3 bucket (for an instance store-backed AMI)
* **Limits**:
  * You can't copy an encrypted AMI that was shared with you from another account. Instead, if the underlying snapshop and encryption key was shared with you, you can copy the snapshot while re-encrypting it with a key of your own. You own the copied snapshot, and can register it as a new AMI.
  * You can't copy an AMI with an associated **billingProduct** code that was shared with you from another account. This includes Windows AMIs and AMIs from the AWS Marketplace. To copy a shared AMI with a **billingProduct** code, launch an EC2 instance in your account using the shared AMMI and then create an AMI from the instance
* https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/CopyingAMIs.html

### Elastic IP

* Use cases:
  * Mask the failure of an instance or software by rapidly remapping the address
* The soft limit is 5 Elastic IP per account (you have to request an increase if there's need for more)
* Overall, try to avoid using Elastic IP, some alternatives are:
  * Possible to use a random Public IP and register a DNS name
  * Load Balancer with a static hostname

### CloudWatch Metrics for EC2

* **AWS Provided metrics (AWS pushes them)**:
  * Basic Monitoring (default): metrics are collected at a 5 minute interval
  * Detailed Monitoring (paid): metrics are collected at a 1 minute interval
  * Includes CPU, Network, Disk and Status Check Metrics

* **Custom metric (yours to push)**:
  * Basic Resolution: 1 minute resolution
  * High Resolution: all the way to 1 second resolution
  * Include RAM, application level metrics
  * Make sure the IAM permissions on the EC2 instance role are correct

