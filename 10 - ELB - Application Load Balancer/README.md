# Lab 10 — ELB: Application Load Balancer

![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow)
![Time](https://img.shields.io/badge/Time-~40min-blue)
![Cost](https://img.shields.io/badge/Cost-<2%20USD-green)
![Service](https://img.shields.io/badge/Service-EC2%20|%20ELB-blue)

> "An Application Load Balancer is like a traffic cop at a busy intersection — it takes incoming requests and spreads them across your servers so no single one gets overwhelmed. Let's build one!" — Rithu ⚖️

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

5. **Inbound rules:**
   - Click **Add rule**
   - **Type:** HTTP
   - **Port range:** 80
   - **Source:** Anywhere-IPv4 (`0.0.0.0/0`)
   - 📸 [Screenshot: Inbound rule: HTTP from 0.0.0.0/0]

6. **Outbound rules:** Allow all traffic (default — leave as is)
7. Click **Create security group**

> 💡 **Rithu's Tip:** The ALB accepts traffic from the entire internet on port 80 (HTTP). That's fine — it's a public-facing load balancer. The important thing is that the EC2 instances behind it are more restricted!

---

### Step 2: Create Security Group for EC2 Instances

The EC2 instances should ONLY accept traffic from the ALB — not directly from the internet.

1. Still on the **Security Groups** page, click **Create security group**
2. Configure:
   - **Security group name:** `ec2-from-alb-sg`
   - **Description:** `Allow HTTP only from ALB security group`
   - **VPC:** Select the **same VPC** as the ALB security group

> 📸 [Screenshot: EC2 security group creation]

3. **Inbound rules:**
   - Click **Add rule**
   - **Type:** HTTP
   - **Port range:** 80
   - **Source:** Select **Custom** → start typing `alb-sg` → select the security group ID of `alb-sg`
     - 📸 [Screenshot: Source field showing alb-sg security group reference]

> 💡 **Rithu's Tip:** This is the key to ALB security! Instead of allowing HTTP from `0.0.0.0/0` on the EC2 instances, we allow it ONLY from the ALB's security group. This means:
> - ✅ ALB → EC2 (allowed)
> - ❌ Internet → EC2 directly (blocked)
>
> This is called a "security group reference" — one security group referencing another. Super powerful!

4. **Outbound rules:** Allow all traffic (default)
5. Click **Create security group**

---

### Step 3: Create a Target Group

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

4. **Health checks:**
   - **Protocol:** HTTP
   - **Path:** `/`
     - 📸 [Screenshot: Health check path set to /]
5. **Advanced health check settings:** Leave as default
6. Click **Next**
7. On the "Register targets" page — **DON'T register any targets yet!** We'll do that after launching the instances.
8. Click **Create target group**

> 💡 **Rithu's Tip:** Health checks are how the ALB knows if your instances are healthy. It sends a request to `/` every few seconds. If the instance responds with a 200 OK, it's "healthy." If it times out or returns an error, the ALB marks it as "unhealthy" and stops sending traffic to it. Smart, right?

---

### Step 4: Launch EC2 Instance 1 (Web Server 1)

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

4. **User data** — expand "Advanced details" and paste this in the **User data** field:

```bash
#!/bin/bash
yum install -y httpd
systemctl start httpd
echo "<h1>Hello from Web Server 1! Hostname: $(hostname)</h1>" > /var/www/html/index.html
```

> 📸 [Screenshot: User data script pasted in advanced details]

> 💡 **Rithu's Tip:** This script installs Apache web server and creates a simple webpage. The `$(hostname)` part shows the server's actual hostname — this lets you see WHICH server responded when you test the load balancer!

5. Click **Launch instance**
6. Wait for the instance to be **Running**

---

### Step 5: Launch EC2 Instance 2 (Web Server 2)

1. **Launch instance** again
2. Configure:
   - **Name:** `web-server-2`
   - **AMI:** Amazon Linux 2023
   - **Instance type:** t2.micro
   - **Key pair:** Same as before

3. **Network settings** → **Edit:**
   - **VPC:** Same VPC
   - **Subnet:** Select a **different** subnet if possible (e.g., `us-east-1b`)
     - 📸 [Screenshot: Instance 2 in us-east-1b subnet]
   - **Auto-assign public IP:** Enable
   - **Security group:** Select existing → `ec2-from-alb-sg`

4. **User data** — paste this:

```bash
#!/bin/bash
yum install -y httpd
systemctl start httpd
echo "<h1>Hello from Web Server 2! Hostname: $(hostname)</h1>" > /var/www/html/index.html
```

5. Click **Launch instance**
6. Wait for both instances to be **Running** and pass status checks (2/2)

> 📸 [Screenshot: Both instances running with status checks passed]

> 💡 **Rithu's Tip:** In production, you'd launch instances in different Availability Zones (AZs) for high availability. If one AZ goes down, the other keeps serving traffic. We're doing that by using us-east-1a and us-east-1b!

---

### Step 6: Register Instances in the Target Group

1. Go to **EC2** → **Target Groups** (left sidebar)
2. Click on `ravi-target-group`
3. Click the **Targets** tab
4. Click **Register target**
5. Select both `web-server-1` and `web-server-2`
6. Click **Include as pending below**
7. Click **Register pending targets**

> 📸 [Screenshot: Both instances registered as "pending" in target group]

8. Wait about 30-60 seconds, then refresh. Both targets should show **Health status: healthy** ✅

> 📸 [Screenshot: Both targets showing "healthy" status]

> 💡 **Rithu's Tip:** If a target shows "unhealthy," it usually means:
> - The web server didn't start properly (check your user data script)
> - The security group blocks port 80 from the ALB
> - The health check path `/` doesn't return a 200 response
>
> Give it at least 30 seconds — health checks take a moment!

---

### Step 7: Create the Application Load Balancer

Now for the main event! 🎉

1. Go to **EC2** → **Load Balancers** (left sidebar)
2. Click **Create Load Balancer**
3. Select **Application Load Balancer**

Configure:
- **Load balancer name:** `ravi-alb`
- **Scheme:** Internet-facing
- **Type:** Application Load Balancer
- **IP address type:** IPv4

> 📸 [Screenshot: ALB basic configuration]

4. **Network mapping:**
   - **VPC:** Select the same VPC as everything else
   - **Mappings:**
     - Check **us-east-1a** → select a public subnet
     - Check **us-east-1b** → select a public subnet
     - 📸 [Screenshot: ALB mapped to both AZs with public subnets]

> 💡 **Rithu's Tip:** The ALB MUST be in at least two AZs. This is a requirement, not a suggestion. If one AZ goes down, the ALB continues working from the other AZ. That's the whole point!

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

> 📸 [Screenshot: ALB creation in progress]

9. Wait for the ALB state to change to **Active** (this takes 2-5 minutes)

---

### Step 8: Verify — Watch the Magic Happen! 🎉

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
> 📸 [Screenshot: Browser showing "Hello from Web Server 2!" after another refresh]

> 💡 **Rithu's Tip:** 🎉 Congratulations, Ravi! You just set up load balancing! The ALB is distributing requests between your two servers using a round-robin algorithm. In the real world, this means:
> - If one server crashes, the ALB sends all traffic to the healthy one
> - If you get a traffic spike, you can add more servers behind the ALB
> - Users always get a response, even during deployments or maintenance

---

### Step 9: Check Target Group Health

1. Go to **EC2** → **Target Groups**
2. Click on `ravi-target-group`
3. Click the **Targets** tab
4. Both targets should show:
   - **Status:** ✅ healthy
   - **Description:** Target registration succeeded

> 📸 [Screenshot: Target group showing both targets as healthy]

> 💡 **Rithu's Tip:** The ALB health checks every 30 seconds by default. If an instance fails 2 consecutive health checks, it's marked unhealthy and the ALB stops sending traffic to it. When it recovers, the ALB automatically adds it back. Self-healing infrastructure!

---

### Step 10: Verify Your Work

- [ ] ALB `ravi-alb` is in **Active** state
- [ ] ALB has an **internet-facing** DNS name
- [ ] Both EC2 instances are **healthy** in the target group
- [ ] Pasting the ALB DNS name in a browser shows a webpage
- [ ] Refreshing the page alternates between "Web Server 1" and "Web Server 2"
- [ ] Security group `ec2-from-alb-sg` only accepts HTTP from `alb-sg`

---

## ✅ Validation Checklist

| # | Check | Status |
|---|-------|--------|
| 1 | Security group `alb-sg` with HTTP from 0.0.0.0/0 | ☐ |
| 2 | Security group `ec2-from-alb-sg` with HTTP from alb-sg only | ☐ |
| 3 | Target group `ravi-target-group` created with health checks | ☐ |
| 4 | `web-server-1` running with Apache in public subnet | ☐ |
| 5 | `web-server-2` running with Apache in different AZ | ☐ |
| 6 | Both instances registered and healthy in target group | ☐ |
| 7 | ALB `ravi-alb` active and internet-facing | ☐ |
| 8 | ALB DNS name loads the webpage in browser | ☐ |
| 9 | Page alternates between Server 1 and Server 2 on refresh | ☐ |

---

## 🧹 Cleanup (IMPORTANT!)

> ⚠️ **Delete everything to stop all charges!** The ALB and EC2 instances will keep running (and costing) until you delete them.

### Step 1: Delete the Application Load Balancer

1. Go to **EC2** → **Load Balancers**
2. Select `ravi-alb`
3. Click **Actions** → **Delete load balancer**
4. Type `confirm` and click **Delete**

> 📸 [Screenshot: ALB deletion confirmed]

> 💡 **Rithu's Tip:** Delete the ALB first — it has dependencies. If you try to delete the target group first, AWS will complain that it's still in use by the load balancer!

### Step 2: Delete the Target Group

1. Go to **EC2** → **Target Groups**
2. Select `ravi-target-group`
3. Click **Actions** → **Delete**
4. Confirm

### Step 3: Terminate Both EC2 Instances

1. Go to **EC2** → **Instances**
2. Select `web-server-1` and `web-server-2`
3. Click **Instance state** → **Terminate instance**
4. Click **Terminate**

### Step 4: Delete Both Security Groups

1. Go to **EC2** → **Security Groups**
2. Select `alb-sg` → **Actions** → **Delete security groups** → confirm
3. Select `ec2-from-alb-sg` → **Actions** → **Delete security groups** → confirm

> 📸 [Screenshot: Clean EC2 console — no load balancers, instances, or unnecessary security groups]

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

## 🔗 What's Next?

You've covered S3, VPC, and Load Balancing! Here are some ideas for your next labs:

- 🔒 **Lab 11 — HTTPS with ALB** (add an SSL certificate with ACM)
- 📊 **Lab 12 — Auto Scaling Groups** (automatically add/remove EC2 instances based on traffic)
- 🗄️ **Lab 13 — RDS: Relational Database Service** (managed databases!)
- 🐳 **Lab 14 — ECS: Containers on AWS** (Docker containers in the cloud)

> 💡 **Rithu's Tip:** You've come a long way, Ravi! From creating your first S3 bucket to building a load-balanced web application. Keep going — you're building real cloud engineering skills! 🚀

---

## ❓ Troubleshooting

| Problem | Solution |
|---------|----------|
| ALB shows "Provisioning" for a long time | This is normal — ALBs take 2-5 minutes to provision. Be patient! |
| Browser shows "This site can't be reached" | Check: (1) ALB state is Active, (2) You're using HTTP (not HTTPS), (3) You copied the full DNS name |
| Page loads but shows one server only | Refresh multiple times. If still one server, check the other instance's health status |
| Target shows "unhealthy" | Check: (1) Apache is running (user data script worked), (2) EC2 security group allows port 80 FROM alb-sg, (3) Wait 60 seconds for re-check |
| "No instances registered" in target group | Go to Target Group → Targets tab → Register targets manually |
| "Invalid security group" error during ALB creation | Make sure alb-sg and ec2-from-alb-sg are in the same VPC as the ALB |
| User data didn't install Apache | SSH into the instance and check: `systemctl status httpd`. If not running, try running the commands manually |
| Can't delete target group | Make sure the ALB is deleted first — the target group is still attached |
| Security group won't delete | Make sure it's not attached to any instances or the ALB |

> 💡 **Rithu's Tip:** The most common issue is the security group configuration. Double-check:
> 1. `alb-sg` allows HTTP (80) from `0.0.0.0/0`
> 2. `ec2-from-alb-sg` allows HTTP (80) from `alb-sg` ONLY
>
> If the EC2 security group doesn't reference `alb-sg` as the source, health checks will fail! 🔍
