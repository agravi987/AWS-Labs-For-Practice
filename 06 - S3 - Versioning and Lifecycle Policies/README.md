<div align="center">

<img src="https://img.shields.io/badge/Lab%2006-S3%20Versioning%20%26%20Lifecycle-3498DB?style=for-the-badge&labelColor=232F3E" />

</div>

<div align="center">

# Lab 06 — S3: Versioning and Lifecycle Policies

<img src="https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=flat-square" />
<img src="https://img.shields.io/badge/Time-~20min-blue?style=flat-square" />
<img src="https://img.shields.io/badge/Cost-%3C1%20USD-green?style=flat-square" />
<img src="https://img.shields.io/badge/Service-S3-orange?style=flat-square" />

</div>

> "Versioning is like having a time machine for your files. Accidentally deleted something? No worries — it's still there!" — Rithu 🚀

---

## 🎯 Objective

In this lab, you'll learn how S3 Versioning protects your data from accidental deletion and overwrites, and how Lifecycle Policies automatically move your objects between storage classes to save money. You'll upload files, create versions, delete and restore objects, and set up a lifecycle policy that automates storage management.

---

## 🧠 Prerequisites

- [x] Completed [Lab 05 — S3 Basics](../05%20-%20S3%20Basics/README.md)
- [x] AWS account with console access
- [x] Basic familiarity with the S3 console

---

## 💰 Cost Warning

> ⚠️ **This lab costs less than $1.** You're using S3 Standard storage with a tiny test file. However, always remember: S3 charges per GB per month. A lifecycle policy that moves data to Glacier is cheap, but leaving tons of data in Standard-IA when you don't need it adds up. **Always clean up after yourself!**

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────┐
│              S3 Bucket                          │
│         ravi-versioning-lab-12345                │
│                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │ index.html│  │ index.html│  │ index.html│     │
│  │  v1 (old) │  │  v2 (old) │  │  v3 (current)│ │
│  └──────────┘  └──────────┘  └──────────┘      │
│                                                 │
│  ┌─────────────────────────────────────┐        │
│  │       Lifecycle Policy              │        │
│  │  30 days → Standard-IA             │        │
│  │  7 days (noncurrent) → Glacier     │        │
│  │  90 days (noncurrent) → Delete     │        │
│  └─────────────────────────────────────┘        │
└─────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

> <img src="https://img.shields.io/badge/Step%201-Create%20an%20S3%20Bucket-2ECC71?style=for-the-badge" />

1. Log in to the [AWS Management Console](https://console.aws.amazon.com/)
2. In the search bar at the top, type **S3** and click on **S3** under Services
3. Click the orange **Create bucket** button
4. Under **Bucket name**, type: `ravi-versioning-lab-12345`
   - 📸 [Screenshot: Bucket name field filled in]
5. Leave **AWS Region** as **US East (N. Virginia) us-east-1**
6. Scroll down and leave all other settings as default
7. Click the orange **Create bucket** button at the bottom

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Bucket names must be globally unique across ALL of AWS. The `12345` suffix helps, but if it's taken, try a different number!

---

> <img src="https://img.shields.io/badge/Step%202-Enable%20Versioning%20on%20the%20Bucket-3498DB?style=for-the-badge" />

1. Click on your newly created bucket name: `ravi-versioning-lab-12345`
2. Click on the **Properties** tab (next to Objects, at the top)
3. Scroll down until you find **Bucket Versioning**
4. Click the **Edit** button
5. Select **Enable**
6. Click the orange **Save changes** button
7. You should see a green banner saying "Successfully edited bucket versioning"

> 📸 [Screenshot: Bucket Versioning showing "Enabled"]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Once you enable versioning, you can NEVER go back to " suspended" — you can only suspend it, which stops new versions but doesn't delete old ones. Think of it like a one-way valve!

---

> <img src="https://img.shields.io/badge/Step%203-Upload%20Version%201-E67E22?style=for-the-badge" />

1. Click on the **Objects** tab
2. Click the orange **Upload** button
3. Click **Add files**
4. Open a text editor on your computer (Notepad, VS Code, whatever you like) and create a file called `index.html` with this content:

```html
<h1>Hello from Version 1</h1>
```

5. Save it as `index.html` somewhere you can find it (like your Desktop)
6. Back in the S3 console, click **Add files** and select your `index.html` file
7. Scroll down and click the orange **Upload** button
8. You should see a green banner: "Upload succeeded"

> 📸 [Screenshot: Upload succeeded screen]

---

> <img src="https://img.shields.io/badge/Step%204-Upload%20Version%202-9B59B6?style=for-the-badge" />

1. Click on your bucket name to go back to the Objects tab
2. Click the orange **Upload** button
3. Open the SAME `index.html` file on your computer and change the content to:

```html
<h1>Hello from Version 2 — Updated!</h1>
```

4. Save the file
5. Click **Add files** and select the updated `index.html`
6. Click the orange **Upload** button
7. You should see "Upload succeeded" again

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Notice that S3 didn't complain about "overwriting" the file? That's because versioning is enabled — it creates a NEW version instead of replacing the old one!

---

> <img src="https://img.shields.io/badge/Step%205-Upload%20Version%203-E74C3C?style=for-the-badge" />

1. Go back to the Objects tab
2. Click **Upload**
3. Change `index.html` content to:

```html
<h1>Hello from Version 3 — Final Version!</h1>
```

4. Save, upload, and wait for success

---

> <img src="https://img.shields.io/badge/Step%206-View%20All%20Versions-1ABC9C?style=for-the-badge" />

1. On the Objects tab, you should see `index.html` listed — but it looks like there's only ONE file, right? 😏
2. Look above the file list — you'll see a toggle or link that says **List versions** or **Show versions**
3. Click on it!
4. Now you'll see THREE versions of `index.html` — each with a different **Version ID**
5. Notice which one has the checkmark icon — that's the **current** version (v3)

> 📸 [Screenshot: All 3 versions of index.html displayed with Version IDs]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> This is the magic of versioning! S3 kept every single version. If you ever accidentally upload the wrong file, you can roll back to any previous version. It's like Git, but for your files!

---

> <img src="https://img.shields.io/badge/Step%207-Delete%20the%20File%20(Soft%20Delete!)-F39C12?style=for-the-badge" />

1. Make sure **List versions** is still turned ON
2. Select `index.html` (the current version — the one with the checkmark)
3. Click the **Delete** button at the top
4. In the confirmation box, you'll see:
   - "Permanently delete" vs "Delete"
   - Make sure **Delete** is selected (NOT "Permanently delete")
5. Type `delete` in the confirmation field
6. Click the orange **Delete** button

7. Now look at your file list — wait, `index.html` is STILL there! 😮
8. But look closely — there's now a **Delete marker** at the top (with a grey "x" icon)
9. The three versions are still listed below it

> 📸 [Screenshot: Delete marker shown with all 3 versions still present]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> When you "delete" a versioned object, S3 doesn't actually delete anything. It just adds a **Delete Marker** — think of it like putting a sticky note that says "pretend this file doesn't exist." The actual data is still safe underneath!

---

> <img src="https://img.shields.io/badge/Step%208-Restore%20a%20Previous%20Version-34495E?style=for-the-badge" />

Now let's pretend we panicked and want our file back!

1. You should still have **List versions** enabled
2. Find the version you want to restore (let's pick v1 — "Hello from Version 1")
3. Click the checkbox next to that specific version
4. Click **Actions** → **Restore**
   - OR: Click the **Download** button next to that version to download it
   - OR: Click on the **Version ID** link to see the object details, then use **Open** or **Download**
5. To make v1 the current version:
   - Download v1, then re-upload it (S3 will create a NEW current version from it)

> 📸 [Screenshot: Downloading/restoring a previous version]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Versioning saved the day! Ravi, imagine you're a developer and you accidentally deployed broken code to production. With versioning, you just restore the last known good version. No stress!

---

> <img src="https://img.shields.io/badge/Step%209-Remove%20the%20Delete%20Marker-16A085?style=for-the-badge" />

Let's clean up the delete marker so our file is "visible" again:

1. Make sure **List versions** is ON
2. Find the **Delete marker** (it has a grey "x" icon)
3. Select it
4. Click **Delete**
5. Type `delete` to confirm
6. Click **Delete**

Now the most recent version of `index.html` should be "current" again (visible without listing versions).

---

> <img src="https://img.shields.io/badge/Step%2010-Set%20Up%20a%20Lifecycle%20Policy-8E44AD?style=for-the-badge" />

Now for the really cool part — let S3 manage your storage automatically!

1. Click on the **Properties** tab of your bucket
2. Scroll down to find **Lifecycle rules** (under "Bucket polices and rules")
3. Click **Create lifecycle rule**

#### Configure Rule 1: Move Current Versions to Standard-IA

1. **Rule name:** `move-to-standard-ia`
2. Under **Choose a rule scope**, select **Apply to all objects in the bucket**
3. Click **Next**
4. Under **Current version**, check the box for **Move current versions of objects between storage classes**
5. In the dropdown, select **Standard-IA**
6. In the **Days after creation** field, type: `30`
7. Click **Add transition** (if needed)

#### Configure Rule 2: Move Noncurrent Versions to Glacier

1. Still on the same page, scroll to **Noncurrent versions**
2. Check **Move noncurrent versions of objects between storage classes**
3. Select **Glacier Flexible Retrieval**
4. **Days after objects become noncurrent:** `7`

#### Configure Rule 3: Permanently Delete Noncurrent Versions

1. Still under **Noncurrent versions**, check **Permanently delete noncurrent versions of objects**
2. **Days after objects become noncurrent:** `90`

5. Click **Next**
6. Review your rules — you should see:
   - Current → Standard-IA after 30 days
   - Noncurrent → Glacier after 7 days
   - Noncurrent → Delete after 90 days
7. Click **Create rule**

> 📸 [Screenshot: Lifecycle rule configuration showing all three transitions]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Why does this save money? Because different storage classes have different costs:
> - **S3 Standard** — $0.023/GB/month (most accessible, most expensive)
> - **S3 Standard-IA** — $0.0125/GB/month (half the price, but you pay to retrieve)
> - **S3 Glacier** — $0.004/GB/month (cheap, but retrieval takes hours)
>
> If you have old files you rarely access, letting S3 automatically move them to Glacier saves you 80%+!

---

> <img src="https://img.shields.io/badge/Step%2011-Verify%20Your%20Work-2C3E50?style=for-the-badge" />

Let's make sure everything is set up correctly:

- [ ] Bucket `ravi-versioning-lab-12345` exists in us-east-1
- [ ] Versioning is **Enabled** on the bucket
- [ ] You can see at least 1 version of `index.html` in the bucket
- [ ] The lifecycle rule `move-to-standard-ia` exists and is enabled
- [ ] The lifecycle rule shows transitions for both current and noncurrent versions

> 📸 [Screenshot: Properties tab showing Versioning Enabled and Lifecycle rules listed]

---

## ✅ Validation Checklist

| # | Check | Status |
|---|-------|--------|
| 1 | Bucket `ravi-versioning-lab-12345` created | ☐ |
| 2 | Versioning enabled on bucket | ☐ |
| 3 | `index.html` uploaded with 3 different versions | ☐ |
| 4 | All 3 versions visible when listing versions | ☐ |
| 5 | Delete marker created and file "soft deleted" | ☐ |
| 6 | Previous version restored successfully | ☐ |
| 7 | Lifecycle rule created with correct transitions | ☐ |
| 8 | Lifecycle rule applies to all objects in bucket | ☐ |

---

## 🧹 Cleanup (IMPORTANT!)

> ⚠️ **Don't skip this!** Even though the cost is tiny, it's good practice to always clean up. Let's build that habit now!

### Step 1: Empty the Bucket (Delete All Versions)

1. Go to the **Objects** tab of your bucket
2. Click **List versions** to show all versions
3. Select ALL objects (including delete markers) by checking the box next to **Name**
4. Click **Delete**
5. In the confirmation box, type `permanently delete`
6. Click the orange **Permanently delete** button

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> With versioning enabled, you can't just "delete" objects — you have to delete ALL versions. S3 makes you type `permanently delete` to make sure you really mean it. Double-check!

### Step 2: Delete the Bucket

1. Go back to the S3 bucket list (click **Breadcrumbs** → S3)
2. Select your bucket `ravi-versioning-lab-12345`
3. Click **Delete**
4. Type `ravi-versioning-lab-12345` in the confirmation field
5. Click **Delete bucket**

### Step 3: Delete the Lifecycle Rule (If Bucket Still Exists)

If you want to delete the lifecycle rule before emptying the bucket:
1. Go to **Properties** → **Lifecycle rules**
2. Select the rule
3. Click **Delete**

> 📸 [Screenshot: Empty S3 console — no buckets remaining]

---

## 🎓 What You Learned

- **S3 Versioning** keeps every version of a file, protecting against accidental deletion and overwrites
- **Delete Markers** are "soft deletes" — the data is still there, just hidden
- **Restoring previous versions** is easy — just re-upload or download an old version
- **Lifecycle Policies** automate storage class transitions, saving you money over time
- The difference between **S3 Standard**, **Standard-IA**, and **Glacier** storage classes
- How to properly clean up versioned S3 buckets (you must delete ALL versions!)

---

## 🔗 What's Next?

Now that you know how to protect your data within a single region, let's learn how to replicate it across regions for disaster recovery!

➡️ **[Lab 07 — S3: Cross-Region Replication](../07%20-%20S3%20-%20Cross-Region%20Replication/README.md)**

---

<details>
<summary><strong>❓ Troubleshooting</strong></summary>

| Problem | Solution |
|---------|----------|
| Can't create bucket with name `ravi-versioning-lab-12345` | The name is taken! Change the number suffix (e.g., `ravi-versioning-lab-67890`) |
| Versioning toggle is greyed out | Make sure you're on the **Properties** tab, not Objects |
| Can't see "List versions" toggle | Look carefully above the file list — it's a small button/link |
| Lifecycle rule not saving | Make sure you filled in all required fields (days, storage class) |
| Upload keeps failing | Check file size — it should be tiny. Also check your internet connection |
| Can't delete bucket | Make sure the bucket is EMPTY first — including all versions. Turn on "List versions" and delete everything |
| "Access Denied" when creating lifecycle rule | You need S3 full permissions. Check your IAM user/role has `s3:PutLifecycleConfiguration` |

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Stuck? Take a screenshot and check the AWS console error message carefully. 90% of S3 errors are about permissions or the bucket not being empty. You've got this, Ravi! 💪

</details>

---

<div align="center">

<img src="https://img.shields.io/badge/Lab%20Complete!-3498DB?style=for-the-badge&labelColor=232F3E" />

</div>
