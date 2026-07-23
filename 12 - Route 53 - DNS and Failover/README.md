<div align="center">

<img src="https://img.shields.io/badge/Lab%2012-Route%2053%20DNS%20%26%20Failover-2980B9?style=for-the-badge&labelColor=232F3E" />

</div>

<div align="center">

<img src="https://img.shields.io/badge/Difficulty-Medium-F4D03F?style=flat-square" />
<img src="https://img.shields.io/badge/Time-~35min-3498DB?style=flat-square" />
<img src="https://img.shields.io/badge/Cost-%3C%243-2ECC71?style=flat-square" />
<img src="https://img.shields.io/badge/Service-Route%2053-8E44AD?style=flat-square" />

</div>

> "Ravi, every time you type google.com, DNS is working behind the scenes to find the right server. Today you become the DNS wizard. And yes, we'll also teach your domain to switch to a backup when the main server goes down — like a bat signal for your infrastructure!" — Rithu

---

## 🎯 Objective

In this lab, you will:

- Understand how **DNS** works and why Route 53 is AWS's DNS service
- Create a **Health Check** that monitors your server's availability
- Set up a **Simple Routing** record pointing to your EC2 instance
- Configure **Failover Routing** so traffic automatically switches to a backup server when the primary goes down
- Test the failover by deliberately breaking your primary server

---

## 🧠 Prerequisites

Before you start, make sure you have:

- ✅ Completed **Lab 11** (Auto Scaling Groups)
- ✅ An AWS account with Route 53 access
- ✅ At least one **running EC2 instance** with a public IP (you can launch a fresh t2.micro Amazon Linux 2023 if needed)
- ✅ Apache (httpd) installed and running on that EC2 instance
- ✅ Your key pair ready for SSH

---

## 💰 Cost Warning

| Resource | Cost |
|----------|------|
| Route 53 Health Checks | ~$0.50/month per health check (Free Tier: 100 health checks/month) |
| Hosted Zone | $0.50/month per zone |
| DNS Queries | ~$0.40 per million queries |
| **Domain Registration** | **~$12/year** (if you register a new domain) |

> ⚠️ **CRITICAL — Rithu says:** Domain registration costs ~$12 and **cannot be refunded or deleted** once registered. If you don't want to spend that, use **Option B** below. The learning is the same either way! Also, the $0.50 hosted zone fee is NOT covered by the Free Tier. Still cheap, but worth knowing.

### Choose Your Path:

- **Option A:** Register a real domain (~$12) — Great if you want your own `.com`!
- **Option B:** Skip domain registration and practice with health checks only (**Free**) — The failover concepts still apply!

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> I recommend Option B for your first time. You can always register a domain later. The important thing is understanding how DNS and failover routing work!

---

## 🏗️ Architecture

```
        User types: www.ravi-labs.com
                    │
                    ▼
        ┌──────────────────────┐
        │     Route 53 DNS     │
        │    Health Check      │
        │    monitors:80       │
        └──────────┬───────────┘
                   │
         ┌─────────┴─────────┐
         │   Failover Policy  │
         │                    │
    ┌────▼────┐        ┌────▼────┐
    │Primary  │  ──▶   │Secondary│
    │EC2 (OK) │        │EC2(BU) │
    │  :80    │        │  :80    │
    └─────────┘        └─────────┘

    If Primary health check fails →
    Route 53 returns Secondary IP!
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Prepare Your EC2 Instances

> <img src="https://img.shields.io/badge/Step%201-Prepare%20EC2%20Instances-3498DB?style=for-the-badge" />

You need **two EC2 instances** for failover routing — a primary and a secondary.

**Launch Instance 1 (Primary):**

1. Go to **EC2 Console** → **Launch Instance**
2. Configure:

| Field | Value |
|-------|-------|
| Name | `ravi-primary-server` |
| AMI | Amazon Linux 2023 |
| Instance type | t2.micro |
| Key pair | first-key-pair |
| Security group | Create new: `route53-sg` |
| Security group rules | HTTP (80) from Anywhere, SSH (22) from My IP |

3. Under **Advanced details** → **User data**, paste:

```bash
#!/bin/bash
yum install -y httpd
systemctl start httpd
echo "<h1>Hello from PRIMARY server!</h1><p>Instance: $(hostname)</p>" > /var/www/html/index.html
```

4. Click **Launch Instance**

**Launch Instance 2 (Secondary/Backup):**

1. Repeat the same steps but name it `ravi-backup-server`
2. Same AMI, same instance type, same security group
3. Use the same user data but change the message:

```bash
#!/bin/bash
yum install -y httpd
systemctl start httpd
echo "<h1>Hello from BACKUP server!</h1><p>This is the failover instance.</p>" > /var/www/html/index.html
```

4. Click **Launch Instance**

5. Wait for both instances to be **Running** and **2/2 status checks passed**

6. Note down both **Public IPs:**
   - Primary IP: `________________`
   - Backup IP: `________________`

📸 **[Screenshot: Two EC2 instances running — primary and backup]**

---

### Step 2: Create a Health Check

> <img src="https://img.shields.io/badge/Step%202-Create%20Health%20Check-2ECC71?style=for-the-badge" />

Route 53 uses health checks to know if your server is alive. Let's create one for the primary server.

1. Go to **Route 53 Console** → left sidebar → **Health checks**
2. Click **Create health check**
3. Configure:

| Field | Value |
|-------|-------|
| Health check name | `ravi-primary-health-check` |
| What to monitor | **IP address** |
| Protocol | **HTTP** |
| Port | **80** |
| IP address | Paste your **primary server's public IP** |
| Path | `/` |
| Request interval | **Fast (10 seconds)** |
| Failure threshold | **3** |

4. Click **Create health check**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "Fast" interval means Route 53 checks your server every 10 seconds. "Failure threshold 3" means it must fail 3 consecutive checks before marking the server as unhealthy. This prevents false alarms from one slow response!

5. Wait about 1 minute, then check the health check status. It should show **Healthy** (green)

📸 **[Screenshot: Health check showing "Healthy" status]**

---

### Step 3: Create a Hosted Zone (Option A Only)

> <img src="https://img.shields.io/badge/Step%203-Create%20Hosted%20Zone-E74C3C?style=for-the-badge" />

> ⏭️ **Skip this step if you chose Option B** (no domain registration)

If you registered a domain in Step 1:

1. Go to **Route 53** → **Hosted zones**
2. Your domain should already have a hosted zone. Click on it.
3. You'll see default NS and SOA records — those are normal

📸 **[Screenshot: Hosted zone with default records]**

---

### Step 4: Create a Simple DNS Record (Testing)

> <img src="https://img.shields.io/badge/Step%204-Create%20Simple%20DNS%20Record-F39C12?style=for-the-badge" />

Let's first point our domain to the primary server to make sure DNS is working.

> ⚠️ **Option B users:** Without a registered domain, you can't fully test DNS resolution. But you CAN still create health checks and understand the failover concepts by following along!

**If you have a domain (Option A):**

1. In your hosted zone, click **Create record**
2. Configure:

| Field | Value |
|-------|-------|
| Record name | `www` |
| Record type | **A** |
| Value | Primary server's public IP |
| TTL | **300** seconds |
| Routing policy | **Simple routing** |

3. Click **Create records**

4. Test DNS resolution:

```bash
nslookup www.your-domain.com
curl http://www.your-domain.com
```

You should see the primary server's response! 🎉

📸 **[Screenshot: nslookup showing the correct IP address]**

---

### Step 5: Set Up Failover Routing — The Real Deal!

> <img src="https://img.shields.io/badge/Step%205-Set%20Up%20Failover%20Routing-9B59B6?style=for-the-badge" />

Now let's set up automatic failover. This is where Route 53 really shines!

**Step 5a: Create the PRIMARY Failover Record**

1. Go to your hosted zone → **Create record**

| Field | Value |
|-------|-------|
| Record name | `www` |
| Record type | **A** |
| Value | **Primary** server's public IP |
| TTL | **300** |
| Routing policy | **Failover** |
| Failover record type | **Primary** |
| Health check | Select `ravi-primary-health-check` |
| Evaluate target health | **Yes** |

2. Click **Create records**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> The "Evaluate target health" option is crucial! It tells Route 53 to actually check if the primary server is healthy before sending traffic to it.

**Step 5b: Create the SECONDARY Failover Record**

1. Click **Create record** again

| Field | Value |
|-------|-------|
| Record name | `www` |
| Record type | **A** |
| Value | **Backup** server's public IP |
| TTL | **300** |
| Routing policy | **Failover** |
| Failover record type | **Secondary** |
| Health check | (Optional — leave empty or create a separate health check) |
| Evaluate target health | **Yes** |

2. Click **Create records**

📸 **[Screenshot: Route 53 records page showing both Primary and Secondary failover records]**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Notice both records have the SAME name (`www`) but different failover types (Primary vs Secondary). Route 53 knows to check Primary first, and only use Secondary when Primary is unhealthy. It's like having a plan B that automatically kicks in! 🔄

---

### Step 6: Test Failover — Let's Break the Primary! 💥

> <img src="https://img.shields.io/badge/Step%206-Test%20Failover-1ABC9C?style=for-the-badge" />

Time to simulate a disaster and watch Route 53 save the day!

**Step 6a: Verify Primary is Serving Traffic**

1. Open your browser: `http://www.your-domain.com` (or use the primary IP directly if Option B)
2. You should see: **"Hello from PRIMARY server!"**

**Step 6b: Kill the Primary Server's Web Service**

1. SSH into your **primary** server:

```bash
ssh -i "first-key-pair.pem" ec2-user@<PRIMARY_PUBLIC_IP>
```

2. Stop Apache:

```bash
sudo systemctl stop httpd
```

3. Verify it's down:

```bash
curl http://localhost
```

You should get an error — no response from the web server!

4. Exit the SSH session: `exit`

**Step 6c: Wait for Health Check to Fail**

1. Go to **Route 53** → **Health checks**
2. Watch `ravi-primary-health-check` status change:
   - It will go from **Healthy** → **Unhealthy** (after ~30 seconds = 3 checks × 10 seconds)
3. 📸 **[Screenshot: Health check showing "Unhealthy" status]**

**Step 6d: Test Failover**

1. Open a fresh browser tab: `http://www.your-domain.com`
2. You should now see: **"Hello from BACKUP server!"** 🎉

> 🎉 **Ravi, this is HUGE!** Your DNS automatically redirected traffic to the backup server without any manual intervention. In production, this means your users barely notice an outage!

**Step 6e: Test Failback (Primary Comes Back)**

1. SSH back into the **primary** server:

```bash
ssh -i "first-key-pair.pem" ec2-user@<PRIMARY_PUBLIC_IP>
```

2. Start Apache:

```bash
sudo systemctl start httpd
```

3. Exit SSH: `exit`

4. Wait 1-2 minutes for the health check to recover
5. Check Route 53 health check — it should show **Healthy** again
6. Open `http://www.your-domain.com` — you should see **"Hello from PRIMARY server!"** again!

📸 **[Screenshot: Health check back to "Healthy" and primary serving traffic]**

---

### Step 7: Verify Your Work

> <img src="https://img.shields.io/badge/Step%207-Verify%20Your%20Work-E67E22?style=for-the-badge" />

Let's make sure everything is set up correctly:

- [ ] Two EC2 instances are running (primary and backup)
- [ ] Health check exists and shows appropriate status
- [ ] Route 53 records: Two A records with same name, different failover types
- [ ] Primary record points to primary IP with the health check attached
- [ ] Secondary record points to backup IP
- [ ] Stopping httpd on primary causes health check to fail
- [ ] After health check fails, DNS returns backup IP
- [ ] Restarting httpd on primary causes health check to recover
- [ ] After health check recovers, DNS returns primary IP

📸 **[Screenshot: Route 53 records overview showing both failover records]**

---

## ✅ Validation Checklist

- [ ] Two EC2 instances running with different web pages (primary vs backup)
- [ ] Security group allows HTTP (80) and SSH (22)
- [ ] Health check created and monitoring primary server on port 80
- [ ] Simple routing record working (if you have a domain)
- [ ] Primary failover record created with health check attached
- [ ] Secondary failover record created pointing to backup server
- [ ] Stopping httpd on primary → health check fails → traffic goes to backup
- [ ] Restarting httpd on primary → health check recovers → traffic returns to primary

---

## 🧹 Cleanup (IMPORTANT!)

> ⚠️ **Domain registrations cannot be deleted!** They expire after 1 year. If you registered a domain, just let it expire or disable auto-renew.

1. **Delete Route 53 Records:**
   - Go to **Route 53** → **Hosted zones** → click your domain
   - Select the `www` record(s) → **Delete**
   - Type `delete` to confirm
   - Delete ALL records you created (keep the default NS and SOA if the domain is registered)

2. **Delete Health Check:**
   - Go to **Route 53** → **Health checks**
   - Select `ravi-primary-health-check`
   - Click **Delete** → Confirm

3. **Delete Hosted Zone (if you created one separately):**
   - Go to **Route 53** → **Hosted zones**
   - Select the zone → **Delete hosted zone**
   - Type the domain name to confirm

4. **Terminate EC2 Instances:**
   - Go to **EC2** → **Instances**
   - Select `ravi-primary-server` and `ravi-backup-server`
   - Click **Instance state** → **Terminate instance**
   - Confirm

5. **Delete Security Group:**
   - Go to **EC2** → **Security Groups**
   - Select `route53-sg` → **Actions** → **Delete security group**
   - Confirm

📸 **[Screenshot: All Route 53 resources deleted, EC2 instances terminated]**

---

## 🎓 What You Learned

| Concept | What You Now Know |
|---------|-------------------|
| **DNS Basics** | How domain names get translated to IP addresses |
| **Route 53** | AWS's highly available DNS service and how to use it |
| **Health Checks** | How Route 53 monitors your servers and detects failures |
| **Simple Routing** | How to create a basic DNS record pointing to an IP |
| **Failover Routing** | How to set up automatic DNS failover when a server goes down |
| **Primary/Secondary** | How Route 53 prioritizes records and switches between them |
| **Failback** | How traffic returns to the primary server after it recovers |

---

## 🔗 What's Next?

Now that you can direct traffic to instances, let's learn about managing databases in the cloud:

➡️ **Lab 13 — RDS: MySQL on AWS** — Learn how to run a managed MySQL database in the cloud without worrying about patches, backups, or infrastructure!

---

## ❓ Troubleshooting

<details>
<summary><strong>Click to expand Troubleshooting Section</strong></summary>

### My health check stays "Unhealthy" even though the server is running

- Verify the security group allows inbound HTTP (port 80) from anywhere (`0.0.0.0/0`)
- Check that Apache is actually running: SSH in and run `sudo systemctl status httpd`
- Make sure the health check IP matches the primary server's public IP exactly
- Try changing the health check path to something simpler like `/`
- Wait at least 1 minute for the status to update

### DNS resolution is not changing after failover

- DNS caching! Your computer may be caching the old IP
- Flush your DNS cache:
  - Windows: `ipconfig /flushdns`
  - Mac: `sudo dscacheutil -flushcache`
- Try using `nslookup` or `dig` instead of browser (browsers cache more aggressively)
- The TTL is set to 300 seconds — wait that long for caches to expire

### I can't delete my hosted zone

- Make sure all non-default records are deleted first (NS and SOA are default for registered domains)
- You cannot delete the hosted zone for a registered domain — it's automatically managed
- For manually created hosted zones, delete all custom records first, then delete the zone

### Failover doesn't switch — both records return Primary

- Check that the health check is actually attached to the Primary record
- Verify "Evaluate target health" is set to "Yes" on both records
- Make sure you stopped httpd (not just the instance) — `sudo systemctl stop httpd`
- The health check needs to fail 3 consecutive times (30 seconds with Fast interval)

### My secondary server doesn't show the backup page

- Make sure you used the correct user data when launching the backup instance
- SSH into the backup and check: `curl http://localhost`
- If it shows the default page, restart httpd: `sudo systemctl restart httpd`

</details>

> 🎉 **Fantastic work, Ravi!** DNS and failover routing are fundamental to building resilient applications. You now know how to automatically redirect traffic when something goes wrong — a skill that separates beginners from real cloud engineers. Next up, let's tackle databases! 🚀

---

<div align="center">

<img src="https://img.shields.io/badge/Lab%2012-Complete!-27AE60?style=for-the-badge&labelColor=232F3E" />

</div>
