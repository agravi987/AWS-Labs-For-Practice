<div align="center">

<img src="https://img.shields.io/badge/%F0%9F%9A%80%20Lab%2001-EC2%20Launch%20%26%20Connect-FF9900?style=for-the-badge&labelColor=232F3E" />

# ⚡ Amazon EC2: Launch & Connect

### Your First Cloud Server — From Zero to SSH in 30 Minutes!

<img src="https://img.shields.io/badge/Difficulty-🟢%20Easy-2ECC71?style=flat-square" />
<img src="https://img.shields.io/badge/Time-~30%20min-3498DB?style=flat-square" />
<img src="https://img.shields.io/badge/Cost-<%241-95A5A6?style=flat-square" />
<img src="https://img.shields.io/badge/Service-EC2-FF9900?style=flat-square" />

</div>

---

> ### 🗣️ *"Ravi, every cloud engineer's journey starts with launching a virtual server. It's like the 'Hello World' of AWS. Let's get your first EC2 instance up and running!"*
> — **Rithu** ☁️

---

<div align="center">

## 📊 Lab Progress

`[██░░░░░░░░░░░░░░░░░░] 5% — Let's Begin!`

</div>

---

## 🎯 What You'll Accomplish

By the end of this lab, you'll have **your own virtual server running in the cloud** — accessible from your laptop, serving a real webpage to the entire internet.

> <img src="https://img.shields.io/badge/🧠-What%20You'll%20Learn-8E44AD?style=flat-square" />
>
> ✅ Launch an EC2 instance from the AWS Console
> ✅ Create & manage SSH key pairs
> ✅ Connect to your server via SSH
> ✅ Install Apache httpd web server
> ✅ Host a custom webpage — live on the internet!

---

## 🧠 Prerequisites

| Requirement | Details |
|:---:|---|
| ☁️ | An **AWS account** with administrative access |
| 🖥️ | Basic familiarity with the AWS Management Console |
| 🎓 | No prior EC2 experience needed — this is ground zero! |

---

## 💸 Cost Warning

<table>
<tr>
<td width="50" align="center"><h1>💰</h1></td>
<td>

All resources in this lab are **Free Tier eligible** (t2.micro). However, as our favorite greenhorn once said:

> *"I forgot to terminate my instance and got a surprise **$15 AWS bill** the next month."*

**Never forget to terminate.** Seriously. There's a Cleanup section at the bottom. **USE IT.**

</td>
</tr>
</table>

---

## 🏗️ Architecture Overview

```
╔══════════════════════════════════════════════════════╗
║                    PUBLIC SUBNET                      ║
║                                                      ║
║   ┌──────────────────────────────────────────────┐   ║
║   │            🖥️  t2.micro EC2                  │   ║
║   │         Amazon Linux 2023 AMI                 │   ║
║   │                                              │   ║
║   │   🔐 Security Group:                         │   ║
║   │      SSH (22) ──▶ My IP Only                 │   ║
║   │      HTTP (80) ──▶ 0.0.0.0/0                │   ║
║   │                                              │   ║
║   │   🌐 httpd installed & running               │   ║
║   │   📄 index.html served on port 80            │   ║
║   └──────────────────────┬───────────────────────┘   ║
║                          │                            ║
║                    ┌─────▼─────┐                      ║
║                    │    IGW    │ ◀── Internet Gateway  ║
║                    └─────┬─────┘                      ║
║                          │                            ║
╚══════════════════════════╪════════════════════════════╝
                           │
                    ┌──────▼──────┐
                    │  🌍 Browser │
                    │  http://<IP>│
                    └─────────────┘
```

---

## 🛠️ Step-by-Step Walkthrough

---

### Step 1: Launch the EC2 Instance

> <img src="https://img.shields.io/badge/Step%201-🚀%20Launch%20Instance-27AE60?style=for-the-badge" />

**1.** Go to the **AWS Management Console** → search for **EC2** → click it.

**2.** Click the orange **`Launch instance`** button.

<div align="center">
<img src="screenshots/01-ec2-dashboard-launch-button.png" width="700" alt="EC2 Dashboard showing the Launch instance button" />
<br/>
<sub>📸 EC2 Dashboard — Click that beautiful orange button!</sub>
</div>

<br/>

**3.** **Name your instance:** Enter `first-ec2-instance`

**4.** **Choose your OS:**
- Click **Amazon Linux** (usually the default)
- Select **Amazon Linux 2023 AMI** *(Free Tier eligible ✅)*

**5.** **Instance type:** Select **t2.micro**
> That little asterisk saying "Free tier eligible" is like winning the lottery 🎰

**6.** **Key pair (your door key):**
- Click **Create new key pair**
- Name: `first-key-pair` | Type: **RSA** | Format: **.pem**
- Click **Create key pair** — the `.pem` file downloads automatically

> <img src="https://img.shields.io/badge/💡-Rithu's%20Tip-FFC300?style=flat-square" />
> Move that `.pem` file to `~/.ssh/` on Mac/Linux or a dedicated folder on Windows.
> **Don't lose it. You can't SSH without it.** 🔑

**7.** **Network settings →** Click **Edit:**
- VPC & Subnet: leave as default
- **Auto-assign public IP:** ✅ Enable
- **Firewall:** Select "Create security group"
  - Name: `launch-wizard-1`
  - Click **Add security group rule:**
    - Type: **SSH** | Source: **My IP** | Description: `SSH from my IP`

**8.** **Storage:** Leave default 8 GiB gp2/gp3

**9.** Click **`Launch instance`** at the bottom → **View all instances**

<div align="center">
<img src="screenshots/02-instance-running-with-status-checks.png" width="700" alt="Instance list showing first-ec2-instance with status checks" />
<br/>
<sub>📸 Your first EC2 instance — running and healthy! 🎉</sub>
</div>

---

### Step 2: Wait for Instance Ready

> <img src="https://img.shields.io/badge/Step%202-⏳%20Wait%20for%20Ready-3498DB?style=for-the-badge" />

- Instance shows **Pending** → then **Running**
- Wait for **2/2 status checks** to pass (refresh every 30s)

```
┌──────────────────────┬──────────┬───────────┬────────────┐
│ Instance State       │ Status   │ Checks    │ Public IP  │
├──────────────────────┼──────────┼───────────┼────────────┤
│ ✅ Running           │ 2/2 OK   │  ✅ PASS  │ 54.x.x.x  │
└──────────────────────┴──────────┴───────────┴────────────┘
```

---

### Step 3: Find Your Public IP

> <img src="https://img.shields.io/badge/Step%203-🔍%20Find%20IP-E67E22?style=for-the-badge" />

1. Click `first-ec2-instance` in the list
2. Go to **Details** tab
3. Copy the **Public IPv4 address** — you'll need this!
4. Also note the **Public IPv4 DNS** — it works too

---

### Step 4: Connect via SSH

> <img src="https://img.shields.io/badge/Step%204-🔐%20SSH%20Connect-8E44AD?style=for-the-badge" />

Pick **ONE** option based on your OS:

---

<details>
<summary><b>🍎 Option A: Mac / Linux — Terminal</b></summary>

```bash
# 1. Lock down key permissions (SSH is paranoid — and it should be!)
chmod 400 /path/to/your/first-key-pair.pem

# 2. SSH into your cloud server!
ssh -i /path/to/your/first-key-pair.pem ec2-user@<YOUR-PUBLIC-IP>
```

When prompted:
```
The authenticity of host 'xx.xx.xx.xx' can't be established.
Are you sure you want to continue connecting (yes/no)?
```
Type `yes` and press Enter.

You should see:
```
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
[ec2-user@ip-xxx-xxx-xxx-xxx ~]$
```

> <img src="https://img.shields.io/badge/💡-Rithu's%20Tip-FFC300?style=flat-square" />
> If you see `Permissions 0644 are too open`, you forgot `chmod 400`. Fix it!

</details>

---

<details>
<summary><b>🪟 Option B: Windows — PuTTY</b></summary>

PuTTY uses `.ppk` files, not `.pem`. Convert first:

1. Open **PuTTYgen** → click **Load** → select your `.pem` file
2. Click **OK** → **Save private key** → **Yes** (no passphrase)
3. Save as `first-key-pair.ppk`

Now connect with PuTTY:

| Setting | Value |
|---------|-------|
| Host Name | `ec2-user@<YOUR-PUBLIC-IP>` |
| Port | `22` |
| Connection type | SSH |
| Auth → Private key | `first-key-pair.ppk` |

Click **Open** → **Accept** the fingerprint → You're in! 🎉

</details>

---

<details>
<summary><b>🌐 Option C: AWS EC2 Instance Connect (Browser)</b></summary>

The easiest option — no setup needed:

1. Select your instance → **Connect** button
2. Go to **EC2 Instance Connect** tab
3. Click **Connect**
4. New browser tab opens with a terminal — magic! ✨

> <img src="https://img.shields.io/badge/💡-Rithu's%20Tip-FFC300?style=flat-square" />
> Great for quick tasks, but falls flat in private subnets.
> **Learn the CLI methods — they'll save you someday!**

</details>

---

### Step 5: Install Apache Web Server

> <img src="https://img.shields.io/badge/Step%205-🌐%20Install%20httpd-E74C3C?style=for-the-badge" />

You're inside your EC2 instance now. Let's install Apache:

```bash
# Update all packages (always do this first!)
sudo yum update -y
```

```bash
# Install the Apache web server
sudo yum install -y httpd
```

```bash
# Start the web server
sudo systemctl start httpd
```

```bash
# Enable auto-start on reboot (survives restarts!)
sudo systemctl enable httpd
```

```bash
# Verify it's running
sudo systemctl status httpd
```

You should see: **`active (running)`** — the green light! 🟢

---

### Step 6: Create Your Custom Web Page

> <img src="https://img.shields.io/badge/Step%206-📝%20Custom%20Page-16A085?style=for-the-badge" />

```bash
echo "<h1>Hello from Ravi's first EC2 instance!</h1>" | sudo tee /var/www/html/index.html
```

> <img src="https://img.shields.io/badge/💡-Rithu's%20Tip-FFC300?style=flat-square" />
> `tee` writes to a file AND prints to terminal — like a microphone + speaker.
> `sudo` is needed because `/var/www/html/` is owned by `root`.

---

### Step 7: Verify Your Work 🎉

> <img src="https://img.shields.io/badge/Step%207-🎉%20Verify%20It!-F39C12?style=for-the-badge" />

1. Open your browser
2. Go to: **`http://<YOUR-PUBLIC-IP>`**
3. **You should see:** `Hello from Ravi's first EC2 instance!`

> <img src="https://img.shields.io/badge/💡-Rithu's%20Tip-FFC300?style=flat-square" />
> Notice it's `http://` NOT `https://`. SSL comes later. Patience, grasshopper. 🦗

---

<details>
<summary><b>⚠️ Page not loading? Click here!</b></summary>

**Most likely cause:** Your security group doesn't have an HTTP rule!

**Fix it now:**
1. EC2 Console → Security Groups → select your SG
2. **Inbound rules** → **Edit inbound rules** → **Add rule:**
   - Type: **HTTP** | Source: **0.0.0.0/0** | Description: `HTTP from anywhere`
3. **Save rules** → Refresh browser → **Magic!** ✨

</details>

---

## ✅ Validation Checklist

Before moving on, confirm ALL of these:

- [ ] 🖥️ EC2 instance launched from Amazon Linux 2023 AMI
- [ ] 🔑 Key pair `first-key-pair` created and downloaded
- [ ] 🔐 SSH connection established to the instance
- [ ] 🌐 Apache httpd installed and running (`active (running)`)
- [ ] 📄 Custom `index.html` visible at `http://<public-ip>`
- [ ] 🔥 HTTP inbound rule added to security group

---

## 🧹 Cleanup (DO NOT SKIP!)

> <img src="https://img.shields.io/badge/⚠️-CLEANUP%20OR%20PAY!-E74C3C?style=for-the-badge" />

Don't skip this. **Future Ravi will thank you.**

### 1. Terminate the Instance

EC2 Console → Instances → `first-ec2-instance` → **Instance state** → **Terminate**

### 2. Delete the Key Pair

EC2 Console → Network & Security → **Key Pairs** → `first-key-pair` → **Delete**

Also delete the `.pem` / `.ppk` file from your computer.

### 3. Clean Up Security Group

EC2 Console → **Security Groups** → select your SG → **Delete security groups**

> <img src="https://img.shields.io/badge/💡-Rithu's%20Tip-FFC300?style=flat-square" />
> You may need to wait for instance termination to complete before deleting the SG.
> Get in the muscle memory early! 💪

---

## 🎓 What You Learned

<table>
<tr><th>Concept</th><th>Key Takeaway</th></tr>
<tr><td>🖥️ EC2 Launch</td><td>t2.micro is the Free Tier king</td></tr>
<tr><td>🔑 Key Pairs</td><td>RSA .pem for Mac/Linux, convert to .ppk for PuTTY</td></tr>
<tr><td>🔥 Security Groups</td><td>Stateful firewall — allow only what's needed</td></tr>
<tr><td>🔐 SSH Methods</td><td>Mac/Linux CLI, Windows PuTTY, EC2 Instance Connect</td></tr>
<tr><td>🌐 Apache httpd</td><td>Install → start → enable for automatic boot</td></tr>
<tr><td>📝 User Data</td><td>Not covered here, but remember: scripts run at launch!</td></tr>
</table>

---

## 🔗 What's Next?

> <img src="https://img.shields.io/badge/➡️-Next%20Lab-2ECC71?style=for-the-badge" />

This was the warm-up! Next up, we play with **firewalls**.

👉 **[Lab 02 — EC2 Security Groups Deep Dive](../02%20-%20EC2%20-%20Security%20Groups%20Deep%20Dive/README.md)**

We'll lock down traffic, open ports on-demand, and reference security groups within each other.
It'll be like being a **cloud bouncer**. 🫡

---

## ❓ Troubleshooting

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| `Permission denied (publickey)` | Wrong key or permissions | `chmod 400` on `.pem`, verify key name |
| PuTTY: `No supported auth algorithms` | `.pem` used directly | Convert to `.ppk` with PuTTYgen |
| Website won't load | Missing HTTP rule in SG | Add inbound HTTP (80) from `0.0.0.0/0` |
| Connection timeout | Wrong IP or instance down | Verify Public IP, check instance state |
| `sudo: yum: command not found` | Amazon Linux 2023 uses dnf | Try `sudo dnf install -y httpd` |
| Browser shows default Apache page | Custom index.html failed | Re-run the `echo \| sudo tee` command |

---

<div align="center">

### 🏆 Lab Complete!

<img src="https://img.shields.io/badge/✅-Lab%2001%20DONE!-2ECC71?style=for-the-badge" />

---

*Built with ☕ and a lot of patience — Rithu* ☁️

</div>
