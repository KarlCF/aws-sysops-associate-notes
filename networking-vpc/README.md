## Network - VPC

### Default VPC Summary

* New instances are launched into default VPC if no subnet is specified
* Default VPC have internet connectivity and all instances have public IP
* We also get a public and a private DNS name
* DNS resolution Enabled
* DNS hostnames Enabled

### VPC Overview

* You can have multiple VPCs in a region (max 5 per region, **soft limit**)
* Max CIDR per VPC is 5. For each CIDR
  * Min size is /28 = 16 IP Addresses
  * Max size is /16 = 65536 Ip Addresses
* Because VPC is private, only the private IP ranges are allowed
  * 10.0.0.0/8 | 172.16.0.0/12 | 192.168.0.0/16
* Your VPC CIDR should not overlap with your other networks (ex: corporate)

### Subnets

* AWS reservers 5 IPs address (first 4 and last 1 IP address) in each Subnet
* These 5 IPs are not available for use and cannot be assigned
* Ex: On CIDR block 10.0.0.0/24 reserved IP are:
  * 10.0.0.0: Network address
  * 10.0.0.1: Reserved by AWS for the VPC router
  * 10.0.0.2: Reserved by AWS for mapping to Amazon-provided DNS
  * 10.0.0.3: Reserved by AWS for future use
  * 10.0.0.255: Network broadcast address. AWS does not support broadcast in a VPC, therefore the address is reserved
* Exam tip: Be mindful of the -5 IPs.

### Internet Gateways

* Internet gateways helps  our VPC instances connect with the internet
* It scales horizontally and is HA and redundant
* Must be created separately from VPC
* One VPC can only be attached to one IGW and vice versa
* Iternet Gateway is also a NAT for the instances that have a public IPv4
* Internet Gateways on their own do not allow internet access
* Route tables must be edited

### NAT Instances - Network Address Translation

* Allow instances in the private subnets to connect to the internet
* Must be launched in a public subnet
* Must disable EC2 flag: Source / Destination Check
* Must have Elastic IP Attached to it
* Route table must be configured to route traffic from private subnets to NAT instance
* Amazon Linux AMI pre-configured are available
* Not highly available / resilient setup out of the box
* => would need to create ASG in multi AZ + resilient user-data script
* Internet traffic bandwidth depends on EC2 instance performance
* Must manage security groups & rules:
  * Inbound:
    * Allow HTTP / HTTPS traffic coming from Private Subnets
    * Allow SSH from your home network (access is provided through Internet Gateway)
  * Outbound
    * Allow HTTP / HTTPS traffic to the internet

### NAT Gateway

* AWS managed NAT, higher bandwidth, better availability, no admin
* Pay by the hour for usage and bandwidth
* NAT is created in a specific AZ, uses an Elastic IP
* Cannot be used by an instance in that subnet (only from other subnets)
* Requires an IGW (Private Subnet => NAT => IGW)
* 5 Gbps of bandwidth with automatic scaling up to 45 Gbps
* No security group to manage / required
* Nat Gateway is resilient within a single-AZ
* Must create **multiple NAT Gateway** in **multiple AZ** for fault-tolerance
* There is no cross AZ failover needed, because if an AZ goes down it doesn't need NAT

### NAT Instance vs Gateway

* https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-comparison.html

### DNS Resolution in VPC

* enableDnsSupport: (= DNS Resolution setting)
  * Default True
  * Helps decide if DNS resolution is supported for the VPC
  * If True, queries the AWS DNS server at 169.254.169.253
* enableDnsHostname: (= DNS Hostname setting)
  * False by default for newly created VPC, True by default for Default VPC
  * Won't do anything unless enableDnsSupport = true
  * If True, assign public hostname to EC2 instance if it has a public IP
* If you custom DNS domain in a private zone in Route 53, you must set both these attributes to true

### Network ACLs 

* NACL are like a firewall which control traffic from and to subnet
* Default NACL allows everything outbound and everything inbound
* One NACL per Subnet, new Subnets are assigned the Default NACL
* Define NACL rules:
  * Rules have a number (1-32766) and higher precedence with a **lower number**
  * If you define #100 allow \<IP> and #200 DENY \<IP>, the IP will be allowed
  * Last rule is always an "*" and denies a request in case of no rule match
  * AWS recommends adding rules by increment of 100
* Newly created NACL will deny everything
* NACL are a great way to blocking a specific IP at the subnet level

### VPC Peering

* Connect two VPC, privately using AWS' network
* Make them behave as if they were in the same network
* Must not have overlapping CIDR
* VPC Peering connection is **not transitive**
* You can do VPC peering with another AWS account
* You must update route tables **in each VPC's subnets** to ensure instances can communicate
* VPC peering can work inter-region, cross-account
* You can reference a security group of a peered VPC (e.g.: "securitygroupname"/"accountID")

### VPC Endpoints

* Endpoints allow you to connect to AWS Services using a private network instead of the public www network
* They scale horizontally and are redundant
* They remove the need of IGW, NAT, etc... to access AWS Services
* **Interface:** provisions an ENI (private IP address) as an entry point (must attach security group) - most AWS services
* **Gateway:** provisions a target and must be used in a route table - used by S3 and DynamoDB
* In case of issues:
  * Check DNS Setting Resolution in your VPC
  * Check Route Tables

### Flow Logs

* Capture information about IP traffic going into your interfaces
  * VPC Flow Logs
  * Subnet Flow Logs
  * Elastic Network Interface Flow Logs
* Helps to monitor & troubleshoot connectivity issues
* Flow logs data can go to S3 / CloudWatch Logs
* Captures network information from AWS managed interfaces too: ELB, RDS, ElastiCache, Redshift, WorkSpaces

#### Flow Log Syntax

* \<version>\<account-id>\<interface-id>\<srcaddr>\<dstaddr>\<srcport>\<dstport>\<protocol>\<packets>\<bytes>\<start>\<end>\<action>\<log-status>
* srcaddr, dstaddr help identify problematic IP
* srcport, dstport help identify problematic ports
* Action: success or failure of the request due to Security Group / NACL
* Can be used for analytics on usage patterns, or malicious behavior
* Flow logs example: https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html#flow-log-records
* Query VPC flow logs using Athena on S3 or CloudWatch Logs Insights

### Bastion Hosts

* Use a Bastion Host to SSH into our private instances
* The bastion is in the public subnet which is then conected to all other private subnets
* Bastion Host security group mnust be tightened
* Make sure bastion host only has port 22 traffic from the IP you need, nott from the security groups of your other instances

### Site to Site VPN

#### Virtual Private Gateway

* VPN concentrator on the AWS side of the VPN connection
* VGW is created and attached to the VPC from which you want to create the Site-to-site VPN connection
* Possibility to customize the ASN

#### Customer Gateway

* Software application or physical device on customer side of the VPN connection
* VGW is created and attached to the VPC from which you want to create the Site-to-Site VPN connection
* Possibility to customize the ASN

#### Customer Gateway

* Software application or physical device on customer side of the VPN connection
* https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html
* IP address:
  * Use static, internet-routable IP address for your customer gateway device
  * If behind a CGW behind NAT (with NAT-T), use the public IP address of the NAT

### Direct Connect

* Provides a dedicated **private** connection from a remote network to your VPC
* Dedicated connection must be setup between your DC and AWS Direct Connect locations
* You need to setup a Virtual Private Gateway on your VPC
* Access public resources (S3) and private (EC2) on same connection
* Use cases:
  * Increase bandwidth throughput - working with large data sets - lower cost
  * More consistent network experience - applications using real-time data feeds
  * Hybrid Environments (on prem + cloud)
* Supports both IPv4 and IPv6

### Direct Connect Gateway

* If you want to setup a Direct Connect to one or more VPC in many different regions (same account), you must use a Direct Connect Gateway

#### Direct Connect - Connection Types

* **Dedicated Connections:** 1 Gbps and 10 Gbps capacity
  * Physical ethernet port dedicated to a customer
  * Request made to AWS first, then completed by AWS Direct Connect Partners
* **Hosted Connections:** 50 Mbps, 500 Mbps, to 10 Gbps
  * Connection requests are made via AWS Direct Connect Partners
  * Capacity can be added or removed on demand
  * 1, 2, 5, 10 Gbps available at select AWS Direct Connect Partners
* Lead times are often longer than 1 month to establish a new connection

#### Direct Connect - Encryption

* Data in transit is not encrypted but is private
* AWS Direct Connect + VPN provides an IPsec-encrypted private connection
* Good for an extra level of security, but slightly more complex to put in place

### Egress Only Internet Gateway

* Egress only Internet Gateway is for IPv6 only
* Similar function as a NAT, but a NAT is for IPv4
* IPv6 are all public addresses, therefore all our instances with IPv6 are publicly accessibly
* Egress Only Internet Gateway gives our IPv6 instances access to the internet, but they won't be directly reachable by the internet
* After creating an Egress Only Internet Gateway, edit the route tables

