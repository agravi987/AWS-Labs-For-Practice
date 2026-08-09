<div align="center">

<img src="https://img.shields.io/badge/Lab%2018-Lambda%20S3%20Trigger-F39C12?style=for-the-badge&labelColor=232F3E" />

# Lab 18 — Lambda: S3 Triggered Function

<img src="https://img.shields.io/badge/Difficulty-Medium-orange?style=flat-square" />
<img src="https://img.shields.io/badge/Time-~35min-blue?style=flat-square" />
<img src="https://img.shields.io/badge/Cost-<_%241-yellow?style=flat-square" />
<img src="https://img.shields.io/badge/Service-Lambda%20%2F%20S3-purple?style=flat-square" />

</div>

> "Lambda is like a vending machine — put something in, get something out, and you don't have to worry about what's inside the machine." — Rithu

---

<details>
<summary><b>🎭 Ravi & Rithu's Coffee Break Chat</b></summary>

**Ravi:** "Lambda runs code without a server?"

**Rithu:** "Correct! It's like ordering takeout - you get the food without owning a kitchen."

**Ravi:** "What if my code crashes?"

**Rithu:** "Lambda just retries. No servers were harmed in the making of your bug."

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

> **What is this, really?** Lambda is **code that runs on its own when events happen** — no servers to manage, nothing to keep running. Upload a file to S3 → Lambda fires automatically, processes the file, and goes back to sleep. It's like a fire alarm that calls the fire department *by itself*. 🚒
>
> 🌍 **Why you should care:** Event-driven serverless is how modern apps process images, resize videos, validate uploads, and trigger pipelines — paying only for the milliseconds the code actually runs.

---

## 🎯 Objective

In this lab, you'll write a **Python Lambda function** that automatically runs every time a file is uploaded to an S3 bucket. This is one of the most common serverless patterns in AWS — event-driven processing at scale!

**By the end of this lab you will be able to:**
- Create and configure an S3 bucket as an event source for Lambda
- Write a Lambda function in Python that processes S3 events
- Read and write CloudWatch Logs for debugging
- Understand IAM roles for Lambda execution
- Implement event-driven, serverless architectures

---

## 🧠 Prerequisites

- [ ] Completed [Lab 17 — SNS and SQS: Messaging](../17%20-%20SNS%20and%20SQS%20-%20Messaging/README.md)
- [ ] AWS account with root or admin access
- [ ] A `.jpg` or `.png` image file to upload (any image, even a small one)
- [ ] Basic familiarity with Python (don't worry — we'll provide the code!)

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> You don't need to be a Python expert for this lab. The code is provided — your job is to understand what it does and how it connects to S3.

---

## 💰 Cost Warning

**This lab costs less than $1!** 💸

AWS Lambda Free Tier (always free):
- **1 million requests** per month
- **400,000 GB-seconds** of compute time per month

S3 Free Tier (12 months):
- **5 GB** of storage
- **20,000 GET** requests and **2,000 PUT** requests per month

⚠️ **IMPORTANT:** Delete the Lambda function, S3 bucket, and IAM role after the lab. While Lambda itself is free within limits, abandoned resources can accumulate unexpected charges over time.

> **Ravi's Mistake of the Day:** I wrote a Lambda function that made an HTTP call to an external API, but forgot to handle timeouts. The function ran for 15 minutes (max timeout), burning memory the whole time.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────┐
│                                                  │
│   User uploads file                              │
│        │                                         │
│        ▼                                         │
│   ┌──────────────┐     Event      ┌───────────┐ │
│   │   S3 Bucket   │ ──────────→   │   Lambda   │ │
│   │ravi-lambda-   │               │  Function  │ │
│   │trigger-bucket │               │s3-image-   │ │
│   └──────────────┘               │processor  │ │
│                                   └─────┬─────┘ │
│                                         │       │
│                                         ▼       │
│                                   ┌───────────┐ │
│                                   │ CloudWatch │ │
│                                   │   Logs     │ │
│                                   └───────────┘ │
└─────────────────────────────────────────────────┘
```

> **Did You Know?** Lambda functions can run code in response to over 200 AWS service events. S3 uploads, DynamoDB changes, API calls, IoT sensor data - Lambda reacts to all of them.

---

## 🛠️ Step-by-Step Instructions

> <img src="https://img.shields.io/badge/Step%201-Create%20S3%20Bucket-2ECC71?style=for-the-badge" />

1. Sign in to the **AWS Management Console**.
2. In the search bar, type **S3** and click on **S3**.
3. Click **Create bucket**.
4. **Bucket name:** `ravi-lambda-trigger-bucket-12345`
   - ⚠️ S3 bucket names are **globally unique** — no two buckets in the world can have the same name. Replace `12345` with random numbers to ensure uniqueness (e.g., `ravi-lambda-trigger-bucket-88421`).
5. **Region:** `us-east-1` (N. Virginia)
6. **Block Public Access settings:** Keep all options checked ✅ (public access blocked)
7. Leave other settings as defaults.
8. Click **Create bucket**.

> 📸 [Screenshot: The S3 bucket creation form with ravi-lambda-trigger-bucket-12345 entered]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Bucket names can only contain lowercase letters, numbers, hyphens, and periods. No spaces, no underscores, no capital letters. Think of it like a DNS name — it has to be globally unique!

---

> <img src="https://img.shields.io/badge/Step%202-Create%20Lambda%20Function-3498DB?style=for-the-badge" />

1. In the search bar, type **Lambda** and click on **AWS Lambda**.
2. Click the orange **Create function** button.
3. **Author from scratch** should be selected (it's the default).
4. **Function name:** `s3-image-processor`
5. **Runtime:** Select **Python 3.12** from the dropdown.
6. **Architecture:** `x86_64`
7. **Execution role:**
   - Select: ⚫ **Create a new role with basic Lambda permissions**
   - **Role name:** `lambda-s3-role`
8. Click **Create function**.

> 📸 [Screenshot: The Lambda function creation form showing Python 3.12 and the new role option]

After creation, you'll be taken to the function configuration page. You should see a success message.

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "Serverless" doesn't mean there's no server — it means YOU don't manage the server. AWS handles everything: the OS, the patches, the scaling, the availability. You just write code and deploy it. It's like getting a taxi instead of owning a car — someone else handles the maintenance! 🚕

---

> <img src="https://img.shields.io/badge/Step%203-Write%20Lambda%20Code-E67E22?style=for-the-badge" />

1. In the Lambda function page, scroll down to the **Code source** section.
2. You'll see a default file `lambda_function.py` with some example code.
3. **Delete all the existing code** and **replace it** with the following:

```python
import json
import urllib.parse
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = urllib.parse.unquote_plus(record['s3']['object']['key'])
        size = record['s3']['object']['size']

        print(f"New file uploaded!")
        print(f"Bucket: {bucket}")
        print(f"Key: {key}")
        print(f"Size: {size} bytes")

        # Get file metadata
        response = s3.head_object(Bucket=bucket, Key=key)
        content_type = response.get('ContentType', 'unknown')
        print(f"Content Type: {content_type}")

    return {
        'statusCode': 200,
        'body': json.dumps(f'Processed {len(event["Records"])} file(s)')
    }
```

4. Click the **Deploy** button (orange button above the code editor).

> 📸 [Screenshot: The Lambda code editor showing the Python code with the Deploy button highlighted]

**What does this code do?**

| Line | Purpose |
|------|---------|
| `import json, urllib.parse, boto3` | Import AWS SDK and utilities |
| `s3 = boto3.client('s3')` | Create an S3 client to interact with S3 |
| `for record in event['Records']` | Loop through each S3 event (one per file) |
| `bucket = record['s3']['bucket']['name']` | Get the bucket name |
| `key = record['s3']['object']['key']` | Get the file name (object key) |
| `size = record['s3']['object']['size']` | Get the file size |
| `s3.head_object(...)` | Get metadata (content type, etc.) |
| `print(...)` | Write to CloudWatch Logs for debugging |
| `return {...}` | Return a response (for Lambda logs) |

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> The `event` parameter is the star of the show here. When S3 triggers Lambda, it passes a JSON payload describing what happened — which bucket, which file, what action, what time, etc. This is the "event-driven" part of serverless!

---

> <img src="https://img.shields.io/badge/Step%204-Add%20S3%20Trigger-27AE60?style=for-the-badge" />

1. In your Lambda function page, scroll to the **Function overview** section at the top.
2. Click **+ Add trigger** (under the "Function overview" diagram).
3. **Source:** Select **S3** from the dropdown.
4. **Bucket:** Select `ravi-lambda-trigger-bucket-12345` (the bucket you created in Step 1).
5. **Event type:** ⚫ All object create events
   - This means the Lambda fires whenever a file is created, overwritten, or copied into the bucket.
6. **Suffix (optional filter):** `.jpg`
   - This ensures the Lambda only runs for `.jpg` files, not every file type.
7. You'll see a warning box about **Recursive invocation**:
   - ⚠️ "This S3 bucket is configured to invoke this Lambda function. If this function writes to the same bucket, it could cause an infinite loop of invocations."
   - Check the acknowledgment box: ☑ **I acknowledge that using the same S3 bucket for input and output is not recommended.**
8. Click **Add**.

> 📸 [Screenshot: The S3 trigger configuration showing bucket, event type, suffix filter, and the recursive invocation warning]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> That recursive invocation warning is no joke! If your Lambda writes a file back to the same bucket with the same suffix, it triggers itself again, which writes again, which triggers again... forever, until you hit the Lambda invocation limit and AWS sends you an angry bill. Always use separate input and output buckets in production!

---

> <img src="https://img.shields.io/badge/Step%205-Test-E74C3C?style=for-the-badge" />

Now for the exciting part — let's test it!

#### Upload a File:

1. Go to **S3 → Buckets → ravi-lambda-trigger-bucket-12345**.
2. Click **Upload**.
3. Click **Add files** and select any `.jpg` image from your computer.
   - 💡 Don't have a `.jpg`? Open Paint, draw anything, save it as a `.jpg`.
4. Click **Upload**.
5. Wait for the upload to complete (you'll see "Upload succeeded").

#### Check Lambda Logs:

1. Go back to **Lambda → Functions → s3-image-processor**.
2. Click the **Monitor** tab.
3. Click **Logs** → **Log groups**.
4. You should see a log group called `/aws/lambda/s3-image-processor`. Click on it.
5. Click on the **most recent log stream** (the one at the top with the latest timestamp).
6. Look for lines that say:

```
New file uploaded!
Bucket: ravi-lambda-trigger-bucket-12345
Key: your-image-name.jpg
Size: 12345 bytes
Content Type: image/jpeg
```

> 📸 [Screenshot: CloudWatch Logs showing the Lambda function output with the S3 file details]

🎉 **CONGRATULATIONS!** Your Lambda function successfully:
- Detected the file upload to S3
- Read the file metadata
- Logged everything to CloudWatch

#### Test with Another File:

1. Upload another `.jpg` file to the S3 bucket.
2. Go back to CloudWatch Logs → Click **Actions → Refresh** (or just go back and re-enter the log stream).
3. A new log entry should appear for the second file!

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> CloudWatch Logs are your best friend when debugging Lambda functions. If something doesn't work, ALWAYS check the logs first. In the console, you can also use the **Test** tab to invoke the function manually with a sample event.

---

> <img src="https://img.shields.io/badge/Step%206-Add%20IAM%20Permission-9B59B6?style=for-the-badge" />

If your Lambda function can't read S3 metadata (you see an error in CloudWatch Logs), you need to add the `s3:GetObject` permission to the Lambda execution role.

1. Go to **IAM → Roles**.
2. Search for `lambda-s3-role` and click on it.
3. Click **Add permissions → Attach policies**.
4. Search for `AmazonS3ReadOnlyAccess`.
5. Check the box → **Add permissions**.

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> The basic Lambda permissions role only includes CloudWatch Logs access — the `s3:GetObject` permission for reading file metadata is added in the step below. In production, always create the smallest policy possible.

---

> <img src="https://img.shields.io/badge/Step%207-Simple%20Transformation%20(Advanced)-1ABC9C?style=for-the-badge" />

Want to take it further? Let's modify the Lambda to **copy the uploaded file to a destination bucket**.

#### Create a Destination Bucket:
1. Go to **S3 → Create bucket**.
2. **Bucket name:** `ravi-lambda-processed-bucket-12345` (use a unique name).
3. Keep all defaults → **Create bucket**.

#### Update the Lambda Code:
1. Go to **Lambda → s3-image-processor → Code**.
2. Replace the code with:

```python
import json
import urllib.parse
import boto3

s3 = boto3.client('s3')

DESTINATION_BUCKET = 'ravi-lambda-processed-bucket-12345'  # Replace with your bucket name

def lambda_handler(event, context):
    for record in event['Records']:
        source_bucket = record['s3']['bucket']['name']
        key = urllib.parse.unquote_plus(record['s3']['object']['key'])
        size = record['s3']['object']['size']

        print(f"Processing: {key} ({size} bytes) from {source_bucket}")

        # Copy file to destination bucket
        copy_source = {'Bucket': source_bucket, 'Key': key}
        destination_key = f"processed/{key}"
        s3.copy_object(
            Bucket=DESTINATION_BUCKET,
            CopySource=copy_source,
            Key=destination_key
        )

        print(f"Copied to: {DESTINATION_BUCKET}/{destination_key}")

    return {
        'statusCode': 200,
        'body': json.dumps(f'Processed {len(event["Records"])} file(s)')
    }
```

3. Click **Deploy**.
4. Upload a new `.jpg` to the trigger bucket.
5. Check the `ravi-lambda-processed-bucket-12345` — the file should appear in a `processed/` folder!

> 📸 [Screenshot: The destination bucket showing the copied file in the processed/ folder]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> This pattern — copy file, process it, save to another location — is exactly how real-world data pipelines work. Think about Instagram: you upload a photo, Lambda resizes it into multiple sizes (thumbnail, medium, large), and stores them all. That's Lambda + S3 in production!

---

> <img src="https://img.shields.io/badge/Step%208-Verify%20Your%20Work-F39C12?style=for-the-badge" />

1. **Lambda function exists:** Lambda → Functions → `s3-image-processor` appears.
2. **S3 trigger configured:** Lambda → Configuration → Triggers → S3 trigger for `ravi-lambda-trigger-bucket-12345` with `.jpg` suffix.
3. **Upload triggers Lambda:** Upload a `.jpg` to the S3 bucket → Lambda runs.
4. **CloudWatch Logs show output:** Logs show bucket name, file key, size, and content type.
5. **(Optional) File copied:** If you did Step 7, the processed file appears in the destination bucket.

---

## ✅ Validation Checklist

| # | Check | Status |
|---|-------|--------|
| 1 | S3 bucket `ravi-lambda-trigger-bucket-*` created | ☐ |
| 2 | Lambda function `s3-image-processor` created with Python 3.12 | ☐ |
| 3 | Lambda code deployed with S3 event processing logic | ☐ |
| 4 | S3 trigger configured with `.jpg` suffix filter | ☐ |
| 5 | Uploading a `.jpg` triggers the Lambda | ☐ |
| 6 | CloudWatch Logs show file metadata (bucket, key, size, content type) | ☐ |
| 7 | IAM role `lambda-s3-role` has S3 permissions | ☐ |
| 8 | (Optional) Processed file copied to destination bucket | ☐ |

---

> **Achievement Unlocked:** Serverless Wizard! Lambda + S3 magic!

---

## 🧹 Cleanup (IMPORTANT!)

> ⚠️ **CLEAN UP NOW to avoid charges!** Lambda is cheap but S3 storage and CloudWatch logs add up if forgotten.

| Step | Action | How |
|:-----|:-------|:----|
| 1 | Delete Lambda function | Lambda → Functions → `s3-image-processor` → Actions → Delete |
| 2 | Empty & delete S3 buckets | S3 → empty each bucket first, then delete |
| 3 | Delete IAM role | IAM → Roles → `lambda-s3-role` → Delete |
| 4 | Delete CloudWatch log group | CloudWatch → Logs → Log groups → `/aws/lambda/s3-image-processor` → Delete |
| 5 | Remove S3 trigger (if not auto-deleted) | Lambda → Triggers → remove S3 trigger |

### Detailed Steps:

1. **Delete the Lambda Function:**
   - Go to **Lambda** → **Functions** → `s3-image-processor`
   - Click **Actions** → **Delete**
   - Type `delete` to confirm → **Delete**

2. **Empty and Delete S3 Buckets:**
   - Go to **S3** → **Buckets**
   - Click on `ravi-lambda-trigger-bucket-12345`
   - **Empty the bucket first:**
     - Select all files → Click **Delete** → Type `permanently delete` → **Delete objects**
   - Go back to the bucket list → Click on the bucket → **Delete** → Type the bucket name → **Delete bucket**
   - Repeat for `ravi-lambda-processed-bucket-12345` if you created it

3. **Delete the IAM Role:**
   - Go to **IAM** → **Roles**
   - Search for `lambda-s3-role`
   - Click on it → **Delete** → Type the role name → **Delete role**

4. **Delete CloudWatch Log Group:**
   - Go to **CloudWatch** → **Logs** → **Log groups**
   - Find `/aws/lambda/s3-image-processor`
   - Select it → **Delete log group**
   - ⚠️ CloudWatch logs can accumulate over time and cost money if not cleaned up

5. **Remove S3 Trigger (if still attached):**
   - Go to **Lambda** → `s3-image-processor` → **Configuration** → **Triggers**
   - Select the S3 trigger → **Delete**
   - ⚠️ This step may be automatic when you delete the function, but verify it's gone

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Always empty S3 buckets before deleting them — AWS won't let you delete a bucket that still has objects in it. It's like trying to throw away a box without taking out the stuff inside first! And don't forget CloudWatch logs — they're sneaky costs that accumulate quietly!

---

## 🧠 Memory Tips

Stick these in your brain and they'll never leave. 🧲

| 🧠 Memory Hook | Remember it like... |
|---|---|
| **Lambda = serverless** | There are **no servers to patch, babysit, or pay for while idle**. Code appears, runs, vanishes. 🪄 |
| **`lambda_handler(event, context)`** | The **front door** of every Python Lambda. `event` = what happened; `context` = runtime info. 🚪 |
| **Execution role = ID badge** | Lambda needs an **IAM role** to touch S3/CloudWatch. No badge, no access. 🪪 |
| **Event payload = the story** | S3 sends a JSON story about what happened (bucket, key, size). Your function reads it like a detective. 🕵️ |
| **Logs → CloudWatch** | Every `print()` lands in CloudWatch Logs — your debugging window. 👀 |

> 🗣️ **Rithu:** *"If you can write a function that reacts to an S3 upload, you can build half the modern internet's plumbing."

---

## 🎓 What You Learned

In this lab, you learned:

1. **AWS Lambda** — Running code without managing servers (serverless compute).
2. **Event-Driven Architecture** — S3 events trigger Lambda functions automatically.
3. **S3 Event Notifications** — S3 can notify Lambda (and other services) when objects are created, deleted, or modified.
4. **Lambda Execution Role** — IAM roles grant Lambda permission to access other AWS services.
5. **CloudWatch Logs** — Lambda automatically logs to CloudWatch for debugging and monitoring.
6. **Lambda Handler Function** — The `lambda_handler(event, context)` function is the entry point for all Lambda functions.
7. **S3 Event Payload** — Understanding the JSON structure of S3 events (bucket, key, size, etc.).

---

## 🎮 Test Yourself! (No Peeking 👀)

**Q1:** What triggers the Lambda function in this lab?

<details><summary>👀 Show answer</summary>

**A:** An **S3 event** — when an object is uploaded to the bucket, S3 fires a notification that invokes your function. 📬

</details>

**Q2:** What is the entry point of a Python Lambda function?

<details><summary>👀 Show answer</summary>

**A:** **`lambda_handler(event, context)`** — the named handler you set in the Lambda configuration. 🚪

</details>

**Q3:** Where can you see your function's `print()` output for debugging?

<details><summary>👀 Show answer</summary>

**A:** **CloudWatch Logs** — Lambda writes all logs to a log group named after your function, with a new stream per invocation. 👀

</details>

### 🔥 Bonus Challenge

Modify the function to also log the **file size** from the event payload, upload a big file, and check CloudWatch Logs to confirm the size appears. Then add a second event type (e.g., `s3:ObjectRemoved:Delete`) and watch the function react to deletions too. 🔄

> 💪 **Rithu:** *"Reading the event JSON is the superpower. Log it, pretty-print it, study it — that's where all the answers live."

---

### 🆚 Pro Tip vs Noob Tip

| | Approach |
|---|---|
| **Noob Tip** | Run a 24/7 server to watch a bucket and process files |
| **Pro Tip** | Event-driven Lambda: zero cost when idle, infinite scale when busy |

---

## 🔗 What's Next?

You've built a serverless function triggered by storage events — now let's build an **API** that anyone in the world can call!

👉 **[Lab 19 — Lambda: API Gateway REST API](../19%20-%20Lambda%20-%20API%20Gateway%20REST%20API/README.md)**

In the next lab, you'll create a REST API with API Gateway that invokes Lambda functions for each HTTP endpoint — the foundation of modern serverless applications!

---

## ❓ Troubleshooting

<details>
<summary><strong>Lambda doesn't trigger when I upload a file</strong></summary>

**Cause:** The S3 trigger isn't configured correctly, or the suffix filter doesn't match your file type.
**Fix:** 
- Go to Lambda → Configuration → Triggers → verify the S3 trigger exists.
- Make sure you're uploading a `.jpg` file (if you set the suffix filter).
- Check the S3 bucket event notification settings.

</details>

<details>
<summary><strong>CloudWatch Logs are empty</strong></summary>

**Cause:** The Lambda function hasn't been invoked, or the log stream is in a different region.
**Fix:**
- Make sure you're looking in the same region as the Lambda function.
- Check CloudWatch Logs → Log groups → `/aws/lambda/s3-image-processor`.
- Verify the S3 trigger is configured.

</details>

<details>
<summary><strong>Lambda fails with "Access Denied" or "AccessDeniedException"</strong></summary>

**Cause:** The Lambda execution role doesn't have permission to access S3.
**Fix:** Go to IAM → Roles → `lambda-s3-role` → Add the `AmazonS3ReadOnlyAccess` policy.

</details>

<details>
<summary><strong>Lambda fails with "Unable to import module 'lambda_function'"</strong></summary>

**Cause:** There's a syntax error in your Python code.
**Fix:** Go to Lambda → Code → check for typos. Common issues: missing colons, wrong indentation, unmatched parentheses.

</details>

<details>
<summary><strong>Recursive invocation warning / Lambda keeps running</strong></summary>

**Cause:** Your Lambda is writing to the same bucket it's reading from.
**Fix:** Use a different destination bucket, or remove the file copy logic. If your Lambda is already stuck in a loop, go to Lambda → Configuration → Triggers → disable the S3 trigger immediately!

</details>

<details>
<summary><strong>File doesn't appear in destination bucket (Step 7)</strong></summary>

**Cause:** The destination bucket name is wrong, or the Lambda role doesn't have write permissions.
**Fix:** Check the `DESTINATION_BUCKET` variable in your code matches the actual bucket name. Ensure the Lambda role has `AmazonS3FullAccess` (for lab purposes).

</details>

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Lambda failures are very common when learning — don't get discouraged! The key is to ALWAYS check CloudWatch Logs first. The logs will tell you exactly what went wrong. Think of CloudWatch as your Lambda's diary — it writes down everything that happens, good or bad! 📝

---

<div align="center">

<img src="https://img.shields.io/badge/Lab%2018-Complete!-F39C12?style=for-the-badge&labelColor=232F3E" />

> 🎉 **Incredible work, Ravi!** You've built your first event-driven serverless application! S3 + Lambda is one of the most powerful patterns in AWS. You're now thinking like a cloud architect! 🚀

</div>
