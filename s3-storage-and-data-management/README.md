## S3

### S3 Encryption for Objects

* There are 4 methods of encrypting objects in S3
  * SSE-S3: encrypts S3 objects using keys handled & managed by AWS
  * SSE-KMS: leverage AWS KMS to manage encryption keys
  * SSE-C: when you want to manage your own encryption keys
  * Client Side Encryption

#### SSE-S3

* Object is encrypted server side
* AES-2256 encryption type
* Must set header: **"x-amz-server-side-encryption":"AES256"**

#### SSE-KMS

* KMS Advantages: user control + audit trail
* Object is encrypted server side
* Must set header: **"x-amz-server-side-encryption":"aws:kms"**

#### SSE-C

* Amazon S3 does not store the encryption key you provide
* **HTTPS must be used**
* Encryption key must be provided in HTTP headers, for every HTTP request made

#### Client Side Encryption

* Client library such as the Amazon S3 Encryption Client
* Clients must encrypt data themselves before sending to S3
* Clients must decrypt data themselves when retrieving from S3
* Customer fully manages the keys and encryption cycle

#### Encryption in transit (SSL/TLS)

* Amazon S3 exposes:
  * HTTP endpoint: non encrypted
  * HTTPS endpoint: encryption in flight
* Most clients use HTTPS endpoint by default
* HTTPS is mandatory for SSE-C
* Encryption in flight is also called SSL / TLS

 ### S3 Security

 * **User based**
   * IAM policies - which API calls should be allowed for a specific user from IAM console
 * **Resource Based**
   * Bucket Policies - bucket wide rules from the S3 console - allows cross account
   * Object Access Control List (ACL) - finer grain
   * Bucket Access Control List (ACL) - less common

### S3 Bucket Policies

* Use S3 bucket policy to:
  * Grant public access to the bucket
  * Force objects to be encrypted at upload
  * Grant access to another account (Cross Account)

### Bucket settings for Block Public Access

* Block public access to buckets and objects granted through
  * new access control lists (ACLs)
  * any access control lists (ACLs)
  * new public bucket or access point policies
* Block public and cross-account access to buckets and objects through any public bucket or access point policies
* These settings were created to prevent company data leaks
* If you know your bucket should never be public, enable Block Public Access
* Can be set at the account level

### S3 Security - Other

* Networking:
  * Supports VPC Endpoints ( for instances in VPC without www internet)
* Logging and Audit:
  * S3 Access Logs can be stored in other S3 bucket
  * API calls can be logged in AWS CloudTrail
* User Security:
  * MFA Delete: MFA can be required in versioned buckets to delete objects
  * Pre-Sgined URL: URLs that are valid only for a limited time(ex: premium video service for logged users)

### Amazon S3 - Consistency Model

* Read after write consistency for PUTS of new objects
  * As soon as a new object is written we can retrieve it (PUT 200 > GET 200), except if we did a GET before to see if the object existed (GET 404 > PUT 200 > GET 404)
* Eventual Consistency for DELETES and PUTS of existing objects
  * If we read an object after updating, we might get the older version, also might be able to retrieve a file shortly after it was deleted
* It is impossible to request "strong consistency"

## S3 Storage and Data Management For SysOps

### S3 Versioning Advanced

* S3 Versioning creates a new version each time you change a file, that includes when you encrypt a file
* **It's a nice way to get protected against hackers wanting to encrypt our data**
* Deleting a file in the S3 bucket just adds a delete marker on the versioning
* To delete a bucket, you need to remove all the file version within it

### S3 MFA-Delete

* MFA forces user to generate a code on a device before doing important operations on S3
* Versioning is required to use MFA-Delete
* You will need MFA to:
  * permanently delete an object version
  * suspend versioning on the bucket
* You won't need MFA for:
  * enabling versioning
  * listing deleted versions
* Only the bucket owner (root account) can enable/disable MFA-Delete
* Currently MFA-Delete can only be enabled using the CLI

### S3 Access Logs

* For audit purposes, you may want to log all access to S3 buckets
* Any request made to S3, from any account, authorized or denied, will be logged into another S3 bucket
* That data can be analyzed using data analysis tools, like Amazon Athena
* https://docs.aws.amazon.com/AmazonS3/latest/dev/LogFormat.html

### S3 Replication (CRR & SRR)

* **Must enable versioning** in source and destination
* Cross Region Replication (CRR)
  * Use cases: Compliance, lower latency access, replication across accounts
* Same Region Replication (SRR)
  * log aggregation, live replication between production and test accounts
* Buckets can be in different accounts
* Copying is asynchronous
* Must give propoer IAM permissions to S3
* After activating, only new objects are replicated
* For DELETE operations:
  * If you delete without a version ID, it adds a delete marker, not replicated
  * If you delete with a version ID, it deletes in the source, not replicated
* There is no "chaining" of replication
  * If bucket 1 has replication into bucket 2, which has replication into bucket 3, does not mean that bucket 1 has replication into bucket 3

### S3 Policies

*  Review and analyze:
   *  https://docs.aws.amazon.com/AmazonS3/latest/dev/example-bucket-policies.html

### S3 pre-sgined URLs

* Can generate pre-signed URLs using SDK or CLI
  * For downloads (easy, can use the CLI)
  * For uploads (harder, must use the SDK)
* Valid for a default of 3600 seconds, can change timeout with --expires-in [TIME_BY_SECONDS] argument
* Users given a pre-signed URL inherit the permissions of the person who generated the URL for GET / PUT
* Use cases:
  * Alow only logged-in users to download a premium video of your S3 bucket
  * Allow an ever changing list of users to download files by generating URLs dynamically
  * Allow temporarily a user to upload a file to a precise location in our bucket

### AWS CloudFront

* Content Delivery Network
* Improves read performance, content is cached at the edge
* DDoS protection, integration with Shield, AWS Web Application Firewall
* Can expose external HTTPS and talk to internal HTTPS backends

#### CLoudFront - Origins

* S3 bucket
  * Enhanced security with CloudFront **Origin Access Identity (OAI)**
  * CloudFront can be used as an ingress (to upload files to S3)
* Custom Origin (HTTP)
  * Application Load Balancer
  * EC2 instance
  * S3 website (must first enable the bucket as a static S3 website)
  * Any HTTP backend you want
  * http://d7uri8nf7uskq.cloudfront.net/tools/list-cloudfront-ips

#### CloudFront Geo Restriction

* You can restrict who can access your distribution with **Whitelists** and **Blacklists**
* The "country" is determined using a 3rd party Geo-IP database
* Use case: Copyright Laws to control access to conents

#### CloudFront Access Logs

* CloudFront access logs: logs every request made to CloudFront into a logging S3 bucket

#### CloudFront Reports

* It's possible to generate reports on:
  * Cache Statistics Report
  * Popular Objects Report
  * Top Referrers Report
  * Usage Reports
  * Viewers Report
* These reports are based on the data from the Access Logs (Access Logs don't have to be enabled to have access to the reports)

#### CloudFront Troubleshooting

* CloudFront caches HTTP 4xx and 5xx status codes returned by S3 (or the origin server)
* 4xx error code indicates that user doesn't have access to the underlying bucket (403) or the object the user is requesting is not found (404)
* 5xx error codes indicates Gateway issues

### S3 Inventory

* Amazon S3 inventory helps manage your storage
* Audit and report on the replication and encryption status of your objects
* Use cases:
  * Business
  * Compliance
  * Regulartory needs
* You can query all the data using Amazon Athena, Redshift, Presto, Hive, Spark...
* Can setup multiple inventories
* Data goes from a source bucket to a target bucket (need to setup policy)

## S3 Storage Classes

### Amazon Glacier

* Low Cost object storage meant for archiving / backup
* Data is retained for the longer term (10s of years)
* Alternative to on-premise magnetic tape storage
* Average annual durability is 99.999999999%
* Cost per storage per month ($0.004 / GB) + retrieval cost
* Each item in Glacier is called "Archive" (up to 40TB)
* Archives are stored in "Vaults"

### Amazon Glacier Storage Tier

#### Amazon Glacier

* Retrieval options:
  * Expedited (1 to 5 minutes)
  * Standard (3 to 5 hours)
  * Bulk (5 to 12 hours)
  * Minimum storage duration of 90 days

#### Amazon Glacier Deep Archive

* For long term storage - cheaper
* Retrieval options:
  * Standard (12 hours)
  * Bulk (48 hours)
  * Minimum storage duration of 180 days

### S3 Lifecycle Rules

* **Transition actions**: it defines when objects are transitioned to another storage class
  * Move objects to Standard IA class 60 days after creation
  * Move to Glacier for archiving after 6 months
* **Expiration actions**: configure objects to expire (delete) after some time
  * Access log files can b eset to delete after 365 days
  * **Can be used to delete old versions of files (if versioning is enabled)**
  * Can be used to delete incomplete multi-part uploads
* Rules can be created for a certain prefix (e.g.: s3://mybucket/temp/*)
* Rules can be created for certain object tags (e.g.: Department: Finance)

### S3 Performance

* Amazon S3 automatically scales to high request rates, latency 100-200 ms
* Your application can achieve at least 3500 PUT/COPY/POST/DELETE and 5500 GET/HEAD requests per second per prefix in a bucket
* There are no limits to the number of prefixes in a bucket
  * Example (object path => prefix):
    * bucket/folder1/sub1/file => /folder1/sub1/
    * bucket/1/file => /1/

#### S3 - KMS Limitation

* If you use SSE-KMS you may be impacted by the KMS limits
* When you upload, it calls the **GenerateDataKey** KMS API
* When you donwnload, it calls the **Decrypt** KMS API
* Count towards the KMS quota per second (5500 / 10000 / 30000 requests/second based on region)

#### Multi-Part upload

* Recommended for files > 100 MB, **must use for files > 5G B**
* Can help parallelize uploads (speed up transfers)

#### S3 Transfer Acceleration (upload only)

* Increase transfer speed by transferring file to an AWS edge location which will forward the data to the S3 bucket in the target region
* Compatible with multi-part upload

#### S3 Byte-Range Fetches

* Parallelize GETsx by request specific byte ranges
* Better resilience in case of failures
* Can be used to retrieve only partial data (for example the head of a file)

### S3 Select & Glacier Select

* Retrieve less data using SQL by performing **server side filtering**
* Can filter by rows & columns (simple SQL statements)
* Less network transfer, less CPU cost client-side

### S3 Event Notifications

* S3:ObjectCreated, S3:ObjectRemoved, S3:ObjectRestore, S3:replication, etc...
* Object name filtering possible (e.g.: *.jpg)
* Targets:
  * SNS
  * SQS
  * Lambda
* Can create as many "S3 events" as desired
* S3 event notifications typically deliver events in seconds, but can take a minute or longer
* If two writes are made to a single non-versioned object at the same time, it is possible that only a single event notification will be sent
* If you want to ensure that event notification is sent for every sucessful write, you need to enable versioning

### S3 Analytics - Storage Class Analysis

* You can setup analytics to help determine when to transition objects from one storage type to another
  * Does not work for ONEZONE_IA or Glacier
* Report is updated on a daily basis
* Takes about 24h to 48h from begining to first start
* This helps to configure Lifecycle Rules

### Glacier

* Low cost object storage meant for archiving / backup
* Data is retained for the longer term (10s of years)
* Alternative to on-premise magnetic tape storage
* Average annual durability is 99.999999999%
* Cost per storage per month ( $0.0004 / GB) + retrieval cost
* Each item in Glacier is called "Archive" (up to 40TB)
* Archives are stored in "Vaults"

#### Glacier Operations

* 3 Glacier operations:
  * Upload - Single operation or by parts (MultiPart upload) for larger archives
  * Download - First initiate a retrieval job for the particular archive, Glacier then prepares it for download. User then has a limited time to download the data from staging server
  * Delete - Use Glacier Rest API or AWS SDKs by specifying archive ID
* Restore links have an expiry date
* 3 retrieval options:
  * Expedited (1 to 5 minutes retrieval) - $0.03 per GB and $0.01 per request
  * Standard (3 to 5 hours) - $0,01 per GB and 0.05 per 1000 requests
  * Bulk (5 to 12 hours) - $0,0025 per GB and $0,025 per 1000 requests

#### Glacier - Vault Policies & Vault Lock

* Vault is a collection of archives
* Each Vault has:
  * One vault access policy
  * One vault lock policy
* Vault Policies are written in JSON
* Vault Access Policy is similar to bucket policy (restrict user / account permissions)
* Vault Lock Policy is a poolicy you **lock**, for regulatory and compliance requirements
  * The policy is **immutable, it can never be changed**(that's why it's called a lock)
  * Ex1: Forbid deleting an archive if less than 1 year old
  * Ex2: Implement WORM policy (write once read many)

