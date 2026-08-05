<div align="center">

# 🎯 Quiz Answer Key

<img src="https://img.shields.io/badge/Answer%20Key-All%2025%20Labs-2ECC71?style=for-the-badge&labelColor=232F3E" />

**The official answer key for every "🎮 Test Yourself!" quiz in the AWS Hands-On Labs.**

> ⚠️ **No peeking before you try!** Attempt the quiz in the lab first — then check your answers here. The struggle is where the learning happens. 💪

</div>

---

## 📚 Quick Jump

| Lab | Topic | Questions |
|:---:|---|:---:|
| [01](#-lab-01--ec2-launch--connect) | EC2 — Launch & Connect | 3 |
| [02](#-lab-02--security-groups-deep-dive) | EC2 — Security Groups | 3 |
| [03](#-lab-03--ebs-volumes-and-snapshots) | EBS — Volumes & Snapshots | 3 |
| [04](#-lab-04--ami-create-and-clone) | AMI — Create & Clone | 3 |
| [05](#-lab-05--s3-static-website-hosting) | S3 — Static Website Hosting | 3 |
| [06](#-lab-06--s3-versioning-and-lifecycle) | S3 — Versioning & Lifecycle | 3 |
| [07](#-lab-07--s3-cross-region-replication) | S3 — Cross-Region Replication | 3 |
| [08](#-lab-08--vpc-build-from-scratch) | VPC — Build from Scratch | 3 |
| [09](#-lab-09--vpc-nat-and-endpoints) | VPC — NAT Gateway & Endpoints | 3 |
| [10](#-lab-10--application-load-balancer) | ELB — Application Load Balancer | 3 |
| [11](#-lab-11--auto-scaling-group) | ASG — Auto Scaling Group | 3 |
| [12](#-lab-12--route-53-dns-and-failover) | Route 53 — DNS & Failover | 3 |
| [13](#-lab-13--rds-mysql-on-aws) | RDS — MySQL on AWS | 3 |
| [14](#-lab-14--dynamodb-crud) | DynamoDB — CRUD Operations | 3 |
| [15](#-lab-15--cloudwatch-alarms-and-dashboards) | CloudWatch — Alarms & Dashboards | 3 |
| [16](#-lab-16--iam) | IAM — Users, Groups, Roles, Policies | 3 |
| [17](#-lab-17--sns-and-sqs) | SNS & SQS — Messaging | 3 |
| [18](#-lab-18--lambda-s3-trigger) | Lambda — S3 Triggered Function | 3 |
| [19](#-lab-19--lambda-api-gateway) | Lambda — API Gateway REST API | 3 |
| [20](#-lab-20--ecs-fargate-nginx) | ECS — NGINX on Fargate | 3 |
| [21](#-lab-21--cloudformation) | CloudFormation — Deploy EC2 | 3 |
| [22](#-lab-22--cloudtrail) | CloudTrail — Enable & Query | 3 |
| [23](#-lab-23--kms) | KMS — Encrypt S3 and EBS | 3 |
| [24](#-lab-24--aws-backup) | AWS Backup — Multi-Service | 3 |
| [25](#-lab-25--capstone) | Capstone — Full Stack on AWS | 4 |

---

## 📗 Lab 01 — EC2: Launch & Connect

| # | Question | Answer |
|---|---|---|
| Q1 | Which EC2 instance type is Free Tier eligible? | **`t2.micro`** (or `t3.micro`) — the tiny workhorse, 750 hrs/month free |
| Q2 | What port does SSH use, and why does it matter for your security group? | **Port 22**. Only allow it from **My IP** so internet bots can't try to break in |
| Q3 | What does `sudo systemctl enable httpd` do that `start` doesn't? | `start` runs it now; **`enable` makes it auto-start after every reboot** |

**🔥 Bonus Challenge:** Add an HTTPS (port 443) rule and observe what the browser says without an SSL cert.

---

## 📗 Lab 02 — Security Groups Deep Dive

| # | Question | Answer |
|---|---|---|
| Q1 | True or false: a security group is **stateless** — every reply needs its own inbound rule? | **False!** Security groups are **stateful** — replies flow back automatically. (Network ACLs are the stateless ones) |
| Q2 | You add a new inbound rule. Does the instance need a restart? | **No** — SG rules apply **immediately**, no restart needed |
| Q3 | Why is referencing another SG safer than allowing a specific IP range? | SG references stay valid even when IPs change (e.g., auto-scaling) — no hardcoded, stale IPs |

**🔥 Bonus Challenge:** Connect two instances using **only** an SG-to-SG reference — no IP ranges allowed.

---

## 📗 Lab 03 — EBS: Volumes and Snapshots

| # | Question | Answer |
|---|---|---|
| Q1 | Volume in `us-east-1a`, instance in `us-east-1b` — can you attach it? | **No!** EBS volumes are locked to their **Availability Zone** (same AZ only) |
| Q2 | Where are EBS snapshots stored, and are they full copies each time? | In **S3**, and they're **incremental** — only changed blocks are copied |
| Q3 | You mount a volume, reboot, and the mount is gone. Why? | `mount` is temporary — add the entry to **`/etc/fstab`** for permanent auto-mounting |

**🔥 Bonus Challenge:** Snapshot → delete the volume → restore a new one from the snapshot → verify your data is back.

---

## 📗 Lab 04 — AMI: Create and Clone

| # | Question | Answer |
|---|---|---|
| Q1 | What's the difference between an AMI and a snapshot? | A snapshot is just the disk data; an **AMI is the full recipe** — snapshots + launch metadata + permissions |
| Q2 | Should you stop the source instance before creating the AMI? | **Yes** — it guarantees a **consistent** image with no half-written files |
| Q3 | Your clone boots but Apache isn't running. What did the source miss? | **`systemctl enable httpd`** — the service wasn't registered to auto-start, so the clone inherited that |

**🔥 Bonus Challenge:** Create an AMI and launch **two** clones from it — both should serve your custom page.

---

## 📗 Lab 05 — S3: Static Website Hosting

| # | Question | Answer |
|---|---|---|
| Q1 | Can two different AWS accounts create a bucket with the same name? | **No** — bucket names are **globally unique across all AWS accounts** |
| Q2 | You get **403 Forbidden** on your S3 site. Likely cause? | Bucket policy missing/wrong, **Block Public Access** still on, or objects still private |
| Q3 | Why must the homepage file be named `index.html` exactly? | Static hosting treats **`index.html` as the root document** served automatically |

**🔥 Bonus Challenge:** Upload a funny custom `error.html` and visit a bogus URL like `/does-not-exist`.

---

## 📗 Lab 06 — S3: Versioning and Lifecycle

| # | Question | Answer |
|---|---|---|
| Q1 | Versioning is ON and you delete a file. Is it gone forever? | **No** — S3 drops a **delete marker**; the data is hidden but alive and restorable |
| Q2 | Why add a lifecycle policy to move objects to Glacier after 90 days? | To **save money automatically** — cold data on cheap storage, no manual work |
| Q3 | AWS won't let you delete your versioned bucket. What's missing? | The bucket still holds **versions + delete markers** — empty it fully with "List versions" ON first |

**🔥 Bonus Challenge:** Overwrite a file 3×, restore version 1, then delete the file and bring it back.

---

## 📗 Lab 07 — S3: Cross-Region Replication

| # | Question | Answer |
|---|---|---|
| Q1 | What is the #1 cause of CRR setup failure? | **Versioning not enabled on BOTH buckets** — CRR literally can't work without it |
| Q2 | You delete an object in the source bucket. What happens in the destination? | **Nothing** — deletions don't replicate by default; the destination keeps its backup copy |
| Q3 | Give three real-world reasons to enable CRR. | **Disaster recovery**, **compliance**, and **data locality** (users near the second region get faster access) |

**🔥 Bonus Challenge:** Upload, wait for replication, delete from source, and prove the destination still has it.

---

## 📗 Lab 08 — VPC: Build from Scratch

| # | Question | Answer |
|---|---|---|
| Q1 | What single component connects your VPC to the public internet? | The **Internet Gateway (IGW)** — the front door of your neighborhood |
| Q2 | What does the route `0.0.0.0/0 → IGW` mean? | "**Any destination** → send it through the internet gateway" — the anywhere wildcard |
| Q3 | Your instance in a custom subnet has no public IP. What did you forget? | **Auto-assign public IP** — it's OFF by default on custom subnets; flip the switch (porch light 💡) |

**🔥 Bonus Challenge:** Add a second public subnet in a different AZ and launch an instance there — now you're multi-AZ.

---

## 📗 Lab 09 — VPC: NAT Gateway and Endpoints

| # | Question | Answer |
|---|---|---|
| Q1 | Which subnet does the NAT Gateway live in? | The **public subnet** — it needs an internet path (via IGW) to relay traffic |
| Q2 | Can the internet initiate a connection to a private instance behind NAT? | **No** — NAT allows **outbound only**; nobody can reach in |
| Q3 | Cheaper for private S3 access: NAT Gateway or Gateway VPC Endpoint? | The **Gateway VPC Endpoint — it's free!** NAT Gateways cost per-hour + data |

**🔥 Bonus Challenge:** Remove the NAT, run `sudo yum update` on a private instance (fails!), then recreate it.

---

## 📗 Lab 10 — Application Load Balancer

| # | Question | Answer |
|---|---|---|
| Q1 | What does a health check actually do? | Pings each target and **stops sending traffic** to any that fail — sick servers get benched |
| Q2 | At which OSI layer does an ALB work? | **Layer 7** (HTTP/HTTPS) — it understands URLs, paths, and headers |
| Q3 | Why run instances across multiple Availability Zones? | **High availability** — one AZ dies, the other keeps serving. No single point of failure |

**🔥 Bonus Challenge:** Stop one instance and watch the ALB keep the site alive using only the survivor.

---

## 📗 Lab 11 — Auto Scaling Group

| # | Question | Answer |
|---|---|---|
| Q1 | What is a Launch Template, in one sentence? | A **reusable recipe** for instances — AMI, instance type, SG, key pair, user data |
| Q2 | CPU stays above target for 10 minutes. What does the ASG do? | **Scale out** — launch new instances to spread the load |
| Q3 | You manually terminate an ASG-managed instance. What happens? | The ASG notices the count dropped and **launches a replacement** automatically |

**🔥 Bonus Challenge:** Use `stress` to peg the CPU and watch the ASG scale out live, then stop it and watch scale-in.

---

## 📗 Lab 12 — Route 53: DNS and Failover

| # | Question | Answer |
|---|---|---|
| Q1 | Why is it called "Route 53"? | Because **DNS runs on port 53** — it's the route for DNS traffic |
| Q2 | Which record maps a domain name to an IPv4 address? | An **A record** ("Address" record). IPv6 uses AAAA |
| Q3 | Your primary server crashes. What does failover routing do? | The health check fails → Route 53 **switches traffic to the backup server** automatically |

**🔥 Bonus Challenge:** Stop the primary, watch the health check go red and traffic fail over — then watch failback.

---

## 📗 Lab 13 — RDS: MySQL on AWS

| # | Question | Answer |
|---|---|---|
| Q1 | What does a DB Subnet Group actually control? | **Which Availability Zones** your database may live in (pick ≥2 for HA) |
| Q2 | Which port does MySQL use, and where should you open it? | **3306** — only to your app server's SG (or My IP for the lab), never the world |
| Q3 | Why pay for RDS instead of running MySQL yourself? | AWS handles **patching, backups, monitoring, and failover** automatically |

**🔥 Bonus Challenge:** Create a second table and run a real `SELECT ... JOIN ...` across both.

---

## 📗 Lab 14 — DynamoDB: CRUD Operations

| # | Question | Answer |
|---|---|---|
| Q1 | Which is more efficient: a Query or a Scan? | **Query** — it uses an index; a Scan reads every item (slow & expensive at scale) |
| Q2 | What is a Global Secondary Index (GSI) for? | **Querying on a non-key attribute** — an extra index tab for a different lookup path |
| Q3 | Do all items in a table need the same attributes? | **No!** Items are schema-free — each can have different fields |

**🔥 Bonus Challenge:** Build a GSI on a `status` field and run a Query against it.

---

## 📗 Lab 15 — CloudWatch: Alarms and Dashboards

| # | Question | Answer |
|---|---|---|
| Q1 | Basic vs detailed monitoring — what intervals? | Basic = **5 minutes** (free); Detailed = **1 minute** (paid) |
| Q2 | What must an alarm connect to so it actually notifies you? | An **SNS topic** (e.g., an email subscription) |
| Q3 | Where do application logs get centralized? | **CloudWatch Log Groups** — searchable home for all your logs |

**🔥 Bonus Challenge:** Build a dashboard with 3+ widgets and watch CPU spike with `stress`.

---

## 📗 Lab 16 — IAM: Users, Groups, Roles, Policies

| # | Question | Answer |
|---|---|---|
| Q1 | IAM User vs IAM Role? | A **user** is a person with long-term credentials; a **role** is a temporary identity assumed by a service (like EC2) |
| Q2 | What does "least privilege" mean in practice? | Grant **only the minimum permissions** needed — no more, no less |
| Q3 | When use a managed policy vs a custom one? | **Managed** = AWS pre-built (use first); **custom** = your own JSON for exact, narrow needs |

**🔥 Bonus Challenge:** Give a role only `s3:ListBucket`/`GetObject`, try to delete an object (fails!), then grant `DeleteObject` (works!).

---

## 📗 Lab 17 — SNS and SQS: Messaging

| # | Question | Answer |
|---|---|---|
| Q1 | Which service *pushes* and which *pulls*? | **SNS pushes** (loudspeaker 📣); **SQS pulls** (mailbox 📬) |
| Q2 | What is the fan-out pattern? | One message published to SNS is **delivered to many subscribers at once** (email, SQS, Lambda, SMS) |
| Q3 | A worker grabs a message but crashes halfway. What happens? | The **visibility timeout expires** and the message returns to the queue for another worker |

**🔥 Bonus Challenge:** Add a second SQS queue as an SNS subscriber and watch ONE message land in BOTH.

---

## 📗 Lab 18 — Lambda: S3 Triggered Function

| # | Question | Answer |
|---|---|---|
| Q1 | What triggers the Lambda in this lab? | An **S3 event** — a notification fired when an object is uploaded to the bucket |
| Q2 | What is the entry point of a Python Lambda? | **`lambda_handler(event, context)`** |
| Q3 | Where can you see your function's `print()` output? | **CloudWatch Logs** — a log group named after your function |

**🔥 Bonus Challenge:** Log the file **size** from the event payload, then add a delete-object trigger too.

---

## 📗 Lab 19 — Lambda: API Gateway REST API

| # | Question | Answer |
|---|---|---|
| Q1 | In `GET /students`, what's the resource and the method? | `/students` = the **resource** (path); `GET` = the **method** (verb) |
| Q2 | 200 OK vs 201 Created? | **200** = successful read/update; **201** = a new resource was **created** |
| Q3 | Your API still returns old behavior after editing Lambda. Why? | You didn't **Deploy the API to the stage** — edits alone don't go live |

**🔥 Bonus Challenge:** Add `DELETE /students/{id}` handling the `{id}` path parameter, and re-enable CORS.

---

## 📗 Lab 20 — ECS: Deploy NGINX on Fargate

| # | Question | Answer |
|---|---|---|
| Q1 | Task definition vs service? | Task definition = **recipe** (image, CPU, memory, ports); service = **manager** keeping the desired count running |
| Q2 | What makes Fargate different from the EC2 launch type? | **Fargate is serverless** — no EC2 instances to pick, patch, or manage |
| Q3 | One of your tasks crashes. What happens? | The **service replaces it automatically** — self-healing containers |

**🔥 Bonus Challenge:** Change desired count 2 → 4 → 2 and watch tasks scale live.

---

## 📗 Lab 21 — CloudFormation: Deploy EC2

| # | Question | Answer |
|---|---|---|
| Q1 | Template vs stack? | Template = the **YAML plan**; stack = the **live resources** created from it |
| Q2 | How do you delete ALL resources a stack created? | **Delete the stack** — CloudFormation tears everything down in one click |
| Q3 | Why Infrastructure as Code? | **Reproducible, reviewable, versionable, recoverable** — code beats clicking |

**🔥 Bonus Challenge:** Add an `AWS::S3::Bucket` to the template, update the stack, then delete the stack and watch it vanish.

---

## 📗 Lab 22 — CloudTrail: Enable and Query

| # | Question | Answer |
|---|---|---|
| Q1 | Management events vs data events? | **Management** = API calls that change resources; **data** = calls that touch data inside (e.g., S3 object reads) |
| Q2 | How far back does Event History go in the console? | **90 days** — longer requires streaming logs to S3 |
| Q3 | Best long-term, searchable storage for trail logs? | **S3** (permanent archive) + **CloudWatch Logs** (search with Logs Insights) |

**🔥 Bonus Challenge:** Launch + terminate an instance, then find both events — including who did it and from where.

---

## 📗 Lab 23 — KMS: Encrypt S3 and EBS

| # | Question | Answer |
|---|---|---|
| Q1 | Customer-managed vs AWS-managed key? | **Customer-managed** = you control it (~$1/month); **AWS-managed** = free but limited control |
| Q2 | You delete a KMS key. What happens to the data, and when? | After a **7-day waiting period** the key is gone — and **data encrypted with it becomes unrecoverable** |
| Q3 | Why does KMS encryption feel "transparent"? | S3/EBS **encrypt/decrypt automatically** — apps use data normally, never handling keys |

**🔥 Bonus Challenge:** Enable **Default EBS Encryption** and verify a new instance's volume shows `Encrypted: Yes`.

---

## 📗 Lab 24 — AWS Backup: Multi-Service Backup

| # | Question | Answer |
|---|---|---|
| Q1 | What does a backup plan define? | **What** to back up, **when** (schedule), and **how long** to keep it (retention) |
| Q2 | Scheduled vs on-demand backup? | **Scheduled** = automatic per the plan; **on-demand** = manual "back up now" |
| Q3 | You restore a resource from a backup. What happens to the original? | **Nothing — restore is non-destructive**; AWS creates a new resource from the backup |

**🔥 Bonus Challenge:** Change a file, run an on-demand backup, restore to a NEW instance, and confirm the file is back.

---

## 📗 Lab 25 — Capstone: Full Stack on AWS

| # | Question | Answer |
|---|---|---|
| Q1 | Why is the database in a private subnet while web servers are public? | **Defense in depth** — web servers must face the internet, but the DB should only be reachable by your app layer |
| Q2 | One web server dies — which TWO services keep the app online? | The **ALB** (stops sending it traffic) and the **ASG** (launches a replacement) |
| Q3 | App is slow: which service tells you WHERE, and which tells you WHO? | **CloudWatch** = metrics & alarms (where); **CloudTrail** = audit log of API calls (who) |
| Q4 | What makes this architecture *production-like*? | **Multi-tier design, multi-AZ HA, auto-scaling, managed DB, monitoring, and audit logging** |

**🔥 Final Challenge:** Kill the primary web instance and watch ALB + ASG recover, find it in CloudTrail, then tear everything down.

---

<div align="center">

### 🏆 You've Got This, Ravi!

**Try the quiz in the lab first — then check here. That's how knowledge sticks.**

*Built with ☕ and a lot of patience — Rithu* ☁️

</div>
