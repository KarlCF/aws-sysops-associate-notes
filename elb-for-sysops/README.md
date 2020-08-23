### Elastic Beanstalk - SysOps

* Beanstalk can put application logs into CloudWatch Logs
* You manage the application, AWS will manage the infrastructure
* Know the different deployment modes for your application
* Custom domain: Route 53 ALIAS or CNAME on top of Beanstalk URL
* You are not responsible for patching runtimes (Node.js, PHP, etc...)

#### How Beanstalk deploys applications (e.g.: Rolling)

* EC2 has a base AMI (can configure)
* EC2 gets the new code of the app
* EC2 resolves the app dependencies (can take a while)
* Apps get swapped on the EC2 instance
* To save time on resolving dependencies, we can use **Golden AMI** to fix the problem

#### Golden AMI

* If your application has a lot of application or OS dependencies, and you want to deploy as quickly as possible, you should create a **Golden AMI**
* Golden AMI = standardized company-specific AMI with:
  * Package OS dependencies
  * Package APP dependencies
  * Package comapany-wide software
* By using a Golden AMI to deploy to Beanstalk (in combination of blue / green new ASG deployment), our application won't need to resolve dependencies or a long time to configure

#### Troubleshooting Beanstalk

* If the health of your environment changes to red, try the following:
  * Review environment events
  * Pull logs to view recent log file entries
  * Roll back to a previous, working version of the application
* When accessing external resources, make sure the security groups are correctly configured
* In case of command timeouts, you can increase the deployment timeout

