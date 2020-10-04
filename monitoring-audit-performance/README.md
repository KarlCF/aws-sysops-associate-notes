### AWS CloudWatch Metrics

* Metric is a variable to monitor (CPUUtilization, NetworkIn...)
* Metrics belong to namespaces
* Dimension is an attribute of a metric (instance id, environment, etc...) 
* Up to 10 dimensions per metric
* Metrics have timestamps
* Can create CloudWatch dashboards of metrics

#### CloudWatch Custom Metrics

* Possibility to define and send your own custom metrics to CloudWatch
* Abilitiy to use dimensions (attributes) to segment metrics
  * Instance.id
  * Environment.name
* Metric resolution:
  * Standard: 1 minute
  * High Resolution: up to 1 second  (StorageResolution API parameter) [ Higher cost ]
* Use API call PutMetricData
* Use exponential back off in case of throttle errors

### CloudWatch Dashboards

* Great way to setup dashboards for quick access to keys metrics
* Dashboards are global
* Dashboards can include graphs from different regions
* You canchange the time zone & time range of the dashboards
* You can setup automatic refresh (10s, 1m, 2m, 5m, 15m)
* Pricing:
  * 3 dashboards (up to 50 metrics) for free
  * $3/dashboard/month afterwards

### AWS CloudWatch Logs

* Applications can send logs to CloudWatch using the SDK
* CloudWatch can collect log from:
  * Elastic Beanstalk: collection of logs from application
  * ECS: collection from containers
  * AWS Lambda: collection from function logs
  * VPC Flow Logs: VPC specific logs
  * API Gateway
  * CloudTrail based on filter
  * CloudWatch log agents: for example on EC2 machines
  * Route53: Log DNS queries
* CloudWatch Logs can go to: 
  * Batch exporter to S3 for archival
  * Stream to ElasticSearch cluster for further analytics
* Log storage architecture:
  * Log groups: arbitrary name, usually representing an application
  * Log stream: instances within application / log files / containers
* Can define log expiration policies (never expire, 30 days, etc...)
* Using the AWS CLI we can tail CloudWatch logs
* To send logs to CloudWatch, make sure IAM permissions are correct
* Security
  * Encryption of logs using KMS at the Group Level

#### CloudWatch Logs Metrics Filter & Insights

* CloudWatch Logs can use filter expressions
  * For example, find a specific IP inside of a log
  * Metric filters can be used to trigger alarms
* CloudWatch Logs Insights can be used to query logs and addq queries to CloudWatch Dashboards

### AWS CloudWatch Alarms

* Alarms are used to trigger notifications for any metric
* Alarms are used to trigger notifications for any metric
* Alamrs can go to Auto Scaling, EC2 Actions, SNS notifications
* Various options (sampling, %, max, min, etc...)
* Alarm States:
  * OK
  * INSUFFICIENT_DATA
  * ALARM
* Period: Lenght of time in seconds to evaluate the metric
  * High resolution custom metrics, can only choose 10 sec or 30 sec

#### CloudWatch Alarm Targets

* Stop, Terminate, Reboot or Recover an EC2 instance
* Trigger Auto Scaling Action
* Send notification to SNS (from which you can do pretty much anything)

#### Good to Know

* Alarms can be created based on CloudWatch Logs Metrics Filters
* CloudWatch doesn't test or validate the actions that is assigned
* To test alarms and notifications, set the alarm state to Alarm with the CLI
  * aws cloudwatch set-alarm-state --alarm-name "alarmname" --state-value ALARM --state-reason "testing"

### AWS CloudWatch Events

* Source + Rule => Target
* Schedule: Cron jobs
* Event Pattern: Event rulest to react to a service doing something
  * Ex: CodePipeline state changes!
* Triggers to Lambda functions, SQS / SNS / Kinesis Messages
* CloudWatch Event creates a small JSON document to give information about the change

### CloudTrail

* CloudTrail shows the past 90 days of activity
* The default UI only shows "Create", "Modify" or "Delete" events
* **CloudTrail Trail**:
  * Get a detailed list of all the events you choose
  * Ability to store these events in S3 for further analysis
  * Can be region specific or global
* CloudTrail Logs have SSE-S3 encryption when placed into s3
* Control access to S3 using IAM, Bucket Policy, etc..

### AWS Config

* Helps with auditing and compliance of your AWS resources
* Helps record configurations and changes over time
* Helps record compliance over time
* Probability of storing AWS Config data into S3 (can be queried by Athena)
* Questions that can be solved by AWS Config:
  * Is there unrestricted SSH access to my security groups?
  * Do my buckets have any public access?
  * How has my ALB configuration changed over time?
* You can receive alerts (SNS notifications) for any changes
* AWS Config is a per-region service
* Can be aggregated across regions and accounts

#### Config Rules

* Can use AWS managed config rules (over 75)
* Can make custom config rules (must be defined in AWS Lambda)
  * Evaluate if each EBS disk is of type gp2
  * Evaluate if each EC2 instance is t2.micro
* Rules can be evaluated triggered:
  * For each config change
  * And / or: at regular time intervals
* Pricing
  * No free tier
  * ~$2 USD per active rule per region per month (less after 10 rules)

