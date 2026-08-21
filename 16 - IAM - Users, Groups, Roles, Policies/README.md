# 🔐 Lab 16 - IAM: Users, Groups, Roles, Policies

> 📅 **Updated:** 21 August 2026 | ⏱️ **Duration:** ~30 minutes | 📊 **Level:** Beginner

![IAM](https://img.shields.io/badge/IAM-Users%20%7C%20Groups%20%7C%20Roles%20%7C%20Policies-3498DB?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-2ECC71?style=flat-square)
![Time](https://img.shields.io/badge/Time-~30%20minutes-2F80ED?style=flat-square)
![Cost](https://img.shields.io/badge/Cost-FREE-2ECC71?style=flat-square)

> ### 🗣️ *"IAM is the bouncer of your AWS club — nobody gets in without checking with the bouncer first!"*
> — **Rithu** 🪪

<details>
<summary><b>🎭 Ravi & Rithu's Coffee Break Chat</b></summary>

**Ravi:** "IAM seems complicated..."

**Rithu:** "It's like a permission slip system. Who can do what, where, and when."

**Ravi:** "Can I just give everyone admin access?"

**Rithu:** "You CAN. It's also the fastest way to get your AWS account compromised. So... don't."

</details>

---

## 🎯 What You'll Master

| Skill | Description |
|-------|-------------|
| 👤 **IAM Users** | Console vs programmatic access, credentials |
| 👥 **User Groups** | Permissions follow the team, not the person |
| 📝 **Custom Policies** | Visual editor: S3 actions + ARNs + IP condition |
| 👒 **IAM Roles** | Temporary hats for services — no stored keys |
| 🧪 **Permission Testing** | Prove read-only works AND write fails |
| 🔍 **Access Analyzer** | Find accidentally-exposed resources |

> 💡 **Pro Tip:** This is the most important lab in the series. Every AWS breach you've ever read about came down to IAM mistakes. Get this right and you're 90% safer than most AWS users.

---

## 🚦 Before You Start

### ✅ Prerequisites Checklist
- [ ] ✅ **[Lab 15](../15%20-%20CloudWatch%20-%20Alarms%20and%20Dashboards/README.md)** complete *(Rithu: "If you haven't finished Lab 15, go do that first. I'll wait. No really, I'll be here. ☕")*
- [ ] 🔑 Root or admin console access
- [ ] 💻 AWS CLI configured (optional, for Step 4)
- [ ] 🔐 Existing SSH key pair

### 📦 What You Need (and Don't)
| Required | Optional |
|----------|----------|
| ~30 minutes | CLI configured locally |
| Incognito browser windows for testing | |

---

## 💰 Cost & Safety First

**This lab is 100% FREE!** 🎉 IAM never charges. But the resources IAM entities access (EC2!) can — so cleanup still matters.

> 💸 **Ravi's Mistake of the Day:** *"I created an IAM user with AdministratorAccess and used those credentials in a public GitHub repo. AWS automatically flagged it within minutes. Don't commit credentials. Ever. Period."*

### 🏷️ Naming Convention

| Resource | Name |
|----------|------|
| 👤 User 1 | `ravi-developer` (console) |
| 👤 User 2 | `ravi-readonly` (CLI keys) |
| 👥 Groups | `Developers` · `ReadOnlyUsers` |
| 📝 Policy | `S3DeveloperAccess` |
| 👒 Role | `EC2-S3-ReadOnly-Role` |
| 🖥️ Test Instance | `ravi-iam-test` |
| 🔍 Analyzer | `ravi-access-analyzer` |

---

## 🧠 How It All Fits Together

```mermaid
graph TD
    subgraph USERS["👥 Identity Layer"]
        DEV["👤 ravi-developer<br/>console login"]
        RO["👤 ravi-readonly<br/>access keys only"]
    end
    subgraph GROUPS["🧩 Group Layer"]
        G1["👥 Developers<br/>AdministratorAccess"]
        G2["👥 ReadOnlyUsers<br/>ReadOnlyAccess"]
    end
    POL["📝 S3DeveloperAccess<br/>custom policy · ravi-dev-bucket-*<br/>+ aws:SourceIp condition"] -->|"attached"| G1
    DEV --> G1
    RO --> G2
    subgraph SERVICE["🖥️ Service Layer"]
        EC2["🖥️ ravi-iam-test<br/>t2.micro"]
        ROLE["👒 EC2-S3-ReadOnly-Role<br/>AmazonS3ReadOnlyAccess"]
    end
    ROLE -->|"instance profile"| EC2
    ROLE -->|"read only"| S3["🪣 S3 Buckets"]

    style DEV fill:#3498DB,color:#fff
    style RO fill:#3498DB,color:#fff
    style G1 fill:#E67E22,color:#fff
    style G2 fill:#27AE60,color:#fff
    style POL fill:#F39C12,color:#fff
    style EC2 fill:#FF9800,color:#fff
    style ROLE fill:#9B59B6,color:#fff
    style S3 fill:#2ECC71,color:#fff
```

### 🔑 Key Concepts
| Concept | Remember It Like... |
|---------|---------------------|
| **Users = people, Roles = hats** | A user has a password/keys. A role is a temporary hat a service wears. 👒 |
| **Policy = permission slip** | JSON saying "allow `s3:ListBucket` on bucket X." Show slip, get access. 📝 |
| **Least privilege** | One room key for the contractor — never the master key. 🔑 |
| **Managed vs custom** | Managed = AWS pre-built. Custom = your own JSON. Start managed. 🏗️ |
| **Root = master key** | Can do anything → locked in an MFA vault, used almost never. 🏦 |

> 💡 Policies max out at 6,144 characters — but shorter is always better.

---

## 🪜 Step-by-Step Guide

### 🟢 Step 1: Create Two IAM Users 👤

<details>
<summary><b>👤 Expand for user creation</b></summary>

**User 1 — `ravi-developer` (console access):**

1. Console search → **IAM** → left nav **Users** → ➕ **Create user**
2. User name: `ravi-developer`
3. ☑ Provide user access to the AWS Management Console → ⚫ I want to create an IAM user
4. ⚫ Custom password → strong password (e.g., `RaviLabs2024!`) — leave *"Users must create a password at next sign-in"* **unchecked** so it still works in Step 5. Write it down!
5. **Next** → Set permissions: ⚫ Add user to group → **Create group**: `Developers` ☑ AdministratorAccess → Create group
6. **Next → Next → Create user**
7. ⚠️ **Download .csv file** — you need the sign-in URL + credentials for Step 5!

**User 2 — `ravi-readonly` (programmatic only):**

1. IAM → Users → ➕ **Create user** → `ravi-readonly`
2. Do **NOT** check console access — CLI/API only
3. **Next** → ⚫ Add user to group → **Create group**: `ReadOnlyUsers` ☑ ReadOnlyAccess → Create group
4. **Next → Create user**
5. Open `ravi-readonly` → **Security credentials** tab → Access keys → **Create access key**
6. ⚫ Command Line Interface (CLI) → ☑ confirmation → **Next → Create access key**
7. ⚠️ **Copy Access Key ID + Secret NOW** — the secret is shown exactly once!

```
Access Key ID:     AKIAIOSFODNN7EXAMPLE    ← looks like this
Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY  ← looks like this
```

</details>

![The user creation success page showing ravi-developer with the download CSV option](screenshots/iam-user-creation-success.png)

> 🗣️ **Rithu's Tip:** *"NEVER give AdministratorAccess to a developer in production! We do it here so you can explore freely. Production = least privilege. And store those access keys in a password manager — never commit them to Git. Never. I mean it, Ravi. 🔒"*

---

### 🟢 Step 2: Verify the Groups 👥

<details>
<summary><b>👥 Expand for group verification</b></summary>

1. IAM → **User groups**
2. You should see both: `Developers` (AdministratorAccess) and `ReadOnlyUsers` (ReadOnlyAccess)
3. Open each → check the **Permissions** tab for the attached managed policy

</details>

![The Developers group permissions tab showing AdministratorAccess policy](screenshots/iam-developers-group-permissions.png)

> 🗣️ **Rithu's Tip:** *"Groups are teams. Someone joins → add to group. Someone leaves → remove. Permissions follow the group, not the individual — that's how you manage permissions at scale!"*

---

### 🟢 Step 3: Create a Custom Policy (Visual Editor) 📝

<details>
<summary><b>📝 Expand for policy creation</b></summary>

1. IAM → **Policies** → ➕ **Create policy** → keep **Visual** tab selected
2. Service: **S3**
3. Actions:
   - **List** → ☑ `ListBucket`
   - **Read** → ☑ `GetObject`
   - **Write** → ☑ `PutObject`
4. Resources (Add ARN):
   - `ListBucket` → Bucket: `ravi-dev-bucket-*`
   - `GetObject` / `PutObject` → Bucket: `ravi-dev-bucket-*`, Object: `*`
5. **Next** → *(optional)* **Add condition**: key `aws:SourceIp` · operator `IpAddress` · value = your public IP ("what is my IP")
6. **Next** → Name: `S3DeveloperAccess` · Description: `Allows listing, getting, and putting objects in ravi-dev-bucket-* with IP restriction`
7. ✅ **Create policy**

**Attach to the Developers group:**

8. **User groups → Developers** → Permissions tab → **Add permissions → Attach policies** → search `S3DeveloperAccess` → ☑ → **Add permissions**

</details>

![The visual policy editor showing S3 actions selected and the policy summary](screenshots/iam-s3developer-policy.png)

> 🗣️ **Rithu's Tip:** *"That IP condition is gold! Even if someone steals the developer's credentials, they only work from your IP. In production add MFA, VPC endpoints... Security is a layer cake, Ravi — you want LOTS of layers! 🎂"*

---

### 🟢 Step 4: Create a Role & Attach It to EC2 👒

<details>
<summary><b>👒 Expand for role creation + EC2 test</b></summary>

**Create the role:**

1. IAM → **Roles** → ➕ **Create role**
2. Trusted entity type: ⚫ **AWS service** · Use case: **EC2** → Next
3. Permissions: ☑ `AmazonS3ReadOnlyAccess` → Next
4. Name: `EC2-S3-ReadOnly-Role` · Description: `Allows EC2 instances to read S3 buckets` → ✅ **Create role**

**Launch a test instance wearing the hat:**

5. EC2 → Launch instance → Name: `ravi-iam-test` · AL2023 · `t2.micro` · your key pair · SSH 22 from My IP
6. **Advanced details → IAM instance profile:** select `EC2-S3-ReadOnly-Role`
7. Launch → wait for Running + 2/2 checks

**Prove least privilege from the CLI:**

```bash
ssh -i your-key.pem ec2-user@<your-public-ip>
```

Read should **succeed**:

```bash
aws s3 ls
```

Write should **fail**:

```bash
aws s3 mb s3://test-bucket-fail-$(date +%s)
```

Expected:

```
An error occurred (AccessDenied) when calling the CreateBucket operation: Access Denied
```

🎉 **Seeing AccessDenied means the role works perfectly!**

</details>

![The role creation page showing EC2 as the trusted entity with S3ReadOnlyAccess policy](screenshots/iam-ec2-role-creation.png)

![Terminal showing the successful aws s3 ls and the failed aws s3 mb command](screenshots/iam-role-cli-test.png)

> 🗣️ **Rithu's Tip:** *"A role is a temporary badge you hand the instance. No credentials stored on disk — AWS handles auth behind the scenes. WAY better than hardcoding keys!"*

---

### 🟢 Step 5: Test Both Users' Permissions 🧪

<details>
<summary><b>🧪 Expand for permission tests</b></summary>

**Test `ravi-developer`:**

1. New incognito window → sign-in URL from the CSV (`https://<account-id>.signin.aws.amazon.com/console`)
2. Username `ravi-developer` + your custom password
3. Full access almost everywhere (AdministratorAccess) — create instances, buckets, explore!

**Test `ravi-readonly`:**

4. This user has no console access (by design) — its power lives in the CLI keys. If you'd enabled console access, you'd see everything but be unable to change anything.

</details>

> 🗣️ **Rithu's Tip:** *"Different users see different things in the same account — that's IAM in action! HR sees billing, developers see pipelines, security sees CloudTrail. Everyone gets exactly what they need — nothing more, nothing less."*

---

### 🟢 Step 6: Bonus — Access Analyzer 🔍

<details>
<summary><b>🔍 Expand for Access Analyzer</b></summary>

1. Back in the **root/admin** session → IAM → **Access analyzer**
2. ➕ **Create analyzer** → Name: `ravi-access-analyzer` · Region: current → **Create analyzer**
3. Wait 1–2 minutes → review **Findings**: any resources publicly accessible or shared with external accounts

</details>

![IAM Access Analyzer findings page](screenshots/iam-access-analyzer-findings.png)

> 🗣️ **Rithu's Tip:** *"Access Analyzer is a guard watching your account 24/7, telling you when you've left the front door open. Run it periodically in production!"*

---

## ✅ Validation Checklist

| # | Check | Status |
|---|-------|--------|
| 1️⃣ | `ravi-developer` exists with console access | ☐ ✅ |
| 2️⃣ | `ravi-readonly` exists with programmatic access | ☐ ✅ |
| 3️⃣ | `Developers` group has AdministratorAccess | ☐ ✅ |
| 4️⃣ | `ReadOnlyUsers` group has ReadOnlyAccess | ☐ ✅ |
| 5️⃣ | `S3DeveloperAccess` custom policy exists | ☐ ✅ |
| 6️⃣ | `EC2-S3-ReadOnly-Role` exists | ☐ ✅ |
| 7️⃣ | EC2 launched with the role attached | ☐ ✅ |
| 8️⃣ | `aws s3 ls` succeeded from EC2 | ☐ ✅ |
| 9️⃣ | `aws s3 mb` failed with AccessDenied | ☐ ✅ |
| 🔟 | Access Analyzer created + findings reviewed | ☐ ✅ |

---

## 🧹 Cleanup (Follow Order!)

Even though IAM is free — clean up! A messy account is a messy desk. 🧹

| Step | Action | Console Location |
|------|--------|------------------|
| 1️⃣ 🖥️ | Terminate `ravi-iam-test` | EC2 → Instances |
| 2️⃣ 👤 | Delete `ravi-developer` + `ravi-readonly` (type name to confirm) | IAM → Users |
| 3️⃣ 👥 | Delete `Developers` + `ReadOnlyUsers` groups | IAM → User groups |
| 4️⃣ 📝 | Delete `S3DeveloperAccess` policy (type name to confirm) | IAM → Policies |
| 5️⃣ 👒 | Delete `EC2-S3-ReadOnly-Role` | IAM → Roles |
| 6️⃣ 🔍 | Delete `ravi-access-analyzer` | IAM → Access analyzer |

---

## 🚀 Level Ups (Post-Core Lab)

| Challenge | What to Try | Notes |
|-----------|-------------|-------|
| 🎯 **Denial Testing** | Role with ONLY `s3:ListBucket` + `s3:GetObject` → try deleting an object (fails!) → grant `s3:DeleteObject` → succeeds | Security isn't what you CAN do — it's what you CAN'T. Test the denials. |
| 🔐 **MFA Enforcement** | Add an MFA-required condition to a policy | Layer cake, remember? |
| 🏷️ **Permission Boundaries** | Cap a user's max permissions with a boundary policy | Advanced guardrails |

---

## 🆘 Troubleshooting Quick Reference

| 🔍 Issue | 💡 Likely Cause | 🔧 Fix |
|-------|--------------|-----|
| ❌ Access Denied creating users/groups | Signed in without IAM permissions | Use root or an AdministratorAccess user |
| ⚠️ "Entity already exists" | IAM names are unique per account | Rename it or delete the old entity first |
| 👒 EC2 can't assume the role | Role not attached at launch / bad trust policy | Roles can't be added at launch-time retroactively via old consoles — terminate & relaunch with the instance profile |
| 🔑 Access keys lost/"pending" | Secret shown only once | Delete the key, create a new one from Security credentials |
| ❌ `aws s3 ls` fails from EC2 | No role attached / missing S3 perms | Check the instance's IAM profile; relaunch if missing |

> 🗣️ **Rithu's Tip:** *"IAM issues are the #1 cause of 'it worked for me but not for you.' When something breaks, check IAM first — 9 times out of 10, that's the culprit!"*

---

## 🎮 Test Yourself! (No Peeking 👀)

**Q1:** Difference between an IAM User and an IAM Role?

<details><summary>👀 Show answer</summary>

**A:** A **user** has long-term credentials. A **role** is a temporary identity a service assumes — no passwords or keys to leak. 👒

</details>

**Q2:** What does "least privilege" mean in practice?

<details><summary>👀 Show answer</summary>

**A:** Grant **only the minimum** needed. Read-only job → no full admin. 🔑

</details>

**Q3:** Managed vs custom policy — when to use which?

<details><summary>👀 Show answer</summary>

**A:** **Managed** for common jobs (AWS-maintained). **Custom** for narrow, specific permissions. 📝

</details>

---

## 📚 Official Documentation

- 🔐 [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- 📝 [IAM JSON Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html)
- 👒 [IAM Roles for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)
- 🔍 [IAM Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)

---

## 🎓 What You Learned

> **The bouncer's rulebook:**
> - 👤 **Users** → individual identities (console password and/or access keys)
> - 👥 **Groups** → shared permission sets; people come and go, permissions stay
> - 📝 **Policies** → JSON allow/deny documents; managed vs custom
> - 👒 **Roles** → temporary service credentials instead of hardcoded keys
> - 🔍 **Access Analyzer** → finds externally-shared resources

**Golden Habit:** Least privilege + groups + roles → tiny blast radius when credentials leak. 🔐

| | Approach |
|---|---|
| 👶 **Noob Way** | Everyone gets AdministratorAccess "so they never get stuck" |
| 🧙 **Pro Way** | Least privilege + groups + roles — leaks hurt as little as possible |

---

## ➡️ What's Next?

You control WHO does WHAT. Next: decouple your architecture with pub/sub notifications and message queues. 📬

🎯 **[Lab 17 - SNS and SQS: Messaging](../17%20-%20SNS%20and%20SQS%20-%20Messaging/README.md)**

---

<div align="center">

### ⭐ Enjoyed this lab? Star the repo & share your feedback!

**Happy Learning!** 🚀☁️

</div>