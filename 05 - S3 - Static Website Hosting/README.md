<div align="center">

<img src="https://img.shields.io/badge/Lab%2005-S3%20Static%20Website-2ECC71?style=for-the-badge&labelColor=232F3E" />

</div>

<div align="center">

# Lab 05 — S3: Static Website Hosting

<img src="https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=flat-square" />
<img src="https://img.shields.io/badge/Time-~25%20min-blue?style=flat-square" />
<img src="https://img.shields.io/badge/Cost-%3C%241-lightgrey?style=flat-square" />
<img src="https://img.shields.io/badge/Service-S3-yellow?style=flat-square" />

</div>

> *"S3 is like a magical empty filing cabinet in the sky. You can stuff files in it, and if you configure it right, the whole world can read them. Today, we're building a full static website inside that cabinet. No EC2. No Apache. Just S3."* — Rithu

---

<details>
<summary><b>🎭 Ravi & Rithu's Coffee Break Chat</b></summary>

**Ravi:** "Wait, I can host a WEBSITE on S3?!"

**Rithu:** "Yep! No server to manage, no OS to patch. Just pure static files served globally."

**Ravi:** "So S3 is basically a fancy USB stick that also hosts websites?"

**Rithu:** "That's... actually not a terrible analogy. I'm both proud and concerned."

</details>

---

## 🎯 Objective

Create an S3 bucket, configure it for static website hosting, upload HTML files, set a bucket policy for public read access, and verify the site loads in a browser. You'll ALSO test the custom error document by trying to reach a non-existent page.

## 🧠 Prerequisites

- Completion of **[Lab 04 — AMI: Create and Clone](../04%20-%20AMI%20-%20Create%20and%20Clone/README.md)**
- Basic HTML knowledge (none required really)
- AWS Console familiarity

## 💰 Cost Warning

- S3 offers 5 GB of standard storage FREE for the first 12 months.
- This lab uses maybe 0.01 GB. You're fine.
- **Public access buckets** mean anyone with the URL can access your objects.
- S3 bucket policies are FREE but misconfiguration might accidentally expose sensitive data. This lab is intentionally public; real workloads need stricter controls.

**Still. DELETE THE BUCKET when done. Orphan buckets (especially public ones) are how breaches happen.**

> **Ravi's Mistake of the Day:** I enabled public access on an S3 bucket containing test data that had real customer emails in it. S3 misconfigurations are the #1 cause of AWS data breaches. Don't be like me.

## 🏗️ Architecture

```
┌───────────────────────────────────┐
│           S3 Bucket               │
│  ravi-static-website-12345        │
│  ┌─────────────────────────────┐  │
│  │ Bucket Policy               │  │
│  │ { "Principal": "*",         │  │
│  │   "Action": "s3:GetObject", │  │
│  │   "Resource": "arn:aws:...  │  │
│  │  "*"}                       │  │
│  └─────────────────────────────┘  │
│                                   │
│  Files:                           │
│  index.html  (root)               │
│  error.html  (custom error)       │
│                                   │
│  Website Endpoint:                │
│  http://ravi-static-website       │
│     -12345.s3-website-us-east-    │
│     1.amazonaws.com               │
└───────────────────────────────────┘
         │
         ▼
   Internet Browser
   http://<endpoint>       → index.html loads
    http://<endpoint>/nope  → error.html loads
```

> **Did You Know?** S3 stands for "Simple Storage Service." It was launched in 2006 and stores over 100 TRILLION objects. That's about 13,000 objects for every human on Earth.

## 🛠️ Step-by-Step Instructions

> <img src="https://img.shields.io/badge/Step%201-Create%20an%20S3%20Bucket-2ECC71?style=for-the-badge" />

1. AWS Management Console → search for **S3** → click it.
2. Click the orange **Create bucket** button.

📸 [Screenshot: S3 Console with the Create bucket button highlighted]
![S3 Console with the Create bucket button highlighted](screenshots/create-bucket-button.png)

3. General configuration:

| Field | Value |
|-------|-------|
| Bucket name | `ravi-static-website-12345` |
| AWS Region | **US East (N. Virginia) us-east-1** |

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> The bucket name MUST be globally unique across ALL AWS accounts and ALL regions. Not just your account. ALL OF AWS. Pick something nobody else would think of. If `ravi-static-website-12345` is taken (unlikely), add more numbers: your birthday, favorite number, whatever.

4. **Object Ownership:**
   - Leave default: **ACLs disabled**.
   - Untick "ACLs enabled", leave "Object Ownership" as "Bucket Owner Enforced".

5. **Block Public Access settings for this bucket:**

   ⚠️ **This is the most important step.**

   - UNCHECK **Block all public access**.
   - A warning banner appears: "Turning off Block all public access is prohibited for new buckets in this account."
   - Read the warning. Acknowledge it by checking: **I acknowledge that the current settings may result in a bucket and objects becoming public.**

   📸 [Screenshot: Block all public access section with all boxes UNCHECKED and the acknowledgment checked]
  ![Block all public access section with all boxes UNCHECKED and the acknowledgment checked](screenshots/public-access-settings.png)


   > <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
   > Normally, you NEVER want a bucket public. We're making an exception because STATIC WEBSITES require PUBLIC access to deliver content to browsers. In real life, MOST buckets should stay private. Use CloudFront (covered much later) to serve S3 content securely.

6. **Bucket Versioning:** Leave **Disable**.

7. **Tags (optional):** Add one or two:

   | Key | Value |
   |-----|-------|
   | Name | `ravi-static-website` |
   | Project | `AWS-Hands-On-Labs` |

8. **Default encryption:** Leave at **Disable** (SSE-S3 active by default). AWS encrypts S3 objects by default now.

9. Click **Create bucket** at the bottom.

Congratulations, you own a cloud bucket! 🪣

> <img src="https://img.shields.io/badge/Step%202-Enable%20Static%20Website%20Hosting-3498DB?style=for-the-badge" />

1. Click on your bucket name `ravi-static-website-12345`.
2. Go to the **Properties** tab.
3. Scroll down to the **Static website hosting** card.
4. Click **Edit**.

📸 [Screenshot: Properties tab showing Static website hosting section]

5. Configure:

| Field | Value |
|-------|-------|
| Static website hosting | **Enable** |
| Hosting type | **Host a static website** |
| Index document | `index.html` |
| Error document | `error.html` |

6. Click **Save changes**.

Now look at the **Static website hosting** card again. You'll see a **Bucket website endpoint**:

```
http://ravi-static-website-12345.s3-website-us-east-1.amazonaws.com
```

📸 [Screenshot: Static website hosting card showing the endpoint URL]
![Static website hosting card showing the endpoint URL](screenshots/static-website-endpoint.png)

Copy that URL. You'll need it later.

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Notice it's `http://` (unencrypted). S3 static website hosting doesn't support HTTPS natively. For HTTPS, you'd use CloudFront as a front CDN. File that away, young padawan.

> <img src="https://img.shields.io/badge/Step%203-Prepare%20Your%20Website%20Files-E67E22?style=for-the-badge" />

Create two HTML files on your LOCAL machine.

**index.html:**

```html
<!DOCTYPE html>
<html>
<head>
<title>Ravi's Website</title>
<style>
body {
  font-family: Arial, sans-serif;
  text-align: center;
  padding: 50px;
  background: #f0f0f0;
}
h1 {
  color: #ff9900;
}
p {
  color: #333;
}
</style>
</head>
<body>
<h1>Welcome to Ravi's S3 Website!</h1>
<p>Hosted on Amazon S3 - Built by Ravi with guidance from Rithu</p>
<p>This website runs entirely on S3. No servers. No maintenance. Just magic.</p>
</body>
</html>
```

**error.html:**

```html
<!DOCTYPE html>
<html>
<head>
<title>Oops - Not Found</title>
<style>
body {
  font-family: Arial, sans-serif;
  text-align: center;
  padding: 50px;
  background: #ffe6e6;
}
h1 {
  color: #cc0000;
}
p {
  color: #555;
}
</style>
</head>
<body>
<h1>404 - Page Not Found</h1>
<p>Ravi says: This page doesn't exist!</p>
<p>Looks like you took a wrong turn somewhere. Try the <a href="index.html">homepage</a>.</p>
</body>
</html>
```

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> You can write these files in any text editor. Notepad (Windows) works. VS Code works. Heck, you can type them in a `README.md` and copy them out. Just make sure you save them as `.html` and not `.html.txt`.

> <img src="https://img.shields.io/badge/Step%204-Upload%20the%20Website%20Files-9B59B6?style=for-the-badge" />

1. S3 Console → bucket `ravi-static-website-12345`.
2. Go to the **Objects** tab.
3. Click **Upload**.

📸 [Screenshot: Upload page in S3 Objects tab]
!Upload page in S3 Objects tab](screenshots/upload-page.png)

4. Click **Add files**.
5. Select **both** `index.html` and `error.html`.
6. Click **Upload** (blue button at bottom).

Alternative: drag and drop files from your computer onto the Upload panel.

Once uploaded, you should see both files listed in the Objects tab.

📸 [Screenshot: S3 Objects tab showing index.html and error.html listed, size ~300 bytes each]
![ S3 Objects tab showing index.html and error.html listed, size ~300 bytes each](screenshots/uploaded-objects.png)

> <img src="https://img.shields.io/badge/Step%205-Add%20Bucket%20Policy%20for%20Public%20Read-E74C3C?style=for-the-badge" />

Even though the bucket is not fully blocked from public access, objects are STILL private by default. When a browser visits your website endpoint, S3 checks policy first.

1. S3 Console → your bucket **ravi-static-website-12345**.
2. Go to the **Permissions** tab.
3. Scroll to **Bucket Policy**.
4. Click **Edit**.
5. Paste the following:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::ravi-static-website-12345/*"
    }
  ]
}
```

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Let's decode this JSON:
> - `"Principal": "*"` → Anyone in the world
> - `"Action": "s3:GetObject"` → can read objects
> - `"Resource": "arn:aws:s3:::`ravi-static-website-12345`/*"` → from my bucket
> This gives anonymous internet users read access to EVERY object in the bucket. For a public static site, this is exactly what we want.

6. Click **Save changes**.

You might see an error if there are conflicts with account-level block public access settings. Since we disabled block all public access earlier, this should save cleanly.

📸 [Screenshot: Bucket policy editor showing the JSON pasted and saved]
![ Bucket policy editor showing the JSON pasted and saved](screenshots/bucket-policy-editor.png)

> <img src="https://img.shields.io/badge/Step%206-Verify%20Your%20Work-1ABC9C?style=for-the-badge" />

1. Open a fresh browser tab.
2. Paste the **Bucket website endpoint** URL you copied earlier.
3. Press Enter.

You should see your custom HTML page with:

> **Welcome to Ravi's S3 Website!**
>
> Hosted on Amazon S3 - Built by Ravi with guidance from Rithu
>
> This website runs entirely on S3. No servers. No maintenance. Just magic.

📸 [Screenshot: Browser showing the index.html page loaded from S3 website endpoint]
![ Browser showing the index.html page loaded from S3 website endpoint](screenshots/website-homepage.png)

**Now test the error page:**

1. Append `/nonexistent-page` to the URL in your browser:
   ```
   http://ravi-static-website-12345.s3-website-us-east-1.amazonaws.com/nonexistent-page
   ```
2. You should see:

> **404 - Page Not Found**
>
> Ravi says: This page doesn't exist!

📸 [Screenshot: Browser showing the custom error page at a non-existent URL]
![Browser showing the custom error page at a non-existent URL](screenshots/error-page.png)

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Without the error document configured, S3 would return a generic XML error page. Ugly. With `error.html` configured, S3 intercepts all 4xx errors and returns YOUR page. Your site, your branding.

### Alternative verification

In your terminal, you can also use `curl`:

```bash
curl http://ravi-static-website-12345.s3-website-us-east-1.amazonaws.com
```

## ✅ Validation Checklist

- [ ] S3 bucket named `ravi-static-website-12345` created in us-east-1
- [ ] Block all public access UNCHECKED and acknowledged
- [ ] Static website hosting ENABLED with index.html and error.html
- [ ] index.html uploaded to bucket
- [ ] error.html uploaded to bucket
- [ ] Bucket policy granting `s3:GetObject` to `Principal: "*"` applied
- [ ] index.html loads in browser at S3 website endpoint
- [ ] error.html loads when visiting a non-existent path
- [ ] Website endpoint URL works for anyone on the internet

> **POV:** Your S3 website is live and you tell everyone you're "basically a web developer now."

<div align="center">

> **Achievement Unlocked:** Web Host! Your first website is live on AWS!

</div>

## 🧹 Cleanup (IMPORTANT!)

Public buckets left running = potential bill and security risk.

1. **Delete objects:**
   - S3 Console → your bucket → **Objects** tab.
   - Select both `index.html` and `error.html`.
   - Click **Delete** → enter `permanently delete` in the confirmation → Click **Delete objects**.

2. **Delete the bucket:**
   - Go back to bucket listing (S3 Console → Buckets).
   - Select `ravi-static-website-12345`.
   - Click **Delete**.
   - Enter the bucket name in the confirmation field: `ravi-static-website-12345`.
   - Click **Delete bucket**.

   > <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
   > If the bucket contains objects that failed to delete previously, enable **Show versions** and delete ALL versions and ALL delete markers. Then try bucket deletion again. AWS won't let you delete a non-empty bucket.

## 🎓 What You Learned

| Concept | Takeaway |
|---------|----------|
| Object storage | S3 stores files as objects in buckets, not as block devices |
| Globally unique bucket names | No two accounts can have the same bucket name |
| Static website hosting | S3 serves index.html at root and error.html for 404 |
| Bucket policies | JSON statements that grant/deny access at the BUCKET level |
| Public access | Useful for websites. Dangerous for private data. Use CloudFront |
| S3 website endpoint | URL format: `http://<bucket-name>.s3-website-<region>.amazonaws.com` |
| Custom error documents | Provide user-friendly 4xx pages |

## 🔗 What's Next?

Excellent work hosting a static website! You've graduated from servers to serverless. The world, however, has MANY S3 superpowers.

Check out upcoming labs —
- **Lab 06** — S3 Versioning and Lifecycle Policies
- **Lab 07** — S3 Cross-Region Replication
- **Lab 08** — VPC: Build from Scratch

Or skip ahead. Whatever fuels your cloud engine. 🚀

<details>
<summary><strong>❓ Troubleshooting</strong></summary>

| Problem | Likely Cause | Fix |
|---------|-------------|------|
| Browser shows **403 Forbidden** | Bucket policy missing or objects still private | Paste the Bucket Policy JSON exactly; verify object permissions |
| Browser shows **404 Not Found** | index.html filename wrong, or URL path wrong | Upload file named `index.html` exactly. Check S3 listing |
| Error.html appears as raw HTML | Error page URL doesn't match error document name | error.html must use `.html` or whatever you configured |
| Bucket creation fails with "name already taken" | Someone else globally owns that bucket name | Add extra numbers/characters to bucket name |
| Bucket policy has warning yellow/red | Policy conflicts with account-level public access settings | Go to Permissions → Block Public Access → verify ALL checkboxes are UNCHECKED |
| `Unable to Update, the S3 web endpoint always returns "The specified bucket does not have a website configuration"` | Static website hosting NOT enabled | Properties → Static website hosting → Enable |
| Internet users still get Access Denied | Bucket Policy syntax error or resource ARN mismatch | Double-check `Resource` ends with `/*` (the wildcard covers all objects) |
| Objects show ACLs disabled, bucket policy is wrong place | ACLs not needed for static sites | Bucket Owner Enforced + Bucket Policy = the way |

</details>

---

<div align="center">

<img src="https://img.shields.io/badge/Lab%20Complete!-2ECC71?style=for-the-badge&labelColor=232F3E" />

*Written by Rithu, after accidentally hosting a bucket named "rithu-fun-bucket-ohno" publicly for 3 weeks. Ravi's bucket maybe safer. maybe.*

</div>
