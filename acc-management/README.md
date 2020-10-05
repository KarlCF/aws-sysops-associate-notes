## AWS Account Management

### AWS Status - Service Health Dashboards

* Shows all regions, all services health
* Shows historical information each day
* Has an RSS feed you can subscribe to
* https://status.aws.amazon.com/

### AWS Personal Health Dashboard

* Global service
* Show how AWS outages directly impact you
* Shows impact on your resources
* List issues and actions you can do to remediate them

### AWS Organizations

* Global service
* Allows to manage multiple AWS accounts
* The main account is the master account - you can't change it
* Other accounts are member accounts
* Member accounts can only be part of one organization
* Consolidated Billing across all accounts - single payment method
* Pricing benefits from aggregated usage (volume discount for EC2, S3...)
* API is available to automate AWS account creation

#### Multi Account Strategies

* Create accounts per **department**, per **cost center**, per dev / test / prod, based on regulatory restrictions (using SCP), for bettter resource isolation (ex: VPC), to have separate per-account service limits, isolated account for logging
* Multi Account vs One Account Multi VPC
* Use tagging standards for billing purposes
* Enable CloudTrail on all accounts, send logs to central S3 account
* Send CloudWatch Logs to central logging account
* Establish Cross Account Roles for Admin purposes

#### Service Control Policies (SCP)

* Whitelist or blacklist IAM actions
* Applied at the OU or Account Level
* Does not apply to the Master Account
* SCP is applied to all the **Users and Roles** of the Account, including Root
* The SCP does not affect service-linked roles
  * Service-linked roles enable other AWS services to integrate with AWS Organizations and can't be restricted by SCPs
* SCP must have an explicit Allow (does not allow anything by default)
* Use cases:
  * Restrict access to certain services
  * Enforce PCI compliance by explicitly disabling services

#### AWS Organization - Moving Accounts

* To migrate accounts from one organization to another
  1. Remove the member account from the Oold organization
  2. Send an invite to the new organization
  3. Accept the invite to the new organization from the member account
* If you want the master account of the old organization to also join the new organization, do the following:
  1. Remove the member accounts from the organizations using procedure above
  2. Delete the old organization
  3. Repeat the process above to invite the old master account to the new org

### AWS Service Catalog

* Create and manage catalogs of IT services that are approved on AWS
* The "products" are CloudFormation templates
* Ex: Virtual machine images, Servers, Software, Databases, Regions, IP address ranges
* CloudFormation helps ensure consistency, and standardization by Admins
* They are assgined to Portfolios (teams)
* Teams are presented a self-service portal where they can launch the products
* All the deployed products are managed deployed services
* Helps with gfovernance, compliance and consistency
* Can give user access to launching products without requiring deep AWS knowledge
* Integrations with "self-service portals" such as ServiceNow

### AWS Billing Alarms

* Billing data metric is stored in CloudWatch us-east-1
* Billing data are for overall worldwide AWS costs
* It's for actual cost, not for project cost

### AWS Cost Explorer

* A graphical tool to view and analyze your costs and usage
* Review charges and cost associated with your AWS account or org
* Forecast spending for the next 3 months
* Get recommendations for which EC2 Reserved Instances to purchase
* Access to default reports
* API to build custom cost management applications

### AWS Budgets

* Create budget and send alarms when costs exceeds the budget
* 3 types of budgets: Usage, Cost, Reservation
* For Reserved Instances (RI)
  * Track utilization
  * Supports EC2, ElastiCache, RDS, Redshift
* Up to 5 SNS notifications per budget
* Can filter by: Service, Linked Account, Tag, Purchase Option, Instance Type, Region, Availability Zone, API Operation, etc...
* Same options as AWS Cost Explorer

### AWS Cost Allocation Tags

* With Tags we can track resources that relate to each other
* With Cost Allocation Tags we can enabled detailed costing reports
* **Just like Tags, but they show up as columns in Reports**
* AWS Generated Cost Allocation Tags
  * Automatically applied to resource you create
  * Starts with Prefix **aws:** (e.g.: aws:cr
  * They're not applied to resources created before the activation
* User tags
  * Defined by the user
  * Starts with Prefix **user:**
* **Cost Allocation Tags just appear in the Billing Console**
* Takes up to 24 hours for the tags to show up in the report

