## CloudFormation For SysOps

### User Data in EC2 for CloudFormation

* We can have user data at EC2 instance launch through the console
* We can also include it in CloudFormation
* The important thing is to pass the entire script through the function Fn::Base64
* user data script log is in **/var/log/cfn-init-output.log**

### cfn-init

* AWS::CloudFormation::Init must be in the Metadata of a resource
* With the cfn-init script, it helps make complex EC2 configurations readable
* The EC2 instance will query the CloudFormation service to get init data
* Logs go to **/var/log/cfn-init.log**

### cfn-signal & wait conditions

* We still don't know how to tell CloudFormation that the EC2 instance got properly configured after **cfn-init**
* For this, use **cfn-signal** script
  * Run cfn-signal right after cfn-init
  * Tell CloudFormation service to keep on going or fail
* We need to define **WaitCondition**:
  * Block the template until it receives a signal from cfn-signal
  * We attach a **CreationPolicy** (also works on EC2, ASG)

#### Wait condition didn't receive the required number of signals from Amazon EC2 Instance

* Ensure that the AMI you're using has the AWS CloudFormation helper scripts installed. If the AMI doesn't include the helper scripts, you can also download them to your instance
* Verify that the cfn-init & cfn-signal command was successfully run on the instance. You can view logs, such as /var/log/cloud-init.log or /var/log/cfn-init.log, to help you debug the instance launch
* You can retrieve the logs by logging in to your instance, but you must **disable rollback on failure** or else AWS CloudFormation deletes the instance after your stack fails to create
* Verify that the instance has a connection to the Internet. If the instance is in a VPC, the instnace should be able to connect to the internet through a NAT device if it's a private subnet or through an Internet Gateway if it's in a public subnet
  * To check, run: curl -l https://aws.amazon.com

### CloudFormation Rollbacks

* Stack Creation Fails: 
  * Default: everything rolls back (gets deleted). We can look at the events log
  * Option to disable rollback and troubleshoot what happened
* Stack Update Fails:
  * The stack automatically rolls back to the previous known working state
  * Ability to see in the log what happened and error messages

### Nested stacks

* Nested stacks are stacks as part of the other stacks
* They allow you to isolate repeated patterns / common components in separate stacks and call them from other stacks
* E.g.:
  * Load Balancer configuration that is re-used
  * Security Group that is re-used
* Nested stacks are considered best practice
* To update a nested stack, always update the parent (root stack)

### ChangeSets

* When you update a stack, you need to know what changes before it happens, for greater confidence
* ChangeSets won't say if the update will be sucessful

### Retaining Data on Deletes

* You can put a DeletionPolicy on any resource to control what happens when the CloudFormation template is deleted
* **DeletionPolicy=Retain**
  * Specify on resources to preserve / backup in case of CloudFormation deletes
  * To keep a resource, specify **Retain** (works for any resource / nested stack)
* **DeletionPolicy=Snapshot**
  * EBS Volume, ElastiCache Cluster, ElastiCache Replication Group
  * RDS DBInstance, RDS DBCluster, Redshift Cluster
* **DeletionPolicy=Delete (default behavior):**
  * Note: for AWS::RDS::DBCluster resources, the default policy is **Snapshot**
  * Note: to delete an S3 bucket, you need to first empty the bucket of its content

### Termination Protection on Stacks

* To prevent accidental deletes of CloudFormation templates, use TerminationProtection

