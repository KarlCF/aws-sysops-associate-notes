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

