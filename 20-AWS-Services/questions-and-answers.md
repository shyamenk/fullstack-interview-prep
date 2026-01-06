# AWS Services & Architecture Interview Questions & Answers

## Table of Contents

### Fundamentals & Architecture
1. [IaaS vs PaaS vs SaaS on AWS](#q1-what-is-the-difference-between-iaas-paas-and-saas)
2. [Regions vs Availability Zones (AZs) vs Edge Locations](#q2-what-is-the-difference-between-regions-azs-and-edge-locations)
3. [CapEx vs OpEx (Cost Trade-offs)](#q3-explain-capex-vs-opex-in-cloud-computing)
4. [The 6 Pillars of Well-Architected Framework](#q4-what-are-the-pillars-of-the-aws-well-architected-framework)
5. [Multi-Cloud vs Hybrid Cloud](#q5-what-is-the-difference-between-multi-cloud-and-hybrid-cloud)

### Networking (VPC)
6. [Security Groups vs Network ACLs (NACLs)](#q6-what-is-the-difference-between-security-groups-and-nacls)
7. [NAT Gateway vs NAT Instance](#q7-what-is-the-difference-between-nat-gateway-and-nat-instance)
8. [VPC Peering vs Transit Gateway](#q8-when-should-you-use-transit-gateway-over-vpc-peering)
9. [Direct Connect vs VPN](#q9-what-is-the-difference-between-aws-direct-connect-and-vpn)
10. [Public vs Private Subnets](#q10-what-makes-a-subnet-public-or-private)

### Compute (EC2)
11. [EC2 Pricing Models (Spot, On-Demand, Reserved, Savings Plans)](#q11-explain-ec2-pricing-models-and-when-to-use-each)
12. [Instance Store vs EBS](#q12-what-is-the-difference-between-instance-store-and-ebs)
13. [Vertical vs Horizontal Scaling](#q13-what-is-the-difference-between-vertical-and-horizontal-scaling)
14. [ALB vs NLB vs CLB](#q14-what-are-the-types-of-elastic-load-balancers)

### Storage (S3 & EFS)
15. [S3 Storage Classes & Lifecycle Policies](#q15-explain-s3-storage-classes-and-lifecycle-policies)
16. [EBS vs EFS vs S3](#q16-when-should-you-use-ebs-vs-efs-vs-s3)
17. [S3 Consistency Model](#q17-what-is-the-data-consistency-model-of-s3)

### Security (IAM)
18. [IAM Role vs IAM User](#q18-what-is-the-difference-between-an-iam-role-and-an-iam-user)
19. [S3 Bucket Policy vs IAM Policy](#q19-what-is-the-difference-between-s3-bucket-policy-and-iam-policy)
20. [AWS WAF vs AWS Shield](#q20-what-is-the-difference-between-waf-and-shield)
21. [KMS vs CloudHSM](#q21-what-is-the-difference-between-kms-and-cloudhsm)

### Databases & Caching
22. [RDS vs Aurora](#q22-what-is-the-difference-between-rds-and-aurora)
23. [ElastiCache Strategy (Lazy Loading vs Write-Through)](#q23-what-are-caching-strategies-in-elasticache)
24. [DynamoDB vs RDS](#q24-when-should-you-choose-dynamodb-over-rds)

### Integration & Decoupling
25. [SQS vs SNS](#q25-what-is-the-difference-between-sqs-and-sns)
26. [Standard Queue vs FIFO Queue](#q26-what-is-the-difference-between-standard-and-fifo-sqs-queues)
27. [EventBridge vs SNS](#q27-when-should-you-use-eventbridge-over-sns)

### Monitoring & Management
28. [CloudWatch vs CloudTrail](#q28-what-is-the-difference-between-cloudwatch-and-cloudtrail)
29. [Route53 Routing Policies](#q29-what-are-the-routing-policies-in-route53)
30. [AWS Config vs Trusted Advisor](#q30-what-is-the-difference-between-aws-config-and-trusted-advisor)

---

## Q1. What is the difference between IaaS, PaaS, and SaaS?
Concept:
- **IaaS (Infrastructure as a Service):** You manage OS, runtime, and data. AWS manages hardware/virtualization. (e.g., EC2).
- **PaaS (Platform as a Service):** You manage code and data. AWS manages OS, patching, and scaling. (e.g., Elastic Beanstalk, RDS).
- **SaaS (Software as a Service):** You just use the software. AWS manages everything. (e.g., WorkMail, Connect).

Analogy:
- **IaaS:** Renting a car (You drive, fill gas, maintain insurance).
- **PaaS:** Taking a taxi (You tell driver where to go, they handle driving/maintenance).
- **SaaS:** Riding a bus (Fixed route, you just get on and off).

⚡ One-liner to remember:
IaaS = Host it; PaaS = Build it; SaaS = Consume it.

## Q2. What is the difference between Regions, AZs, and Edge Locations?
Concept:
- **Region:** A separate geographic area (e.g., us-east-1). Completely independent.
- **Availability Zone (AZ):** Isolated data centers *within* a region. Connected by low-latency links. Failure domain.
- **Edge Location:** Content Delivery Network (CDN) endpoints for CloudFront. Caches content closer to users.

Analogy:
- **Region:** A Country (USA, UK).
- **AZ:** Cities within that country (New York, Boston). If power fails in NY, Boston is fine.
- **Edge Location:** Local convenience stores in every neighborhood holding popular items.

⚡ One-liner to remember:
Region = Geographic Area; AZ = Data Center Cluster; Edge = Caching Point.

## Q3. Explain CapEx vs OpEx in Cloud Computing.
Concept:
- **CapEx (Capital Expenditure):** Upfront cost to buy physical servers/datacenters. You pay before you use. (On-Premises model).
- **OpEx (Operational Expenditure):** Pay-as-you-go. You pay only for what you use. No upfront cost. (Cloud model).

Analogy:
- **CapEx:** Buying a house (Big down payment, you own the roof repairs).
- **OpEx:** Renting an apartment (Monthly fee, move out whenever, landlord fixes roof).

⚡ One-liner to remember:
CapEx is "Buy first, use later"; OpEx is "Pay as you go".

## Q4. What are the pillars of the AWS Well-Architected Framework?
Concept:
Six pillars to ensure systems are built correctly:
1.  **Operational Excellence:** Monitoring, automation.
2.  **Security:** IAM, encryption, data protection.
3.  **Reliability:** Recovery from failure (Multi-AZ).
4.  **Performance Efficiency:** Using right instance types, serverless.
5.  **Cost Optimization:** Spot instances, rightsizing.
6.  **Sustainability:** Minimizing environmental impact.

Analogy:
Building a Skyscraper: It needs a maintenance crew (Ops), guards (Security), strong foundation (Reliability), fast elevators (Performance), within budget (Cost), and green energy (Sustainability).

⚡ One-liner to remember:
Ops, Security, Reliability, Performance, Cost, Sustainability.

## Q5. What is the difference between Multi-Cloud and Hybrid Cloud?
Concept:
- **Hybrid Cloud:** Connecting on-premises private data centers to public cloud (AWS) via Direct Connect/VPN.
- **Multi-Cloud:** Using multiple public cloud providers (e.g., AWS + Azure + Google Cloud) simultaneously to avoid vendor lock-in.

Analogy:
- **Hybrid:** You own a garage (On-prem) but rent a storage unit (Cloud) for overflow.
- **Multi-Cloud:** You rent apartments in New York (AWS) and London (Azure) to live in both.

⚡ One-liner to remember:
Hybrid = On-Prem + Cloud; Multi-Cloud = AWS + Azure/GCP.

## Q6. What is the difference between Security Groups and NACLs?
Concept:
- **Security Group (SG):** Instance-level firewall. **Stateful** (Return traffic automatically allowed). Deny rules not possible (only Allow).
- **NACL (Network ACL):** Subnet-level firewall. **Stateless** (Must explicitly allow return traffic). Supports Allow and Deny rules.

Analogy:
- **SG:** A bouncer at the club door (Instance). If he lets you in, he remembers you and lets you out.
- **NACL:** The border patrol at the country border (Subnet). They check your passport on entry AND exit every time.

⚡ One-liner to remember:
SG = Stateful Instance Guard; NACL = Stateless Subnet Guard.

## Q7. What is the difference between NAT Gateway and NAT Instance?
Concept:
Both allow private subnets to access the internet.
- **NAT Gateway:** Managed by AWS. Highly available. Auto-scales. Less maintenance.
- **NAT Instance:** EC2 instance managed by you. Single point of failure (unless configured for HA). Cheaper but requires patching/management.

Analogy:
- **Gateway:** A municipal water pipe. It just works, scales with demand, you don't touch it.
- **Instance:** A water pump in your backyard. You have to fuel it, fix it if it breaks, and it has a max capacity.

⚡ One-liner to remember:
Always prefer NAT Gateway for production; it's managed and HA.

## Q8. When should you use Transit Gateway over VPC Peering?
Concept:
- **VPC Peering:** 1:1 connection between two VPCs. Non-transitive (A<->B and B<->C doesn't mean A<->C). Good for simple setups (< 10 VPCs).
- **Transit Gateway (TGW):** Hub-and-spoke model. Connects thousands of VPCs and on-prem networks through a central hub. Transitive routing.

Analogy:
- **Peering:** Running a separate cable between every employee's desk. Messy if you have 100 employees.
- **Transit Gateway:** A central Wi-Fi router. Everyone connects to the router to talk to each other.

⚡ One-liner to remember:
Peering for simple 1-to-1; Transit Gateway for complex many-to-many.

## Q9. What is the difference between AWS Direct Connect and VPN?
Concept:
- **VPN:** Connects via public internet. Encrypted (IPsec). Quick to set up. Variable latency. Cheap.
- **Direct Connect (DX):** Dedicated physical fiber connection. Does not use public internet. Consistent low latency. High bandwidth. Expensive. Takes weeks to setup.

Analogy:
- **VPN:** Driving on a public highway (Internet) with a police escort (Encryption). Traffic jams (Latency) can happen.
- **DX:** Building your own private tunnel directly to the destination. No traffic, just you.

⚡ One-liner to remember:
VPN = Encrypted tunnel over internet; Direct Connect = Private physical fiber cable.

## Q10. What makes a subnet Public or Private?
Concept:
- **Public Subnet:** Has a Route Table entry pointing to an **Internet Gateway (IGW)**. Resources have public IPs.
- **Private Subnet:** Does NOT have a route to IGW. Accesses internet via **NAT Gateway**. Resources have private IPs only.

Analogy:
- **Public:** A house with a front door opening to the main street.
- **Private:** A room inside the house with no windows; you must go through the hallway (NAT) to leave.

⚡ One-liner to remember:
It's all about the Route Table: Route to IGW = Public.

## Q11. Explain EC2 pricing models and when to use each.
Concept:
1.  **On-Demand:** Pay by second. No commitment. (Unpredictable workloads).
2.  **Reserved Instances (RI):** 1-3 year commitment. Up to 72% discount. (Steady-state databases).
3.  **Spot Instances:** Bid for unused capacity. Up to 90% discount. Can be terminated with 2 min warning. (Batch jobs, stateless apps).
4.  **Savings Plans:** Commit to $X/hour usage. Flexible. (Modern replacement for RIs).

Analogy:
- **On-Demand:** Hotel room rate (Expensive, flexible).
- **Reserved:** Year-long lease (Cheaper, stuck with it).
- **Spot:** Last-minute unsold tickets (Super cheap, might get bumped).

⚡ One-liner to remember:
Spot for fault-tolerant; Reserved/Savings for steady; On-Demand for spiky/unknown.

## Q12. What is the difference between Instance Store and EBS?
Concept:
- **EBS (Elastic Block Store):** Network drive. Persistent (data survives reboot/stop). Can be detached/attached. Slower than Instance Store.
- **Instance Store (Ephemeral):** Physically attached to the host server. Very fast (IOPS). **Ephemeral** (Data is LOST if instance stops/terminates).

Analogy:
- **EBS:** A USB hard drive. You can unplug it and plug it into another computer.
- **Instance Store:** The RAM/Scratchpad inside the computer. If you turn it off, the scratchpad is wiped.

⚡ One-liner to remember:
Instance Store = Fast & Temporary; EBS = Durable & Persistent.

## Q13. What is the difference between Vertical and Horizontal Scaling?
Concept:
- **Vertical (Scale Up):** Increasing the size of the instance (e.g., t2.micro -> t2.large). Has a limit (hardware ceiling). Requires downtime (usually).
- **Horizontal (Scale Out):** Adding *more* instances (e.g., 1 server -> 3 servers). Infinite scale. No downtime. Handled by Auto Scaling Groups.

Analogy:
- **Vertical:** Making a single horse stronger to pull a heavier cart. Eventually, the horse can't get bigger.
- **Horizontal:** Adding 10 horses to pull the cart.

⚡ One-liner to remember:
Scale Up = Bigger machine; Scale Out = More machines (Preferred for Cloud).

## Q14. What are the types of Elastic Load Balancers?
Concept:
1.  **ALB (Application LB):** Layer 7 (HTTP/HTTPS). Path-based routing, Host-based routing. Good for Microservices/Containers.
2.  **NLB (Network LB):** Layer 4 (TCP/UDP). Ultra-high performance (millions of requests/sec). Static IP support.
3.  **CLB (Classic LB):** Legacy (Layer 4/7). Avoid using.

Analogy:
- **ALB:** Receptionist reading the letter content (HTTP headers) to decide department.
- **NLB:** Mail sorter looking only at the address label (IP/Port) and throwing packages fast.

⚡ One-liner to remember:
ALB for HTTP/Web Apps; NLB for TCP/Performance.

## Q15. Explain S3 Storage Classes and Lifecycle Policies.
Concept:
Classes trade availability/speed for cost:
1.  **Standard:** Hot data. High cost.
2.  **Standard-IA (Infrequent Access):** Lower storage cost, retrieval fee.
3.  **Intelligent Tiering:** Moves data automatically based on access patterns.
4.  **Glacier:** Archive. Minutes/Hours to retrieve. Very cheap.
5.  **Glacier Deep Archive:** Long-term compliance. 12 hours retrieval. Lowest cost.

**Lifecycle Policy:** Rules to automate moving objects (e.g., "Move to Glacier after 30 days").

Analogy:
- **Standard:** Desk drawer (Immediate access).
- **Glacier:** Basement box (Takes time to find).
- **Deep Archive:** Off-site warehouse (Takes a day to fetch).

⚡ One-liner to remember:
Move cold data to colder storage classes (Glacier) to save money.

## Q16. When should you use EBS vs EFS vs S3?
Concept:
- **EBS:** Block storage. Attached to **ONE** EC2 instance. Boot volume, Database.
- **EFS:** File storage (NFS). Attached to **MULTIPLE** EC2 instances. Linux only. Shared code/files.
- **S3:** Object storage. Accessed via API/Internet. Unlimited scale. Static assets, backups, media.

Analogy:
- **EBS:** Your laptop's internal hard drive.
- **EFS:** A shared network folder (Google Drive for servers).
- **S3:** Dropbox/Internet storage.

⚡ One-liner to remember:
EBS = Single Disk; EFS = Shared Linux Drive; S3 = Web Object Storage.

## Q17. What is the data consistency model of S3?
Concept:
Since Dec 2020, S3 provides **Strong Read-After-Write Consistency** for all requests.
Previously, it was eventually consistent for overwrites. Now, if you write a file and immediately read it, you get the new version.

Analogy:
Writing on a whiteboard. As soon as the marker leaves the board, everyone sees the new text instantly.

⚡ One-liner to remember:
S3 is now Strongly Consistent (Write -> Read always returns new data).

## Q18. What is the difference between an IAM Role and an IAM User?
Concept:
- **IAM User:** Represents a person or service with long-term credentials (password/access keys).
- **IAM Role:** Represents a set of permissions. It is *assumed* temporarily. Has no password. Used by AWS Services (EC2/Lambda) or for Cross-Account access.

Analogy:
- **User:** An ID Badge assigned to "Bob". Bob takes it home.
- **Role:** A "Safety Vest" labeled "Engineer". Bob puts it on to enter the construction site. When he takes it off, he loses those permissions. Anyone with permission can put on the vest.

⚡ One-liner to remember:
Users have permanent keys; Roles provide temporary credentials.

## Q19. What is the difference between S3 Bucket Policy and IAM Policy?
Concept:
- **Identity-based (IAM Policy):** Attached to User/Role. "What can this user do?" (e.g., Bob can read Bucket A).
- **Resource-based (Bucket Policy):** Attached to the Resource (S3). "Who can access this bucket?" (e.g., Allow Bob to read *this* bucket).
- Both must allow (or at least one Allow and no Deny) for access to work.

Analogy:
- **IAM Policy:** A key on your keychain. It opens specific doors.
- **Bucket Policy:** A sign on the door saying "Only Bob can enter".

⚡ One-liner to remember:
IAM = "I can do X"; Bucket Policy = "X can be done to me".

## Q20. What is the difference between WAF and Shield?
Concept:
- **AWS WAF (Web Application Firewall):** Protects against Layer 7 attacks (SQL Injection, XSS, Bad Bots). You define rules.
- **AWS Shield:** Protects against DDoS attacks (Layer 3/4).
    - **Standard:** Free, auto-on.
    - **Advanced:** Paid ($3k/mo), protects against massive enterprise attacks, cost protection.

Analogy:
- **WAF:** Security guard checking IDs and bag contents (Application layer).
- **Shield:** Concrete bollards preventing trucks from ramming the building (Network flood).

⚡ One-liner to remember:
WAF blocks hackers (SQLi/XSS); Shield blocks DDoS floods.

## Q21. What is the difference between KMS and CloudHSM?
Concept:
- **KMS (Key Management Service):** Managed service for creating/controlling encryption keys. Shared hardware (multi-tenant). Integration with most AWS services.
- **CloudHSM:** Dedicated hardware security module (single-tenant). You manage the hardware. Required for specific compliance (FIPS 140-2 Level 3).

Analogy:
- **KMS:** A bank safety deposit box room. Highly secure, but you share the room with others.
- **CloudHSM:** Buying your own titanium safe and keeping it in your house. Only you have the combination.

⚡ One-liner to remember:
Use KMS by default; use CloudHSM only if compliance requires dedicated hardware.

## Q22. What is the difference between RDS and Aurora?
Concept:
- **RDS:** Managed Database for MySQL, Postgres, etc. Uses EBS volumes. One master, read replicas. Failover takes ~60-120s.
- **Aurora:** AWS-native relational DB. Compatible with MySQL/Postgres. storage scales automatically. 6 copies of data across 3 AZs. Failover < 30s. More expensive but faster.

Analogy:
- **RDS:** A standard car with a professional driver.
- **Aurora:** A Formula 1 car built specifically for the track (Cloud) with a pit crew.

⚡ One-liner to remember:
Aurora is RDS on steroids (Cloud-native, faster replication, auto-scaling storage).

## Q23. What are caching strategies in ElastiCache?
Concept:
1.  **Lazy Loading:** App looks in cache -> Miss -> DB -> Update Cache. Data can be stale.
2.  **Write-Through:** Update DB -> Immediately update Cache. Data always fresh, but write latency is higher.

Analogy:
- **Lazy:** Checking the fridge. Empty? Go to store. Next time it's there.
- **Write-Through:** Every time you buy groceries, you immediately restock the fridge.

⚡ One-liner to remember:
Lazy Loading = Read penalty on miss; Write-Through = Write penalty on update.

## Q24. When should you choose DynamoDB over RDS?
Concept:
Choose **DynamoDB** (NoSQL) when:
- Schema is flexible/unknown.
- You need single-digit millisecond latency at any scale.
- High write throughput.
- Simple queries (Key-Value).

Choose **RDS** (SQL) when:
- Complex joins/relationships are needed.
- ACID transactions are complex (though DynamoDB supports basic transactions).
- Fixed schema.

Analogy:
- **DynamoDB:** A hashtable/dictionary (Fast lookups, no relations).
- **RDS:** An Excel spreadsheet with linked tabs (Complex relations).

⚡ One-liner to remember:
DynamoDB for scale and speed; RDS for complex relationships.

## Q25. What is the difference between SQS and SNS?
Concept:
- **SQS (Simple Queue Service):** Queue (Pull). One-to-One. Decouples producer/consumer. Consumer polls for messages.
- **SNS (Simple Notification Service):** Pub/Sub (Push). One-to-Many. Fan-out architecture (e.g., 1 msg -> Email + Lambda + SQS).

Analogy:
- **SQS:** An email inbox. You read (poll) messages when you have time.
- **SNS:** A megaphone. You shout, and everyone listening (subscribers) hears it instantly.

⚡ One-liner to remember:
SQS = Buffer/Queue (Pull); SNS = Broadcaster (Push).

## Q26. What is the difference between Standard and FIFO SQS queues?
Concept:
- **Standard:** Unlimited throughput. At-least-once delivery (duplicates possible). Best-effort ordering (might be out of order).
- **FIFO:** Exactly-once processing. Strict ordering (First-In-First-Out). Limited throughput (300/s without batching).

Analogy:
- **Standard:** A mosh pit entrance. Everyone gets in eventually, maybe not in order.
- **FIFO:** A single-file line at the bank. No cutting.

⚡ One-liner to remember:
Use FIFO only if order is critical; otherwise Standard for performance.

## Q27. When should you use EventBridge over SNS?
Concept:
**EventBridge** is a serverless event bus (evolution of CloudWatch Events).
- Content-based filtering (Route based on JSON body).
- Schema Registry.
- Integration with SaaS (Salesforce, Zendesk).
- **SNS** is simple text-based filtering and high-throughput broadcasting.

Analogy:
- **SNS:** A radio broadcast. Simple filtering (Channel).
- **EventBridge:** A smart mail sorter. Reads the letter content and routes it to the exact department based on rules.

⚡ One-liner to remember:
EventBridge for complex routing/SaaS integration; SNS for simple high-volume fan-out.

## Q28. What is the difference between CloudWatch and CloudTrail?
Concept:
- **CloudWatch:** Monitoring. Metrics, Logs, Alarms. "What is the performance?" (CPU usage, Errors).
- **CloudTrail:** Auditing. "Who did what?" (API calls history). records every API call made to AWS.

Analogy:
- **CloudWatch:** The dashboard in your car (Speed, RPM, Fuel).
- **CloudTrail:** The dashcam recording who drove the car and where they went.

⚡ One-liner to remember:
CloudWatch monitors Performance; CloudTrail audits API Activity.

## Q29. What are the routing policies in Route53?
Concept:
1.  **Simple:** One record, one IP.
2.  **Weighted:** Split traffic (e.g., 80% Blue, 20% Green).
3.  **Latency:** Route to region with lowest latency for user.
4.  **Failover:** Primary/Secondary (Active-Passive).
5.  **Geolocation:** Route based on user's location (Country).

Analogy:
- **Weighted:** A/B testing.
- **Failover:** Spare tire.
- **Latency:** Go to the nearest store.

⚡ One-liner to remember:
Route53 is not just DNS; it's a traffic load balancer (Global Traffic Manager).

## Q30. What is the difference between AWS Config and Trusted Advisor?
Concept:
- **AWS Config:** Records configuration history. "What did my firewall rule look like yesterday?" Compliance auditing.
- **Trusted Advisor:** Recommends best practices. "You are spending too much money" or "Security group open to 0.0.0.0".

Analogy:
- **Config:** A CCTV camera recording changes to the room layout.
- **Trusted Advisor:** An inspector telling you "That chair is broken" or "You can save money on lights".

⚡ One-liner to remember:
Config tracks history/compliance; Trusted Advisor gives optimization tips.