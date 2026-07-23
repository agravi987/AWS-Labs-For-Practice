<div align="center">

<img src="https://img.shields.io/badge/Lab%2007-S3%20Cross-Region%20Replication-9B59B6?style=for-the-badge&labelColor=232F3E" />

</div>

<div align="center">

# Lab 07 — S3: Cross-Region Replication

<img src="https://img.shields.io/badge/Difficulty-Medium-yellow?style=flat-square" />
<img src="https://img.shields.io/badge/Time-~40min-blue?style=flat-square" />
<img src="https://img.shields.io/badge/Cost-%3C2%20USD-green?style=flat-square" />
<img src="https://img.shields.io/badge/Service-S3%20|%20IAM-orange?style=flat-square" />

</div>

> "Disasters don't send calendar invites. Cross-region replication means your data is safe even if an entire region goes down!" — Rithu 🌍

---

## 🎯 Objective

In this lab, you'll set up **Cross-Region Replication (CRR)** — a feature that automatically copies every object you upload in one S3 bucket to a bucket in a completely different AWS region. This is essential for disaster recovery, compliance, and reducing latency for users in different geographic locations.

---

## 🧠 Prerequisites

- [x] Completed [Lab 06 — S3 Versioning and Lifecycle Policies](../06%20-%20S3%20-%20Versioning%20and%20Lifecycle%20Policies/README.md)
- [x] AWS account with console access
- [x] Basic understanding of IAM roles (don't worry — we'll walk through it!)

---

## 💰 Cost Warning

> ⚠️ **CRR has storage costs in BOTH regions!** You're paying for the same data in two places. For our tiny test file, this is basically free. But in production, always calculate the cost of replicating terabytes of data across regions. **Delete both buckets and the IAM role when you're done!**

---

## 🏗️ Architecture

```
┌──────────────────────┐          ┌──────────────────────┐
│   SOURCE BUCKET      │          │  DESTINATION BUCKET   │
│  us-east-1 (N.VA)    │ ──────▶  │  us-west-2 (OREGON)  │
│                      │  CRR     │                      │
│  ravi-source-        │  Rule    │  ravi-dest-           │
│  replication-12345   │          │  replication-12345    │
│                      │          │                      │
│  [Versioning: ON]    │          │  [Versioning: ON]     │
└──────────────────────┘          └──────────────────────┘
         ▲
         │
    ┌────┴────┐
    │ IAM Role│
    │s3-rep-  │
    │lication- │
    │role     │
    └─────────┘
```

---

## 🛠️ Step-by-Step Instructions

> <img src="https://img.shields.io/badge/Step%201-Create%20the%20Source%20Bucket-2ECC71?style=for-the-badge" />

1. Log in to the [AWS Management Console](https://console.aws.amazon.com/)
2. In the search bar, type **S3** and click on **S3**
3. Click **Create bucket**
4. Configure:
   - **Bucket name:** `ravi-source-replication-12345`
   - **AWS Region:** `US East (N. Virginia) us-east-1`
5. Leave all other settings as default
6. Click **Create bucket**

> 📸 [Screenshot: Source bucket created successfully]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> us-east-1 is the default region for most AWS services and often the cheapest. It's our "home base" for this lab!

---

> <img src="https://img.shields.io/badge/Step%202-Enable%20Versioning%20on%20Source-3498DB?style=for-the-badge" />

1. Click on `ravi-source-replication-12345`
2. Click the **Properties** tab
3. Find **Bucket Versioning** → click **Edit**
4. Select **Enable**
5. Click **Save changes**

> ⚠️ **IMPORTANT:** Versioning MUST be enabled on BOTH buckets for replication to work. Don't skip this!

---

> <img src="https://img.shields.io/badge/Step%203-Create%20the%20Destination%20Bucket-E67E22?style=for-the-badge" />

1. Go back to the S3 console (click **S3** in the breadcrumbs)
2. Click **Create bucket**
3. Configure:
   - **Bucket name:** `ravi-dest-replication-12345`
   - **AWS Region:** `US West (Oregon) us-west-2`
     - 📸 [Screenshot: Region dropdown showing Oregon selected]
4. Leave all other settings as default
5. Click **Create bucket**

---

> <img src="https://img.shields.io/badge/Step%204-Enable%20Versioning%20on%20Destination-9B59B6?style=for-the-badge" />

1. Click on `ravi-dest-replication-12345`
2. Click the **Properties** tab
3. Find **Bucket Versioning** → click **Edit**
4. Select **Enable**
5. Click **Save changes**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Both buckets now have versioning enabled. This is a hard requirement for CRR — AWS won't let you replicate between non-versioned buckets. Why? Because replication needs to track which version of each object to copy!

---

> <img src="https://img.shields.io/badge/Step%205-Create%20an%20IAM%20Role-E74C3C?style=for-the-badge" />

Now we need to give S3 permission to read from one bucket and write to another. We do this with an IAM Role.

1. In the search bar at the top, type **IAM** and click on **IAM**
2. In the left sidebar, click **Roles**
3. Click the orange **Create role** button
4. Under **Trusted entity type**, select **AWS Service**
5. Under **Use case**, find and select **S3** (or type `S3` in the search box)
6. Click **Next**

> 📸 [Screenshot: Trusted entity selection showing S3]

7. On the "Add permissions" page, search for `AmazonS3FullAccess`
8. Check the box next to **AmazonS3FullAccess**
   - ⚠️ **Note:** In a real production environment, you would create a custom policy with ONLY the specific permissions needed. For this lab, we're using the managed policy for simplicity.
9. Click **Next**

> 📸 [Screenshot: AmazonS3FullAccess policy selected]

10. For **Role name**, type: `s3-replication-role`
11. Scroll down and click **Create role**

> 📸 [Screenshot: Role created successfully]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> IAM Roles are like giving someone a key card to your building. The role says "S3 service, you have permission to read from the source bucket and write to the destination bucket." Never give more permissions than needed in production!

---

> <img src="https://img.shields.io/badge/Step%206-Set%20Up%20Replication%20Rule-1ABC9C?style=for-the-badge" />

1. Go back to **S3** console
2. Click on `ravi-source-replication-12345`
3. Click the **Management** tab
4. Scroll down to **Replication rules**
5. Click **Create replication rule**

#### Configure the Rule:

1. **Replication rule name:** `replicate-to-west`
2. Under **Scope**, select **Entire bucket** (replicate all objects)
3. Click **Next**

> 📸 [Screenshot: Rule name and scope configuration]

4. **Destination:** Select **Choose a bucket in this account**
5. Browse and select `ravi-dest-replication-12345`
   - 📸 [Screenshot: Destination bucket selected]

6. Click **Next**

7. **IAM Role:** Select `s3-replication-role` from the dropdown
   - 📸 [Screenshot: IAM role selected]

8. **Replication Time Control (RTC):** Leave unchecked (this is optional and adds cost)
9. Click **Next**

10. Review all settings:
    - Source: `ravi-source-replication-12345`
    - Destination: `ravi-dest-replication-12345`
    - Rule name: `replicate-to-west`
    - IAM Role: `s3-replication-role`
11. Click **Save**

> 📸 [Screenshot: Replication rule saved successfully]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Replication doesn't happen instantly — it can take a few minutes to initialize. After that, new objects are typically replicated within 15 minutes. Be patient, Ravi!

---

> <img src="https://img.shields.io/badge/Step%207-Upload%20a%20Test%20File-34495E?style=for-the-badge" />

1. Click on `ravi-source-replication-12345` → **Objects** tab
2. Click **Upload**
3. Create a simple text file on your computer called `hello.txt` with the content:

```
Hello from the source bucket in N. Virginia!
```

4. Upload `hello.txt` to the source bucket
5. Wait for the upload to succeed

> 📸 [Screenshot: hello.txt uploaded to source bucket]

---

> <img src="https://img.shields.io/badge/Step%208-Wait%20and%20Verify%20Replication-F39C12?style=for-the-badge" />

1. Wait **2-5 minutes** (grab a coffee ☕ — replication takes a moment to kick in)
2. Go to the S3 console and click on `ravi-dest-replication-12345`
3. Click on the **Objects** tab
4. If all went well, you should see `hello.txt` in the destination bucket! 🎉
5. Click on `hello.txt` and check the details — it should show the same content

> 📸 [Screenshot: hello.txt appearing in the destination bucket in Oregon]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> If you don't see the file immediately, wait a few more minutes. The first replication can take up to 15 minutes. Check the **Management** tab → **Replication rules** on the source bucket to see the rule status.

---

> <img src="https://img.shields.io/badge/Step%209-Upload%20More%20Files-16A085?style=for-the-badge" />

Let's test with a few more files:

1. Go back to `ravi-source-replication-12345`
2. Create and upload these files:

**file2.txt:**
```
Second file replicated automatically!
```

**file3.html:**
```html
<h1>Replicated HTML File</h1>
<p>This file was automatically copied to us-west-2</p>
```

3. Wait 2-5 minutes
4. Check `ravi-dest-replication-12345` — both files should appear!

> 📸 [Screenshot: All files replicated to destination bucket]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> From now on, EVERY file you upload to the source bucket will be automatically copied to the destination. You don't need to do anything — S3 handles it all behind the scenes!

---

> <img src="https://img.shields.io/badge/Step%2010-Test%20Deletion%20Behavior-E74C3C?style=for-the-badge" />

Here's something interesting: **deleting from the source does NOT delete from the destination.**

1. Go to `ravi-source-replication-12345`
2. Select `hello.txt`
3. Click **Delete** → confirm deletion
4. Go to `ravi-dest-replication-12345`
5. Check — `hello.txt` is still there! 😮

> 📸 [Screenshot: File deleted from source but still present in destination]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> This is a safety feature! If someone accidentally deletes files from the source, the copies in the destination are safe. In production, you can configure delete markers to replicate if you want synced deletions.

---

> <img src="https://img.shields.io/badge/Step%2011-Verify%20Your%20Work-2C3E50?style=for-the-badge" />

- [ ] Source bucket `ravi-source-replication-12345` exists in us-east-1 with Versioning enabled
- [ ] Destination bucket `ravi-dest-replication-12345` exists in us-west-2 with Versioning enabled
- [ ] IAM Role `s3-replication-role` exists
- [ ] Replication rule `replicate-to-west` is active on the source bucket
- [ ] Files uploaded to source appear in destination after a few minutes
- [ ] Deleting files from source does NOT delete them from destination

---

## ✅ Validation Checklist

| # | Check | Status |
|---|-------|--------|
| 1 | Source bucket in us-east-1 with versioning | ☐ |
| 2 | Destination bucket in us-west-2 with versioning | ☐ |
| 3 | IAM Role `s3-replication-role` created | ☐ |
| 4 | Replication rule `replicate-to-west` active | ☐ |
| 5 | Files replicate from source to destination | ☐ |
| 6 | Deletion from source doesn't affect destination | ☐ |

---

## 🧹 Cleanup (IMPORTANT!)

> ⚠️ **This is critical!** You're paying for storage in TWO regions. Delete everything!

### Step 1: Delete All Objects from BOTH Buckets

**Source bucket:**
1. Go to `ravi-source-replication-12345` → **Objects** tab
2. Select all files → **Delete** → type `permanently delete` → confirm
3. If versioning is on, turn on "List versions" and delete ALL versions

**Destination bucket:**
4. Go to `ravi-dest-replication-12345` → **Objects** tab
5. Select all files → **Delete** → type `permanently delete` → confirm
6. Delete ALL versions if versioning is on

### Step 2: Delete the Replication Rule

1. Go to `ravi-source-replication-12345` → **Management** tab
2. Under **Replication rules**, select `replicate-to-west`
3. Click **Delete** → confirm

### Step 3: Delete Both Buckets

1. Go to S3 bucket list
2. Select `ravi-source-replication-12345` → **Delete** → type the bucket name → confirm
3. Select `ravi-dest-replication-12345` → **Delete** → type the bucket name → confirm

### Step 4: Delete the IAM Role

1. Go to **IAM** console → **Roles**
2. Search for `s3-replication-role`
3. Click on it → **Delete** → type `s3-replication-role` → confirm deletion

> 📸 [Screenshot: Clean S3 console and IAM Roles page — nothing left behind]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Always clean up in reverse order: objects → rules → buckets → IAM roles. If you delete the bucket first, the replication rule becomes orphaned and may cause confusing error messages!

---

## 🎓 What You Learned

- **Cross-Region Replication (CRR)** automatically copies objects from one bucket to another in a different region
- **Versioning is required** on both source and destination buckets for CRR to work
- **IAM Roles** provide the permissions S3 needs to read and write across buckets
- **Deletions don't replicate by default** — your destination acts as a backup
- **Replication isn't instant** — it can take a few minutes, especially for the first objects
- CRR is essential for **disaster recovery**, **compliance**, and **data locality**

---

## 🔗 What's Next?

Now that you understand S3 storage and data protection, let's dive into networking! You'll build a Virtual Private Cloud (VPC) from scratch — the foundation of every AWS architecture.

➡️ **[Lab 08 — VPC: Build from Scratch](../08%20-%20VPC%20-%20Build%20from%20Scratch/README.md)**

---

<details>
<summary><strong>❓ Troubleshooting</strong></summary>

| Problem | Solution |
|---------|----------|
| "Replication rule failed" | Make sure Versioning is enabled on BOTH buckets. This is the #1 cause of failure! |
| Files aren't replicating | Wait 5-15 minutes. First-time replication can be slow. Check the rule status in the Management tab |
| "Access Denied" when creating replication role | Make sure you selected `AmazonS3FullAccess` policy and the trusted entity is S3 |
| Can't select destination bucket | Make sure the destination bucket has Versioning enabled. AWS won't show non-versioned buckets as valid destinations |
| Destination bucket not listed | Ensure you're looking in the right region — the destination is in us-west-2 |
| Replication rule shows "Pending" | This is normal! It takes a few minutes to initialize. Give it 5-10 minutes |
| Can't delete bucket | Empty ALL versions first. Turn on "List versions" and delete everything, including delete markers |

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> The most common mistake is forgetting to enable versioning on one of the buckets. If something seems stuck, that's the first thing to check. You've got this! 🎯

</details>

---

<div align="center">

<img src="https://img.shields.io/badge/Lab%20Complete!-9B59B6?style=for-the-badge&labelColor=232F3E" />

</div>
