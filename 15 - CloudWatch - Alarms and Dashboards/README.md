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

`[██░░░░░░░░░░░░░░░░░░] 5% — Let's Begin!`

</div>

---

## 🤔 In Plain English

> **What is this, really?** CloudWatch is the **heart monitor + security cameras + black box** of your AWS account. It collects metrics (CPU, disk, network) from your resources, raises alarms when something's wrong ("CPU > 80%!"), shows everything on a dashboard (your mission control), and collects logs for post-mortems. 📊
>
> 🌍 **Why you should care:** You can't fix what you can't see. CloudWatch is how engineers know *before* customers do that something's on fire.

---

## 🎯 Objective

In this lab, you will:

- Explore **CloudWatch Metrics** for your EC2 instances
- Create a **CloudWatch Alarm** that notifies you when CPU usage is high
- Trigger the alarm using the **stress** tool
- Build a **Custom Dashboard** with multiple widgets to visualize your infrastructure
- Create a **CloudWatch Log Group** and understand how applications send logs
- Understand why monitoring is non-negotiable in cloud engineering

---

## 🧠 Prerequisites

Before you start, make sure you have:

- ✅ Completed **Lab 14** (DynamoDB)
- ✅ An AWS account with CloudWatch access
- ✅ A running **EC2 instance** (t2.micro Amazon Linux 2023) — we'll launch one if needed
- ✅ An email address for receiving alarm notifications
- ✅ Your key pair ready for SSH

---

## 💰 Cost Warning

| Resource | Cost |
|----------|------|
| CloudWatch Metrics (basic) | Free for 10 metrics |
| CloudWatch Alarms | ~$0.10/alarm/month |
| CloudWatch Dashboards | ~$3.00/dashboard/month |
| CloudWatch Logs (first 5 GB) | Free |
| SNS Notifications | Free for email |
| **Estimated total** | **< $1 for this lab** |

> ⚠️ **Rithu says:** CloudWatch Dashboards cost ~$3/month. That's fine for learning, but remember to delete it after the lab! The first 10 basic metrics and 5 GB of logs are free, so most of this lab won't cost you anything. The alarm itself is ~$0.10/month — cheaper than a gumball! 🫧

> **Ravi's Mistake of the Day:** I created a CloudWatch alarm but never subscribed to the SNS topic. The alarm fired beautifully... to an empty void. Nobody was notified. The server died in silence.

---

## 🏗️ Architecture

```
┌───────────────────────────────────────────────────────────────┐
│                        CloudWatch                              │
│                                                                │
│  ┌─────────────┐   ┌──────────────┐   ┌──────────────────┐    │
│  │   Metrics    │   │    Alarms    │   │    Dashboards    │    │
│  │              │   │              │   │                  │    │
│  │ CPU Util %   │──▶│ CPU > 80%?   │──▶│ Line chart,      │    │
│  │ Network In   │   │ Yes → Alert! │   │ Number widgets,  │    │
│  │ Network Out  │   │              │   │ Alarm status     │    │
│  │ Status Check  │   └──────┬───────┘   └──────────────────┘    │
│  └─────────────┘          │                                    │
│                           ▼                                    │
│                   ┌──────────────┐                             │
│                   │   SNS Topic   │                             │
│                   │ "ec2-cpu-"    │                             │
│                   │  "alerts"     │                             │
│                   └──────┬───────┘                             │
│                          │                                     │
│                          ▼                                     │
│                   ┌──────────────┐                             │
│                   │ 📧 Email     │                             │
│                   │ Notification │                             │
│                   └──────────────┘                             │
└───────────────────────────────────────────────────────────────┘
```

> **Did You Know?** CloudWatch can monitor over 70 AWS services and custom metrics. You can even monitor your on-premises servers with the CloudWatch agent. It sees all.

---

## 🛠️ Step-by-Step Instructions

> <img src="https://img.shields.io/badge/Step%201-Launch%20a%20Test%20EC2%20Instance-2ECC71?style=for-the-badge" />

If you don't already have a running EC2 instance, let's launch one!

1. Go to the **EC2 Console** → **Launch Instance**
2. Configure:

| Field | Value |
|-------|-------|
| Name | `cloudwatch-test-ec2` |
| AMI | Amazon Linux 2023 |
| Instance type | t2.micro |
| Key pair | first-key-pair |
| Security group | Create new: `cloudwatch-sg` |
| Inbound rules | SSH (22) from My IP, HTTP (80) from Anywhere |

3. Under **Advanced details** → **User data**, paste:

```bash
#!/bin/bash
yum install -y httpd stress
systemctl start httpd
```

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> We're pre-installing `stress` in the user data so we don't have to install it later via SSH. Efficiency! 🚀

4. Click **Launch Instance**
5. Wait for it to be **Running** and **2/2 status checks passed**

📸 **[Screenshot: EC2 instance cloudwatch-test-ec2 in running state]**

---

> <img src="https://img.shields.io/badge/Step%202-Explore%20CloudWatch%20Metrics-3498DB?style=for-the-badge" />

Let's see what CloudWatch is already tracking for your EC2 instance!

1. Go to the **CloudWatch Console** → left sidebar → **Metrics**
2. Click **All metrics**
3. You'll see metric categories. Click **EC2**
4. Click **Per-Instance Metrics**
5. You should see several metrics for your instance:
   - **CPUUtilization** — How busy the CPU is (0-100%)
   - **NetworkIn** — Bytes received
   - **NetworkOut** — Bytes sent
   - **StatusCheckFailed** — Whether the instance passed status checks
   - **MetadataNoToken** — Metadata access checks

6. Click on **CPUUtilization** — check the box next to it
7. At the bottom, you'll see a graph showing CPU usage over time!

📸 **[Screenshot: CloudWatch metrics page showing CPU utilization graph for the EC2 instance]**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> EC2 automatically sends basic metrics to CloudWatch every 5 minutes for FREE. No agent needed! But if you want 1-minute granularity or custom metrics (like memory usage), you'd install the CloudWatch Agent. For this lab, 5-minute intervals are perfect.

**Explore Other Metrics:**

8. Go back to **Metrics** → **All metrics**
9. Click around — you'll see metrics for ALL your AWS services:
   - RDS (if you still have one running)
   - DynamoDB (if you still have one)
   - S3, Lambda, and more!
10. Each service automatically sends basic metrics to CloudWatch

📸 **[Screenshot: CloudWatch metrics overview showing different service categories]**

---

> <img src="https://img.shields.io/badge/Step%203-Create%20a%20CloudWatch%20Alarm-E67E22?style=for-the-badge" />

This is the star of the show — let's create an alarm that emails you when CPU goes above 80%!

1. Go to **CloudWatch Console** → left sidebar → **Alarms**
2. Click **Create alarm**

**Step 3a: Select Metric**

1. Click **Select metric**
2. Browse to **EC2** → **Per-Instance Metrics**
3. Find your instance `cloudwatch-test-ec2`
4. Select **CPUUtilization**
5. Click **Select metric**

📸 **[Screenshot: Metric selection showing CPUUtilization for cloudwatch-test-ec2]**

**Step 3b: Configure Conditions**

| Field | Value |
|-------|-------|
| Threshold type | **Static** |
| Whenever CPUUtilization is... | **Greater than** |
| than... | **80** |
| Datapoints to alarm | **5 out of 5** consecutive periods |
| Period | **5 minutes** |

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "5 out of 5 consecutive periods" means the CPU must stay above 80% for 5 consecutive 5-minute periods (25 minutes total) before the alarm triggers. This prevents false alarms from brief CPU spikes. You can make it more sensitive by changing to "3 out of 5" — but for this lab, 5/5 works great!

**Step 3c: Configure Notification**

1. Under **Notification type**, select: **Send a notification to an existing SNS topic or create a new one**
2. Click **Create new topic**
3. Configure:

| Field | Value |
|-------|-------|
| Topic name | `ec2-cpu-alerts` |
| Email endpoints | `your-email@example.com` |

> ⚠️ **Use YOUR real email address here!** You'll receive the notification when the alarm triggers.

4. Click **Create topic**
5. ⚠️ **IMPORTANT:** Check your email inbox! You'll receive a confirmation email from AWS. Click the **Confirm subscription** link!

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> If you don't confirm the subscription, you won't receive notifications. Check your spam folder too! The email comes from `no-reply@sns.amazonaws.com`.

**Step 3d: Create the Alarm**

1. Click **Next**
2. **Alarm name:** `ec2-cpu-alarm`
3. **Alarm description:** "Alert when EC2 CPU exceeds 80%"
4. Click **Create alarm**

📸 **[Screenshot: Alarm creation page showing all configured settings]**

---

> <img src="https://img.shields.io/badge/Step%204-Trigger%20the%20Alarm-E74C3C?style=for-the-badge" />

Now let's make the alarm actually fire!

1. SSH into your EC2 instance:

```bash
ssh -i "first-key-pair.pem" ec2-user@<PUBLIC_IP>
```

2. Install stress (if not already installed via user data):

```bash
sudo yum install -y stress
```

3. Start stressing the CPU hard:

```bash
stress --cpu 4 --timeout 600s
```

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> This runs 4 CPU stress workers for 10 minutes. With our alarm set at 5 out of 5 periods at 5 minutes each, we need the CPU above 80% for 25 minutes. So we'll keep the stress running long enough!

4. Open a **second terminal** and SSH into the same instance (or just wait)

5. Watch the CloudWatch metrics:
   - Go to **CloudWatch** → **Metrics** → **EC2** → **Per-Instance Metrics**
   - Select **CPUUtilization** for your instance
   - You should see the graph shoot up to 100%! 📈

📸 **[Screenshot: CloudWatch graph showing CPU spike to 100%]**

6. Go to **CloudWatch** → **Alarms**
7. Watch your alarm state:
   - 🟢 **OK** → 🟡 **Insufficient Data** → 🔴 **ALARM**
8. Wait for the alarm to trigger (this takes about 25 minutes)
9. Check your email — you should receive a notification! 📧

> ⏱️ **Be patient, Ravi!** The alarm needs 5 consecutive periods (25 minutes) of CPU above 80%. Go grab a well-deserved snack while you wait! 🍕

📸 **[Screenshot: Alarm showing "ALARM" state in the CloudWatch console]**

10. Once you get the email notification, stop the stress:

```bash
# Press Ctrl+C in the SSH session, or wait for the 600s timeout
```

11. After CPU drops below 80% for a while, the alarm will return to **OK** state

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> In the real world, you'd set this up with tighter thresholds (like 3 out of 5 periods) and maybe different actions for different severity levels. For example: Warning at 70%, Critical at 90%. But the concept is the same!

---

> <img src="https://img.shields.io/badge/Step%205-Create%20a%20Dashboard-9B59B6?style=for-the-badge" />

Let's build a beautiful dashboard to visualize your infrastructure!

1. Go to **CloudWatch Console** → left sidebar → **Dashboards**
2. Click **Create dashboard**
3. Name: `Ravi-Labs-Dashboard`
4. Click **Create dashboard**

**Widget 1: CPU Utilization Line Chart**

1. Choose widget type: **Line**
2. Click **Configure**
3. Search for metrics: **EC2** → **Per-Instance Metrics** → **CPUUtilization**
4. Select your instance `cloudwatch-test-ec2`
5. Click **Create widget**
6. Resize the widget by dragging the corner to make it wider

📸 **[Screenshot: Dashboard with the first CPU line chart widget added]**

**Widget 2: Network In — Number Widget**

1. Click **Add widget** → **Number**
2. Search for metrics: **EC2** → **Per-Instance Metrics** → **NetworkIn**
3. Select your instance
4. Click **Create widget**
5. This shows the total network bytes received as a big number!

**Widget 3: Alarm Status**

1. Click **Add widget** → **Alarm status**
2. Select your alarm: `ec2-cpu-alarm`
3. Click **Create widget**
4. This widget shows a big green (OK) or red (ALARM) indicator!

**Widget 4: Network Out Line Chart (Bonus)**

1. Click **Add widget** → **Line**
2. Search for **NetworkOut** for your instance
3. Click **Create widget**

5. Click **Save dashboard**

📸 **[Screenshot: Complete Ravi-Labs-Dashboard with all 4 widgets visible]**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Dashboards update automatically every minute! You can share them with your team by clicking the **Share** button. In production, teams often have dashboards on big screens in the office showing real-time infrastructure health. Now you know how to build one! 🎨

**Customize the Dashboard:**

- Drag widgets to rearrange them
- Resize widgets by dragging corners
- Click the **pencil icon** on any widget to edit its configuration
- Add time range selectors at the top

---

> <img src="https://img.shields.io/badge/Step%206-Create%20a%20CloudWatch%20Log%20Group-F39C12?style=for-the-badge" />

Let's learn about CloudWatch Logs — where applications send their log files!

1. Go to **CloudWatch Console** → left sidebar → **Logs** → **Log groups**
2. Click **Create log group**
3. Configure:

| Field | Value |
|-------|-------|
| Log group name | `ravi-app-logs` |
| Retention setting | **7 days** |

4. Click **Create**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Log groups are like folders for your logs. You'd configure your application (Apache, Nginx, your custom app) to send logs here instead of writing them to local files. This is incredibly useful because:
> - Logs are centralized (not scattered across instances)
> - Logs persist even if the instance is terminated
> - You can search and filter logs across all instances
> - You can set up alarms based on log patterns!

**Send a Test Log (via AWS CLI):**

```bash
aws logs put-log-events \
  --log-group-name ravi-app-logs \
  --log-stream-name test-stream \
  --log-events timestamp=$(date +%s000),message="Hello from Ravi's first log entry!"

# If the log stream doesn't exist, create it first:
aws logs create-log-stream \
  --log-group-name ravi-app-logs \
  --log-stream-name test-stream
```

4. Go back to **Log groups** → click `ravi-app-logs`
5. Click on the log stream
6. You should see your test log entry!

📸 **[Screenshot: CloudWatch Logs showing the test log message]**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> In a real application, you'd configure the CloudWatch Agent on your EC2 instances to automatically ship logs to CloudWatch. For example, Apache logs (`/var/log/httpd/access_log`) would be sent here automatically. Then you can search across ALL your servers' logs in one place. Game changer! 🎮

---

> <img src="https://img.shields.io/badge/Step%207-Verify%20Your%20Work-1ABC9C?style=for-the-badge" />

- [ ] EC2 instance `cloudwatch-test-ec2` is running
- [ ] CloudWatch Metrics show CPU, Network, and Status Check metrics
- [ ] CPU Utilization graph shows a spike when stress was running
- [ ] CloudWatch Alarm `ec2-cpu-alarm` was created
- [ ] Alarm triggered (went to ALARM state) during stress test
- [ ] Email notification received from SNS
- [ ] Dashboard `Ravi-Labs-Dashboard` has 4 widgets
- [ ] Log group `ravi-app-logs` exists with 7-day retention
- [ ] Test log message visible in the log stream

📸 **[Screenshot: CloudWatch console showing alarm in OK state after stress stopped]**

---

## ✅ Validation Checklist

- [ ] EC2 instance launched and running
- [ ] CloudWatch metrics visible for the instance
- [ ] CPUUtilization graph shows the stress test spike
- [ ] Alarm created with correct threshold (80%, 5/5 periods)
- [ ] SNS topic created and email subscription confirmed
- [ ] Alarm triggered during stress test
- [ ] Email notification received
- [ ] Dashboard created with multiple widget types (Line, Number, Alarm)
- [ ] Dashboard updates automatically
- [ ] Log group created with 7-day retention
- [ ] Test log entry visible in CloudWatch Logs

---

> **Achievement Unlocked:** Big Brother! CloudWatch sees everything.

---

## 🧹 Cleanup (IMPORTANT!)

CloudWatch costs are small but let's clean up properly!

1. **Delete the Dashboard:**
   - Go to **CloudWatch** → **Dashboards**
   - Select `Ravi-Labs-Dashboard`
   - Click **Delete dashboard**
   - Confirm deletion

2. **Delete the Alarm:**
   - Go to **CloudWatch** → **Alarms**
   - Select `ec2-cpu-alarm`
   - Click **Actions** → **Delete**
   - Confirm

3. **Delete the SNS Topic:**
   - Go to **SNS Console** → **Topics** (or search SNS in the top bar)
   - Select `ec2-cpu-alerts`
   - Click **Delete**
   - Type `delete` to confirm

4. **Delete the Log Group:**
   - Go to **CloudWatch** → **Logs** → **Log groups**
   - Select `ravi-app-logs`
   - Click **Delete**
   - Type the log group name to confirm
   - Click **Delete**

5. **Terminate the EC2 Instance:**
   - Go to **EC2** → **Instances**
   - Select `cloudwatch-test-ec2`
   - Click **Instance state** → **Terminate instance**
   - Confirm

6. **Delete the Security Group:**
   - Go to **EC2** → **Security Groups**
   - Select `cloudwatch-sg`
   - Click **Actions** → **Delete security group**
   - Confirm

📸 **[Screenshot: All CloudWatch resources deleted — empty Dashboards and Alarms pages]**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Always clean up dashboards and alarms! A forgotten dashboard costs ~$3/month and an alarm costs ~$0.10/month. Small amounts add up, and more importantly, it's good cloud hygiene! 🧹

---

## 🧠 Memory Tips

Stick these in your brain and they'll never leave. 🧲

| 🧠 Memory Hook | Remember it like... |
|---|---|
| **Basic vs Detailed** | **Basic** = every **5 min** (free). **Detailed** = every **1 min** (paid). Free = blurrier photo. 📷 |
| **Alarm = "call me when..."** | "When CPU > 80% for 5 minutes → **notify me** (via SNS email)." That's the whole concept. 📟 |
| **Dashboard = mission control** | A wall of widgets showing your entire infra at a glance. 🖥️ |
| **Log groups = the black box** | All your app logs, collected in one searchable place. Debugging without them is archaeology. 🕵️ |

> 🗣️ **Rithu:** *"An alarm without an action is just a fancy notification. Connect it to SNS and make it TEXT you. Future-you will be grateful at 3 AM."

---

## 🎓 What You Learned

| Concept | What You Now Know |
|---------|-------------------|
| **CloudWatch Metrics** | How AWS services automatically send metrics to CloudWatch |
| **Basic vs Detailed Monitoring** | Basic (free, 5-min) vs Detailed (paid, 1-min) metrics |
| **CloudWatch Alarms** | How to set thresholds and get notified when metrics breach them |
| **SNS Integration** | How alarms connect to SNS topics which send email notifications |
| **Dashboards** | How to build visual dashboards with multiple widget types |
| **Log Groups** | How to centralize application logs in CloudWatch |
| **Metric Granularity** | Understanding periods, datapoints, and evaluation windows |

---

## 🎮 Test Yourself! (No Peeking 👀)

**Q1:** Basic monitoring gives you a datapoint every how many minutes? What about detailed?

<details><summary>👀 Show answer</summary>

**A:** Basic = **5 minutes** (free), Detailed = **1 minute** (paid). Pay for detail only when you truly need it. 📷

</details>

**Q2:** What must an alarm be connected to so that it actually *notifies* you?

<details><summary>👀 Show answer</summary>

**A:** An **SNS topic** (e.g., an email subscription). Alarm fires → SNS pushes → your inbox/phone gets pinged. 📟

</details>

**Q3:** Where do application logs get centralized for searching?

<details><summary>👀 Show answer</summary>

**A:** **CloudWatch Log Groups** — a searchable, centralized home for your logs (often queried with Logs Insights). 🕵️

</details>

### 🔥 Bonus Challenge

Build a **custom dashboard** with 3+ widgets: CPU, network in/out, and a text widget with your name and the date. Then use `stress` to spike the CPU and watch the widget go red in near-real-time. You're monitoring like a pro now. 📈

> 💪 **Rithu:** *"Dashboards are the cockpit. Pilots check instruments BEFORE trouble — so should you."

---

### 🆚 Pro Tip vs Noob Tip

| | Approach |
|---|---|
| **Noob Tip** | Alarms that only send email to an inbox nobody checks |
| **Pro Tip** | Alarms → SNS → SMS/chat integration. Metrics on a dashboard you actually open daily |

---

## 🔗 What's Next?

Congratulations on completing all 15 labs! Here's where you can go from here:

➡️ **Next Steps:**
- **AWS Certification:** These labs cover topics from the AWS Cloud Practitioner and Solutions Architect Associate exams!
- **Practice with different regions:** Try deploying resources in `us-west-2` or `eu-west-1`
- **Explore Lambda:** Serverless compute is the next frontier
- **Set up AWS Budgets:** Get alerts when your spending exceeds a threshold
- **Build a project:** Combine everything you've learned — EC2 + ASG + RDS + Route 53 + CloudWatch = a real production-ready architecture!

---

## ❓ Troubleshooting

<details>
<summary><strong>I don't see any metrics for my EC2 instance</strong></summary>

- Metrics take **5-10 minutes** to appear for a new instance
- Make sure you're looking in the correct region
- Check that the instance is **Running** (not stopped or terminated)
- Try refreshing the metrics page

</details>

<details>
<summary><strong>Alarm is stuck in "Insufficient Data" state</strong></summary>

- This means CloudWatch doesn't have enough data points yet
- Wait at least 5 minutes (one metric period)
- The alarm needs data from at least 1 period before it can evaluate

</details>

<details>
<summary><strong>I didn't receive the email notification</strong></summary>

- Check your **spam/junk folder**
- Verify you confirmed the SNS subscription (check your inbox for the confirmation email)
- Make sure the alarm actually triggered (showing "ALARM" state, not "Insufficient Data")
- Check the SNS topic has your email listed correctly

</details>

<details>
<summary><strong>The stress test didn't push CPU above 80%</strong></summary>

- t2.micro only has 1 vCPU — `stress --cpu 4` may still max it out
- Check that stress is actually running: `top` in another terminal
- Some instance types have burstable CPU — t2.micro can burst above baseline but may throttle

</details>

<details>
<summary><strong>Dashboard widgets show "No data"</strong></summary>

- Wait a few minutes — widgets need time to populate
- Make sure the metrics exist and are being reported
- Check the time range selector on the dashboard (might be set to the wrong time period)

</details>

<details>
<summary><strong>Log group creation fails</strong></summary>

- Check that the log group name doesn't already exist
- Verify you have CloudWatch Logs permissions
- Make sure you're in the correct region

</details>

---

<div align="center">

<img src="https://img.shields.io/badge/Lab%2015-Complete!-E67E22?style=for-the-badge&labelColor=232F3E" />

> 🎉 **AMAZING work, Ravi!** You've completed all 15 labs! From launching your first EC2 instance to building auto-scaling architectures, configuring DNS failover, managing databases, and now monitoring everything with CloudWatch — you've built a solid foundation in AWS. You should be incredibly proud of yourself! The cloud journey never really ends, but you've taken the most important steps. Keep building, keep learning, and remember — Rithu believes in you! 🚀✨

</div>
