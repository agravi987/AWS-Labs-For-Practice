# Lab 24 — AWS Backup: Multi-Service Backup

![Difficulty](https://img.shields.io/badge/Difficulty-Medium-yellow)
![Time](https://img.shields.io/badge/Time-~35min-blue)
![Cost](https://img.shields.io/badge/Cost-%3C%242-orange)
![Service](https://img.shields.io/badge/Service-AWS%20Backup-red)

> "Ravi, backups aren't exciting — until you NEED one. Then they're the most exciting thing in the world!" — Rithu

---

## 🎯 Objective

By the end of this lab, you will:
- Understand why backups are critical and how AWS Backup centralizes them
- Create a backup plan with scheduling and retention rules
- Assign multiple AWS resources to a backup plan
- Run an on-demand backup and restore from it
- View backup compliance reports

AWS Backup is your **central backup dashboard** for AWS. Instead of manually snapshotting each service, you create one plan that covers EC2, RDS, S3, DynamoDB, and more. It's like setting one alarm clock for every morning instead of five.

---

## 🧠 Prerequisites

- [ ] Completed Lab 23 (KMS)
- [ ] AWS Console access with appropriate permissions
- [ ] An EC2 key pair exists (from previous labs)
- [ ] Free Tier eligible for RDS

---

## 💰 Cost Warning

AWS Backup charges for stored backups and storage:

| What | Cost |
|------|------|
| EBS backup storage | $0.05/GB/month for first copy |
| RDS backup storage | Included with RDS Free Tier (up to 20GB) |
| DynamoDB backup | $0.10/GB/month |
| S3 backup (via AWS Backup) | Standard S3 pricing |
| On-demand backups | Same as service-native backup pricing |

Estimated total lab cost: **< $2** if cleaned up within 1 hour.

> ⚠️ **CRITICAL**: AWS Backup charges for stored backups! If you leave backups lying around, they WILL cost money. Delete all backup plans, vaults, and backups before leaving!

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│                  AWS Backup                          │
│                                                     │
│  ┌──────────────┐    ┌──────────────────────────┐   │
│  │ Backup Plan  │───▶│   Backup Jobs            │   │
│  │              │    │                          │   │
│  │ Schedule:    │    │  ┌─────┐  ┌─────┐       │   │
│  │ Daily 12:00  │    │  │ EC2 │  │ RDS │ ...   │   │
│  │ Retention:   │    │  │ Snap│  │ Snap│       │   │
│  │ 35 days      │    │  └─────┘  └─────┘       │   │
│  └──────────────┘    └──────────────────────────┘   │
│                                                     │
│  ┌──────────────┐    ┌──────────────────────────┐   │
│  │ On-Demand    │───▶│   Restore Jobs           │   │
│  │ Backup       │    │                          │   │
│  └──────────────┘    └──────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Create Resources to Back Up

Before we can back things up, we need something TO back up! Let's create a few resources.

**Launch an EC2 instance with an EBS volume:**

1. Go to **EC2** → **Launch instance**
2. **Name**: Type `ravi-backup-ec2`
3. **AMI**: Amazon Linux 2023 (Free Tier eligible)
4. **Instance type**: t2.micro
5. **Key pair**: Select your existing key pair
6. Under **Configure storage**:
   - Click **Advanced details** (expand)
   - **Size**: Change to `10` GiB (to have a meaningful volume to back up)
7. Click **Launch instance** → **View all instances**

Wait for the instance to be **Running**.

**Create an RDS database (Free Tier):**

8. Go to **RDS** in the AWS Console
9. Click **Create database**
10. **Creation method**: Select **Standard create**
11. **Engine type**: Select **MySQL**
12. **Engine version**: Select latest Free Tier eligible version
13. **Templates**: Select **Free tier**
14. **DB instance identifier**: Type `ravi-backup-db`
15. **Master username**: Type `admin`
16. **Master password**: Create a secure password (write it down!)
17. **Confirm password**: Re-enter the password
18. Scroll down to **Storage**:
    - **Storage type**: gp2
    - **Allocated storage**: 20 GB (Free Tier max)
    - **Storage autoscaling**: Uncheck to stay in Free Tier
19. Under **Connectivity**:
    - **Public access**: Select **No** (we'll connect from EC2)
20. Under **Additional configuration**:
    - **Initial database name**: Type `capstone_app`
21. Click **Create database**
22. Wait 5-10 minutes for the RDS instance to be **Available**

**Create an S3 bucket with files:**

23. Go to **S3** → **Create bucket**
24. **Bucket name**: Type `ravi-backup-bucket-12345` (add random numbers)
25. Click **Create bucket**
26. Click on the bucket → **Upload** → Add a few files from your computer
27. Click **Upload**

**Create a DynamoDB table:**

28. Go to **DynamoDB** in the AWS Console
29. Click **Create table**
30. **Table name**: Type `ravi-backup-table`
31. **Partition key**: Type `id` (String)
32. Click **Create table**
33. Wait for the table to be **Active**
34. Click on the table → **Explore table items** → **Create item**
35. Add an item with `id` = `1`, `name` = `Test User`
36. Add another item with `id` = `2`, `name` = `Another User`

> 💡 **Rithu's Tip**: "We're creating multiple resource types to show how AWS Backup can protect everything from a single dashboard. In the real world, you might back up databases, file systems, and virtual machines all from one backup plan."

📸 [Screenshot: Four AWS resources created — EC2, RDS, S3, DynamoDB]

---

### Step 2: Enable AWS Backup

Let's see what AWS Backup already knows about your resources.

1. Search for **AWS Backup** in the AWS Console search bar
2. Click on **AWS Backup**
3. Click **Protected resources** in the left sidebar
4. You might already see some of your resources listed here

**Understanding Protected Resources:**

- Some services are automatically discovered by AWS Backup
- Others need to be explicitly assigned to a backup plan
- EBS volumes and RDS databases are commonly discovered
- DynamoDB and S3 may need manual assignment

> 💡 **Rithu's Tip**: "AWS Backup auto-discovers some resources, but not all. Don't worry if you don't see everything yet — we'll assign them to our backup plan in the next steps."

📸 [Screenshot: AWS Backup Protected resources page showing discovered resources]

---

### Step 3: Create a Backup Plan

Now let's create the backup plan — this is the heart of AWS Backup!

1. Click **Backup plans** in the left sidebar
2. Click **Create backup plan** (top right)
3. Select **Start with a template** → Choose **Daily - 35 day retention**

**Configure the plan:**

4. **Backup plan name**: Type `ravi-backup-plan`
5. You'll see a default rule has been created. Click on the rule to edit it:

**Configure backup rule:**

6. **Rule name**: Type `daily-backup`
7. **Backup vault**: Select **Default**
8. **Backup schedule**: Select **Custom cron expression** and type:
   ```
   0 12 * * ? *
   ```
   (This runs every day at 12:00 UTC)

9. **Backup window**: Leave as default (4 hour window)
10. **Lifecycle**:
    - **Move to cold storage after**: Leave unchecked for this lab (in production, this saves costs)
    - **Delete after**: Type `35` days
11. **Copy to another AWS Region**: Leave unchecked (this is for cross-region backup — advanced)
12. Click **Create plan**

Your backup plan is created! It will run daily at noon UTC and retain backups for 35 days.

> 💡 **Rithu's Tip**: "The lifecycle settings are important for cost management. Cold storage (like Glacier) is cheaper for long-term retention. For this lab, we're keeping things simple, but in production, you'd definitely use lifecycle rules!"

📸 [Screenshot: Backup plan creation page showing the schedule and retention settings]

---

### Step 4: Assign Resources to Backup Plan

Now let's tell AWS Backup which resources to protect!

1. Click on `ravi-backup-plan` in the Backup plans list
2. Click the **Resources** tab
3. Click **Assign resources** (top right)

**Configure resource assignment:**

4. **Assignment name**: Type `ravi-backup-assignment`
5. **Resource type**: Select **EBS Volumes** and **RDS Databases** (you can select multiple)
6. **IAM role**: Select **Create new IAM role**
   - Leave the default name: `ravi-backup-role`
   - Click **Create role**
7. **Define resource selection**:
   - For EBS: Select your `ravi-backup-ec2` instance's volume
   - For RDS: Select your `ravi-backup-db` instance
8. Click **Assign resources**

**Wait for the backup to start:**

9. Go to **Backup jobs** in the left sidebar
10. You should see backup jobs starting for your resources
11. Wait for the jobs to show **Completed** status (this may take 5-15 minutes)

> 💡 **Rithu's Tip**: "The first backup is always a 'full backup' — it copies everything. Subsequent backups are incremental (only changed data), which is much faster and cheaper."

📸 [Screenshot: Backup jobs page showing completed backup jobs for EBS and RDS]

---

### Step 5: Run an On-Demand Backup

Don't want to wait for the scheduled backup? Run one right now!

1. Click **Protected resources** in the left sidebar
2. Click **Create on-demand backup** (top right)

**Configure the on-demand backup:**

3. **Resource type**: Select **EBS** (or the resource you want to back up)
4. **Resource ID**: Select your EBS volume (the one attached to `ravi-backup-ec2`)
5. **Backup vault**: Select **Default**
6. **Retention period**: Type `7` days
7. **IAM role**: Select the role we created earlier (`ravi-backup-role`)
8. Click **Create on-demand backup**

**Watch the backup:**

9. Go to **Backup jobs** in the left sidebar
10. You should see a new backup job with status **Running**
11. Wait for it to show **Completed**
12. Click on the job to see details like:
    - Backup creation time
    - Backup size
    - Backup vault where it's stored

> 💡 **Rithu's Tip**: "On-demand backups are great before risky operations. About to run a database migration? Take an on-demand backup first. About to test a destructive script? Back up first. The 'oops' insurance policy!"

📸 [Screenshot: On-demand backup job showing Completed status with backup details]

---

### Step 6: Restore from Backup

This is the moment of truth! Let's restore from a backup.

1. Go to **Backup vaults** in the left sidebar
2. Click on the **Default** vault
3. You should see a list of your backups
4. Select one of your EBS backups (the most recent one)
5. Click **Restore** (top right)

**Configure the restore:**

6. **Restore type**: Select **New resource** (this creates a new EBS volume — doesn't overwrite the original)
7. **Encryption**: Leave as default
8. **Availability Zone**: Select the same AZ as your original (e.g., us-east-1a)
9. **Volume type**: Select **gp3**
10. **Size**: Leave as default (same as original)
11. Click **Restore**

**Wait for restore:**

12. Go to **Restore jobs** in the left sidebar
13. You should see a restore job with status **Running**
14. Wait for it to show **Completed** (takes 2-5 minutes)

**Verify the restored resource:**

15. Go to **EC2** → **Volumes**
16. You should see a new volume — this is your restored backup!
17. It should have the same size and encryption as the original
18. You could attach this to an EC2 instance and access the data

> 💡 **Rithu's Tip**: "Notice we chose 'New resource' for the restore. This is the safe option — it creates a separate volume without touching the original. In an emergency, you could restore to the original location (destructive), but always prefer non-destructive restores!"

📸 [Screenshot: Restored EBS volume visible in EC2 Volumes console]

---

### Step 7: View Backup Reports (Optional)

AWS Backup provides compliance and audit reports!

1. Click **Audit and reporting** in the left sidebar
2. Click **Compliance reports**
3. You should see reports showing:
   - Which resources are compliant (backed up on schedule)
   - Which resources are non-compliant (missed a backup)
   - Backup job history

**What the reports show:**

- **Compliance**: Resources that are being backed up according to the plan
- **Job history**: All backup and restore jobs with timestamps
- **Resource coverage**: Which services are covered by backup plans

> 💡 **Rithu's Tip**: "Compliance reports are crucial for regulated industries. Imagine an auditor asking: 'Can you prove your database was backed up every day for the last year?' These reports answer that question instantly!"

📸 [Screenshot: AWS Backup compliance report showing resource backup status]

---

## ✅ Validation Checklist

Before moving on, confirm all of these:

- [ ] EC2 instance `ravi-backup-ec2` is running with 10GB EBS
- [ ] RDS instance `ravi-backup-db` is available
- [ ] S3 bucket `ravi-backup-bucket-12345` has files
- [ ] DynamoDB table `ravi-backup-table` has items
- [ ] Backup plan `ravi-backup-plan` exists with daily schedule
- [ ] Resources are assigned to the backup plan
- [ ] At least one backup job shows Completed
- [ ] On-demand backup completed
- [ ] Restored EBS volume exists in EC2

---

## 🧹 Cleanup (IMPORTANT!)

AWS Backup charges for stored backups. Delete everything!

**Delete backup plans and assignments:**

1. Go to **AWS Backup** → **Backup plans**
2. Select `ravi-backup-plan`
3. Click **Delete** (top right)
4. Type `ravi-backup-plan` to confirm
5. Click **Delete**
6. Confirm that all assignments are deleted with the plan

**Delete backups from the vault:**

7. Go to **AWS Backup** → **Backup vaults**
8. Click on **Default** vault
9. Select all backups in the vault
10. Click **Delete** (or Actions → Delete)
11. Confirm deletion
12. Wait for all backups to be deleted

**Delete the IAM role:**

13. Go to **IAM** → **Roles**
14. Search for `ravi-backup-role`
15. Click on it → Click **Delete**
16. Type the role name to confirm
17. Click **Delete**

**Terminate the EC2 instance:**

18. Go to **EC2** → **Instances**
19. Select `ravi-backup-ec2`
20. Click **Instance state** → **Terminate instance**
21. Confirm termination

**Delete the RDS instance:**

22. Go to **RDS** → **Databases**
23. Select `ravi-backup-db`
24. Click **Actions** → **Delete**
25. Type `delete me` to confirm
26. Uncheck **Create final snapshot** (to save time and money)
27. Check ✅ **Acknowledge**
28. Click **Delete**

**Empty and delete S3 bucket:**

29. Go to **S3** → **Buckets**
30. Click `ravi-backup-bucket-12345`
31. Click **Empty** → Type `permanently delete` → Click **Delete**
32. Go back to bucket list → Select bucket → **Delete** → Type name → **Delete bucket**

**Delete DynamoDB table:**

33. Go to **DynamoDB** → **Tables**
34. Select `ravi-backup-table`
35. Click **Delete** (top right)
36. Type the table name to confirm
37. Click **Delete table**

**Delete the restored EBS volume (if it still exists):**

38. Go to **EC2** → **Volumes**
39. If any volumes remain, select → **Actions** → **Delete volume**

**Verify everything is gone:**

40. Go to AWS Backup → confirm no backup plans, no backups
41. Go to EC2 → confirm no instances or volumes
42. Go to RDS → confirm no databases
43. Go to S3 → confirm no buckets
44. Go to DynamoDB → confirm no tables
45. Go to IAM → confirm no backup role

> 💡 **Rithu's Tip**: "AWS Backup cleanup is easy — delete the plan and all its backups go with it. Compare this to manually managing snapshots across EC2, RDS, and DynamoDB. That's the power of a centralized backup service!"

📸 [Screenshot: AWS Backup console showing no backup plans or backups remaining]

---

## 🎓 What You Learned

In this lab, you learned:

| Concept | What It Means |
|---------|---------------|
| **AWS Backup** | Centralized service for managing backups across AWS services |
| **Backup Plan** | A set of rules that define when and how to back up resources |
| **Backup Schedule** | Cron-based schedule for automatic backups |
| **Retention Policy** | How long backups are kept before automatic deletion |
| **Cold Storage** | Cheaper storage for long-term backup retention |
| **On-Demand Backup** | Manual backup triggered outside the regular schedule |
| **Restore** | Recovering a resource from a backup |
| **Non-Destructive Restore** | Restoring to a new resource without overwriting the original |
| **Compliance Reports** | Auditing which resources are backed up and on schedule |
| **Backup Vault** | Encrypted storage location for backups |

---

## 🔗 What's Next?

You now know how to protect your data with automated backups! In the final lab, we'll bring everything together in a **Capstone Project** — building a complete full-stack application on AWS!

**[Lab 25 — Capstone: Full Stack on AWS →](../25%20-%20Capstone%20-%20Full%20Stack%20on%20AWS/README.md)**

---

## ❓ Troubleshooting

**Problem**: No resources appear in "Protected resources"
- **Fix**: Wait 5-10 minutes after creating resources. AWS Backup needs time to discover them. Also verify the resources are in the same region as your backup plan.

**Problem**: Backup jobs stuck in "Pending" status
- **Fix**: Check that the IAM role has the correct permissions. The backup role needs access to the source resource and the backup vault. Wait 5-10 minutes — the first backup can take time to initialize.

**Problem**: Can't assign RDS to the backup plan
- **Fix**: Make sure the RDS instance is in **Available** state. RDS instances must be available to be backed up. Also check that the backup IAM role has `rds:CreateDBSnapshot` permission.

**Problem**: Restore fails or gets stuck
- **Fix**: Check that you have sufficient service quotas for EBS volumes. Also verify the destination AZ has capacity. Try restoring to a different AZ if one fails.

**Problem**: "Insufficient permissions" error when creating backup
- **Fix**: The IAM role created by AWS Backup might be missing permissions. Go to IAM → Roles → find the backup role → verify it has the AWS managed policy `AWSBackupServiceRolePolicyForBackup` attached.

**Problem**: On-demand backup not appearing in vault
- **Fix**: Wait 5-15 minutes for the backup to complete and be stored in the vault. Check the Backup jobs page to see if the job is still running or if it failed.

**Problem**: Can't delete the backup plan
- **Fix**: First delete all backup jobs and restore jobs associated with the plan. Then delete the plan. If jobs are stuck, they may need to expire (usually after 7 days).

**Problem**: RDS deletion fails
- **Fix**: Make sure no other resources are connected to the RDS instance. Check that you unchecked "Create final snapshot" if you want a quick deletion. The final snapshot deletion takes extra time.

---

> 🎉 **Rithu says**: "You've just completed one of the most important skills in cloud computing — protecting data with backups. In the real world, the engineer who set up proper backups is the hero of every disaster recovery story. You're now that engineer!"
