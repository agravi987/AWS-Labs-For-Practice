<div align="center">

<img src="https://img.shields.io/badge/Lab%2010-Application%20Load%20Balancer-E74C3C?style=for-the-badge&labelColor=232F3E" />

</div>

<div align="center">

<img src="https://img.shields.io/badge/Difficulty-Medium-F4D03F?style=flat-square" />
<img src="https://img.shields.io/badge/Time-~40min-3498DB?style=flat-square" />
<img src="https://img.shields.io/badge/Cost-%3C2%20USD-2ECC71?style=flat-square" />
<img src="https://img.shields.io/badge/Service-EC2%20%7C%20ELB-8E44AD?style=flat-square" />

</div>

> "An Application Load Balancer is like a traffic cop at a busy intersection — it takes incoming requests and spreads them across your servers so no single one gets overwhelmed. Let's build one!" — Rithu ⚖️

---

<details>
<summary><b>🎭 Ravi & Rithu's Coffee Break Chat</b></summary>

**Ravi:** "Why can't users just hit the EC2 instances directly?"

**Rithu:** "Try telling a million users to remember three different IPs and guess which server is alive today."

**Ravi:** "...fair point."

**Rithu:** "The ALB gives them ONE address, spreads the load, and quietly retires sick servers without anyone noticing."

**Ravi:** "So it's a host who seats customers at tables only if the waiter is healthy?"

**Rithu:** "Exactly. And if a waiter collapses, the host just stops sending people to that table." 🍽️

</details>

---

## 📋 Table of Contents

- [🎯 Objective](#-objective)
- [📊 Lab Progress](#-lab-progress)
- [🤔 In Plain English](#-in-plain-english)
- [🧠 Prerequisites](#-prerequisites)
- [💰 Cost Warning](#-cost-warning)
- [🏗️ Architecture](#️-architecture)
- [🛠️ Step-by-Step Instructions](#️-step-by-step-instructions)
- [✅ Validation Checklist](#-validation-checklist)
- [🧹 Cleanup (IMPORTANT!)](#-cleanup-important)
- [🧠 Memory Tips](#-memory-tips)
- [🎓 What You Learned](#-what-you-learned)
- [🎮 Test Yourself](#-test-yourself-no-peeking-)
- [🆚 Pro Tip vs Noob Tip](#-pro-tip-vs-noob-tip)
- [🔗 What's Next?](#-whats-next)
- [❓ Troubleshooting](#-troubleshooting)

---

<div align="center">

## 📊 Lab Progress

`[██░░░░░░░░░░░░░░░░░░] 5% — Let's Begin!`

</div>

---

## 🤔 In Plain English

> **What is this, really?** An ALB is a **traffic cop with one address**. Users type one URL, and the ALB spreads each request across your servers (targets) using a rotation system (round-robin). It constantly pings the servers (health checks) and stops sending traffic to any that don't answer. 🚦
>
> 🌍 **Why you should care:** Real web apps run on multiple servers — because one server can't handle everything and can't survive a crash. The ALB is the front door that makes many servers look like one.

---

## 🎯 Objective

In this lab, you'll create an **Application Load Balancer (ALB)** that distributes incoming HTTP traffic across **two EC2 instances**. You'll set up security groups, a target group, health checks, and verify that traffic is balanced between the instances. This is how real-world web applications handle traffic!

---

## 🧠 Prerequisites

- [x] Completed [Lab 09 — VPC: NAT Gateway and VPC Endpoints](../09%20-%20VPC%20-%20NAT%20Gateway%20and%20VPC%20Endpoints/README.md)
- [x] AWS account with console access
- [x] Basic familiarity with EC2 and security groups

---

## 💰 Cost Warning

> ⚠️ **This lab costs less than $2.** You're using two t2.micro instances (free tier eligible) and an ALB for a short time. The ALB itself has a small hourly charge (~$0.0225/hour). **Delete everything when you're done — especially the ALB, which keeps running (and costing) even with no traffic!**

---

## 🏗️ Architecture

```
                         ┌─────────────┐
                         │  Internet   │
                         └──────┬──────┘
                                │
                    ┌───────────▼───────────┐
                    │  Application LB       │
                    │  ravi-alb              │
                    │  DNS: ravi-alb-xxx.elb │
                    │  .amazonaws.com        │
                    │                        │
                    │  Listener: HTTP :80    │
                    └───┬───────────────┬───┘
                        │               │
            ┌───────────▼───┐   ┌───────▼───────────┐
            │  Target 1     │   │  Target 2         │
            │  web-server-1 │   │  web-server-2     │
            │  (AZ: 1a)     │   │  (AZ: 1b)        │
            │               │   │                   │
            │  "Hello from  │   │  "Hello from      │
            │  Server 1!"   │   │   Server 2!"      │
            └───────────────┘   └───────────────────┘

Security Groups:
  alb-sg:        HTTP (80) from 0.0.0.0/0 → ALB
  ec2-from-alb-sg: HTTP (80) from alb-sg → EC2 instances
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Create Security Group for the ALB

> <img src="https://img.shields.io/badge/Step%201-Create%20ALB%20Security%20Group-3498DB?style=for-the-badge" />

The ALB needs its own security group that accepts HTTP traffic from the internet.

1. Log in to the [AWS Management Console](https://console.aws.amazon.com/)
2. In the search bar, type **EC2** and click on **EC2**
3. In the left sidebar, under **Network & Security**, click **Security Groups**
4. Click **Create security group**

Configure:
- **Security group name:** `alb-sg`
- **Description:** `Security group for Application Load Balancer`
- **VPC:** Select the **default VPC** (or your custom VPC if you prefer)

> 📸 [Screenshot: ALB security group creation]
![ ALB security group creation](screenshots/01-alb-security-group.png)

5. **Inbound rules:**
   - Click **Add rule**
   - **Type:** HTTP
   - **Port range:** 80
   - **Source:** Anywhere-IPv4 (`0.0.0.0/0`)
   - 📸 [Screenshot: Inbound rule: HTTP from 0.0.0.0/0]
   ![Inbound rule: HTTP from 0.0.0.0/0](screenshots/02-alb-inbound-http-from-anywhere.png)

6. **Outbound rules:** Allow all traffic (default — leave as is)
7. Click **Create security group**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> The ALB accepts traffic from the entire internet on port 80 (HTTP). That's fine — it's a public-facing load balancer. The important thing is that the EC2 instances behind it are more restricted!

---

### Step 2: Create Security Group for EC2 Instances

> <img src="https://img.shields.io/badge/Step%202-Create%20EC2%20Security%20Group-2ECC71?style=for-the-badge" />

The EC2 instances should ONLY accept traffic from the ALB — not directly from the internet.

1. Still on the **Security Groups** page, click **Create security group**
2. Configure:
   - **Security group name:** `ec2-from-alb-sg`
   - **Description:** `Allow HTTP only from ALB security group`
   - **VPC:** Select the **same VPC** as the ALB security group


3. **Inbound rules:**
   - Click **Add rule**
   - **Type:** HTTP
   - **Port range:** 80
   - **Source:** Select **Custom** → start typing `alb-sg` → select the security group ID of `alb-sg`
     - 📸 [Screenshot: Source field showing alb-sg security group reference]
     ![Source field showing alb-sg security group reference](screenshots/03-ec2-security-group-reference.png)

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> This is the key to ALB security! Instead of allowing HTTP from `0.0.0.0/0` on the EC2 instances, we allow it ONLY from the ALB's security group. This means:
> - ✅ ALB → EC2 (allowed)
> - ❌ Internet → EC2 directly (blocked)
>
> This is called a "security group reference" — one security group referencing another. Super powerful!

4. **Outbound rules:** Allow all traffic (default)
5. Click **Create security group**

---

### Step 3: Create a Target Group

> <img src="https://img.shields.io/badge/Step%203-Create%20Target%20Group-E74C3C?style=for-the-badge" />

A Target Group tells the ALB which instances to send traffic to.

1. In the search bar, type **EC2** → go to **EC2 Dashboard**
2. In the left sidebar, under **Load Balancing**, click **Target Groups**
3. Click **Create target group**

Configure:
- **Choose a target type:** Instances
- **Target group name:** `ravi-target-group`
- **Protocol:** HTTP
- **Port:** 80
- **VPC:** Select the **same VPC** as your security groups

> 📸 [Screenshot: Target group creation with HTTP port 80]
![Target group creation with HTTP port 80](screenshots/04-target-group-http80.png)

4. **Health checks:** Protocol HTTP, Path `/` (leave advanced settings as default)
5. Click **Next** — on the "Register targets" page, **don't register anything yet** (we'll do that after launching the instances)
6. Click **Create target group**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Health checks are how the ALB knows if your instances are healthy. It sends a request to `/` every few seconds. If the instance responds with a 200 OK, it's "healthy." If it times out or returns an error, the ALB marks it as "unhealthy" and stops sending traffic to it. Smart, right?

---

### Step 4: Launch EC2 Instance 1 (Web Server 1)

> <img src="https://img.shields.io/badge/Step%204-Launch%20Web%20Server%201-F39C12?style=for-the-badge" />

1. Go to **EC2** → **Launch instance**
2. Configure:
   - **Name:** `web-server-1`
   - **AMI:** Amazon Linux 2023
   - **Instance type:** t2.micro
   - **Key pair:** Select your existing key pair (or create one)

3. **Network settings** → **Edit:**
   - **VPC:** Same VPC as the target group and security groups
   - **Subnet:** Select a public subnet (e.g., `us-east-1a`)
   - **Auto-assign public IP:** Enable
   - **Security group:** Select existing → `ec2-from-alb-sg`

> 📸 [Screenshot: Instance 1 network settings with ec2-from-alb-sg]
![Instance 1 network settings with ec2-from-alb-sg](screenshots/05-instance1-network-settings.png)

4. **User data** — expand "Advanced details" and paste this in the **User data** field:

```bash
#!/bin/bash
dnf install -y httpd
systemctl start httpd
echo "<h1>Hello from Web Server 1! Hostname: $(hostname)</h1>" > /var/www/html/index.html
```

> 📸 [Screenshot: User data script pasted in advanced details]
![User data script pasted in advanced details](screenshots/06-user-data-script.png)

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> This script installs Apache web server and creates a simple webpage. The `$(hostname)` part shows the server's actual hostname — this lets you see WHICH server responded when you test the load balancer!

5. Click **Launch instance**
6. Wait for the instance to be **Running**

---

### Step 5: Launch EC2 Instance 2 (Web Server 2)

> <img src="https://img.shields.io/badge/Step%205-Launch%20Web%20Server%202-1ABC9C?style=for-the-badge" />

Repeat Step 4 with three changes:

- **Name:** `web-server-2`
- **Network settings** → **Subnet:** a **different** AZ if possible (e.g., `us-east-1b`)
- **User data** — same script, different page text:

```bash
#!/bin/bash
dnf install -y httpd
systemctl start httpd
echo "<h1>Hello from Web Server 2! Hostname: $(hostname)</h1>" > /var/www/html/index.html
```

Click **Launch instance**, then wait for both instances to be **Running** with status checks 2/2.

> 📸 [Screenshot: Both instances running with status checks passed]
![Both instances running with status checks passed](screenshots/07-both-instances-running.png)

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> In production, spread instances across multiple AZs — if one AZ goes down, the other keeps serving. That's why we're using us-east-1a and us-east-1b!

---

### Step 6: Register Instances in the Target Group

> <img src="https://img.shields.io/badge/Step%206-Register%20Targets-E67E22?style=for-the-badge" />

1. Go to **EC2** → **Target Groups** (left sidebar)
2. Click on `ravi-target-group`
3. Click the **Targets** tab
4. Click **Register target**
5. Select both `web-server-1` and `web-server-2`
6. Click **Include as pending below**
7. Click **Register pending targets**
8. Wait about 30-60 seconds, then refresh. Both targets should show **Health status: healthy** ✅

> 📸 [Screenshot: Both targets showing "healthy" status]
![Both targets showing "healthy" status](screenshots/10-targets-healthy.png)

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> If a target shows "unhealthy," it usually means:
> - The web server didn't start properly (check your user data script)
> - The security group blocks port 80 from the ALB
> - The health check path `/` doesn't return a 200 response
>
> Give it at least 30 seconds — health checks take a moment!

---

### Step 7: Create the Application Load Balancer

> <img src="https://img.shields.io/badge/Step%207-Create%20ALB-9B59B6?style=for-the-badge" />

Now for the main event! 🎉

1. Go to **EC2** → **Load Balancers** (left sidebar)
2. Click **Create Load Balancer**
3. Select **Application Load Balancer**

Configure:
- **Load balancer name:** `ravi-alb`
- **Scheme:** Internet-facing
- **IP address type:** IPv4

> 📸 [Screenshot: ALB basic configuration]
![ALB basic configuration](screenshots/08-alb-basic-configuration.png)

4. **Network mapping:**
   - **VPC:** Select the same VPC as everything else
   - **Mappings:**
     - Check **us-east-1a** → select a public subnet
     - Check **us-east-1b** → select a public subnet

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> The ALB MUST be in at least two AZs. This is a requirement, not a suggestion. If one AZ goes down, the ALB continues working from the other AZ. That's the whole point!

5. **Security groups:**
   - Remove the default security group
   - Select `alb-sg`
   - 📸 [Screenshot: ALB security group set to alb-sg]

6. **Listeners and routing:**
   - **Protocol:** HTTP
   - **Port:** 80
   - **Default action:** Forward to → select `ravi-target-group`
   - 📸 [Screenshot: Listener forwarding to ravi-target-group]

7. Leave all other settings as default
8. Click **Create load balancer**
9. Wait for the ALB state to change to **Active** (this takes 2-5 minutes)
> 📸 [Screenshot: ALB created]
![ALB created](screenshots/09-alb-created.png)

---

### Step 8: Verify — Watch the Magic Happen! 🎉

> <img src="https://img.shields.io/badge/Step%208-Verify%20Load%20Balancing-27AE60?style=for-the-badge" />

1. Click on `ravi-alb` in the Load Balancers list
2. Find the **DNS name** in the details panel — it looks something like:
   ```
   ravi-alb-1234567890.us-east-1.elb.amazonaws.com
   ```
3. **Copy** the DNS name

> 📸 [Screenshot: ALB DNS name in the details panel]

4. **Open your web browser** and paste the DNS name:
   ```
   http://ravi-alb-1234567890.us-east-1.elb.amazonaws.com
   ```
5. You should see:
   ```
   Hello from Web Server 1! Hostname: ip-10-0-1-xxx
   ```
   or
   ```
   Hello from Web Server 2! Hostname: ip-10-0-2-xxx
   ```

6. **REFRESH the page 5-10 times!** 👀
7. You should see the response **alternate between Server 1 and Server 2**!

> 📸 [Screenshot: Browser showing "Hello from Web Server 1!" after refresh]
![ Browser showing "Hello from Web Server 1!" after refresh](screenshots/11-browser-web-server-1.png)
> 📸 [Screenshot: Browser showing "Hello from Web Server 2!" after another refresh]
![Browser showing "Hello from Web Server 2!" after another refresh](screenshots/12-browser-web-server-2.png)

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> 🎉 Congratulations, Ravi! You just set up load balancing! The ALB is distributing requests between your two servers using a round-robin algorithm. In the real world, this means:
> - If one server crashes, the ALB sends all traffic to the healthy one
> - If you get a traffic spike, you can add more servers behind the ALB
> - Users always get a response, even during deployments or maintenance

---

### Step 9: Check Target Group Health

> <img src="https://img.shields.io/badge/Step%209-Check%20Health-3498DB?style=for-the-badge" />

1. Go to **EC2** → **Target Groups**
2. Click on `ravi-target-group`
3. Click the **Targets** tab
4. Both targets should show:
   - **Status:** ✅ healthy
   - **Description:** Target registration succeeded

> 📸 [Screenshot: Target group showing both targets as healthy]
![Target group showing both targets as healthy](screenshots/13-target-group-healthy.png)

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> The ALB health checks targets every 30 seconds by default. After 3 consecutive failures a target is marked unhealthy and traffic stops; when it recovers, it's added back automatically. Self-healing infrastructure!

---

### Step 10: Verify Your Work

> <img src="https://img.shields.io/badge/Step%2010-Verify%20Your%20Work-2ECC71?style=for-the-badge" />

- [ ] ALB `ravi-alb` is in **Active** state
- [ ] ALB has an **internet-facing** DNS name
- [ ] Both EC2 instances are **healthy** in the target group
- [ ] Pasting the ALB DNS name in a browser shows a webpage
- [ ] Refreshing the page alternates between "Web Server 1" and "Web Server 2"
- [ ] Security group `ec2-from-alb-sg` only accepts HTTP from `alb-sg`

---

## ✅ Validation Checklist

| # | ✅ Check | Status |
|---|-------|--------|
| 1 | Security group `alb-sg` with HTTP from 0.0.0.0/0 | ☐ ✅ |
| 2 | Security group `ec2-from-alb-sg` with HTTP from alb-sg only | ☐ ✅ |
| 3 | Target group `ravi-target-group` created with health checks | ☐ ✅ |
| 4 | `web-server-1` running with Apache in public subnet | ☐ ✅ |
| 5 | `web-server-2` running with Apache in different AZ | ☐ ✅ |
| 6 | Both instances registered and healthy in target group | ☐ ✅ |
| 7 | ALB `ravi-alb` active and internet-facing | ☐ ✅ |
| 8 | ALB DNS name loads the webpage in browser | ☐ ✅ |
| 9 | Page alternates between Server 1 and Server 2 on refresh | ☐ ✅ |

---

## 🧹 Cleanup (IMPORTANT!)

> ⚠️ **Delete everything to stop all charges!** The ALB and EC2 instances will keep running (and costing) until you delete them.

### 🗑️ Step 1: Delete the Application Load Balancer

> <img src="https://img.shields.io/badge/Step%201-Delete%20ALB-E74C3C?style=for-the-badge" />

1. Go to **EC2** → **Load Balancers**
2. Select `ravi-alb`
3. Click **Actions** → **Delete load balancer**
4. Type `confirm` and click **Delete**

> 📸 [Screenshot: ALB deletion confirmed]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Delete the ALB first — it has dependencies. If you try to delete the target group first, AWS will complain that it's still in use by the load balancer!

### 🗑️ Step 2: Delete the Target Group

> <img src="https://img.shields.io/badge/Step%202-Delete%20Target%20Group-F39C12?style=for-the-badge" />

1. Go to **EC2** → **Target Groups**
2. Select `ravi-target-group`
3. Click **Actions** → **Delete**
4. Confirm

### 🗑️ Step 3: Terminate Both EC2 Instances

> <img src="https://img.shields.io/badge/Step%203-Terminate%20EC2%20Instances-2ECC71?style=for-the-badge" />

1. Go to **EC2** → **Instances**
2. Select `web-server-1` and `web-server-2`
3. Click **Instance state** → **Terminate instance**
4. Click **Terminate**

### 🗑️ Step 4: Delete Both Security Groups

> <img src="https://img.shields.io/badge/Step%204-Delete%20Security%20Groups-9B59B6?style=for-the-badge" />

1. Go to **EC2** → **Security Groups**
2. Select `alb-sg` → **Actions** → **Delete security groups** → confirm
3. Select `ec2-from-alb-sg` → **Actions** → **Delete security groups** → confirm

> 📸 [Screenshot: Clean EC2 console — no load balancers, instances, or unnecessary security groups]

---

## 🧠 Memory Tips

Stick these in your brain and they'll never leave. 🧲

| 🧠 Memory Hook | Remember it like... |
|---|---|
| **ALB = traffic cop** | One address, many servers, **round-robin** turns. 🚦 |
| **Target group = guest list** | The list of instances the ALB is *allowed* to send traffic to. 📋 |
| **Health check = waiter check-in** | "Are you alive? Still serving?" — failing targets get **removed automatically**. 💔→✅ |
| **Single DNS name** | Users never see your servers — they only know the ALB's address. One front door. 🚪 |
| **Multi-AZ = two escape routes** | Instances spread across AZs; one zone dies, the other keeps serving. 🏙️ |

> 🗣️ **Rithu:** *"If a server fails and the site keeps working — that's not luck. That's the ALB doing its job. Appreciate the traffic cop."*

---

## 🎓 What You Learned

- **Application Load Balancers** distribute HTTP/HTTPS traffic across multiple EC2 instances
- **Target Groups** define which instances the ALB sends traffic to
- **Health checks** automatically detect and remove unhealthy instances
- **Security group references** allow traffic only from the ALB, not directly from the internet
- **Round-robin routing** alternates requests between healthy targets
- **Multi-AZ deployment** ensures high availability — if one AZ goes down, the other keeps working
- The ALB provides a **single DNS name** that clients use — they don't need to know about individual servers
- ALBs handle **SSL/TLS termination**, **path-based routing**, and **host-based routing** (advanced topics for later!)

---

## 🎮 Test Yourself! (No Peeking 👀)

**Q1:** What does a health check actually do?

<details><summary>👀 Show answer</summary>

**A:** It **pings each target** (e.g., HTTP GET on `/`) and if the target fails enough checks, the ALB **stops sending it traffic** — and marks it unhealthy. Sick servers get benched automatically. 💔

</details>

**Q2:** At which OSI layer does an Application Load Balancer work?

<details><summary>👀 Show answer</summary>

**A:** **Layer 7** (HTTP/HTTPS) — it understands URLs, headers, and paths. That's why it can do path-based routing later (like `/api` vs `/web`). 📡

</details>

**Q3:** Why run instances in multiple Availability Zones behind the ALB?

<details><summary>👀 Show answer</summary>

**A:** **High availability** — if one AZ has an outage, the other AZ's instances keep serving. No single point of failure. 🏙️

</details>

### 🔥 Bonus Challenge

**Stop one of your two instances** (not terminate — just stop). Refresh your ALB URL repeatedly. Every request still works — all served by the survivor. Then start the first instance back up and watch traffic rebalance. You just witnessed self-healing. 🩹

> 💪 **Rithu:** *"An ALB without a broken server to test against is an unproven ALB. Break one on purpose!"

---

### 🆚 Pro Tip vs Noob Tip

| | Approach |
|---|---|
| **Noob Tip** | Hand out instance IPs directly and hope nobody dies |
| **Pro Tip** | Always front apps with an ALB: one DNS name, health checks, ready to scale |

---

## 🔗 What's Next?

You've covered S3, VPC, and Load Balancing! Here are some ideas for your next labs:

- ⚙️ **Lab 11 — Auto Scaling Groups** (automatically add/remove EC2 instances based on traffic)
- 🌐 **Lab 12 — Route 53: DNS and Failover** (DNS and automatic failover)
- 🗄️ **Lab 13 — RDS: Relational Database Service** (managed databases!)
- 📊 **Lab 14 — DynamoDB: CRUD Operations** (NoSQL tables in the cloud)

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> You've come a long way, Ravi! From creating your first S3 bucket to building a load-balanced web application. Keep going — you're building real cloud engineering skills! 🚀

---

## ❓ Troubleshooting

<details>
<summary><strong>Click to expand Troubleshooting Table</strong></summary>

| 🔍 Problem | 💡 Solution |
|---------|----------|
| 🔧 ALB shows "Provisioning" for a long time | This is normal — ALBs take 2-5 minutes to provision. Be patient! |
| 🔧 Browser shows "This site can't be reached" | Check: (1) ALB state is Active, (2) You're using HTTP (not HTTPS), (3) You copied the full DNS name |
| 🔧 Page loads but shows one server only | Refresh multiple times. If still one server, check the other instance's health status |
| 🔧 Target shows "unhealthy" | Check: (1) Apache is running (user data script worked), (2) EC2 security group allows port 80 FROM alb-sg, (3) Wait 60 seconds for re-check |
| 🔧 "No instances registered" in target group | Go to Target Group → Targets tab → Register targets manually |
| 🔧 "Invalid security group" error during ALB creation | Make sure alb-sg and ec2-from-alb-sg are in the same VPC as the ALB |
| 🔧 User data didn't install Apache | SSH into the instance and check: `systemctl status httpd`. If not running, try running the commands manually |
| 🔧 Can't delete target group | Make sure the ALB is deleted first — the target group is still attached |
| 🔧 Security group won't delete | Make sure it's not attached to any instances or the ALB |

</details>

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> The most common issue is the security group configuration. Double-check:
> 1. `alb-sg` allows HTTP (80) from `0.0.0.0/0`
> 2. `ec2-from-alb-sg` allows HTTP (80) from `alb-sg` ONLY
>
> If the EC2 security group doesn't reference `alb-sg` as the source, health checks will fail! 🔍

---

<div align="center">

<img src="https://img.shields.io/badge/Lab%2010-Complete!-27AE60?style=for-the-badge&labelColor=232F3E" />

</div>
