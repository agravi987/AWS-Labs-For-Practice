<div align="center">

<img src="https://img.shields.io/badge/Lab%2016-IAM%20Users%20Groups%20Roles-3498DB?style=for-the-badge&labelColor=232F3E" />

# Lab 16 — IAM: Users, Groups, Roles, Policies

<img src="https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=flat-square" />
<img src="https://img.shields.io/badge/Time-~30min-blue?style=flat-square" />
<img src="https://img.shields.io/badge/Cost-Free-00c853?style=flat-square" />
<img src="https://img.shields.io/badge/Service-IAM-orange?style=flat-square" />

</div>

> "IAM is the bouncer of your AWS club — nobody gets in without checking with the bouncer first!" — Rithu

---

<details>
<summary><b>🎭 Ravi & Rithu's Coffee Break Chat</b></summary>

**Ravi:** "IAM seems complicated..."

**Rithu:** "It's like a permission slip system. Who can do what, where, and when."

**Ravi:** "Can I just give everyone admin access?"

**Rithu:** "You CAN. It's also the fastest way to get your AWS account compromised. So... don't."

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

> **What is this, really?** IAM is **AWS's security guard with a clipboard** — it decides *who* (users, services) can do *what* (actions on resources). **Users** = people, **groups** = teams of people, **policies** = permission slips (JSON), and **roles** = temporary hats that AWS services wear to get things done. 🪪
>
> 🌍 **Why you should care:** This is the most important lab in the whole series. Every AWS security breach you've ever read about came down to IAM mistakes. Get this right and you're 90% safer than most AWS users.

---

## 🎯 Objective

In this lab, you'll learn how AWS Identity and Access Management (IAM) controls **who** can access **what** in your AWS account by creating users, groups, custom policies, and roles — the building blocks of AWS security. Pay attention, Ravi! 🎓

**By the end of this lab you will be able to:**
- Create IAM users with different access levels
- Organize users into groups
- Write custom IAM policies using the visual editor
- Create IAM roles and attach them to EC2 instances
- Understand the difference between programmatic and console access

---

## 🧠 Prerequisites

- [ ] Completed [Lab 15](../15%20-%20CloudWatch%20-%20Alarms%20and%20Dashboards/README.md)
- [ ] AWS account with root access
- [ ] AWS CLI configured (optional but helpful for Step 4)
- [ ] Basic understanding of what IAM is (covered in lecture material)

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> If you haven't finished Lab 15, go do that first. I'll wait. No really, I'll be here. ☕

---

## 💰 Cost Warning

**This lab is 100% FREE!** 🎉

IAM is a globally available AWS service that does **not** charge you anything. You can create users, groups, roles, and policies without spending a single penny. However, keep in mind that the **resources those IAM entities access** (like EC2, S3, etc.) may incur charges — which is why cleanup is still important!

> **Ravi's Mistake of the Day:** I created an IAM user with AdministratorAccess and then used those credentials in a public GitHub repo. AWS automatically flagged it within minutes. Don't commit credentials. Ever. Period.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────┐
│                 AWS Account                  │
│                                              │
│  ┌──────────────┐    ┌──────────────┐       │
│  │  Developers   │    │ ReadOnlyUsers │       │
│  │    Group      │    │    Group      │       │
│  └──────┬───────┘    └──────┬───────┘       │
│         │                    │                │
│  ┌──────┴───────┐    ┌──────┴───────┐       │
│  │ravi-developer │    │ravi-readonly  │       │
│  │    User       │    │    User       │       │
│  └──────────────┘    └──────────────┘       │
│                                              │
│  ┌──────────────────────────────────┐       │
│  │     EC2 Instance                  │       │
│  │  ┌─────────────────────────┐     │       │
│  │  │ IAM Role: EC2-S3-Read   │     │       │
│  │  │ Policy: S3ReadOnlyAccess │     │       │
│  │  └─────────────────────────┘     │       │
│  └──────────────────────────────────┘       │
│                                              │
│  ┌──────────────────────────────────┐       │
│  │  Custom Policy: S3DeveloperAccess│       │
│  │  → Attached to Developers Group  │       │
│  └──────────────────────────────────┘       │
└─────────────────────────────────────────────┘
```

> **Did You Know?** IAM policies can be up to 6,144 characters long. But shorter is better. The principle of least privilege means giving ONLY the permissions needed - nothing more.

---

## 🛠️ Step-by-Step Instructions

> <img src="https://img.shields.io/badge/Step%201-Create%20IAM%20Users-2ECC71?style=for-the-badge" />

1. Sign in to the **AWS Management Console** as root (or an admin user).
2. In the search bar at the top, type **IAM** and click on **IAM** under Services.
3. In the left navigation pane, click **Users**.
4. Click the orange **Create user** button.

#### Create User 1: ravi-developer

1. **User name:** `ravi-developer`
2. Check the box: ☑ **Provide user access to the AWS Management Console**
3. Select: ⚫ **I want to create an IAM user**
4. **Console password:** Select ⚫ **Custom password** and type a strong password (e.g., `RaviLabs2024!`). Leave **Users must create a password with next sign-in** **unchecked** so this password still works in Step 5 — write it down somewhere safe!
5. Click **Next**.
6. On the "Set permissions" page, select: ⚫ **Add user to group**
7. Click **Create group**:
   - Group name: `Developers`
   - Check the box: ☑ **AdministratorAccess**
   - Click **Create group**
8. Click **Next** → **Review and create** → **Create user**.
9. ⚠️ **IMPORTANT:** Click **Download .csv file** to save the sign-in credentials (console sign-in URL, username, password) — you'll need them in Step 5.

> 📸 [Screenshot: The user creation success page showing ravi-developer with the download CSV option]
> ![The user creation success page showing ravi-developer with the download CSV option](screenshots/iam-user-creation-success.png)

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> In a real production environment, you would **NEVER** give AdministratorAccess to a developer! We're doing it here so you can explore freely. In production, use the principle of least privilege — give only what's needed.

#### Create User 2: ravi-readonly

1. Go back to **IAM → Users → Create user**.
2. **User name:** `ravi-readonly`
3. Do **NOT** check "Provide user access to the AWS Management Console" — this user will only have programmatic (CLI/API) access.
4. Click **Next**.
5. On the "Set permissions" page, select: ⚫ **Add user to group**
6. Click **Create group**:
   - Group name: `ReadOnlyUsers`
   - Check the box: ☑ **ReadOnlyAccess**
   - Click **Create group**
7. The `ReadOnlyUsers` group is now created and `ravi-readonly` is added to it.
8. Click **Next** → **Create user**.
9. After creation, click on the `ravi-readonly` user → **Security credentials** tab → under "Access keys" click **Create access key**.
10. Select **Command Line Interface (CLI)** → check the confirmation box → **Next** → **Create access key**.
11. ⚠️ **Copy the Access Key ID and Secret Access Key NOW.** You cannot see the secret key again after you leave this page!

```
Access Key ID:     AKIAIOSFODNN7EXAMPLE    ← looks like this
Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY  ← looks like this
```


> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Store those access keys in a password manager or a secure file. Never commit them to Git. Never. I mean it, Ravi. 🔒

---

> <img src="https://img.shields.io/badge/Step%202-Create%20IAM%20Groups-3498DB?style=for-the-badge" />

1. In the left navigation pane, click **User groups**.
2. You should see both groups: `Developers` (AdministratorAccess) and `ReadOnlyUsers` (ReadOnlyAccess).
3. Open each group and check its **Permissions** tab — you'll see the attached managed policy.

> 📸 [Screenshot: The Developers group permissions tab showing AdministratorAccess policy]
> ![The Developers group permissions tab showing AdministratorAccess policy](screenshots/iam-developers-group-permissions.png)

**Understanding Group Policies:**

| Group | Policy Attached | What It Allows |
|-------|----------------|----------------|
| Developers | AdministratorAccess | Full access to everything (all services, all actions, all resources) |
| ReadOnlyUsers | ReadOnlyAccess | Read-only access to all AWS services (can view but cannot modify) |

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Groups are like teams. When someone joins the team, you add them to the group. When they leave, you remove them. The permissions follow the group, not the individual. This makes managing permissions at scale much easier!

---

> <img src="https://img.shields.io/badge/Step%203-Create%20Custom%20IAM%20Policy-E67E22?style=for-the-badge" />

Now let's create a **custom policy** using the visual editor — no JSON required!

1. In the left navigation pane, click **Policies**.
2. Click **Create policy**.
3. You'll see two tabs: **Visual** and **JSON**. Make sure **Visual** is selected.

#### Configure the Policy:

1. **Service:** Search for and select **S3**.
2. **Actions allowed:**
   - Expand **List** → check ☑ `ListBucket`
   - Expand **Read** → check ☑ `GetObject`
   - Expand **Write** → check ☑ `PutObject`
3. **Resources:**
   - For `ListBucket`: Click **Add ARN** → Bucket name: `ravi-dev-bucket-*` (use wildcard)
   - For `GetObject`: Click **Add ARN** → Bucket name: `ravi-dev-bucket-*`, Object name: `*`
   - For `PutObject`: Click **Add ARN** → Bucket name: `ravi-dev-bucket-*`, Object name: `*`
4. Click **Next**.
5. **Optional - Add conditions:**
   - Click **Add condition**
   - Condition key: `aws:SourceIp`
   - Condition operator: `IpAddress`
   - Value: Your current public IP (you can find it by Googling "what is my IP")
6. Click **Next**.
7. **Policy name:** `S3DeveloperAccess`
8. **Description:** `Allows listing, getting, and putting objects in ravi-dev-bucket-* with IP restriction`
9. Click **Create policy**.

> 📸 [Screenshot: S3DeveloperAccess custom policy visual editor]
> ![The visual policy editor showing S3 actions selected and the policy summary](screenshots/iam-s3developer-policy.png)

#### Attach the Custom Policy to Developers Group:

1. Go to **User groups → Developers**.
2. Click the **Permissions** tab → **Add permissions → Attach policies**.
3. Search for `S3DeveloperAccess`.
4. Check the box next to it → **Add permissions**.

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> That IP condition is a great security practice! It means even if someone steals the developer's credentials, they can only use them from your IP address. In production, you'd also use MFA, VPC endpoints, and more. Security is a layer cake, Ravi — and you want LOTS of layers! 🎂

---

> <img src="https://img.shields.io/badge/Step%204-Create%20IAM%20Role-9B59B6?style=for-the-badge" />

1. In the left navigation pane, click **Roles**.
2. Click **Create role**.
3. **Trusted entity type:** ⚫ AWS service
4. **Use case:** Select **EC2** from the dropdown → Click **Next**.
5. **Add permissions:** Search for and check ☑ `AmazonS3ReadOnlyAccess` → Click **Next**.
6. **Role name:** `EC2-S3-ReadOnly-Role`
7. **Description:** `Allows EC2 instances to read S3 buckets`
8. Click **Create role**.

> 📸 [Screenshot: The role creation page showing EC2 as the trusted entity with S3ReadOnlyAccess policy]
> ![The role creation page showing EC2 as the trusted entity with S3ReadOnlyAccess policy](screenshots/iam-ec2-role-creation.png)


#### Attach the Role to an EC2 Instance:

1. Go to **EC2 → Instances → Launch instance**.
2. **Name:** `ravi-iam-test`
3. **AMI:** Amazon Linux 2023 (Free Tier eligible)
4. **Instance type:** `t2.micro` (Free Tier eligible)
5. Under **Key pair (login)**: Select your existing key pair or create a new one.
6. Under **Network settings**: Allow SSH (port 22) from My IP.
7. Under **Advanced details → IAM instance profile:** Select `EC2-S3-ReadOnly-Role`.
8. Click **Launch instance** → **View all instances**.
9. Wait for the instance to reach **Running** state and pass both status checks.

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> The IAM role is like a temporary badge you hand to the EC2 instance. The instance doesn't store any credentials — AWS handles the authentication behind the scenes. This is WAY better than hardcoding access keys in your code!

#### Test the Role from CLI:

1. SSH into your EC2 instance:
   ```bash
   ssh -i your-key.pem ec2-user@<your-public-ip>
   ```

2. Try listing S3 buckets (should **succeed** — read-only access):
   ```bash
   aws s3 ls
   ```
   You should see a list of buckets (or nothing if you have none). No error means it worked!

3. Now try to create a bucket (should **fail** — no write permissions):
   ```bash
   aws s3 mb s3://test-bucket-fail-$(date +%s)
   ```
   Expected output:
   ```
   An error occurred (AccessDenied) when calling the CreateBucket operation: Access Denied
   ```

> 📸 [Screenshot: Terminal showing the successful `aws s3 ls` and the failed `aws s3 mb` command]
> ![Terminal showing the successful `aws s3 ls` and the failed `aws s3 mb` command](screenshots/iam-role-cli-test.png)

🎉 **If you see "Access Denied" on the create command, your IAM role is working perfectly!** The role only allows reading, not writing.

---

> <img src="https://img.shields.io/badge/Step%205-Test%20IAM%20Policies-E74C3C?style=for-the-badge" />

Now let's test the different user permissions.

#### Test ravi-readonly User:

1. Open a **new incognito/private browser window**.
2. Go to the AWS sign-in page: `https://<account-id>.signin.aws.amazon.com/console`
   - Or use the URL from the CSV you downloaded earlier.
3. Sign in as `ravi-readonly` (note: this user has no console access if you didn't enable it, so you may skip this or sign in with a user that has console access).
4. If console access was enabled:
   - Try to go to **EC2** → You should see instances but **cannot** stop/start/terminate them.
   - Try to go to **S3** → You should see buckets (read-only).
   - Try to create anything → You should see **"Unauthorized"** or **"Access Denied"** messages everywhere.

#### Test ravi-developer User:

1. Open another **incognito/private browser window**.
2. Sign in as `ravi-developer` using the Console sign-in URL from the CSV.
3. **Username:** `ravi-developer`
4. **Password:** The password you set earlier.
5. You should have full access to almost everything (thanks to AdministratorAccess).
6. Navigate around — you can create EC2 instances, S3 buckets, etc.

> 📸 [Screenshot: Comparison showing ravi-readonly being denied vs ravi-developer having access]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Notice how different users see different things in the same AWS account? That's IAM in action! In a real company, the HR person sees billing and support, the developer sees code pipeline and databases, and the security team sees CloudTrail and GuardDuty. Everyone gets exactly what they need — nothing more, nothing less.

---

> <img src="https://img.shields.io/badge/Step%206-IAM%20Access%20Analyzer%20(Bonus)-1ABC9C?style=for-the-badge" />

1. Go back to the **root/admin** account.
2. In the IAM dashboard, click **Access analyzer** in the left pane.
3. Click **Create analyzer**.
4. **Analyzer name:** `ravi-access-analyzer`
5. **Region:** Select your current region.
6. Click **Create analyzer**.
7. Wait for the analyzer to run (takes 1-2 minutes).
8. Review the **Findings** — this will show you any resources that are publicly accessible or shared with external accounts.

> 📸 [Screenshot: IAM Access Analyzer findings page]
> ![IAM Access Analyzer findings page](screenshots/iam-access-analyzer-findings.png)

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Access Analyzer is like a security guard that watches your AWS account 24/7 and tells you if you've accidentally left the front door open. Run it periodically in production!

---

> <img src="https://img.shields.io/badge/Step%207-Verify%20Your%20Work-F39C12?style=for-the-badge" />

Let's confirm everything was set up correctly:

1. **IAM Users exist:**
   - Go to IAM → Users → You should see both `ravi-developer` and `ravi-readonly`.

2. **IAM Groups exist:**
   - Go to IAM → User groups → You should see `Developers` and `ReadOnlyUsers`.

3. **Custom Policy exists:**
   - Go to IAM → Policies → Search for `S3DeveloperAccess` → It should appear.

4. **IAM Role exists:**
   - Go to IAM → Roles → Search for `EC2-S3-ReadOnly-Role` → It should appear.

5. **EC2 test confirmed:**
   - `aws s3 ls` succeeded ✅
   - `aws s3 mb` failed with AccessDenied ✅

---

## ✅ Validation Checklist

| # | Check | Status |
|---|-------|--------|
| 1 | `ravi-developer` user exists with console access | ☐ ✅ |
| 2 | `ravi-readonly` user exists with programmatic access | ☐ ✅ |
| 3 | `Developers` group has `AdministratorAccess` | ☐ ✅ |
| 4 | `ReadOnlyUsers` group has `ReadOnlyAccess` | ☐ ✅ |
| 5 | `S3DeveloperAccess` custom policy exists | ☐ ✅ |
| 6 | `EC2-S3-ReadOnly-Role` role exists | ☐ ✅ |
| 7 | EC2 instance launched with the IAM role attached | ☐ ✅ |
| 8 | `aws s3 ls` succeeded from EC2 | ☐ ✅ |
| 9 | `aws s3 mb` failed from EC2 (AccessDenied) | ☐ ✅ |
| 10 | Access Analyzer created and findings reviewed | ☐ ✅ |

---

> **Achievement Unlocked:** Security Sentinel! IAM policies under control.

---

## 🧹 Cleanup (IMPORTANT!)

**Even though IAM is free, you should clean up to keep your account tidy and avoid confusion in future labs!**

### 🗑️ Delete IAM Users:
1. Go to **IAM → Users**.
2. Click on `ravi-developer` → **Delete** → Type the username to confirm → **Delete user**.
3. Click on `ravi-readonly` → **Delete** → Type the username to confirm → **Delete user**.

### 🛑 Delete IAM Groups:
1. Go to **IAM → User groups**.
2. Click on `Developers` → **Delete** → Confirm → **Delete**.
3. Click on `ReadOnlyUsers` → **Delete** → Confirm → **Delete**.

### 🧹 Delete Custom Policy:
1. Go to **IAM → Policies**.
2. Search for `S3DeveloperAccess`.
3. Click on it → **Delete** → Type the policy name to confirm → **Delete policy**.

### 💾 Delete IAM Role:
1. Go to **IAM → Roles**.
2. Click on `EC2-S3-ReadOnly-Role` → **Delete** → Confirm → **Delete role**.

### 🔄 Terminate EC2 Instance:
1. Go to **EC2 → Instances**.
2. Select `ravi-iam-test` → **Instance state → Terminate instance** → **Terminate**.

### 📸 Delete Access Analyzer:
1. Go to **IAM → Access analyzer** → Delete the analyzer.

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Always clean up, even when things are free. A messy AWS account is like a messy desk — you can't find anything when you need it!

---

## 🧠 Memory Tips

Stick these in your brain and they'll never leave. 🧲

| 🧠 Memory Hook | Remember it like... |
|---|---|
| **Users = people, Roles = hats** | A **user** is a person with a password/keys. A **role** is a **temporary hat** worn by a service (like EC2) to get permissions. 👒 |
| **Policy = permission slip** | JSON that says "allow `s3:ListBucket` on bucket X." Show the slip, get access. 📝 |
| **Least privilege** | Give **exactly enough** and nothing more. Like giving a contractor one room key, not the master key. 🔑 |
| **Managed vs custom** | Managed = AWS's pre-built policies. Custom = your own JSON. Start managed, customize when needed. 🏗️ |
| **Root = master key** | Root can do *anything* — so it's locked in a vault (MFA) and used almost never. 🏦 |

> 🗣️ **Rithu:** *"If a service can read your keys from a config file, an attacker can too. Roles are how the pros do it — no keys, no problem."

---

## 🎓 What You Learned

In this lab, you learned:

1. **IAM Users** — Individual identities with their own credentials (console password and/or access keys).
2. **IAM Groups** — Collections of users that share the same permission set.
3. **IAM Policies** — JSON documents that define what actions are allowed or denied on which resources.
4. **Managed Policies vs Custom Policies** — AWS provides pre-built policies; you can also create your own.
5. **IAM Roles** — Temporary credentials assumed by AWS services (like EC2) instead of hardcoding access keys.
6. **Principle of Least Privilege** — Give only the minimum permissions needed.
7. **IAM Access Analyzer** — A tool that finds resources shared externally or publicly.

---

## 🎮 Test Yourself! (No Peeking 👀)

**Q1:** What's the difference between an IAM User and an IAM Role?

<details><summary>👀 Show answer</summary>

**A:** A **user** is a person with long-term credentials. A **role** is a **temporary identity** that a service (like EC2) assumes to get permissions — no passwords or keys to leak. 👒

</details>

**Q2:** What does "principle of least privilege" mean in practice?

<details><summary>👀 Show answer</summary>

**A:** Grant **only the minimum permissions** needed to do the job. If it only needs to read S3, don't give it full admin. 🔑

</details>

**Q3:** When should you use an AWS managed policy vs writing a custom one?

<details><summary>👀 Show answer</summary>

**A:** Use **managed** policies for common jobs (they're maintained by AWS). Write a **custom** policy when you need exactly specific, narrow permissions. 📝

</details>

### 🔥 Bonus Challenge

Create a **role** with ONLY `s3:ListBucket` + `s3:GetObject` on one bucket, attach it to an EC2 instance, then try to *delete* an object from that instance — it should fail with Access Denied. Proving least privilege works. Then grant `s3:DeleteObject` and watch it succeed. 🎯

> 💪 **Rithu:** *"Security isn't about what you can do — it's about what you CAN'T. Test the denials."

---

### 🆚 Pro Tip vs Noob Tip

| | Approach |
|---|---|
| **Noob Tip** | Give every user AdministratorAccess "so they never get stuck" |
| **Pro Tip** | Least privilege + groups + roles. If a credential leaks, the blast radius is tiny |

---

## 🔗 What's Next?

Now that you understand how IAM secures access to AWS, it's time to learn about **messaging and event-driven architectures**!

👉 **[Lab 17 — SNS and SQS: Messaging](../17%20-%20SNS%20and%20SQS%20-%20Messaging/README.md)**

In the next lab, you'll create pub/sub notifications with SNS and message queues with SQS — the backbone of decoupled, scalable architectures!

---

## ❓ Troubleshooting

<details>
<summary><strong>🔍 "Access Denied" when trying to create IAM users/groups</strong></summary>

**Cause:** You're signed in as a user without IAM permissions.
**💡 Fix:** Sign in as root or an IAM user with `AdministratorAccess`.

</details>

<details>
<summary><strong>🔍 "Entity already exists" when creating a user or group</strong></summary>

**Cause:** The name is already taken (IAM names are global across your account).
**💡 Fix:** Use a different name or delete the existing entity first.

</details>

<details>
<summary><strong>🔍 EC2 cannot assume the IAM role</strong></summary>

**Cause:** The role wasn't attached during launch, or the trust policy is wrong.
**🔧 Fix:** You can't attach a role to a running instance. Terminate it and launch a new one with the correct IAM instance profile.

</details>

<details>
<summary><strong>🔍 Access keys show "pending" or don't appear</strong></summary>

**Cause:** Access keys are only available immediately after creation.
**💡 Fix:** If you lost them, delete the old access key and create a new one from the Security credentials tab.

</details>

<details>
<summary><strong>🔍 `aws s3 ls` fails from EC2</strong></summary>

**Cause:** The instance doesn't have an IAM role attached, or the role doesn't have S3 permissions.
**🔧 Fix:** Check the instance's IAM profile in EC2 console. If missing, terminate and relaunch with the correct role.

</details>

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> IAM issues are the #1 cause of "it worked for me but not for you" problems in AWS. When something doesn't work, check IAM permissions first — 9 times out of 10, that's the culprit!

---

<div align="center">

<img src="https://img.shields.io/badge/Lab%2016-Complete!-3498DB?style=for-the-badge&labelColor=232F3E" />

> 🎉 **Fantastic work, Ravi!** You've mastered IAM — the foundation of AWS security. You now know how to create users, groups, custom policies, and roles. Remember: security is never optional in the cloud. Keep building securely! 🔐🚀

</div>
