<div align="center">

<img src="https://img.shields.io/badge/Lab%2015-CloudWatch%20Alarms%20%26%20Dashboards-E67E22?style=for-the-badge&labelColor=232F3E" />

# Lab 15 — CloudWatch: Alarms and Dashboards — Your Infrastructure's Control Center

<img src="https://img.shields.io/badge/Difficulty-Easy-green?style=flat-square" />
<img src="https://img.shields.io/badge/Time-~25_min-blue?style=flat-square" />
<img src="https://img.shields.io/badge/Cost-<%241-green?style=flat-square" />
<img src="https://img.shields.io/badge/Service-CloudWatch-orange?style=flat-square" />

</div>

> "Ravi, you've built amazing infrastructure in the last 14 labs. But how do you know when something breaks at 3 AM? CloudWatch is your 24/7 security camera, fire alarm, and health monitor all rolled into one. Let's set it up!" — Rithu

---

<details>
<summary><b>🎭 Ravi & Rithu's Coffee Break Chat</b></summary>

**Ravi:** "CloudWatch is like a security camera for my servers?"

**Rithu:** "More like a security camera + fitness tracker + health monitor + alarm system."

**Ravi:** "That's a lot of features."

**Rithu:** "That's a lot of servers to keep track of. Trust me, you want this."

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
- [🆚 Pro Tip vs Noob Tip](#-pro-tip-vs-noob-tip)
- [🔗 What's Next?](#-whats-next)
- [❓ Troubleshooting](#-troubleshooting)

---

<div align="center">

## 📊 Lab Progress

`[███████████████░░░░░] 75% — Almost at the Finish Line!`

</div>

---

## 🤔 In Plain English

> **What is this, really?** CloudWatch is the **heart monitor + security cameras + black box** of your AWS account. It collects metrics (CPU, disk I/O, network) from your resources, raises alarms when something is wrong ("CPU > 80%!"), shows everything on a central dashboard (your mission control), and centralizes application logs for post-mortems. 📊
>
> 🌍 **Why you should care:** You can't fix what you can't see. CloudWatch is how cloud engineers discover *before* customers do that something is burning.

---

## 🎯 Objective

In this lab, you will:

- Explore **CloudWatch Metrics** using the updated AWS Console layout
- Create a **CloudWatch Alarm** using the modern 4-step wizard that notifies you when CPU usage is high
- Trigger the alarm using **stress-ng** or a high-CPU script on Amazon Linux 2023
- Build a **Custom Dashboard** with multiple widget types (Line chart, Number, Alarm status, Text header)
- Create a **CloudWatch Log Group**, send test events, and explore **Logs Insights** & **Live Tail**
- Understand why monitoring is non-negotiable in cloud engineering

---

## 🧠 Prerequisites

Before you start, make sure you have:

- ✅ Completed **Lab 14** (DynamoDB)
- ✅ An AWS account with CloudWatch & EC2 access
- ✅ A running **EC2 instance** (t2.micro / t3.micro with Amazon Linux 2023)
- ✅ An active email address for receiving SNS alarm notifications
- ✅ Your SSH key pair ready

---

## 💰 Cost Warning

| Resource | Cost |
|----------|------|
| CloudWatch Metrics (basic) | Free for 10 metrics |
| CloudWatch Alarms | ~$0.10/alarm/month |
| CloudWatch Dashboards | ~$3.00/dashboard/month |
| CloudWatch Logs (first 5 GB) | Free |
| SNS Notifications | Free for email subscriptions |
| **Estimated total** | **< $1 for this lab** |

> ⚠️ **Rithu says:** CloudWatch Dashboards cost ~$3/month. That's fine for learning, but remember to delete it after the lab! The first 10 basic metrics and 5 GB of logs are free. The alarm itself costs ~$0.10/month — cheaper than a gumball! 🫧

> **Ravi's Mistake of the Day:** I created a CloudWatch alarm but forgot to confirm the SNS subscription in my email. The alarm fired beautifully... to an empty void. Nobody was notified and the server died in silence.

---

## 🏗️ Architecture

```
┌───────────────────────────────────────────────────────────────┐
│                        CloudWatch                              │
│                                                                │
│  ┌─────────────┐   ┌──────────────┐   ┌──────────────────┐    │
│  │   Metrics   │   │    Alarms    │   │    Dashboards    │    │
│  │             │   │  (4-Step Wizard)│ │                  │    │
│  │ CPUUtil %   │──▶│ CPU > 80%?   │──▶│ Line chart,      │    │
│  │ NetworkIn   │   │ Yes → Alert! │   │ Number widgets,  │    │
│  │ NetworkOut  │   │              │   │ Alarm status     │    │
│  │ StatusCheck │   └──────┬───────┘   └──────────────────┘    │
│  └─────────────┘          │                                    │
│                           ▼                                    │
│                   ┌──────────────┐                             │
│                   │  SNS Topic   │                             │
│                   │ "ec2-cpu-"   │                             │
│                   │  "alerts"    │                             │
│                   └──────┬───────┘                             │
│                          │                                     │
│                          ▼                                     │
│                   ┌──────────────┐                             │
│                   │ 📧 Email     │                             │
│                   │ Notification │                             │
│                   └──────────────┘                             │
└───────────────────────────────────────────────────────────────┘
```

> **Did You Know?** CloudWatch can monitor over 70 AWS services and custom metrics. You can even monitor your on-premises servers with the CloudWatch Agent.

---

## 🛠️ Step-by-Step Instructions

> <img src="https://img.shields.io/badge/Step%201-Launch%20a%20Test%20EC2%20Instance-2ECC71?style=for-the-badge" />

If you don't already have a running EC2 instance, let's launch one!

1. Open the **EC2 Console** → **Launch Instance**
2. Configure:

| Field | Value |
|-------|-------|
| Name | `cloudwatch-test-ec2` |
| AMI | Amazon Linux 2023 |
| Instance type | t2.micro (or t3.micro) |
| Key pair | Select your existing key pair |
| Network settings | Select default VPC, Auto-assign Public IP: Enable |
| Security group | Create new: `cloudwatch-sg` |
| Inbound rules | SSH (22) from My IP, HTTP (80) from Anywhere |

3. Under **Advanced details** → **User data**, paste:

```bash
#!/bin/bash
dnf update -y
dnf install -y httpd stress-ng || dnf install -y httpd stress
systemctl start httpd
systemctl enable httpd
```

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Amazon Linux 2023 uses `stress-ng` in its repositories! We pre-install `stress-ng` in user data so we can easily generate CPU load later. 🚀

4. Click **Launch Instance**
5. Wait until the instance state is **Running** and **2/2 status checks passed**.

📸 **[Screenshot: EC2 instance cloudwatch-test-ec2 in running state]**
![EC2 instance cloudwatch-test-ec2 in running state](image.png)

---

> <img src="https://img.shields.io/badge/Step%202-Explore%20CloudWatch%20Metrics%20(Modern%20Console)-3498DB?style=for-the-badge" />

Let's see how CloudWatch tracks metrics for your EC2 instance in the updated AWS Console!

1. Open the **CloudWatch Console**
2. In the left navigation sidebar, expand **Metrics** → click **All metrics**
3. Notice the top tabs in the main pane: **Browse**, **Graphed metrics**, and **Query**.
4. In the **Browse** tab under **AWS namespaces**, click **EC2**
5. Click **Per-Instance Metrics**
6. Locate your instance `cloudwatch-test-ec2`:
   - Check the box next to **CPUUtilization** (How busy the CPU is, 0–100%)
   - Take note of other metrics: **NetworkIn**, **NetworkOut**, **StatusCheckFailed**
7. Look at the graph panel at the top:
   - Adjust the time range (e.g., select **1h** or **3h**)
   - Notice the **Period** (default is 5 minutes for basic monitoring)

📸 **[Screenshot: CloudWatch metrics page showing CPU utilization graph for the EC2 instance]**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> **Hypervisor vs OS Metrics:** EC2 automatically sends hypervisor-level metrics (CPU utilization, Network bytes, Disk I/O) every 5 minutes for FREE. However, AWS cannot look inside your operating system's RAM or disk space usage! If you need **Memory (RAM) Utilization** or **Free Disk Space %**, you must install the **CloudWatch Agent** inside the OS. 💡

---

> <img src="https://img.shields.io/badge/Step%203-Create%20a%20CloudWatch%20Alarm%20(4--Step%20Wizard)-E67E22?style=for-the-badge" />

Now, let's create a CloudWatch Alarm using the modern 4-step wizard to notify you when CPU usage exceeds 80%!

1. In the left sidebar of the **CloudWatch Console**, click **Alarms** → **All alarms**
2. Click the orange **Create alarm** button

### 🔹 Step 3a: Step 1 of Wizard — Specify Metric and Conditions

1. Click **Select metric**
2. Choose **EC2** → **Per-Instance Metrics**
3. Search for `cloudwatch-test-ec2` and select **CPUUtilization**
4. Click **Select metric**
5. Configure conditions:

| Setting | Value |
|---------|-------|
| Threshold type | **Static** |
| Whenever CPUUtilization is... | **Greater** (`>`) |
| than... | `80` |
| **Additional configuration** | |
| Datapoints to alarm | **1 out of 1** |
| Period | **5 minutes** |
| Missing data treatment | **Treat missing data as missing** |

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "1 out of 1 consecutive period" means the alarm fires as soon as a single 5-minute evaluation window shows average CPU > 80%. In production, engineers often use "3 out of 5" to prevent false alarms caused by short temporary spikes.

6. Click **Next**

### 🔹 Step 3b: Step 2 of Wizard — Configure Actions

1. Under **Alarm state trigger**, select **In alarm**
2. Under **Send a notification to...**, select **Create new topic** (or choose existing SNS topic if created previously):

| Field | Value |
|-------|-------|
| Create a new topic... | `ec2-cpu-alerts` |
| Email endpoints that will receive notification... | `your-real-email@example.com` |

3. Click **Create topic**
4. ⚠️ **CRITICAL STEP:** Open your email inbox immediately! Look for an email from `AWS Notifications <no-reply@sns.amazonaws.com>` with subject *"AWS Notification - Subscription Confirmation"*. Click **Confirm subscription**.

> 🛑 **If you skip confirming the SNS subscription, you will NEVER receive alarm emails!**

5. Click **Next**

### 🔹 Step 3c: Step 3 of Wizard — Add Name and Description

1. **Alarm name:** `ec2-cpu-alarm`
2. **Alarm description:** `Alert when EC2 CPU exceeds 80%`
3. Click **Next**

### 🔹 Step 3d: Step 4 of Wizard — Preview and Create

1. Review your configured metric, threshold (> 80%), and SNS notification settings.
2. Click **Create alarm** at the bottom right.

📸 **[Screenshot: Alarm creation page showing all configured settings]**

---

> <img src="https://img.shields.io/badge/Step%204-Trigger%20the%20Alarm-E74C3C?style=for-the-badge" />

Let's stress the EC2 CPU and watch the alarm move into **In alarm** state!

1. SSH into your EC2 instance:

```bash
ssh -i "your-key.pem" ec2-user@<PUBLIC_IP>
```

2. Run `stress-ng` to push CPU usage to 100%:

```bash
# If stress-ng is installed:
stress-ng --cpu 4 --timeout 600s

# Or if stress was installed:
stress --cpu 4 --timeout 600s
```

> 💡 **Fallback (No package required):** If for any reason package installation was skipped, run this Python one-liner to max out CPU:
> ```bash
> python3 -c "import multiprocessing as m; [m.Process(target=lambda: [0 for _ in iter(int, 1)]).start() for _ in range(m.cpu_count()*2)]"
> ```

3. Monitor CPU usage in CloudWatch:
   - Navigate to **CloudWatch Console** → **Metrics** → **All metrics** → **Browse** → **EC2** → **Per-Instance Metrics**
   - Check **CPUUtilization** graph — you will see a sharp spike up to 100%! 📈

4. Watch the Alarm State:
   - Go to **CloudWatch** → **Alarms** → **All alarms**
   - Observe the state transition: 🟢 **OK** → 🟡 **Insufficient data** → 🔴 **In alarm**
   - After 5–10 minutes, check your email inbox — you will receive an SNS alert email! 📧

📸 **[Screenshot: Alarm showing "In alarm" state in the CloudWatch console]**

5. Once verified, stop the stress tool (press `Ctrl + C` in SSH or terminate the process).

---

> <img src="https://img.shields.io/badge/Step%205-Create%20a%20Custom%20Dashboard-9B59B6?style=for-the-badge" />

Let's build a central cockpit for your infrastructure using CloudWatch Dashboards!

1. Go to **CloudWatch Console** → left sidebar → **Dashboards**
2. Click **Create dashboard**
3. Dashboard name: `Ravi-Labs-Dashboard`
4. Click **Create dashboard**

Now let's add 4 different widgets:

#### 📊 Widget 1: Line Chart (CPU Utilization)
- In the **Add widget** modal, select **Line** → Click **Next**
- Choose **Metrics** → Click **Next**
- Browse to **EC2** → **Per-Instance Metrics** → select `CPUUtilization` for `cloudwatch-test-ec2`
- Click **Create widget**

#### 🔢 Widget 2: Number Widget (Network In)
- Click the **+** (Add widget) button at top right
- Select **Number** → Click **Next** → Choose **Metrics**
- Browse to **EC2** → **Per-Instance Metrics** → select `NetworkIn`
- Click **Create widget**

#### 🚨 Widget 3: Alarm Status Widget
- Click **Add widget** → Select **Alarm status** → Click **Next**
- Check the box for your alarm: `ec2-cpu-alarm`
- Click **Create widget**

#### 📝 Widget 4: Text Widget (Dashboard Title Header)
- Click **Add widget** → Select **Text** → Click **Next**
- Paste Markdown content:
  ```markdown
  # 🚀 Mission Control Dashboard
  **Owner:** Ravi | **Environment:** Lab 15 Test | **Status:** Active Monitoring
  ```
- Click **Create widget**

5. Drag and resize widgets to arrange your ideal layout.
6. Click **Save dashboard** at the top right!

📸 **[Screenshot: Complete Ravi-Labs-Dashboard with all 4 widgets visible]**

---

> <img src="https://img.shields.io/badge/Step%206-Explore%20CloudWatch%20Logs,%20Insights%20%26%20Live%20Tail-F39C12?style=for-the-badge" />

Now let's explore **CloudWatch Logs**, **Logs Insights**, and **Live Tail**!

1. Go to **CloudWatch Console** → left sidebar → **Logs** → **Log groups**
2. Click **Create log group**
3. Configure:

| Field | Value |
|-------|-------|
| Log group name | `ravi-app-logs` |
| Retention setting | **7 days** (Avoid keeping logs forever to minimize costs) |

4. Click **Create**

5. Click on `ravi-app-logs` → Click **Create log stream** → Name: `test-stream` → Click **Create**.

6. Send a test log entry using AWS CLI (or AWS CloudShell):

```bash
aws logs put-log-events \
  --log-group-name ravi-app-logs \
  --log-stream-name test-stream \
  --log-events timestamp=$(date +%s000),message="[INFO] Lab 15 CloudWatch logging successfully configured by Ravi!"
```

7. **Explore Logs Insights (SQL-like log querying):**
   - In left menu under **Logs**, click **Logs Insights**
   - Select `ravi-app-logs` in the log group dropdown
   - Run the default query:
     ```sql
     fields @timestamp, @message
     | sort @timestamp desc
     | limit 20
     ```
   - Click **Run query** to view formatted log results!

8. **Explore Live Tail (Real-time log streaming):**
   - In left menu under **Logs**, click **Live Tail**
   - Select `ravi-app-logs` to watch incoming log events in real time.

📸 **[Screenshot: CloudWatch Logs showing the test log message]**

---

> <img src="https://img.shields.io/badge/Step%207-Verify%20Your%20Work-1ABC9C?style=for-the-badge" />

- [ ] EC2 instance `cloudwatch-test-ec2` launched & running
- [ ] CloudWatch Metrics viewed under **Browse** tab
- [ ] Alarm `ec2-cpu-alarm` created with static threshold (>80%, 1/1 period)
- [ ] SNS topic created and email subscription confirmed
- [ ] CPU stress test executed and alarm state changed to 🔴 **In alarm**
- [ ] Email alert received in your inbox
- [ ] Dashboard `Ravi-Labs-Dashboard` created with 4 widgets (Line, Number, Alarm status, Text)
- [ ] Log Group `ravi-app-logs` created with 7-day retention & test log queryable in Logs Insights

---

## ✅ Validation Checklist

- [ ] EC2 instance launched and running
- [ ] CloudWatch metrics visible for the instance
- [ ] CPUUtilization graph shows the stress test spike
- [ ] Alarm created with correct threshold (80%, 1/1 period)
- [ ] SNS topic created and email subscription confirmed
- [ ] Alarm triggered during stress test
- [ ] Email notification received
- [ ] Dashboard created with multiple widget types (Line, Number, Alarm, Text)
- [ ] Dashboard updates automatically
- [ ] Log group created with 7-day retention
- [ ] Test log entry visible in CloudWatch Logs Insights

---

> **Achievement Unlocked:** 👁️ Big Brother of AWS! CloudWatch sees everything across your cloud environment.

---

## 🧹 Cleanup (IMPORTANT!)

CloudWatch resources accrue minor costs if left running. Let's clean up!

1. **Delete the Dashboard:**
   - Go to **CloudWatch** → **Dashboards**
   - Select `Ravi-Labs-Dashboard` → Click **Delete** → Confirm.

2. **Delete the Alarm:**
   - Go to **CloudWatch** → **Alarms** → **All alarms**
   - Select `ec2-cpu-alarm` → Click **Actions** → **Delete** → Confirm.

3. **Delete the SNS Topic:**
   - Search **SNS** in the top search bar → Click **Topics**
   - Select `ec2-cpu-alerts` → Click **Delete** → Type `delete` to confirm.

4. **Delete the Log Group:**
   - Go to **CloudWatch** → **Logs** → **Log groups**
   - Select `ravi-app-logs` → Click **Actions** → **Delete log group** → Confirm.

5. **Terminate the EC2 Instance:**
   - Go to **EC2 Console** → **Instances**
   - Select `cloudwatch-test-ec2` → Click **Instance state** → **Terminate instance**.

6. **Delete Security Group:**
   - Go to **EC2** → **Security Groups** → Select `cloudwatch-sg` → Click **Actions** → **Delete security group**.

---

## 🧠 Memory Tips

| 🧠 Memory Hook | Remember it like... |
|---|---|
| **Basic vs Detailed** | **Basic** = every **5 min** (free). **Detailed** = every **1 min** (paid). 📷 |
| **Hypervisor vs OS Metrics** | Hypervisor sees CPU/Network (automatic). OS sees RAM & Disk space (requires **CloudWatch Agent**). 🧠 |
| **Alarm = "Notify me when..."** | When metric > threshold → trigger **SNS Topic** → send Email/SMS. 📟 |
| **Dashboard = Cockpit** | Wall of interactive widgets showing system health at a glance. 🖥️ |
| **Log Groups = Black Box** | Centralized, searchable repository for application and OS logs. 🕵️ |

---

## 🎓 What You Learned

| Concept | What You Now Know |
|---------|-------------------|
| **CloudWatch Metrics** | How AWS services automatically publish hypervisor metrics |
| **Basic vs Detailed Monitoring** | Basic (free, 5-min) vs Detailed (paid, 1-min) metrics |
| **CloudWatch Alarms** | How to configure evaluation periods, thresholds, and missing data rules |
| **SNS Integration** | How alarms send alerts via Simple Notification Service |
| **Dashboards** | How to build visual dashboards with Line, Number, Alarm, and Text widgets |
| **CloudWatch Logs** | Log Group retention, Log Streams, Logs Insights, and Live Tail |

---

## 🎮 Test Yourself! (No Peeking 👀)

**Q1:** Basic monitoring gives you a datapoint every how many minutes? What about detailed?

<details><summary>👀 Show answer</summary>

**A:** Basic = **5 minutes** (free), Detailed = **1 minute** (paid). 📷

</details>

**Q2:** Can CloudWatch collect RAM/Memory usage from an EC2 instance automatically without an agent?

<details><summary>👀 Show answer</summary>

**A:** **No!** RAM and free disk space are OS-level metrics. You must install the **CloudWatch Agent** inside the OS to collect them. 🧠

</details>

**Q3:** What service must a CloudWatch Alarm connect to in order to send email notifications?

<details><summary>👀 Show answer</summary>

**A:** **AWS SNS (Simple Notification Service)** — Alarm triggers SNS, which sends the email/SMS. 📟

</details>

---

### 🆚 Pro Tip vs Noob Tip

| | Approach |
|---|---|
| **Noob Tip** | Alarms set up without confirming SNS email subscriptions, letting servers fail silently. |
| **Pro Tip** | Alarms → SNS → PagerDuty / Slack / Email. Set up Log Retention periods (e.g. 7 or 30 days) to prevent massive CloudWatch storage bills! |

---

## 🔗 What's Next?

➡️ **Proceed to [Lab 16 — IAM: Users, Groups, Roles, Policies](../16%20-%20IAM%20-%20Users,%20Groups,%20Roles,%20Policies/README.md)** — Master identity, access control, and least-privilege permissions in AWS!

---

## ❓ Troubleshooting

<details>
<summary><strong>"dnf install stress" fails on Amazon Linux 2023</strong></summary>

- Amazon Linux 2023 uses `stress-ng`. Run `sudo dnf install -y stress-ng`.
- Alternatively, use the Python high-CPU loop snippet provided in Step 4.

</details>

<details>
<summary><strong>Alarm stays in "Insufficient Data" state</strong></summary>

- Wait 5–10 minutes for CloudWatch to collect at least one full 5-minute metric evaluation window.

</details>

<details>
<summary><strong>No alarm email received</strong></summary>

- Check your spam/junk folder.
- Ensure you clicked **Confirm subscription** in the SNS confirmation email sent to your inbox.

</details>

---

<div align="center">

<img src="https://img.shields.io/badge/Lab%2015-Complete!-E67E22?style=for-the-badge&labelColor=232F3E" />

> 🎉 **AMAZING work, Ravi!** You've mastered CloudWatch monitoring, alarms, dashboards, and centralized logs! You're ready to secure your cloud environment in Lab 16. Keep building! 🚀✨

</div>
