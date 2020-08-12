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

