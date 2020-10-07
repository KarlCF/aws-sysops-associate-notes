## Route 53

### AWS Route 53 Overview

* Managed DNS
* Most common records:
  * A: hostname to IPv4
  * AAAA: hostname to IPv6
  * CNAME: hostname to hostname
  * Alias: hostname to AWS resource
* Route53 can use:
  * public domain names you own (or buy) | application1.mypublicdomain.com
  * private domain names that can be resolved by your instances in your VPCs | application1.company.internal
* Route53 has advanced features such as:
  * Load Balancing (through DNS - also called client load balancing)
  * Health Checks (although limited...)
  * Routing policy: simple, failover, geolocation, latency, weighted, multi value
* You pay $0.50 per month per hosted zone

### Route53 Health Checks

* Have X health checks failed => unhealthy (default 3)
* Have X health checks passed => healthy (default 3)
* Default Health Check Interval: 30s (can set to 10s - higher cost)
* About 15 health checkers will check the endpoint health
* => one request every 2 seconds on average
* Can have HTTP, TCP, and HTTPS health checks (no SSL verification)
* Possibility of integrating the health check with CloudWatch
* Health checks can be linked to Route53 DNS queries

### Route53 as a Registrar

* A domain name registrar is an organization that manages the reservation of internet domain names
* Famous names:
  * GoDaddy
  * Google Domains
  * Route53
  * etc
* Domain Registrar != DNS

### 3rd Party Registrar with AWS Route53

* If you buy your domain on 3rd party website, you can still use Route53
  1. Create Hosted Zone in Route53
  2. Update NS records on 3rd party website to use Route53 name servers
*  Domain Registrar != DNS
*  Each domain registrar usually comes with some DNS features

