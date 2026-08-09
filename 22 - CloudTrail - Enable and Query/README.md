<div align="center">

<img src="https://img.shields.io/badge/Lab%2022-CloudTrail%20Enable%20%26%20Query-2980B9?style=for-the-badge&labelColor=232F3E" />

# Lab 22 — CloudTrail: Enable and Query

<img src="https://img.shields.io/badge/Difficulty-Easy-green?style=flat-square" />
<img src="https://img.shields.io/badge/Time-~20min-blue?style=flat-square" />
<img src="https://img.shields.io/badge/Cost-%3C%241-brightgreen?style=flat-square" />
<img src="https://img.shields.io/badge/Service-CloudTrail-red?style=flat-square" />

</div>

> "In the cloud, someone is always watching. CloudTrail is your security camera — and it never blinks!" — Rithu

---

<details>
<summary><b>🎭 Ravi & Rithu's Coffee Break Chat</b></summary>

**Ravi:** "CloudTrail is like a security camera?"

**Rithu:** "More like a flight recorder for your AWS account. Every API call, every change, logged."

**Ravi:** "Even if I accidentally delete something?"

**Rithu:** "ESPECIALLY if you accidentally delete something. CloudTrail will know exactly who did it... which is you."

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

> **What is this, really?** CloudTrail is the **security camera + black box** of your AWS account. Every API call — who did it, when, from which IP, and what changed — gets recorded. Enable a trail and those records stream to S3 (permanent storage) and CloudWatch Logs (searchable). If something weird happens in your account, CloudTrail is how you find the culprit. 🕵️
>
> 🌍 **Why you should care:** "Who terminated the production database?" is a real interview question. CloudTrail is the answer. Every serious account has it on from day one.

---

## 🎯 Objective

By the end of this lab, you will:
- Understand what CloudTrail is and why every AWS account needs it
- Create a trail to log all management events to S3 and CloudWatch Logs
- Query events using CloudWatch Logs Insights
- View the audit trail of every action taken in your AWS account

CloudTrail is your **security camera for AWS**. It records who did what, when, and from where. Without it, you're flying blind. With it, you have a complete audit trail.

---

## 🧠 Prerequisites

- [ ] Completed Lab 21 (CloudFormation)
- [ ] AWS Console access with appropriate permissions
- [ ] An EC2 key pair exists (from previous labs)

---

## 💰 Cost Warning

CloudTrail has a generous free tier:

| What | Cost |
|------|------|
| First trail — management events | **FREE** (first copy per region) |
| Management events beyond the first | ~$2.00 per 100K events |
| Data events (S3/Lambda) | $0.10 per 100K events (not needed for this lab) |
| CloudWatch Logs ingestion | ~$0.50/GB (minimal for this lab) |
| S3 storage for logs | Standard S3 pricing (pennies) |

Estimated total lab cost: **< $1** if cleaned up within 1 hour.

> ⚠️ **IMPORTANT**: Delete your trail, S3 bucket, and CloudWatch log group before leaving! CloudTrail logs are small but can accumulate over time.

> **Ravi's Mistake of the Day:** I enabled CloudTrail in one region only. Someone made changes in us-west-2, and I had no record of it. Enable CloudTrail for ALL regions.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│                  CloudTrail                          │
│                                                     │
│  ┌────────────┐     ┌──────────┐    ┌───────────┐  │
│  │ AWS API    │────▶│  Trail   │───▶│  S3       │  │
│  │ Calls      │     │          │    │  Bucket   │  │
│  │ (who/what) │     │  Logs    │    │  (JSON)   │  │
│  └────────────┘     │  Events  │    └───────────┘  │
│                     │          │                    │
│                     │          │    ┌───────────┐  │
│                     │          │───▶│ CloudWatch │  │
│                     └──────────┘    │ Logs       │  │
│                                     │ (Insights) │  │
│                                     └───────────┘  │
└─────────────────────────────────────────────────────┘
```

> **Did You Know?** CloudTrail logs are stored in S3 and can be queried with Athena. You can literally ask SQL questions about every API call ever made in your AWS account.

---

## 🛠️ Step-by-Step Instructions

### <img src="https://img.shields.io/badge/Step%201-Create%20a%20Trail-FF6B6B?style=for-the-badge" />

Let's set up CloudTrail to record everything happening in your account.

1. Open the **AWS Console** in your browser
2. Search for **CloudTrail** in the search bar and click on it
3. You should see the CloudTrail dashboard
4. Click **Trails** in the left sidebar
5. Click the orange **Create trail** button (top right)

**Configure your trail:**

6. **Trail name**: Type `ravi-management-trail`
7. **Enable for all accounts in my organization**: Leave unchecked (this is for AWS Organizations)
8. Under **Management events**, make sure it's set to **Read/Write events** (both)
9. Under **Storage location**:
   - Select **CloudWatch Logs** (check the box)
   - **Log group**: Click **Create new log group** → type `ravi-cloudtrail-logs` → click Create
   - **IAM role**: Click **Create new role** → leave the default name → click Create
10. Under **S3 bucket**:
    - Select **Create new S3 bucket**
    - **S3 bucket**: Type `ravi-cloudtrail-bucket-` followed by some random numbers (like `12345`) — bucket names must be globally unique
    - Leave the rest as default
11. Click **Create trail**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> **Good news:** Every trail you create in the CloudTrail **console** is a **multi-Region trail** by default — it logs activity in all enabled Regions automatically, so there's no checkbox to worry about. (A single-Region trail can only be created with the AWS CLI.) Why does this matter? Attackers might try to create resources in a region you don't normally use. Logging everything means no blind spots!

📸 [Screenshot: CloudTrail "Create trail" page with all settings configured as described]

---

### <img src="https://img.shields.io/badge/Step%202-Generate%20Some%20Activity-FFA500?style=for-the-badge" />

CloudTrail is only useful if there's something to log! Let's create some activity.

1. Go to **EC2** in the AWS Console
2. Click **Launch instance** (top right)
3. **Name**: Type `ravi-cloudtrail-test`
4. **AMI**: Amazon Linux 2023 (Free Tier eligible)
5. **Instance type**: t2.micro (Free Tier eligible)
6. **Key pair**: Select your existing key pair
7. Leave everything else as default
8. Click **Launch instance** → **View all instances**

Wait for the instance to be in **Running** state, then:

9. Select the instance → Click **Instance state** → Click **Stop instance**
10. Confirm the stop
11. Wait for the instance to be in **Stopped** state
12. Select the instance → Click **Instance state** → Click **Terminate instance**
13. Confirm the termination

We just created three CloudTrail events:
- `RunInstances` (launching the instance)
- `StopInstances` (stopping the instance)
- `TerminateInstances` (terminating the instance)

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "Every click in the AWS Console generates an API call, and CloudTrail records every one of them. Launching, stopping, terminating — each action has a timestamp, user, and source IP."

📸 [Screenshot: EC2 console showing the instance being terminated]

---

### <img src="https://img.shields.io/badge/Step%203-View%20Events%20in%20CloudTrail-9B59B6?style=for-the-badge" />

Now let's see what CloudTrail recorded!

1. Go back to **CloudTrail** in the AWS Console
2. Click **Event history** in the left sidebar
3. You should see a list of recent events

**Filter the events:**

4. Click the **Filter** dropdown (or search bar)
5. Select **Event name** as the filter type
6. Type `RunInstances` and press Enter
7. You should see the event for when you launched the EC2 instance

**Explore the event details:**

8. Click on any event to expand it
9. You'll see rich details like:
   - **Event time**: When the action happened (UTC)
   - **Event name**: RunInstances
   - **Event source**: ec2.amazonaws.com
   - **User identity**: Who performed the action (your IAM user)
   - **Source IP address**: Where the request came from
   - **Request parameters**: What was requested (instance type, AMI, etc.)
   - **Response elements**: What AWS returned (instance ID, etc.)

10. Try filtering by `StopInstances` and `TerminateInstances` to see those events too

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "The Event History view shows the last 90 days of events. For longer retention, you need the S3 bucket we configured. Think of Event History as a 'recent calls' log on your phone."

📸 [Screenshot: CloudTrail Event History showing the RunInstances event with details expanded]

---

### <img src="https://img.shields.io/badge/Step%204-Query%20Events%20in%20S3-3498DB?style=for-the-badge" />

Your trail sends events to S3 as JSON files. Let's look at them!

1. Go to **S3** in the AWS Console
2. Click on the bucket you created (starts with `ravi-cloudtrail-bucket-`)
3. Navigate through the folder structure:
   ```
   AWSLogs/
   └── [your-account-id]/
       └── CloudTrail/
           └── [your-region]/
               └── [year]/
                   └── [month]/
                       └── [day]/
   ```
4. You should see JSON log files (they end with `.json.gz` — compressed)
5. Click on a file and **Download** it
6. Extract the file (it's gzipped)
7. Open it in a text editor

**What you'll see:**

```json
{
  "Records": [
    {
      "eventVersion": "1.08",
      "eventTime": "2024-01-15T10:30:00Z",
      "awsRegion": "us-east-1",
      "eventSource": "ec2.amazonaws.com",
      "eventName": "RunInstances",
      "userIdentity": {
        "type": "IAMUser",
        "principalId": "AIDA...",
        "arn": "arn:aws:iam::123456789012:user/ravi",
        "accountId": "123456789012"
      },
      "sourceIPAddress": "203.0.113.50",
      "requestParameters": { ... }
    }
  ]
}
```

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "These S3 log files are your permanent audit trail. In a real company, you'd keep these for years for compliance (HIPAA, PCI-DSS, SOC 2). Security teams and auditors love CloudTrail!"

📸 [Screenshot: S3 bucket showing the folder structure and a downloaded log file]

---

### <img src="https://img.shields.io/badge/Step%205-CloudWatch%20Logs%20Insights-1ABC9C?style=for-the-badge" />

This is where CloudTrail gets powerful! Let's query our events using SQL-like syntax.

1. Go to **CloudWatch** in the AWS Console
2. Click **Logs** → **Logs Insights** in the left sidebar
3. You should see a query editor

**Select your log group:**

4. In the **Select log group(s)** dropdown, find and select `ravi-cloudtrail-logs`
5. The query editor should now be ready

**Run your first query:**

6. Copy and paste this query into the editor:

```
fields eventTime, eventName, userIdentity.type, sourceIPAddress
| filter eventName = "RunInstances"
| sort eventTime desc
| limit 10
```

7. Click **Run query** (or press Ctrl+Enter)
8. You should see results showing your RunInstances events!

**Try a more advanced query:**

9. Clear the editor and paste this:

```
fields eventTime, eventName, userIdentity.arn, sourceIPAddress, userAgent
| sort eventTime desc
| limit 20
```

10. Click **Run query**
11. This shows ALL events (not just RunInstances) with who did what and from where

**Try a security-focused query:**

12. Paste this query:

```
fields eventTime, eventName, userIdentity.arn, sourceIPAddress
| filter eventName like /Terminate|Delete|Stop/
| sort eventTime desc
| limit 10
```

13. Click **Run query**
14. This finds all destructive actions — useful for security audits!

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "CloudWatch Logs Insights is like Google for your CloudTrail logs. Instead of scrolling through thousands of events, you write a query and get instant answers. In a real job, you might be asked: 'Who stopped our production server at 3 AM last Tuesday?' — and this is how you'd find out!"

📸 [Screenshot: CloudWatch Logs Insights showing the query and results]

---

### <img src="https://img.shields.io/badge/Step%206-Create%20Metric%20Filter-E74C3C?style=for-the-badge" />

Want to get alerted when suspicious activity happens? Create a metric filter!

1. Go to **CloudWatch** → **Logs** → **Log groups**
2. Click on `ravi-cloudtrail-logs`
3. Click **Metric filters** tab
4. Click **Create metric filter**

**Define pattern:**

5. Enter this filter pattern:
   ```
   { ($.eventName = "ConsoleLogin") && ($.errorMessage = "Failed authentication") }
   ```
6. Click **Next**
7. Filter name: `FailedLoginAttempts`
8. Metric namespace: `CloudTrailMetrics`
9. Metric name: `FailedLogins`
10. Metric value: `1`
11. Click **Next** → **Create metric filter**

Now you can create a CloudWatch Alarm on this metric to get notified of failed login attempts!

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "Metric filters are how you turn logs into actionable alerts. Failed logins, unauthorized API calls, root user activity — all of these can trigger alarms that notify your security team."

📸 [Screenshot: Metric filter creation page with the filter pattern entered]

---

## ✅ Validation Checklist

Before moving on, confirm all of these:

- [ ] Trail `ravi-management-trail` exists and is enabled
- [ ] Trail is logging to S3 bucket and CloudWatch Logs
- [ ] Event history shows your EC2 launch, stop, and terminate events
- [ ] S3 bucket contains JSON log files in the correct folder structure
- [ ] CloudWatch Logs Insights query returns results
- [ ] (Optional) Metric filter for failed logins is created

---

> **Achievement Unlocked:** Audit Trail! CloudTrail activated.

---

## 🧹 Cleanup (IMPORTANT!)

CloudTrail is powerful but logs can accumulate. Clean up everything!

**Delete the EC2 instance (if it's still running):**

1. Go to **EC2** → **Instances**
2. If any test instances remain, select them → **Instance state** → **Terminate instance**

**Delete the CloudTrail trail:**

3. Go to **CloudTrail** → **Trails**
4. Select `ravi-management-trail`
5. Click **Delete** (top right)
6. Confirm the deletion

**Delete the CloudWatch Logs log group:**

7. Go to **CloudWatch** → **Logs** → **Log groups**
8. Find `ravi-cloudtrail-logs`
9. Click the radio button next to it
10. Click **Delete** (or Actions → Delete)
11. Type `delete` to confirm
12. Click **Delete**

**Delete the S3 bucket:**

13. Go to **S3** → **Buckets**
14. Find `ravi-cloudtrail-bucket-...`
15. Click the bucket name
16. Click **Empty** (top right) to delete all objects
17. Type `permanently delete` to confirm
18. Click **Delete**
19. Go back to the bucket list
20. Select the bucket → Click **Delete**
21. Type the bucket name to confirm
22. Click **Delete bucket**

**Delete the metric filter (if created):**

23. Go to **CloudWatch** → **Logs** → **Log groups** → `ravi-cloudtrail-logs`
24. Click **Metric filters** tab
25. Select the filter → Click **Delete metric filter**

**Verify everything is gone:**

26. Go to CloudTrail → confirm no trails exist
27. Go to S3 → confirm the bucket is gone
28. Go to CloudWatch → confirm the log group is gone

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "In production, you'd NEVER delete your CloudTrail. But for this lab, we clean up to avoid any charges. In your real AWS account, always keep at least one trail enabled — it's your security lifeline!"

📸 [Screenshot: CloudTrail console showing no trails remaining]

---

## 🧠 Memory Tips

Stick these in your brain and they'll never leave. 🧲

| 🧠 Memory Hook | Remember it like... |
|---|---|
| **CloudTrail = security camera** | Records **every API call** in your account. Someone did something? The tape has it. 🎥 |
| **Management vs Data events** | **Management** = creating/changing resources (big actions). **Data** = touching data (S3 object reads). 📼 |
| **Event history = 90 days** | The console's built-in replay — free, but only back 90 days. Longer? Send logs to S3. 📅 |
| **Logs → S3 + CloudWatch** | S3 = permanent archive. CloudWatch Logs Insights = search with SQL-ish queries. 🗂️ |
| **Metric filters = alarm triggers** | Turn a log pattern ("AccessDenied") into a metric that can fire alarms. ⏰ |

> 🗣️ **Rithu:** *"Set up CloudTrail before you need it. You can't retroactively watch a camera that was never plugged in."

---

## 🎓 What You Learned

In this lab, you learned:

| Concept | What It Means |
|---------|---------------|
| **CloudTrail** | AWS service that records API activity in your account |
| **Trail** | A configuration that tells CloudTrail where to send logs |
| **Management Events** | API calls that manage AWS resources (create, delete, modify) |
| **Data Events** | API calls that operate on data within resources (S3 object access) |
| **Event History** | Last 90 days of events viewable in the console |
| **S3 Log Storage** | Permanent storage of CloudTrail logs as JSON files |
| **CloudWatch Logs Insights** | SQL-like query engine for searching CloudTrail logs |
| **Metric Filters** | Convert log patterns into CloudWatch metrics and alarms |
| **Security Auditing** | Who did what, when, and from where — the complete picture |

---

## 🎮 Test Yourself! (No Peeking 👀)

**Q1:** What's the difference between management events and data events?

<details><summary>👀 Show answer</summary>

**A:** **Management events** = API calls that *change resources* (create/delete/modify). **Data events** = calls that *touch data inside* resources (like reading an S3 object). 📼

</details>

**Q2:** How far back can you see events in the CloudTrail console's Event History?

<details><summary>👀 Show answer</summary>

**A:** **90 days.** For longer retention (or compliance), stream logs to **S3** permanently. 📅

</details>

**Q3:** Where should CloudTrail logs go for long-term, searchable storage?

<details><summary>👀 Show answer</summary>

**A:** **S3** for permanent archive + **CloudWatch Logs** (with Logs Insights) for powerful searching and alerting. 🗂️

</details>

### 🔥 Bonus Challenge

Launch **and terminate** an EC2 instance, then use CloudTrail's Event History (or Logs Insights) to find the `RunInstances` and `TerminateInstances` events — including **who** did it and **from where**. You've just done a real security audit. 🕵️

> 💪 **Rithu:** *"Every action you've taken in these labs is already recorded. CloudTrail was watching you the whole time — that's the point."

---

### 🆚 Pro Tip vs Noob Tip

| | Approach |
|---|---|
| **Noob Tip** | Enable CloudTrail only after something bad happens |
| **Pro Tip** | Trail on from day one, logging to S3 + CloudWatch, with an alarm on AccessDenied spikes |

---

## 🔗 What's Next?

You now have eyes on everything happening in your AWS account! In the next lab, we'll learn about **KMS** — AWS's Key Management Service for encrypting your data.

**[Lab 23 — KMS: Encrypt S3 and EBS →](../23%20-%20KMS%20-%20Encrypt%20S3%20and%20EBS/README.md)**

---

## ❓ Troubleshooting

<details>
<summary><strong>No events showing in Event History</strong></summary>

**Fix**: Wait 5-10 minutes after creating the trail — it can take time for events to appear. Also make sure you're looking in the correct region.
</details>

<details>
<summary><strong>Trail shows "Logging" but no CloudWatch Logs</strong></summary>

**Fix**: The IAM role might not have permissions. Go to the trail settings and verify the CloudWatch Logs IAM role was created correctly.
</details>

<details>
<summary><strong>CloudWatch Logs Insights query returns no results</strong></summary>

**Fix**: Make sure you selected the correct log group (`ravi-cloudtrail-logs`). Also verify that events have been delivered (can take 5-15 minutes for new trails).
</details>

<details>
<summary><strong>S3 bucket is empty</strong></summary>

**Fix**: CloudTrail delivers logs within 15 minutes. If the bucket is empty after 30 minutes, check the trail status in CloudTrail console.
</details>

<details>
<summary><strong>"Access Denied" when creating the trail</strong></summary>

**Fix**: Ensure your IAM user has the `AWSCloudTrail_FullAccess` policy. You can also attach the `CloudTrailFullAccess` managed policy.
</details>

<details>
<summary><strong>Can't delete the S3 bucket</strong></summary>

**Fix**: You must EMPTY the bucket first (delete all objects), then delete the bucket. S3 buckets cannot be deleted if they contain objects.
</details>

<details>
<summary><strong>Metric filter doesn't seem to work</strong></summary>

**Fix**: After creating the filter, wait 5-10 minutes for it to start processing new log events. The filter only applies to NEW events, not historical ones.
</details>

---

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "You now know more about AWS security auditing than most people! CloudTrail is one of those services that seems boring until something goes wrong — then it's the most important thing in the world. Always keep it enabled!"

---

<div align="center">

<img src="https://img.shields.io/badge/Lab%2022-Complete!-27AE60?style=for-the-badge&labelColor=232F3E" />

**🎉 Congratulations on completing Lab 22! 🎉**

</div>
