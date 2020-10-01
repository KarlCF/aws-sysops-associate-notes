# Databases for SysOps

## RDS

* Compatible Databases:
  * Postgres
  * Oracle
  * MySQL
  * MariaDB
  * Oracle
  * Microsoft SQL Server
  * Aurora (AWS Proprietary database)

### RDS Advantages

* Managed Service:
  * OS Patching level
  * Continuous backups and restore to specific timestamp (Point in Time restore)
  * Monitoring dashboards
* Read replicas for improved read performance
* Multi AZ setup for DR (Disaster Recovery)
* Maintenance windows for upgrades
* Scaling capability (vertical and horizontal)

### Drawback

* Can't SSH into your instances

### RDS Read Replicas 

* Up to 5 Read Replicas
* Within AZ, Cross AZ or Cross Region
* Replication is **ASYNC**, so reads are eventually consistent
* Replicas can be promoted to their own DB
* Applications must update the connection string to leverage read replicas

### RDS Backups

* Backups are automatically enabled in RDS
* Automated Backups:
  * Daily full snapshots of the database
  * Capture transaction logs in real time
  * => ability to restore to any point in tmie
  * 7 days retention (can be increased to 35 days)
* DB Snapshots:
  * Manually triggered by the user
  * Retention of backup for as long as you want

### RDS Encryption

* Encryption at rest capability with AWS KMS - AES-256 encryption
* SSL certificates to encrypt data to RDS in flight
* To **enforce** SSL:
  * **PostgreSQL**: rds.force_ssl=1 in the RDS Console (Parameter Groups)
  * **MySQL**: Within the DB:
    * GRANT USAGE ON *.* TO 'mysqluser'@'%' **REQUIRE SSL**;
* To **connect** using SSL (doesn't mean enforcing!):
  * Provide the SSL Trust certificate (can be downloaded from AWS)
  * Provide SSL options when connecting to database

### RDS Security

* RDS Security works by leveraging security groups - it controls who can **communicate** with RDS
* IAM policies help control who can **manage** AWS RDS
* Login can be with traditional Username and Password
* IAM users can be used to authenticate on RDS (MySQL / Aurora)

### RDS vs Aurora

* PostGres and MySQL are both supported as Aurora DB (that means your drivers will work as if Aurora was a Postgres or MySQL database)
* Aurora is "AWS cloud optimized" and claims 5x performance improvement over MySQL on RDS, over 3x the performance of Postgres on RDS
* Aurora storage automatically grows in increments of 10GB, up to 64 TB
* Aurora can have 15 replicas while MySQL has 5, and the replication process is faster (sub 10ms replica lag)
* Failover in Aurora is instantaneos. It's HA native
* Aurora is 20% more expensive than RDS.

### RDS Multi AZ in depth

* **Multi AZ is not used to support the reads**
* The failover happens only in the following conditions:
  * The primary DB instance fails
  * An Availability Zone outage
  * The DB instance server type is changed
  * The operating system of the DB instance is undergoing software patching
  * A manual failover of the DB instance was initiated using Reboot with failover
* No failover for DB operations: long-running queries, deadlocks or database corruption errors
* Endpoint is the same after failover (no URL change in application needed)
* When to use:
  * Lower maintenance impact it happens on the standby, which is tren promoted to master
  * Backups are created from the standby
  * Multi AZ is only within a single region, not cross region. Region outages impact availability

### RDS Read Replicas in depth

* Read Replicas help scaling read traffic
* A Read Replica can be promoted as a standalone database (manually)
* Read Replicas can be within AZ, Cross AZ or Cross Region
* Each Read Replica has its own DNS endpoint
* You can have Read Replicas of Read Replicas
* Read Replicas can be Multi-AZ
* Read Replicas help with Disaster Recovery by using a cross region RR
* RDS Read Replicas are **not supported for Oracle**
* **Read Replicas can be used to run BI / Analytics for example**

### DB Parameter Groups

* You can configure the DB engine using Parameter Groups
* Dynamic parameters are applied immediately
* Static parameters are applied after instance reboot
* You can modify parameter group associated witha  DB (must reboot)
* See documentation for a list of parameters for a DB technology
* **Must-know parameter**
  * PostgreSQL / SQL Server: **rds.force_ssl=1 **[To force SSL connections]
  * MySQL / MariaDB: **GRANT SELECT ON mydatabase.* TO 'myuser'@'%' IDENTIFIED BY '...' REQUIRE SSL** [To force SSL connections]

### RDS Backups vs Snapshuts

#### Backups

* Backups are "continuous" and allow point in time recovery
* Backups happen during maintenance windows
* When you delete a DB instance, you can retain automated backups
* Backups have a retention period you set between 0 and 35 days

#### Snapshots

* Snapshots takes IO operations and can stop the database from seconds to minutes
* Snapshots taken on Multi AZ don't impact the master - just the standby
* Snapshots are incremental after the first snapshot
* You can copy and share DB Snapshots
* **Manual** Snapshots don't expire
* You can take a "final snapshots" when you delete your DB

### RDS Security for SysOps

* Encryption at rest:
  * Only able to enable when you first create the DB instance
  * From unencrypted to encrypted:
    * Create Snapshot
    * Copy snapshot as encrypted
    * Create DB from encrypted Snapshot
* User responsability:
  * Check the ports / IP / security group inbound rules in DB's SG
  * In-database user creation and permissions
  * Creating a database with or without public access
  * Ensure parameter groups or DB is configured to only allow SSL connections
* AWS responsability:
  * Ensure no SSH access
  * DB patching
  * OS Patching
  * Does not allow audit on underlying instance

### RDS API

* DescribeDBInstances API
  * Helps get a list of all the DB instances you have deployed including Read Replicas
  * Helps to get the DB version
* CreateDBSnapshot API - Make snapshot of a DB
* DescribeEvents API - Helps return information about events in your DB Instance
* RebootDBInstance API - Helps to initiate a 'forced' failover by rebooting DB instace

### RDS with CloudWatch

* CloudWatch metrics associated with RDS (gather from the hypervisor):
  * Database Connections
  * SwapUsage
  * ReadIOPS / WriteIOPS
  * ReadLatency / WriteLatency
  * ReadThroughPut / WriteThroughPut
  * DiskQueueDepth
  * FreeStorageSpace
* Enhanced Monitoring (gathered from an agent on the DB instance)
  * Useful when you need to see how different processes or threads use the CPU
  * Access to over 50 new CPU, memory, file system ad disk I/O metrics

### RDS Performance Insights

* Visualize your data performance and analyze any issues that affect it
* With the Performance Insights dashboard, you can visualize the database load and filter the load:
  * By Waits => find the resource that is the bottleneck (CPU, IO, lock, etc...) 
  * By SQL statements => find the SQL statement that is the problem
  * By Hosts => find the server that is using the most our DB
  * By Users => find the user that is using the most our DB
* DBLoad = the number of active sessions for the DB engine
* You can view the SQL queries that are putting load on your database

### Amazon Aurora

* Aurora is a proprietary technology from AWS (not open sourced!)
* Postgres and MySQL are both supported as Aurora DB (that means your drivers will work as if Aurora was a Postgres or MySQL database)
* Aurora is "AWS Cloud optimized" and claims 5x performance improvement over MySQL, over 3x the performance of Postgres on RDS
* Aurora storage automatically grows in increments of 10GB, up to 64TB
* Aurora can have 15 replicas while MySQL has 5, and the replication process is faster (sub 10ms replica lag)
* Failover in Aurora is instantaneos. Its HA native
* Aurora is 20% more expensive

#### Aurora High Availability and Read Scaling

* 6 copies of your data across 3 AZ:
  * 4 copies out of 6 needed for writes
  * 3 copies out of 6 needed for reads
  * Self healing with peer-to-peer replication
  * Storage is stripped across 100s of volumes 
* One Aurora Instance receives writes (master)
* Automated failover for master in less than 30 seconds
* Master + up to 15 Aurora Read Replicas serve reads

#### Features of Aurora

* Automatic fail-over
* Backup and Recover
* Isolation and security
* Industry compliance
* Push-button scaling
* Automated Patching with Zero Downtime
* Advanced Monitoring
* Routine Maintenance
* Backtrack: restore data at any point of time without using backups

#### Aurora Security

* Encryption at rest using KMS
* Automated backups, snapshots and replicas are also encrypted
* Encryption in flight using SSL (same process as MySQL or Postgres)
* Authentication using IAM
* You are responsible for protecting the instance with security groups
* You can't SSH

#### Aurora Serverless

* No need to choose an instance size
* Only supports MySQL 5.6 > & Postgres (beta)
* Helpful when you can't predict the workload
* DB cluster starts, shutdown and scales automatically based on CPU / connections
* Can migrate from Aurora Cluster to Aurora Serverless and vice versa
* Aurora Serverless usage is measured in ACU (Aurora Capacity Units)

### ElastiCache

* ElastiCache is to get managed Redis or Memcached
* Write Scaling using sharding
* Read Scaling using Read Replicas
* Multi AZ with Failover Capability
* AWS takes care of OS maintenance / patching, optimizations, setup, configuration, monitoring, failure recovery and backups

#### REDIS

* Multi AZ with Auto-Failover
* Read Replicas to scale reads and have **high availability**
* Data Durability using AOF persistence
* Backup and restore features

#### MEMCACHED

* Multi-node for partitioning of data (sharding)
* Non persistent
* No backup and restore
* Multi-threaded architecture