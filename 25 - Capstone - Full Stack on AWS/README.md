<div align="center">

<img src="https://img.shields.io/badge/Lab%2025-Capstone%20Full%20Stack-E74C3C?style=for-the-badge&labelColor=232F3E" />

<br/>

# Lab 25 — Capstone: Full Stack on AWS

![Difficulty](https://img.shields.io/badge/Difficulty-Hard-red?style=for-the-badge&labelColor=232F3E)
![Time](https://img.shields.io/badge/Time-~90min-purple?style=for-the-badge&labelColor=232F3E)
![Cost](https://img.shields.io/badge/Cost-%3C%245-orange?style=for-the-badge&labelColor=232F3E)
![Service](https://img.shields.io/badge/Service-Multiple%20Services-blueviolet?style=for-the-badge&labelColor=232F3E)

> "This is it, Ravi. Everything you've learned, everything you've built — it all comes together now. Let's build something real." — Rithu

</div>

---

<details>
<summary><b>🎭 Ravi & Rithu's Coffee Break Chat</b></summary>

**Ravi:** "We're actually building a FULL application?"

**Rithu:** "The whole enchilada! VPC, EC2, RDS, S3, ALB, Auto Scaling, CloudWatch, CloudTrail."

**Ravi:** "That's... everything we've learned."

**Rithu:** "That's the point. You've leveled up. Now show me what you've got!"

</details>

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
- [🔗 What's Next?](#-whats-next)

---

<div align="center">

## 📊 Lab Progress

`[███░░░░░░░░░░░░░░░░░] 8% — The Final Boss Awaits!`

</div>

---

## 🤔 In Plain English

> **What is this, really?** The Capstone is **everything you've learned, assembled into one real application**. A VPC with public/private subnets, EC2 servers behind an ALB, an Auto Scaling Group, an RDS database, S3 for storage, CloudWatch watching it all, and CloudTrail logging everything. It's the same architecture real companies run — built by you, from scratch. 🏗️
>
> 🌍 **Why you should care:** This is your **portfolio piece**. When someone asks "what have you built on AWS?", this is the answer. Take your time — you've earned the right to be proud of this one.

---

## 🎯 Objective

By the end of this capstone lab, you will:
- Design and deploy a complete multi-tier application architecture on AWS
- Integrate VPC, EC2, RDS, S3, ALB, Auto Scaling, CloudWatch, and CloudTrail
- Experience a real-world deployment from infrastructure to monitoring
- Document and clean up a production-like environment

This is your **final exam and celebration**. You'll build a full-stack web application that ties together every skill from Labs 01-24. Take your time, enjoy the process, and be proud of how far you've come!

**Architecture Overview:**
```
Route 53 → ALB → EC2 (Auto Scaling) → RDS (MySQL) → S3 (static assets) → CloudWatch (monitoring) → CloudTrail (auditing)
```

---

## 🧠 Prerequisites

- [ ] Completed Labs 01-24 (all previous labs)
- [ ] Solid understanding of VPC, EC2, RDS, S3, Security Groups, and IAM
- [ ] AWS Console access with administrator permissions
- [ ] At least 90 minutes of uninterrupted time
- [ ] A notepad to track resource names (you'll create many!)

---

## 💰 Cost Warning

This lab uses **multiple AWS services**. Estimated costs:

| Service | Estimated Cost |
|---------|---------------|
| EC2 (2x t2.micro, ~1.5 hours) | ~$0.03 |
| RDS (db.t3.micro, ~1.5 hours) | ~$0.05 |
| ALB (Application Load Balancer) | ~$0.02 |
| NAT Gateway (~1.5 hours) | ~$0.04 |
| S3 (minimal storage) | ~$0.01 |
| CloudWatch (dashboard + alarms) | ~$0.01 |
| Data transfer (minimal) | ~$0.01 |
| **Total estimated** | **~$3-5** |

> <img src="https://img.shields.io/badge/Warning-STOP!-E74C3C?style=flat-square" /> **CRITICAL WARNING**: This lab creates MANY resources across multiple services. **You MUST follow the cleanup instructions at the end** or you WILL incur ongoing charges. The NAT Gateway is particularly expensive (~$0.045/hr = ~$32/month if left running). **CLEAN UP EVERYTHING!**

> **Ravi's Mistake of the Day:** In the capstone, I launched everything and cleaned up in the wrong order. Tried to delete the VPC before the NAT Gateway. AWS said "nah" and I spent 45 minutes untangling dependencies. Follow. The. Cleanup. Order.

---

## 🏗️ Architecture

```
                              ┌─────────────┐
                              │  Route 53    │
                              │  (DNS)       │
                              └──────┬──────┘
                                     │
                              ┌──────▼──────┐
                              │  ALB         │
                              │  (Load       │
                              │   Balancer)  │
                              └──────┬──────┘
                                     │
                    ┌────────────────┼────────────────┐
                    │                │                 │
             ┌──────▼──────┐  ┌─────▼───────┐        │
             │  Public      │  │  Public      │        │
             │  Subnet      │  │  Subnet      │        │
             │  10.0.1.0/24 │  │  10.0.2.0/24│        │
             │  (us-east-1a)│  │  (us-east-1b)│        │
             └──────┬──────┘  └─────┬───────┘        │
                    │                │                 │
             ┌──────▼──────┐  ┌─────▼───────┐        │
             │  EC2         │  │  EC2         │        │
             │  (Auto       │  │  (Auto       │        │
             │   Scaling)   │  │   Scaling)   │        │
             └──────┬──────┘  └─────┬───────┘        │
                    │                │                 │
                    └────────────────┼────────────────┘
                                     │
                              ┌──────▼──────┐
                              │  Private     │
                              │  Subnet      │
                              │  (RDS)       │
                              └──────┬──────┘
                                     │
                              ┌──────▼──────┐
                               │  RDS MySQL   │
                               │  (Multi-AZ)  │
                               └─────────────┘
```

> **Did You Know?** The capstone architecture you're building mirrors what companies like Netflix, Airbnb, and Spotify use at scale. The difference? They run millions of instances. You're running 2. But the architecture is the same!

---

## 🛠️ Step-by-Step Instructions

### Step 1: Create VPC

<img src="https://img.shields.io/badge/Step%201-Create%20VPC-27AE60?style=for-the-badge" />

Let's build the network foundation for our full-stack app!

1. Go to **VPC** in the AWS Console
2. Click **Your VPCs** in the left sidebar
3. Click **Create VPC** (top right)
4. Select **VPC and more** (the visual option)

**Configure the VPC:**

5. **Name tag auto-generation**: Type `ravi-capstone-vpc`
6. **IPv4 CIDR block**: `10.0.0.0/16`
7. **Number of AZs**: Select **2**
8. **Number of public subnets**: Select **2**
9. **Number of private subnets**: Select **2**
10. **NAT gateways**: Select **1 per AZ** (we only need 1 for this lab)
11. **VPC endpoints**: Select **None**
12. **DNS hostnames**: Check ✅ (should be enabled by default)

You should see a visual showing:
- VPC: `10.0.0.0/16`
- Public subnets: `10.0.1.0/24` (us-east-1a), `10.0.2.0/24` (us-east-1b)
- Private subnets: `10.0.3.0/24` (us-east-1a), `10.0.4.0/24` (us-east-1b)
- Internet Gateway
- NAT Gateway (in public subnet)
- Route tables

13. Click **Create VPC**


**Wait for the VPC to be created**, then verify:
14. Go to **Your VPCs** → You should see `ravi-capstone-vpc`
15. Go to **Subnets** → You should see 4 subnets (2 public, 2 private)
16. Go to **Internet Gateways** → You should see one attached to the VPC
17. Go to **NAT Gateways** → You should see one in a public subnet

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" /> "The VPC is your virtual data center. Everything we build from here lives inside this VPC. The public subnets can talk to the internet; the private subnets are isolated. This is how real production architectures work!"

📸 [Screenshot: VPC console showing the visual topology with all subnets, IGW, and NAT Gateway]

---

### Step 2: Create S3 Bucket for Static Assets

<img src="https://img.shields.io/badge/Step%202-Create%20S3%20Bucket-27AE60?style=for-the-badge" />

Our application needs a place to store static files (CSS, images, logos).

1. Go to **S3** in the AWS Console
2. Click **Create bucket**

**Configure the bucket:**

3. **Bucket name**: Type `ravi-capstone-assets-12345` (add random numbers — must be globally unique)
4. **AWS Region**: Same region as your VPC (e.g., us-east-1)
5. **Block Public Access settings**: Uncheck **Block all public access** (we need the bucket accessible for static assets)
6. Check ✅ the acknowledgment that says you understand the bucket will be public
7. Under **Default encryption**:
   - Check ✅ **Enable**
   - **Encryption key type**: Select **Server-side encryption with AWS KMS keys**
   - **AWS KMS key**: Select **aws/s3** (S3 managed key — free)
8. Click **Create bucket**

**Upload test files:**

9. Click on the bucket name
10. Click **Upload** → **Add files**
11. Create a simple text file called `style.css` with this content:
    ```css
    body { font-family: Arial; background-color: #f0f0f0; }
    h1 { color: #333; }
    ```
12. Upload the file
13. Click **Upload** → **Close**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" /> "In a real application, you'd store images, CSS, JavaScript, and fonts in S3. You can even configure S3 for static website hosting — serving HTML/CSS/JS directly from S3 without any server!"

📸 [Screenshot: S3 bucket with the style.css file uploaded and encryption enabled]

---

### Step 3: Create RDS MySQL

<img src="https://img.shields.io/badge/Step%203-Create%20RDS%20MySQL-27AE60?style=for-the-badge" />

Let's create the database for our application!

1. Go to **RDS** in the AWS Console
2. Click **Create database**

**Configure the database:**

3. **Creation method**: Standard create
4. **Engine type**: MySQL
5. **Engine version**: Latest Free Tier eligible version
6. **Templates**: Free tier
7. **DB instance identifier**: Type `ravi-capstone-db`
8. **Master username**: Type `admin`
9. **Master password**: Create a strong password (write it down!)
10. **Confirm password**: Re-enter

**Configure storage:**

11. **Storage type**: gp3
12. **Allocated storage**: 20 GB
13. **Storage autoscaling**: Uncheck (stay in Free Tier)

**Configure connectivity:**

14. **VPC**: Select `ravi-capstone-vpc`
15. **DB subnet group**: Create new → name it `ravi-capstone-subnet-group`
    - Select both **private** subnets (10.0.3.0/24 and 10.0.4.0/24)
16. **Public access**: No
17. **Security group**: Create new → name it `ravi-rds-sg`
18. **Availability Zone**: Select first AZ (us-east-1a)

**Configure additional settings:**

19. **Initial database name**: Type `capstone_app`
20. **Backup**: Uncheck **Enable automated backups** (for this lab — saves time on deletion)
21. **Deletion protection**: Uncheck (so we can delete it easily)

22. Click **Create database**
23. Wait 5-10 minutes for the RDS instance to be **Available**

**After the RDS instance is available:**

24. Click on `ravi-capstone-db`
25. Note the **Endpoint** (something like `ravi-capstone-db.xxxx.us-east-1.rds.amazonaws.com`)
26. Go to the **Security groups** link → click on `ravi-rds-sg`
27. Click **Edit inbound rules**
28. Click **Add rule**
29. **Type**: MySQL/Aurora
30. **Port**: 3306
31. **Source**: Select **Custom** → type the security group ID of `ravi-ec2-sg` (we'll create this next — if it doesn't exist yet, add `10.0.0.0/16` temporarily)
32. Click **Save rules**

**Connect to RDS and set up the database:**

33. SSH into one of your EC2 instances (or launch a temporary one)
34. Install MySQL client:
    ```bash
    sudo dnf install -y mariadb105
    ```
    > 💡 On Amazon Linux 2023, the `mysql` package isn't available. `mariadb105` provides the `mysql` command and is fully compatible with RDS MySQL.
35. Connect to the database:
    ```bash
    mysql -h ravi-capstone-db.xxxx.us-east-1.rds.amazonaws.com -u admin -p
    ```
36. Create a table and insert data:
    ```sql
    USE capstone_app;

    CREATE TABLE users (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(100) NOT NULL,
        email VARCHAR(100) NOT NULL,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    );

    INSERT INTO users (name, email) VALUES
        ('Ravi', 'ravi@example.com'),
        ('Rithu', 'rithu@example.com'),
        ('AWS Learner', 'learner@example.com');

    SELECT * FROM users;
    EXIT;
    ```

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" /> "The RDS instance is in a private subnet — it has NO public IP. Only EC2 instances in the VPC can connect to it. This is a critical security pattern: databases should never be directly accessible from the internet!"

📸 [Screenshot: RDS instance available, security group configured, table created with data]

---

### Step 4: Create Security Groups

<img src="https://img.shields.io/badge/Step%204-Create%20Security%20Groups-27AE60?style=for-the-badge" />

Security Groups act as firewalls. Let's create them properly!

1. Go to **EC2** → **Security Groups** (left sidebar, under Network & Security)

**ALB Security Group:**

2. Click **Create security group**
3. **Security group name**: Type `ravi-alb-sg`
4. **Description**: Type `Security group for Application Load Balancer`
5. **VPC**: Select `ravi-capstone-vpc`
6. **Inbound rules**:
   - Click **Add rule**
   - **Type**: HTTP
   - **Port**: 80
   - **Source**: Anywhere-IPv4 (0.0.0.0/0)
7. Click **Create security group**

**EC2 Security Group:**

8. Click **Create security group**
9. **Security group name**: Type `ravi-ec2-sg`
10. **Description**: Type `Security group for EC2 instances`
11. **VPC**: Select `ravi-capstone-vpc`
12. **Inbound rules**:
    - Click **Add rule** → **Type**: HTTP → **Port**: 80 → **Source**: Select `ravi-alb-sg` (the ALB can send traffic to EC2)
    - Click **Add rule** → **Type**: SSH → **Port**: 22 → **Source**: Select **My IP** (only you can SSH)
13. Click **Create security group**

**RDS Security Group:**

14. Click **Create security group**
15. **Security group name**: Type `ravi-rds-sg`
16. **Description**: Type `Security group for RDS database`
17. **VPC**: Select `ravi-capstone-vpc`
18. **Inbound rules**:
    - Click **Add rule** → **Type**: MySQL/Aurora → **Port**: 3306 → **Source**: Select `ravi-ec2-sg` (only EC2 can talk to RDS)
19. Click **Create security group**

**Update RDS security group:**

20. Go to **RDS** → **Databases** → `ravi-capstone-db`
21. Click **Modify** (scroll to bottom)
22. Under **Connectivity** → **Security group** → Remove old SG, add `ravi-rds-sg`
23. Click **Continue** → **Apply immediately** (for this lab)
24. Click **Modify DB instance**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" /> "Notice the security group chain: Internet → ALB (port 80) → EC2 (port 80) → RDS (port 3306). Each layer only allows traffic from the layer above it. This is called 'defense in depth' — even if one layer is compromised, the next layer is still protected!"

📸 [Screenshot: Three security groups showing their inbound rules chained together]

---

### Step 5: Create Launch Template

<img src="https://img.shields.io/badge/Step%205-Create%20Launch%20Template-27AE60?style=for-the-badge" />

A Launch Template defines what EC2 instances look like when Auto Scaling creates them.

1. Go to **EC2** → **Launch Templates** (left sidebar)
2. Click **Create launch template**

**Configure the template:**

3. **Launch template name**: Type `ravi-capstone-template`
4. **Template version description**: Type `Capstone project launch template`
5. **AMI**: Amazon Linux 2023 (Free Tier eligible)
6. **Instance type**: t2.micro
7. **Key pair**: Select your existing key pair
8. **Network settings**:
   - Select **Select existing security group**
   - **Security groups**: Select `ravi-ec2-sg`
9. **Storage**:
   - **Size**: 8 GiB (default)

**Add User Data (startup script):**

10. Expand **Advanced details**
11. In the **User data** field, paste this entire script:

```bash
#!/bin/bash
dnf update -y
dnf install -y httpd php php-mysqlnd mariadb105
systemctl start httpd
cat > /var/www/html/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>Capstone Project</title></head>
<body>
<h1>Welcome to Ravi's Capstone Project!</h1>
<p>This full-stack app runs on AWS!</p>
<p>Architecture: Route 53 &rarr; ALB &rarr; EC2 &rarr; RDS</p>
<p>Services used: EC2, RDS, S3, ALB, VPC, CloudWatch, CloudTrail</p>
</body>
</html>
EOF
systemctl enable httpd
```

12. Click **Create launch template**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" /> "The User Data script runs every time an instance launches. It installs Apache (httpd) and PHP, then creates a simple HTML page. In production, you'd use a proper deployment pipeline, but this works great for a demo!"

📸 [Screenshot: Launch template creation page with User Data script visible]

---

### Step 6: Create Auto Scaling Group

<img src="https://img.shields.io/badge/Step%206-Create%20Auto%20Scaling%20Group-27AE60?style=for-the-badge" />

Auto Scaling ensures we always have enough EC2 instances running!

1. Go to **EC2** → **Auto Scaling Groups** (left sidebar)
2. Click **Create Auto Scaling group**

**Configure the ASG:**

3. **Name**: Type `ravi-capstone-asg`
4. **Launch template**: Select `ravi-capstone-template`
5. Click **Next**

**Configure VPC and subnets:**

6. **VPC**: Select `ravi-capstone-vpc`
7. **Availability Zones**: Select both public subnets:
   - `10.0.1.0/24 (us-east-1a)`
   - `10.0.2.0/24 (us-east-1b)`
8. Click **Next**

**Configure advanced options:**

9. **Load balancing**: Select **Attach to an existing load balancer** (we'll create the ALB first, so if it's not ready, select **Skip to next step** and come back)
   - If the ALB isn't created yet, choose **Skip to next step** and we'll attach it later
10. **Health check type**: Select **ELB** (if ALB is attached) or **EC2**
11. **Health check grace period**: 300 seconds
12. Click **Next**

**Configure group size and scaling:**

13. **Desired capacity**: Type `2`
14. **Minimum capacity**: Type `2`
15. **Maximum capacity**: Type `4`
16. **Scaling policies**: Select **Target tracking scaling policy**
17. **Metric type**: Average CPU utilization
18. **Target value**: `50`
19. Click **Next**

**Add notifications (optional):**

20. Click **Skip to next step**

**Review:**

21. Click **Create Auto Scaling group**

The ASG will launch 2 EC2 instances across the two public subnets. Wait 2-3 minutes for them to be in service.

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" /> "Auto Scaling with target tracking is like cruise control for your servers. If CPU goes above 50%, it adds instances. If it drops below, it removes them. Your app always has exactly the right amount of capacity!"

📸 [Screenshot: Auto Scaling Group showing 2 instances InService across both AZs]

---

### Step 7: Create Application Load Balancer

<img src="https://img.shields.io/badge/Step%207-Create%20ALB-27AE60?style=for-the-badge" />

The ALB distributes traffic across our EC2 instances!

1. Go to **EC2** → **Load Balancers** (left sidebar)
2. Click **Create Load Balancer**
3. Select **Application Load Balancer**

**Configure the ALB:**

4. **Name**: Type `ravi-capstone-alb`
5. **Scheme**: Internet-facing
6. **IP address type**: IPv4
7. **VPC**: Select `ravi-capstone-vpc`
8. **Mappings**: Check ✅ both availability zones (us-east-1a, us-east-1b)
9. **Security groups**: Select `ravi-alb-sg`

**Configure listener:**

10. **Listener 1**: HTTP on port 80
11. **Default action**: Forward to target group
12. **Target group**: Select **Create a target group**
    - **Name**: Type `ravi-capstone-tg`
    - **Target type**: Instances
    - **Protocol**: HTTP
    - **Port**: 80
    - **VPC**: `ravi-capstone-vpc`
    - **Health check path**: Type `/`
    - **Health check protocol**: HTTP
13. Click **Create target group** (this opens in a new tab)
14. Go back to the ALB creation tab
15. Refresh the target group dropdown → select `ravi-capstone-tg`
16. Click **Create load balancer**

**Wait for the ALB to be active:**

17. Wait 2-3 minutes for the ALB status to change to **Active**
18. Copy the **DNS name** (something like `ravi-capstone-alb-xxxx.us-east-1.elb.amazonaws.com`)

**Verify the ALB is working:**

19. Open a new browser tab
20. Paste the ALB DNS name
21. You should see the HTML page from one of your EC2 instances!

**Test load balancing:**

22. Refresh the page 5-10 times
23. You might see different responses if the instances are different (or same if the User Data is identical — that's expected)

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" /> "The ALB is the front door to your application. It distributes traffic, performs health checks, and can even handle SSL/TLS termination. If one EC2 instance fails, the ALB automatically routes traffic to healthy instances."

📸 [Screenshot: ALB active with DNS name, browser showing the capstone web page]

---

### Step 8: Set Up CloudWatch

<img src="https://img.shields.io/badge/Step%208-Set%20Up%20CloudWatch-27AE60?style=for-the-badge" />

Let's monitor our application!

1. Go to **CloudWatch** in the AWS Console

**Create a dashboard:**

2. Click **Dashboards** in the left sidebar
3. Click **Create dashboard**
4. **Dashboard name**: Type `Capstone-Dashboard`
5. Click **Create dashboard**

**Add widgets:**

6. Click **Add widget** → Select **Line** → **Next**
7. Select **EC2** metrics → **Per-Instance Metrics** → Select CPU Utilization for both instances → Click **Create widget**
8. Click **Add widget** → Select **Number** → **Next**
9. Select **Application Load Balancer** → **Per App, Target Group, Load Balancer** → Select **RequestCount** → **Create widget**
10. Click **Add widget** → Select **Line** → **Next**
11. Select **RDS** → **Database connections** → Select your RDS instance → **Create widget**
12. Click **Add widget** → Select **Number** → **Next**
13. Select **Application Load Balancer** → **UnHealthyHostCount** → **Create widget**

14. Click **Save dashboard**

**Create a CloudWatch Alarm:**

15. Click **Alarms** in the left sidebar
16. Click **Create alarm**
17. Click **Select metric**
18. Search for `EC2` → **Per-Instance Metrics** → Select **CPUUtilization** for one instance
19. Click **Select metric**

**Configure the alarm:**

20. **Statistic**: Average
21. **Period**: 5 minutes
22. **Threshold type**: Static
23. **Whenever CPUUtilization is**: Greater than `80`
24. **Datapoints to alarm**: 2 out of 3
25. Click **Next**

**Configure notification:**

26. **Alarm notification**: Select **Create new SNS topic**
27. **Topic name**: Type `ravi-capstone-alerts`
28. **Email endpoints**: Type your email address
29. Click **Create topic**
30. Click **Next** → **Next** → **Create alarm**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" /> "Dashboards give you a bird's-eye view of your application's health. Alarms alert you when something goes wrong. Together, they're your early warning system. In production, you'd have alarms for CPU, memory, disk, error rates, and latency!"

📸 [Screenshot: CloudWatch dashboard showing EC2 CPU, ALB requests, RDS connections, and UnhealthyHostCount]

---

### Step 9: Enable CloudTrail

<img src="https://img.shields.io/badge/Step%209-Enable%20CloudTrail-27AE60?style=for-the-badge" />

Let's add audit logging for all API activity!

1. Go to **CloudTrail** in the AWS Console
2. Click **Trails** → **Create trail**

**Configure the trail:**

3. **Trail name**: Type `ravi-capstone-trail`
4. Under **Management events**: Read/Write events
5. **Storage location**:
   - Select **Create new S3 bucket**
   - **S3 bucket**: Type `ravi-capstone-audit-12345` (add random numbers)
6. Click **Create trail**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" /> "CloudTrail is your black box recorder. If something goes wrong — unauthorized access, accidental deletion, strange behavior — CloudTrail tells you exactly what happened, who did it, and when. Never fly without it!" (Console-created trails are multi-Region by default, so every region gets logged automatically.)

📸 [Screenshot: CloudTrail trail created and showing Logging status]

---

### Step 10: Configure Route 53 (Optional)

<img src="https://img.shields.io/badge/Step%2010-Route%2053%20(Optional)-3498DB?style=for-the-badge" />

If you have a domain name, you can point it to your ALB!

**Option A: You have a domain name**

1. Go to **Route 53** → **Hosted zones**
2. Click on your domain name
3. Click **Create record**
4. **Record name**: Leave blank (for root domain) or type `www`
5. **Record type**: A
6. **Alias**: Toggle ON
7. **Route traffic to**: Alias to Application Load Balancer → select your region → select `ravi-capstone-alb`
8. Click **Create records**

**Option B: You don't have a domain (that's fine!)**

Just use the ALB DNS name directly. The architecture still works perfectly!

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" /> "Route 53 is AWS's DNS service. It translates domain names (like example.com) to IP addresses or AWS resources. For this lab, the ALB DNS name works great. In production, you'd use a friendly domain name!"

📸 [Screenshot: Route 53 record created pointing to ALB (or note: using ALB DNS directly)]

---

### Step 11: Full Verification

<img src="https://img.shields.io/badge/Step%2011-Full%20Verification-27AE60?style=for-the-badge" />

Time to verify everything is working! Let's check each component.

**Test the web application:**

1. Open a browser tab
2. Go to your ALB DNS name (or your Route 53 domain)
3. You should see the capstone HTML page
4. Refresh multiple times — traffic should be load-balanced across instances

**Check Auto Scaling:**

5. Go to **EC2** → **Auto Scaling Groups**
6. Verify 2 instances are **InService**
7. Go to **EC2** → **Instances**
8. You should see 2 instances with the `ravi-capstone-template` prefix

**Check CloudWatch:**

9. Go to **CloudWatch** → **Dashboards** → `Capstone-Dashboard`
10. You should see metrics flowing (CPU, requests, connections)
11. Go to **Alarms** → Check that the CPU alarm is in **OK** state

**Check CloudTrail:**

12. Go to **CloudTrail** → **Event history**
13. Filter by **Event source** = `ec2.amazonaws.com`
14. You should see all the EC2 actions you took during this lab

**Check RDS:**

15. Go to **RDS** → **Databases** → `ravi-capstone-db`
16. Verify it's **Available**
17. Check **Database connections** metric in CloudWatch — it should show connection activity

**Check S3:**

18. Go to **S3** → `ravi-capstone-assets-12345`
19. Verify the `style.css` file is there
20. Check that default encryption is enabled

**Check VPC:**

21. Go to **VPC** → **Your VPCs** → Verify `ravi-capstone-vpc` exists
22. Go to **Subnets** → Verify 4 subnets (2 public, 2 private)
23. Go to **NAT Gateways** → Verify 1 NAT Gateway is active

📸 [Screenshot: Browser showing capstone page, CloudWatch dashboard, CloudTrail events, and all services running]

---

### Step 12: Document Your Architecture

<img src="https://img.shields.io/badge/Step%2012-Document%20Architecture-27AE60?style=for-the-badge" />

Great engineers document their work! Create a simple architecture summary.

**Resources created:**

```
VPC:
  - ravi-capstone-vpc (10.0.0.0/16)
  - Public Subnets: 10.0.1.0/24 (1a), 10.0.2.0/24 (1b)
  - Private Subnets: 10.0.3.0/24 (1a), 10.0.4.0/24 (1b)
  - Internet Gateway
  - NAT Gateway

Compute:
  - EC2 Auto Scaling Group (2-4 instances, t2.micro)
  - Launch Template: ravi-capstone-template

Database:
  - RDS MySQL: ravi-capstone-db (db.t3.micro, 20GB)

Load Balancing:
  - ALB: ravi-capstone-alb (HTTP:80)
  - Target Group: ravi-capstone-tg

Storage:
  - S3: ravi-capstone-assets-12345 (static assets)

Monitoring:
  - CloudWatch Dashboard: Capstone-Dashboard
  - CloudWatch Alarm: CPU > 80%
  - SNS Topic: ravi-capstone-alerts

Auditing:
  - CloudTrail: ravi-capstone-trail → S3 bucket

Security Groups:
  - ravi-alb-sg (HTTP from anywhere)
  - ravi-ec2-sg (HTTP from ALB, SSH from My IP)
  - ravi-rds-sg (MySQL from EC2)

Estimated Cost: ~$3-5 for this lab session
```

📸 [Screenshot: Your architecture documentation — ASCII art or a screenshot of a diagram tool]

---

## ✅ Validation Checklist

Before cleaning up, confirm ALL of these:

<table>
<tr>
<th>Status</th>
<th>Check</th>
</tr>
<tr>
<td>- [ ]</td>
<td>ALB DNS name loads the capstone web page in browser</td>
</tr>
<tr>
<td>- [ ]</td>
<td>2 EC2 instances are running (Auto Scaling Group)</td>
</tr>
<tr>
<td>- [ ]</td>
<td>RDS instance is Available</td>
</tr>
<tr>
<td>- [ ]</td>
<td>S3 bucket has the style.css file</td>
</tr>
<tr>
<td>- [ ]</td>
<td>CloudWatch dashboard shows metrics</td>
</tr>
<tr>
<td>- [ ]</td>
<td>CloudTrail is logging events</td>
</tr>
<tr>
<td>- [ ]</td>
<td>VPC has 4 subnets, IGW, and NAT Gateway</td>
</tr>
<tr>
<td>- [ ]</td>
<td>Security groups are chained: ALB → EC2 → RDS</td>
</tr>
</table>

---

> **POV:** You look at your AWS bill after the capstone and realize you actually stayed under $5. Clean-up skills: unlocked.

---

> **Achievement Unlocked:** AWS Builder! Full stack complete! You are LEGENDARY!

---

## 🧹 Cleanup (IMPORTANT!)

Follow this EXACT order to avoid dependency issues and ensure everything is deleted!

> <img src="https://img.shields.io/badge/Warning-Cleanup-E74C3C?style=flat-square" /> **Do NOT skip any step. Missing cleanup = ongoing charges!**

**Step 1: Route 53 (if you created records)**

1. Go to **Route 53** → **Hosted zones**
2. Select any records you created → **Delete**
3. If you created health checks, delete them too

**Step 2: CloudWatch**

4. Go to **CloudWatch** → **Dashboards**
5. Select `Capstone-Dashboard` → **Delete dashboard**
6. Go to **Alarms** → Select your alarm → **Delete**
7. Go to **SNS** → **Topics** → Select `ravi-capstone-alerts` → **Delete topic**

**Step 3: CloudTrail**

8. Go to **CloudTrail** → **Trails**
9. Select `ravi-capstone-trail` → **Delete**
10. Go to **S3** → Find the audit bucket → **Empty** → **Delete bucket**

**Step 4: Application Load Balancer**

11. Go to **EC2** → **Load Balancers**
12. Select `ravi-capstone-alb` → **Delete**
13. Go to **Target Groups** → Select `ravi-capstone-tg` → **Delete**

**Step 5: Auto Scaling Group**

14. Go to **EC2** → **Auto Scaling Groups**
15. Select `ravi-capstone-asg` → **Edit**
16. Change **Desired capacity** to `0` → **Update**
17. Wait for instances to terminate
18. Select the ASG → **Delete**

**Step 6: Launch Template**

19. Go to **EC2** → **Launch Templates**
20. Select `ravi-capstone-template` → **Delete**

**Step 7: EC2 Instances**

21. Go to **EC2** → **Instances**
22. Select any remaining instances → **Terminate**

**Step 8: RDS Database**

23. Go to **RDS** → **Databases**
24. Select `ravi-capstone-db` → **Actions** → **Delete**
25. Uncheck **Create final snapshot**
26. Check ✅ **Acknowledge**
27. Type `delete me` to confirm
28. Click **Delete**
29. Wait 5-10 minutes for deletion to complete

**Step 9: S3 Bucket**

30. Go to **S3** → **Buckets**
31. Find `ravi-capstone-assets-12345`
32. Click → **Empty** → Type `permanently delete` → **Delete**
33. Go back → Select bucket → **Delete** → Type name → **Delete bucket**

**Step 10: NAT Gateway (CRITICAL!)**

34. Go to **VPC** → **NAT Gateways**
35. Select the NAT Gateway → **Delete**
36. Confirm deletion

> <img src="https://img.shields.io/badge/Warning-NAT%20Gateway-E74C3C?style=flat-square" /> NAT Gateway costs ~$0.045/hr = ~$32/month! Do NOT skip this!

**Step 11: VPC Resources**

37. Go to **VPC** → **Subnets** → Delete private subnets first, then public subnets
38. Go to **VPC** → **Route tables** → Delete custom route tables (keep the main one)
39. Go to **VPC** → **Internet Gateways** → Detach from VPC → Delete
40. Go to **VPC** → **Your VPCs** → Select `ravi-capstone-vpc` → **Delete**

**Step 12: Security Groups**

41. Go to **EC2** → **Security Groups**
42. Delete `ravi-alb-sg`, `ravi-ec2-sg`, `ravi-rds-sg` (one by one — can't delete the default)

**Step 13: RDS Subnet Group**

43. Go to **RDS** → **Subnet groups** → Delete `ravi-capstone-subnet-group`

**Step 14: Final Verification**

44. Go through each service one more time:

<table>
<tr>
<th>Service</th>
<th>Expected State</th>
</tr>
<tr>
<td>EC2</td>
<td>No instances</td>
</tr>
<tr>
<td>RDS</td>
<td>No databases</td>
</tr>
<tr>
<td>S3</td>
<td>No buckets</td>
</tr>
<tr>
<td>VPC</td>
<td>No VPCs (except default)</td>
</tr>
<tr>
<td>CloudTrail</td>
<td>No trails</td>
</tr>
<tr>
<td>CloudWatch</td>
<td>No dashboards</td>
</tr>
<tr>
<td>ALB</td>
<td>No load balancers</td>
</tr>
<tr>
<td>NAT Gateway</td>
<td>None active</td>
</tr>
</table>

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" /> "The cleanup order matters! You can't delete a VPC if resources are still in it. You can't delete subnets if route tables reference them. Follow the order and everything will clean up smoothly!"

📸 [Screenshot: All AWS services showing no resources remaining]

---

## 🧠 Memory Tips

Final exam edition — the mnemonics that should now be muscle memory. 🧲

| 🧠 Memory Hook | Remember it like... |
|---|---|
| **VPC = gated neighborhood** | Subnets = streets, IGW = front gate, route tables = street signs. 🏘️ |
| **ALB = traffic cop, ASG = robot manager** | ALB spreads traffic; ASG keeps the right number of servers alive. 🚦🤖 |
| **SG = bouncer, RDS = managed vault** | Firewall rules at the door; database in a locked, backed-up room. 🚪🗄️ |
| **S3 = cabinet, CloudWatch = cameras, CloudTrail = black box** | Storage, monitoring, and audit — your observability trio. 📦🎥🕵️ |
| **Defense in depth** | Layers of security: VPC isolation + SG rules + private subnets. No single door = the whole castle. 🏰 |

> 🗣️ **Rithu:** *"If you can explain every piece of this architecture to a friend without notes, you've genuinely learned AWS. That's the real final exam."

---

## 🎓 What You Learned

In this capstone lab, you brought together EVERY skill from Labs 01-24:

| Service | Skill Applied |
|---------|--------------|
| **VPC** | Designed network architecture with public/private subnets |
| **EC2** | Launched instances with User Data bootstrapping |
| **Auto Scaling** | Created self-healing, scalable infrastructure |
| **ALB** | Distributed traffic across multiple instances |
| **RDS** | Deployed a managed MySQL database |
| **S3** | Stored static assets with encryption |
| **Security Groups** | Implemented defense-in-depth networking |
| **CloudWatch** | Monitored application health with dashboards and alarms |
| **CloudTrail** | Enabled audit logging for compliance |
| **Route 53** | (Optional) Configured DNS for the application |

**Architecture patterns you now understand:**
- Multi-tier application design (web → app → database)
- High availability across multiple Availability Zones
- Auto Scaling for fault tolerance and cost optimization
- Defense-in-depth security with chained Security Groups
- Centralized monitoring and audit logging

| | Approach |
|---|---|
| **Noob Tip** | Click through the console to create everything manually |
| **Pro Tip** | Use CloudFormation/IaC. Reproducible, documented, and version-controlled.

---

<div align="center">

## 🎮 Test Yourself! (No Peeking 👀)

**Q1:** Why does the database live in a **private subnet** while the web servers live in a **public** one?

<details><summary>👀 Show answer</summary>

**A:** **Defense in depth** — web servers *must* face the internet, but the database should only be reachable by your app layer. Private subnet + strict SG = the crown jewels stay hidden. 🏰

</details>

**Q2:** If one web server dies, name the TWO services that keep the app online.

<details><summary>👀 Show answer</summary>

**A:** The **ALB** stops sending it traffic (health check), and the **ASG** launches a replacement automatically. Traffic cop + robot manager working as a team. 🤝

</details>

**Q3:** The app is getting slow. Which service tells you WHERE to look, and which one tells you WHO did what?

<details><summary>👀 Show answer</summary>

**A:** **CloudWatch** shows you the metrics (CPU, latency, alarms) — the "where". **CloudTrail** logs every API action — the "who". Observability + auditability. 📊🕵️

</details>

**Q4:** What makes this architecture *production-like* compared to the earlier labs?

<details><summary>👀 Show answer</summary>

**A:** **Multi-tier design, high availability (multi-AZ), auto-scaling, managed database, monitoring, and audit logging** — the full enterprise checklist, not just a single service demo. 🏆

</details>

### 🔥 Final Challenge

**Break it, then prove it recovers:** kill the primary web instance and watch the ALB + ASG recover. Then terminate an instance and find the event in **CloudTrail**. Finally, delete the entire stack piece by piece following the cleanup checklist. When the account is empty, you've completed AWS Bootcamp. 👏

> 💪 **Rithu:** *"You didn't watch tutorials — you BUILT. That puts you ahead of 90% of people who 'want to learn AWS.' Go be a cloud engineer."

---

## 🔗 What's Next?

![Complete](https://img.shields.io/badge/CAPSTONE%20COMPLETE!-E74C3C?style=for-the-badge&labelColor=232F3E)
![Badge](https://img.shields.io/badge/AWS%20Builder-You%20Did%20It!-F1C40F?style=for-the-badge&labelColor=232F3E)
![Celebration](https://img.shields.io/badge/Labs%2001--24-COMPLETE-27AE60?style=for-the-badge&labelColor=232F3E)

> **CONGRATULATIONS, RAVI!**
>
> You did it! You've gone from launching your first EC2 instance to building a complete full-stack application on AWS. You should be incredibly proud of yourself.
>
> The cloud journey never ends — keep exploring, keep building, keep learning. Here are some ideas for what to do next:
>
> - **Explore more services**: Lambda, API Gateway, ECS, EKS, Step Functions
> - **Get certified**: AWS Cloud Practitioner or Solutions Architect Associate
> - **Build something real**: Deploy a blog, a portfolio site, or a small SaaS app
> - **Join the community**: AWS re:Post, Reddit r/aws, local meetups
>
> **You are now an AWS Builder. Go build something amazing!** 🚀
>
> — Rithu

![Done](https://img.shields.io/badge/Mission-Accomplished-2ECC71?style=for-the-badge&labelColor=232F3E)

</div>

---

<details>
<summary><img src="https://img.shields.io/badge/FAQ-Troubleshooting-3498DB?style=flat-square" /> <b>Problem: VPC creation fails or times out</b></summary>
<br/>

- **Fix**: Try creating with fewer subnets first, then add more. Make sure you have enough IP addresses in your CIDR block. Try a different region if us-east-1 is having issues.

</details>

<details>
<summary><img src="https://img.shields.io/badge/FAQ-Troubleshooting-3498DB?style=flat-square" /> <b>Problem: RDS instance won't create</b></summary>
<br/>

- **Fix**: Check that you're using the Free Tier template. Ensure the subnet group selects private subnets. If you hit a quota limit, go to Service Quotas and request an increase for RDS instances.

</details>

<details>
<summary><img src="https://img.shields.io/badge/FAQ-Troubleshooting-3498DB?style=flat-square" /> <b>Problem: EC2 instances show as "Unhealthy" in ALB</b></summary>
<br/>

- **Fix**: Wait 2-3 minutes after launch for the UserData script to complete. Check that the security group allows traffic from the ALB security group. Verify the health check path `/` returns HTTP 200.

</details>

<details>
<summary><img src="https://img.shields.io/badge/FAQ-Troubleshooting-3498DB?style=flat-square" /> <b>Problem: Can't SSH into EC2 instances</b></summary>
<br/>

- **Fix**: Ensure your security group allows SSH (port 22) from "My IP". Make sure you're using the correct key pair. Check that the instance has a public IP (it should, since it's in a public subnet).

</details>

<details>
<summary><img src="https://img.shields.io/badge/FAQ-Troubleshooting-3498DB?style=flat-square" /> <b>Problem: ALB returns 502 Bad Gateway</b></summary>
<br/>

- **Fix**: The target group health checks are failing. Check that EC2 instances are running and Apache is started. Verify security groups allow traffic from ALB → EC2 on port 80.

</details>

<details>
<summary><img src="https://img.shields.io/badge/FAQ-Troubleshooting-3498DB?style=flat-square" /> <b>Problem: RDS connection refused from EC2</b></summary>
<br/>

- **Fix**: Verify the RDS security group allows MySQL (3306) from the EC2 security group. Make sure you're using the correct endpoint. Check that the RDS instance is in the same VPC as the EC2 instances.

</details>

<details>
<summary><img src="https://img.shields.io/badge/FAQ-Troubleshooting-3498DB?style=flat-square" /> <b>Problem: Auto Scaling won't launch new instances</b></summary>
<br/>

- **Fix**: Check that the launch template is valid. Ensure you haven't hit EC2 instance limits (check Service Quotas). Verify the subnets have available IP addresses.

</details>

<details>
<summary><img src="https://img.shields.io/badge/FAQ-Troubleshooting-3498DB?style=flat-square" /> <b>Problem: CloudWatch dashboard shows no data</b></summary>
<br/>

- **Fix**: Wait 5-10 minutes for metrics to appear. CloudWatch metrics can take up to 15 minutes to populate. Ensure the correct region is selected in the CloudWatch console.

</details>

<details>
<summary><img src="https://img.shields.io/badge/FAQ-Troubleshooting-3498DB?style=flat-square" /> <b>Problem: Can't delete VPC</b></summary>
<br/>

- **Fix**: This is usually because resources still exist in the VPC. Delete NAT Gateway, subnets, route tables, and security groups first. Then try deleting the VPC again.

</details>

<details>
<summary><img src="https://img.shields.io/badge/FAQ-Troubleshooting-3498DB?style=flat-square" /> <b>Problem: NAT Gateway won't delete</b></summary>
<br/>

- **Fix**: Make sure all resources using it (EC2 instances, etc.) are terminated first. The NAT Gateway must be detached from the subnet before deletion.

</details>

<details>
<summary><img src="https://img.shields.io/badge/FAQ-Troubleshooting-3498DB?style=flat-square" /> <b>Problem: S3 bucket won't delete</b></summary>
<br/>

- **Fix**: You must EMPTY the bucket first (delete all objects), then delete the bucket itself. S3 buckets cannot be deleted if they contain any objects.

</details>

---

> **Rithu's Real Talk:** If you've made it to Lab 25, you're ahead of 90% of people who "want to learn AWS." You didn't just watch tutorials - you BUILT things. That's what makes you different. Keep building.

---

<div align="center">

> **Rithu says**: "I'm incredibly proud of you, Ravi. You started this journey knowing nothing about AWS, and now you've built a production-grade application architecture. You are officially a cloud engineer. Keep building, keep learning, and remember — every expert was once a beginner. You've got this!"

![End](https://img.shields.io/badge/End%20of%20Lab-25-27AE60?style=for-the-badge&labelColor=232F3E)

</div>
