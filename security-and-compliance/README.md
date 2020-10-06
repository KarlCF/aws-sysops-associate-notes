## Security and Compliance

### DDOS Protection on AWS

* AWS Shield Standard: protects against DDOS attack for your website and application, for all customers at no additional cost
* AWS Shield Advnaced: 24/7 premium DDoS protection
* AWS WAF: Filter specific requests based on rules
* CloudFront and Route 53:
  * Availability protection using global edge network
  * Combined with AWS Shield, provides attack mitigation at the edge
* Be ready to scale - leverage AWS Auto Scaling
* Separate static resources (S3 / CloudFront) from dynamic ones (EC2 / ALB)
* Whitepaper: https://d1.awsstatic.com/whitepapers/Security/DDoS_White_Paper.pdf

### AWS Shield

* AWS Shield Standard:
  * Free service that is activated for every AWS customer
  * Provides protection from atttacks such as SYN/UDP Floods, Reflection attacks and other layer 3 / layer 4 attacks
* AWS Shield Advanced:
  * Optional DDoS mitigation service ($3.000 per month per organization)
  * Protect against more sophisticated attack on CloudFront, Route 53, Classic, Application & Network Load Balancer (select regions), Elastic IP / EC2
  * 24/7 access to AWS DDoS response team (DRP)
  * **Protect against higher fees during usage spikes due to DDoS**

### AWS WAF - Web Application Firewall

* Protects your web applications from common web exploits
* Define customizable web security rules:
  * Control which traffic to allow or block to your web applications
  * Rules can include: IP addresses, HTTP headers, HTTP body, or URI strings
  * Protect from common attack - SQL injection and Cross-Site Scripting (XSS)
  * Protect against bots, bad user agents, etc...
  * Size contrainsts
  * Geo Match
* Deploy on CloudFront, Application Load Balancer or API Gateway
* Leverage existing marketplace rules

### AWS Inspector

* Only for EC2 instances
* Analyze against known vulnerabilities
* Analyze against uninteded network accessibility
* AWS Inspector Agent must be installed on OS in EC2 instances
* Define template (rules package, duration, attributes, SNS topics)
* No own custom rules possible - only use AWS managed rules
* After the acssessment, you'll get a report with a list of vulnerabilities

#### What does AWS Inspector evaluate?

* For Network assesment:
  * Netowork Reachability
* For Host assesments:
  * Common Vulnerabilities and Exposures
  * Center for Internet Security (CIS) Benchmarks
  * Security Best Practices
  * Runtime Behavior Analysis

### Logging in AWS for security and compliance

* Service logs include: 
  * CloudTrail trails - trace all API calls
  * Config Rules - for config & compliance over time
  * CloudWatch Logs - for full data retention
  * VPC Flow Logs - IP traffic within your VPC
  * ELB Access Logs - metadata of requests made to your load balancers
  * CloudFront Logs - web distribution access logs
  * WAF Logs - full logging of all requests analyzed by the service
* Logs can be analyzed using AWS Athena if they're stored in S3
* You should encrypt logs in S3, control access usign  IAM & Bukcet Policies, MFA
* Move Logs to Glacier for cost savings
* Read whitepaper if interested at: 
  * https://d1.awsstatic.com/whitepapers/compliance/AWS_Security_at_Scale_Logging_in_AWS_Whitepaper.pdf

### AWS GuardDuty

* Intelligent Threat discovery to Protect AWS Account
* Uses Machine Learning algorithms, anomaly detection, 3rd party data
* One click to enable
* 30 days free trial, after that the service can be expensive
* Input data includes:
  * CloudTrail Logs: unusual API calls, unauthorized deployments
  * VPC Flow Logs: unusual internal traffic, unusual IP address
  * DNS Logs: compromised EC2 instances sending encoded data within DNS queries
* Notifies you in case of findings
* Integration with AWS Lambda

### Trusted Advisor

* No need to install - high level AWS account assessment
* Analyze your AWS accounts and provides recommendation:
  * Cost Optimization
  * Performance
  * Security
  * Fault Tolerance
  * Service Limits
* Core checks and recommendations - free for all customers (although some are disabled by default)
* Can enable weekly email notification from the console
* Full Trusted Advisor - Available for Business & Enterprise support plans
  * Ability to set CloudWatch alarms when reaching limits

### Encryption in AWS Services

* Requires migration (through Snapshot / Backup)
  * EBS Volumes
  * RDS databases
  * ElastiCache
  * EFS network file ystem
* In-place encryption:
  * S3

### Cloud HSM

* CloudHSM => AWS **provisions** encryption **hardware**
* Dedicated Hardware (HSM = Hardware Security Module)
* You manage your own encryption keys entirely (not AWS)
* The CloudHSM hardware device is tamper resistant
* FIPS 140-2 Level 3 compliance
* CloudHSM clusters are spread across multi AZ (HA)
* Supports both symmetric and **asymmetric** encryption (SSL/TLS keys)
* No free tier available
* Must use the CloudHSM Client Software

### IAM + MFA

* MFA for root user can be configured from IAM dashboard
* MFA can also be configured from CLI
* You can setup MFA for individual Users
* **Credentials Report**:
  * A CSV report file on all the IAM users and credentials
  * This shows all who have enabled MFA

### IAM PassRole Option

* In order to assign a role to a AWS service, you need IAM:PassRole

### AWS STS - Security Token Service

* ALlows to grant limited and temporary access to AWS resources
* Token is valid for up to one hour (must be refreshed)
* **Cross Account Access**
  * Allows users from one AWS account access resources in another
* **Federation (AD)**
  * Provides a non-AWS user with temporary AWS access by linking users Active Directory credentials
  * Uses SAML (Security Assertion markup language)
  * Allows SSO, which enables users to log in to AWS console without assigning IAM credentials
* **Federation with third party providers / Cognito**
  * Used mainly in web and mobile applications
  * Makes use of Facebook / Google / Amazon etc to federate them

#### Cross Account Access

* Define an IAM role for another acccount to access
* Define which accounts can access this IAM Role
* Use AWS STS to retrieve credentials and impersonate the IAM Role you have access to (AssumeRole API)
* Temporary credentials can be valid between 15 inutes to 1 hour

#### What's Identity Federation

* Federation lets users outside of AWS to assume temporary role for accessing AWS resources
* These users assume identity provided access role
* **Federation assumes a form of 3rd party authentication**
  * LDAP
  * Microsoft Active Directory (~= SAML)
  * Single Sign On
  * Open ID
  * Cognito
* **Using federation, you don't need to create IAM users (user management is outside of AWS)**

#### SAML Federation **For Enterprises**

* To integrate Active Directory / ADFS with AWS (or any SAML 2.0)
* Provide access to AWS Console or CLI (through temporary creds)
* No need to create an IAM user for each of your employees

#### Custom Identity Broker Application **For Enterprises**

* Use only if identity provider is not compatible with SAML 2.0
* The identity broker must determine the appropriate IAM policy

####  AWS Cognito - Federated Identity Pools **For Public Applications**

* Goal: 
  * Provide direct access to AWS Resources from the Client Side
* How:
  * Log in to federated identity provider - or remain anonymous
  * Get temporary AWS credentials back from the Federated Identity Pool
  * These credentials come with a pre-defined IAM
* Example: 
  * Provide (temporary) access to write to S3 bucket using Facebook login
* Note:
  * **Web Identity Federation is an alternative to use Cognito but AWS recommends against it**

### AWS Compliance Certifications

* AWS being compliant does not mean you are automatically compliant
* You need to get audited as well

### AWS Artifact

* Portal that provides customer with on-demand access to AWS compliance documentation and AWS agreements
* Artifact Reports - Allows you to download AWS security and compliance documents, like AWS ISO certifications, Payment Card Industry (PCI), and System and Organization Control (SOC) reports
* Artifact Agreements - Allows you to review, accept, and track the status of AWS agreements such as the Business Associate Addendum (BAA)
* Can be used to support internal audit or compliance

