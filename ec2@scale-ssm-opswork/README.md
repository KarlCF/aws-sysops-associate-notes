## AWS System Manager

### AWS System Manager Overview

* Helps you manage your EC2 and On-Premise systems at scale
* Get operational insights about the state of your infrastructure
* Easily detect problems
* Patching automation for enhanced compliance
* Works for both Windows and Linux OS
* Integrated with CloudWatch metrics / dashboards
* Integrated with AWS Config
* Free service 

#### AWS System Manager Features

* Resource Groups
* Insights:
  * Insights Dashboard
  * Inventory: discover and audit the software installed
  * Compliance
* Parameter Store
* Actions:
  * Automation (shut down EC2, create AMIs)
  * Run Command
  * Session Manager
  * Patch Manager
  * Maintenance Windows
  * State Manager: define and maintaining configuration of OS and applications

#### How Systems Manager Works

* We need to install the SSM agent onto the systems we control
* Installed by default on Amazon Linux AMI & some Ubuntu AMI
* If an instance can't be controlled with SSM, it's probably an issue with the SSM agent!

### AWS Tags

* You can add Text Key / Value Pairs called Tags to many AWS resources
* Commonly used in EC2
* Free naming, common tags are: Name, Environment, Team...
* They're used for:
  * Resource grouping
  * Automation
  * Cost Allocation
* Overall: Better to have too many tags than too few

### AWS System Manager - Resource Groups

* Create, view or manage logical group of resources thanks to **tags**
* Allows the creation of logical groups of resources such as:
  * Applications
  * Different layers of an application stack
  * Production versus development environment
* Regional service
* Works with EC2, S3, DynamoDB, Lambda, etc...

### SSM Documents

* Documents can be in JSON or YAML format
* You define parameters
* You define actions
* Many documents already exist in AWS

### AWS Systems Manager - Run Command

* Excute a document (= script) or just run a command
* Run command across multiple instances (using resource groups)
* Run Control / Error Control
* Integrated with IAM & CloudTrail
* No need for SSH
* Results in the console

### Using AWS System Manager to Patch

* Inventory => List software on an instance
* Inventory + Run Command => Patch Software
* Patch Manager + Maintenance Window => Patch OS
* Patch Manager => Gives you compliance
* State manager => Ensure instances are in a consistent state (compliance)

### AWS SYstems Manager - Session Manager

* Session manager allows you to start a secure shell on your VM
* Does not use SSH access and bastion hosts
* Only EC2 for now, but will be available On Premise soon
* Log actions done through secure shells to S3 and CloudWatch
* IAM permissions: access SSM + write to S3 + write to CluodWatch
* CloudTrail can intercept StartSession events
* AWS Secure Shell versus SSH:
  * No need to open the port 22 at all anymore (+ security)
  * No need fo bastion hosts
  * All commands are logged to S3 / CloudWatch (auditing)
  * Access to Secure Shell is done through User IAM, not SSH keys

### What if I lost my SSH key for EC2?

* (Traditional method) If the instance is EBS backed:
  * Stop the instance, detach the root volume
  * Attach the root volume to another instance as a data volume
  * Modify the **~/.ssh/authorized_keys** file with your new key
  * Move the volume back to the stopped instance
  * Start the instance and you can SSH into it again
* (New Method) If the instance is EBS:
  * Run the **AWSSupport-ResetAccess** automation document in SSM
* Instance Store backed EC2:
  * You can't stop the instance (otherwise data is lost) - **AWS recommends termination**
  * Stephane's tip: Use **Session Manager** access and edit the ~/.ssh/authorized_keys file directly

### AWS Parameter Store - Overview

* Secure storage for configuration and secrets
* Optional Seamless Encryption using KMS
* Serverless, scalable, durable, easy SDK, free
* Version tracking of configurations / secrets
* Configuration management using path & IAM
* Notifications with CloudWatch Events
* Integration with CloudFormation

#### AWS Parameter Store Hierarchy

* /my-department/
  * my-app/
    * dev/
      * db-url
      * db-password
    * prod/
      * db-url
      * db-password
  * other-app/
* /other-department/

### AWS Opsworks

* Chef & Puppet help you perform server configuration automatically, or repetitive actions
* They work great with EC2 & On Premise VM
* AWS Opsworks = Managed Chef & Puppet
* It's an alternative to AWS SSM

#### Quick word on Chef / Puppet

* & Help with managing configuration as code
* Helps in having consistent deployments
* Works with Linux / Windows
* Can automate: user accounts, cron, ntp, packages, services...
* They leverage "Recipes" or "Manifests"
* Chef / Puppet have similarities with SSM / Beanstalk / CloudFormation but they're open source tools that work cross-cloud

