<div align="center">

<img src="https://img.shields.io/badge/Lab%2013-RDS%20MySQL%20on%20AWS-8E44AD?style=for-the-badge&labelColor=232F3E" />

</div>

<div align="center">

<img src="https://img.shields.io/badge/Difficulty-Medium-F4D03F?style=flat-square" />
<img src="https://img.shields.io/badge/Time-~35min-3498DB?style=flat-square" />
<img src="https://img.shields.io/badge/Cost-%3C%243-2ECC71?style=flat-square" />
<img src="https://img.shields.io/badge/Service-RDS-8E44AD?style=flat-square" />

</div>

> "Ravi, installing MySQL on your laptop is like building a shelf from IKEA instructions — possible, but painful. Amazon RDS gives you a fully managed MySQL database in the cloud. No patching, no backing up at 3 AM, no drama. Just pure database joy!" — Rithu

---

<details>
<summary><b>🎭 Ravi & Rithu's Coffee Break Chat</b></summary>

**Ravi:** "Why not just install MySQL on my EC2 instance?"

**Rithu:** "You can! And then YOU patch it, YOU back it up, YOU babysit the backups, YOU handle failover at 3 AM."

**Ravi:** "...that sounds like a lot of YOU."

**Rithu:** "Exactly why RDS exists. AWS does all the boring-but-critical parts. You just write SQL and sleep."

**Ravi:** "Sleep? In this economy?"

**Rithu:** "With RDS? Yes. That's the whole point. 😴"

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

> **What is this, really?** RDS is a **managed MySQL database in the cloud** — AWS installs it, patches it, backs it up, and (if you ask nicely) duplicates it across AZs. You get a normal database address (host, port 3306, user, password) and just connect with any MySQL client. It's like renting a fully-serviced apartment vs. building your own house. 🏠
>
> 🌍 **Why you should care:** Running your own database is a full-time job. RDS turns it into a 5-minute setup — which is why nearly every production app on AWS uses it.

---

## 🎯 Objective

In this lab, you will:

- Create a **DB Subnet Group** to control where your database lives
- Set up a **Security Group** for database access
- Launch a **MySQL RDS instance** on the Free Tier
- **Connect** to your database from an EC2 instance
- Create tables, insert data, and run queries
- Understand why managed databases are worth every penny

---

## 🧠 Prerequisites

Before you start, make sure you have:

- ✅ Completed **Lab 12** (Route 53)
- ✅ An AWS account with RDS access
- ✅ At least one **running EC2 instance** with MySQL client (or be ready to install one)
- ✅ Your key pair ready for SSH
- ✅ A strong password ready (you'll need it for the database!)

---

## 💰 Cost Warning

| Resource | Cost |
|----------|------|
| RDS db.t3.micro | ~$0.017/hour (~$12/month) |
| Storage (20 GB gp3) | ~$2.30/month |
| **Estimated total for this lab** | **< $3** (if cleaned up within an hour!) |

> ⚠️ **CRITICAL — Rithu says:** RDS instances keep running and keep charging even after you close your browser! Unlike EC2 where you can stop an instance and stop paying, RDS **charges you even when stopped** (for the storage). You MUST **delete** the RDS instance after this lab. I cannot stress this enough! 💸

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> The Free Tier gives you 750 hours/month of `db.t3.micro` or `db.t4g.micro` for 12 months (legacy accounts). But storage and backups are NOT Free Tier eligible. Delete after the lab to avoid surprises!

---

## 🏗️ Architecture

```
    ┌──────────────────┐        ┌──────────────────────┐
    │   Your Computer   │        │   Amazon RDS MySQL    │
    │   (or EC2 instance)│──────▶│   ravi-mysql-db       │
    │                    │ :3306 │   db.t3.micro         │
    │  mysql -h ravi...  │       │   20 GB gp3           │
    └──────────────────┘        │   Database: ravilabs   │
                                └──────────────────────┘
                                            │
                                  ┌─────────┴─────────┐
                                  │  DB Subnet Group    │
                                  │  us-east-1a + 1b    │
                                  └───────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Create a DB Subnet Group

> <img src="https://img.shields.io/badge/Step%201-Create%20DB%20Subnet%20Group-3498DB?style=for-the-badge" />

A DB Subnet Group tells RDS which subnets (and therefore which Availability Zones) it can place your database in.

1. Go to the **RDS Console** → left sidebar → **Subnet groups**
2. Click **Create DB subnet group**
3. Fill in:

| Field | Value |
|-------|-------|
| DB subnet group name | `ravi-db-subnet-group` |
| Description | "Subnet group for Ravi's RDS lab" |
| VPC | **Default VPC** |

4. Under **Add subnets:**
   - Select **us-east-1a** and **us-east-1b** (click both!)
   - These should show the default subnets for your VPC

5. Click **Create**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Just like with ASG, we're putting our database in 2 AZs for high availability. If one AZ has issues, RDS can failover to the other. Even for a lab, it's good practice! 🏗️

📸 **[Screenshot: DB Subnet Group creation page showing 2 AZs selected]**

---

### Step 2: Create a Security Group for RDS

> <img src="https://img.shields.io/badge/Step%202-Create%20RDS%20Security%20Group-2ECC71?style=for-the-badge" />

Your database needs its own security group. We'll allow MySQL access from anywhere (for lab purposes only!).

1. Go to **EC2 Console** → **Security Groups** → **Create security group**
2. Configure:

| Field | Value |
|-------|-------|
| Security group name | `rds-sg` |
| Description | "Security group for RDS MySQL lab" |
| VPC | Default VPC |

3. **Inbound rules:**

| Type | Port | Source |
|------|------|--------|
| MySQL/Aurora | 3306 | Anywhere (`0.0.0.0/0`) |

4. **Outbound rules:** Keep default (All traffic)

5. Click **Create security group**

> ⚠️ **Production Reality Check:** In the real world, you would NEVER open MySQL to the entire internet! You'd restrict it to specific security groups or IP ranges. But for learning, `0.0.0.0/0` lets us connect from our local machine without VPC peering headaches.

📸 **[Screenshot: Security group with MySQL port 3306 open]**

---

### Step 3: Create the RDS Instance

> <img src="https://img.shields.io/badge/Step%203-Create%20RDS%20Instance-E74C3C?style=for-the-badge" />

This is the main event — let's launch a MySQL database!

1. Go to **RDS Console** → **Databases** (left sidebar)
2. Click **Create database**

**Choose a database creation method:**
- Select **Standard create** (not Easy create — we want control!)

**Engine options:**
- Select **MySQL**
- Version: **MySQL 8.0.x** (latest available)
- Template: **Free tier** ✅

**Settings:**

| Field | Value |
|-------|-------|
| DB instance identifier | `ravi-mysql-db` |
| Master username | `admin` |
| Master password | **Use a strong password! Write it down!** |

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Pick a password you'll remember but that's not "password123". I recommend something like `Ravi#MySQL2024!` — mix of uppercase, lowercase, numbers, and symbols. WRITE IT DOWN somewhere safe!

**Instance configuration:**

| Field | Value |
|-------|-------|
| DB instance class | **db.t3.micro** (Free Tier) |
| Storage type | **General Purpose SSD (gp3)** |
| Allocated storage | **20 GB** |
| Storage autoscaling | ❌ Uncheck this (keep costs predictable) |

**Connectivity:**

| Field | Value |
|-------|-------|
| VPC | Default VPC |
| DB subnet group | `ravi-db-subnet-group` |
| Public access | **Yes** ⚠️ |
| Security group | `rds-sg` |
| Availability zone | Don't specify (let RDS choose) |

> ⚠️ **Production Reality Check:** "Public access: Yes" means your database can be reached from the internet (protected by the security group). In production, this should almost ALWAYS be "No" — you'd connect through a VPN or from within the VPC. But for this lab, we need public access.
>
> 💡 **Note:** Free Tier supports `db.t3.micro` and `db.t4g.micro`. The console lists `db.t3.micro` first. `gp3` is the current default storage type (cheaper and faster than the older `gp2`).

**Database options:**

| Field | Value |
|-------|-------|
| Initial database name | `ravilabs` |
| DB parameter group | default |
| Backup | **Disable** (for lab — explain below) |
| Enable automated backups | ❌ |

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> In production, automated backups are essential! They let you restore to any point in time. But for a lab, we don't need them, and they cost extra storage.

**Monitoring & Maintenance:**
- Leave all defaults
- Click **Create database**

4. Wait for the database to be created — this takes **5-10 minutes** ⏱️

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Go grab a coffee while you wait! ☕ RDS is setting up MySQL, configuring networking, setting up the security group, and more. This is the "managed" part of a managed database — AWS does all the boring stuff for you.

📸 **[Screenshot: RDS console showing the database with "Creating" status]**

---

### Step 4: Wait for DB to Be Available

> <img src="https://img.shields.io/badge/Step%204-Wait%20for%20DB-F39C12?style=for-the-badge" />

1. In the **RDS Console** → **Databases** → click `ravi-mysql-db`
2. Watch the **Status** column:
   - 🔄 Creating → ⏳ Backing up → ✅ **Available**
3. Once it says **Available**, you're ready to connect!

4. Copy the **Endpoint** from the database details:
   - It looks like: `ravi-mysql-db.xxxxxxxxxxxx.us-east-1.rds.amazonaws.com`
   - Note the **Port**: `3306`

📸 **[Screenshot: RDS database showing "Available" status and the endpoint]**

---

### Step 5: Connect to Your RDS Database

> <img src="https://img.shields.io/badge/Step%205-Connect%20to%20RDS-1ABC9C?style=for-the-badge" />

You have two options for connecting:

**Option A: Connect from Your Local Computer (if you have MySQL installed)**

```bash
mysql -h ravi-mysql-db.xxxxxxxxxxxx.us-east-1.rds.amazonaws.com -u admin -p
```

Enter your password when prompted.

**Option B: Connect from an EC2 Instance (recommended)**

1. SSH into an EC2 instance (or launch a new t2.micro Amazon Linux 2023):

```bash
ssh -i "first-key-pair.pem" ec2-user@<EC2_PUBLIC_IP>
```

2. Install the MySQL client:

> 💡 On Amazon Linux 2023 the `mysql` package isn't available. AWS's recommended client package is **`mariadb105`** (MariaDB), which provides the `mysql` command and is fully compatible with RDS MySQL:

```bash
sudo dnf install -y mariadb105
```

3. Connect to RDS:

```bash
mysql -h ravi-mysql-db.xxxxxxxxxxxx.us-east-1.rds.amazonaws.com -u admin -p
```

4. Enter your password when prompted

5. You should see the MySQL prompt:

```
mysql>
```

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> If you get "Access denied", double-check your password. If you get "Can't connect", check that the security group allows port 3306 from your IP or the EC2 instance's private IP.

📸 **[Screenshot: Terminal showing successful MySQL connection]**

---

### Step 6: Create Tables and Insert Data

> <img src="https://img.shields.io/badge/Step%206-Create%20Tables%20%26%20Insert%20Data-E67E22?style=for-the-badge" />

Now let's do some real database work! Run these commands in your MySQL prompt:

```sql
-- Create a database (if not created automatically)
CREATE DATABASE IF NOT EXISTS ravilabs;

-- Switch to our database
USE ravilabs;

-- Create a students table
CREATE TABLE students (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100),
  topic VARCHAR(100),
  score INT
);

-- Insert some data
INSERT INTO students (name, topic, score) VALUES
  ('Ravi', 'EC2', 95),
  ('Ravi', 'S3', 90),
  ('Ravi', 'VPC', 88);

-- Verify the data was inserted
SELECT * FROM students;
```

You should see:

```
+----+------+-------+-------+
| id | name | topic | score |
+----+------+-------+-------+
|  1 | Ravi | EC2   |    95 |
|  2 | Ravi | S3    |    90 |
|  3 | Ravi | VPC   |    88 |
+----+------+-------+-------+
3 rows in set (0.01 sec)
```

🎉 **Your first cloud database records!**

📸 **[Screenshot: MySQL terminal showing the SELECT query results]**

Let's do a few more operations:

```sql
-- Count how many records we have
SELECT COUNT(*) AS total_records FROM students;

-- Find Ravi's highest score
SELECT name, topic, MAX(score) AS highest_score FROM students GROUP BY name;

-- Update a score
UPDATE students SET score = 98 WHERE topic = 'EC2' AND name = 'Ravi';

-- Verify the update
SELECT * FROM students WHERE name = 'Ravi';
```

Exit MySQL:

```sql
EXIT;
```

---

### Step 7: View RDS Metrics in CloudWatch

> <img src="https://img.shields.io/badge/Step%207-View%20CloudWatch%20Metrics-9B59B6?style=for-the-badge" />

RDS automatically sends metrics to CloudWatch. Let's check them out!

1. Go to **RDS Console** → **Databases** → click `ravi-mysql-db`
2. Scroll down to **Monitoring** tab
3. You'll see graphs for:
   - CPU Utilization
   - Database connections
   - Free storage space
   - Read/write IOPS
4. Click **View all CloudWatch metrics** for more detail

📸 **[Screenshot: RDS monitoring tab showing CPU and connection metrics]**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> In production, you'd set up CloudWatch alarms on these metrics — like alerting when CPU goes above 80% or free storage drops below 1 GB. We'll cover that in Lab 15!

---

### Step 8: Verify Your Work

> <img src="https://img.shields.io/badge/Step%208-Verify%20Your%20Work-27AE60?style=for-the-badge" />

Let's confirm everything worked:

- [ ] DB Subnet Group `ravi-db-subnet-group` exists with 2 AZs
- [ ] Security group `rds-sg` allows MySQL (3306)
- [ ] RDS instance `ravi-mysql-db` shows **Available** status
- [ ] Successfully connected to MySQL from EC2 or local machine
- [ ] Created database `ravilabs`
- [ ] Created table `students` with 3 rows of data
- [ ] SELECT query returned the expected results
- [ ] RDS metrics visible in CloudWatch

📸 **[Screenshot: RDS database details page showing all configurations]**

---

## ✅ Validation Checklist

- [ ] DB Subnet Group spans 2 Availability Zones
- [ ] Security group allows MySQL (3306) inbound
- [ ] RDS instance is running and shows "Available"
- [ ] Endpoint and port noted down
- [ ] Successfully connected using `mysql` client
- [ ] Database `ravilabs` created with `students` table
- [ ] Data inserted and queryable
- [ ] CloudWatch metrics visible for the RDS instance

---

## 🧹 Cleanup (IMPORTANT!)

> ⚠️ **CRITICAL: RDS keeps charging even when stopped! DELETE the instance!**

| Step | Action | How |
|:-----|:-------|:----|
| 1 | Delete RDS instance | RDS → Databases → `ravi-mysql-db` → Actions → Delete (uncheck final snapshot) |
| 2 | Delete DB Subnet Group | RDS → Subnet groups → `ravi-db-subnet-group` → Delete |
| 3 | Delete Security Group | EC2 → Security Groups → `rds-sg` → Actions → Delete |
| 4 | Terminate EC2 (if launched) | EC2 → Instances → select instance → Instance state → Terminate |

### Detailed Steps:

1. **Delete the RDS Instance:**
   - Go to **RDS Console** → **Databases**
   - Select `ravi-mysql-db`
   - Click **Actions** → **Delete**
   - ⚠️ **UNCHECK** "Create final snapshot" (we don't need it for a lab)
   - ⚠️ **UNCHECK** "Acknowledge..." confirmation
   - Type `delete me` in the confirmation box
   - Click **Delete**

   > ⏱️ Deletion takes 5-10 minutes

2. **Delete the DB Subnet Group:**
   - Go to **RDS** → **Subnet groups**
   - Select `ravi-db-subnet-group`
   - Click **Delete**

3. **Delete the Security Group:**
   - Go to **EC2** → **Security Groups**
   - Find `rds-sg`
   - Click **Actions** → **Delete security group**
   - Confirm

4. **Terminate any EC2 instance you launched for this lab:**
   - Go to **EC2** → **Instances**
   - Select any instance you launched for this lab (e.g., for MySQL client access)
   - Click **Instance state** → **Terminate instance**
   - Confirm termination
   - ⚠️ If you used an existing instance from a previous lab, skip this step

📸 **[Screenshot: RDS console with instance deleted and subnet group removed]**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> After deletion, your RDS charges stop immediately. Storage is released and you stop paying. Always double-check that the database is gone! You can verify by going to RDS → Databases and confirming the list is empty (or doesn't contain your lab DB). Also verify EC2 instances are terminated so you don't pay for compute you're not using!

---

## 🧠 Memory Tips

Stick these in your brain and they'll never leave. 🧲

| 🧠 Memory Hook | Remember it like... |
|---|---|
| **RDS = "Really Don't Stress"** | Patching, backups, HA — all AWS's job. You just query. 😎 |
| **DB Subnet Group = real estate choice** | Tells RDS **which AZs it may live in**. Pick at least two for high availability. 🏘️ |
| **MySQL = port 3306** | The database's secret knock. Your security group must open **3306** — to your EC2 only! 🔑 |
| **Publicly Accessible = lab only!** | In production, databases stay **private**. Public DBs are a hacker's dream. 🚫 |
| **Managed = no SSH** | You can't SSH into an RDS instance. It's AWS's server — you only speak SQL to it. 🗣️ |

> 🗣️ **Rithu:** *"The RDS security group should say: only my app servers may enter. Everything else? Get lost."

---

## 🎓 What You Learned

| Concept | What You Now Know |
|---------|-------------------|
| **DB Subnet Groups** | How to control which AZs your database lives in |
| **RDS Security Groups** | How to control network access to your database |
| **RDS Instance Creation** | How to launch a managed MySQL database with proper configuration |
| **Publicly Accessible** | What it means and when to use it (lab only!) |
| **MySQL Client** | How to connect to a remote database from EC2 or local machine |
| **CRUD Operations** | Create, Read, Update, Delete in a cloud database |
| **CloudWatch Integration** | How RDS metrics are automatically tracked |

---

## 🎮 Test Yourself! (No Peeking 👀)

**Q1:** What does a DB Subnet Group actually control?

<details><summary>👀 Show answer</summary>

**A:** **Which Availability Zones** your database may be placed in (and which subnets it uses). Multi-AZ DBs survive a single-AZ outage. 🏘️

</details>

**Q2:** Which port does MySQL listen on, and where should you open it?

<details><summary>👀 Show answer</summary>

**A:** **3306** — and you should open it **only to your EC2 instance's security group** (or My IP for the lab), never to the world. 🔑

</details>

**Q3:** Why would a company pay for RDS instead of running MySQL on their own EC2 instance?

<details><summary>👀 Show answer</summary>

**A:** Because AWS handles **patching, backups, monitoring, and failover** automatically. No 3 AM backup alarms, no manual upgrades. Worth every penny. 💰

</details>

### 🔥 Bonus Challenge

Create a second table, insert a few rows with a `JOIN`-friendly foreign key, and run a `SELECT ... JOIN ...` query across both tables. You're not just clicking now — you're doing real database engineering in the cloud. 🗄️

> 💪 **Rithu:** *"A database is only as good as the queries you can write. Go JOIN something."

---

### 🆚 Pro Tip vs Noob Tip

| | Approach |
|---|---|
| **Noob Tip** | Set the DB security group to 0.0.0.0/0 "so I can connect from anywhere" |
| **Pro Tip** | Lock 3306 to your app server's security group only. Databases are the crown jewels |

---

## 🔗 What's Next?

You've mastered relational databases in the cloud. Now let's explore the NoSQL world:

➡️ **Lab 14 — DynamoDB: CRUD Operations** — Learn about AWS's serverless NoSQL database that scales automatically and has a generous Free Tier!

---

## ❓ Troubleshooting

<details>
<summary><strong>Click to expand Troubleshooting Section</strong></summary>

### "Can't connect to MySQL server" error

- Verify the RDS instance status is **Available** (not "Backing up" or "Creating")
- Check that `rds-sg` security group allows inbound on port 3306
- If connecting from your local machine, your home IP must be in the security group (or use `0.0.0.0/0`)
- Make sure you're using the full endpoint (not just the identifier)
- Check that "Public access" is set to **Yes**

### "Access denied for user 'admin'" error

- Double-check your master password (it's case-sensitive!)
- If you forgot the password, you can modify the RDS instance and set a new master password
- Make sure you're connecting as `admin`, not `root` or `mysql`

### RDS instance is stuck in "Creating" for more than 15 minutes

- This is unusual — check the **Events** tab for error messages
- You may have hit a limit (check Service Quotas → RDS)
- Try deleting and recreating with a different identifier
- Make sure your default VPC has subnets in at least 2 AZs

### "Public access" option is greyed out

- This happens when the subnet group only has private subnets
- Make sure your DB Subnet Group includes subnets that are in the default VPC (which should have public subnets)

### Storage is filling up quickly

- RDS doesn't auto-delete data — you need to manage storage manually
- For this lab, 20 GB should be more than enough
- In production, enable storage autoscaling with a maximum limit

</details>

> 🎉 **Incredible work, Ravi!** You just set up a production-grade MySQL database in the cloud. RDS handles all the boring stuff — patching, backups, monitoring — so you can focus on building awesome applications. Next, we'll explore the wild world of NoSQL with DynamoDB! 🚀

---

<div align="center">

<img src="https://img.shields.io/badge/Lab%2013-Complete!-27AE60?style=for-the-badge&labelColor=232F3E" />

</div>
