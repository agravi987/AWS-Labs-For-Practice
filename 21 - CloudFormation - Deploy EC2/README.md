<div align="center">

<img src="https://img.shields.io/badge/Lab%2021-CloudFormation%20Deploy%20EC2-E67E22?style=for-the-badge&labelColor=232F3E" />

# Lab 21 — CloudFormation: Deploy EC2

<img src="https://img.shields.io/badge/Difficulty-Medium-yellow?style=flat-square" />
<img src="https://img.shields.io/badge/Time-~30min-blue?style=flat-square" />
<img src="https://img.shields.io/badge/Cost-%3C%241-brightgreen?style=flat-square" />
<img src="https://img.shields.io/badge/Service-CloudFormation-orange?style=flat-square" />

</div>

> "Stop clicking around in the console, Ravi. Let's learn to write code that builds AWS for us!" — Rithu

---

<details>
<summary><b>🎭 Ravi & Rithu's Coffee Break Chat</b></summary>

**Ravi:** "CloudFormation is Infrastructure as Code?"

**Rithu:** "Yep! Instead of clicking around the console, you write YAML/JSON and AWS builds it."

**Ravi:** "Like LEGO instructions but for cloud infrastructure?"

**Rithu:** "Perfect analogy! Except the LEGO pieces cost money and some are invisible."

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

> **What is this, really?** CloudFormation is **Infrastructure as Code**: you write a YAML recipe describing what AWS resources you want (an EC2 instance, a security group, a website), and CloudFormation builds them all for you — in the right order, every time. Delete the recipe's "stack" and **everything it created disappears**. It's a contractor working from your blueprint instead of you pointing at materials. 🏗️
>
> 🌍 **Why you should care:** Clicking the console doesn't scale. Code does — it's reviewable, versionable, and reproducible. This is how real companies deploy.

---

## 🎯 Objective

By the end of this lab, you will:
- Understand what Infrastructure as Code (IaC) means and why it matters
- Write a CloudFormation template in YAML to deploy an EC2 instance with a web server
- Create, update, and delete a CloudFormation stack using the console and CLI
- Experience the magic of one-click infrastructure teardown

CloudFormation is **FREE** — you only pay for the resources it creates. Think of CloudFormation as an architect that never forgets, never makes typos, and works while you sleep. ☁️

---

## 🧠 Prerequisites

- [ ] Completed Lab 01 (EC2) — you'll reuse its `first-key-pair` key pair
- [ ] AWS Console access with appropriate permissions

---

## 💰 Cost Warning

CloudFormation itself costs **$0**. You are only charged for the resources it creates:

| Resource | Cost |
|----------|------|
| t2.micro EC2 instance | ~$0.0116/hr (Free Tier eligible) |
| Data transfer (minimal) | ~$0.00 |

Estimated total lab cost: **< $1** if cleaned up within 1 hour.

> ⚠️ **IMPORTANT**: Delete your stack before leaving! CloudFormation stacks don't auto-delete. Leaving a t2.micro running for 24 hours costs ~$0.28. Always clean up!

> **Ravi's Mistake of the Day:** I deleted a CloudFormation stack and it tried to delete the S3 bucket with it. The bucket wasn't empty, so the stack got stuck in DELETE_FAILED state for an hour. Empty buckets before deleting stacks.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────┐
│              CloudFormation                  │
│           (The Orchestrator)                │
│                                             │
│  ┌──────────────┐    ┌──────────────────┐   │
│  │  Parameters   │───▶│   Resources      │   │
│  │  (Inputs)     │    │                  │   │
│  │  - KeyPair    │    │  - SecurityGroup │   │
│  │  - InstanceType│   │  - EC2 Instance  │   │
│  │  - LatestAmiId│    │  - UserData      │   │
│  └──────────────┘    └──────────────────┘   │
│                                             │
│  ┌──────────────┐    ┌──────────────────┐   │
│  │   Outputs     │◀──│   Events Log     │   │
│  │  (Results)    │    │  (Watch tab)     │   │
│  └──────────────┘    └──────────────────┘   │
└─────────────────────────────────────────────┘
```

> **Did You Know?** CloudFormation supports Drift Detection. It can tell you if someone manually changed resources that were supposed to be managed by your template. Big Brother for infrastructure.

---

## 🛠️ Step-by-Step Instructions

### <img src="https://img.shields.io/badge/Step%201-Understand%20CloudFormation-FF6B6B?style=for-the-badge" />

**Infrastructure as Code (IaC)** means defining AWS resources in a text file (YAML or JSON) instead of clicking around the console:

- **Console clicking** = cooking without a recipe (messy, hard to repeat)
- **CloudFormation** = following a precise recipe (repeatable, shareable, version-controlled)

**Why it matters:**
- **Repeatable**: Deploy the same stack in 10 regions with one command
- **Version-controlled**: Store your template in Git — see who changed what
- **Automated**: Deploy entire environments with a single API call
- **Self-documenting**: Your template IS your infrastructure documentation
- **Clean**: Delete the stack = every resource it created is gone

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "Think of CloudFormation as a spell book. You write the spell (template), cast it (create stack), and AWS builds everything. Break the spell (delete stack), and everything vanishes."

📸 [Screenshot: None needed for this step — just understanding!]

---

### <img src="https://img.shields.io/badge/Step%202-Create%20a%20Template-FFA500?style=for-the-badge" />

Now let's write our first CloudFormation template!

1. Open **Notepad**, **VS Code**, or any text editor on your computer (NOT Word — use a plain text editor!)
2. Create a new file called `ec2-stack.yaml`
3. Copy and paste the ENTIRE template below:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: CloudFormation Lab - Launch an EC2 Instance with Apache

Parameters:
  InstanceType:
    Type: String
    Default: t3.micro
    AllowedValues:
      - t3.micro
      - t3.small
      - t2.micro
    Description: EC2 instance type

  KeyPairName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Name of an existing EC2 key pair

  LatestAmiId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64

Resources:
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable HTTP and SSH access
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyPairName
      ImageId: !Ref LatestAmiId
      SecurityGroupIds:
        - !Ref WebServerSecurityGroup
      Tags:
        - Key: Name
          Value: CloudFormation-WebServer
      UserData:
        Fn::Base64: |
          #!/bin/bash -xe
          dnf update -y
          dnf install -y httpd
          systemctl enable --now httpd
          cat > /var/www/html/index.html <<'HTML'
          <h1>Deployed by CloudFormation!</h1>
          HTML

Outputs:
  InstanceId:
    Description: Instance ID
    Value: !Ref WebServerInstance

  PublicIP:
    Description: Public IP of the instance
    Value: !GetAtt WebServerInstance.PublicIp

  WebsiteURL:
    Description: Website URL
    Value: !Sub "http://${WebServerInstance.PublicIp}"
```

4. **Save the file** somewhere easy to find (like your Desktop)

Let's break down what each section does:

- **Parameters** — Inputs you provide when creating the stack (like filling in a form)
- **Resources** — The actual AWS resources to create (Security Group + EC2 Instance)
- **Outputs** — Information displayed after the stack is created (like a receipt)
- **!Ref** and **!Sub** — Special CloudFormation functions (like variables)

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "The `LatestAmiId` parameter uses SSM Parameter Store to automatically fetch the latest Amazon Linux 2023 AMI. No more hardcoding AMI IDs!"

📸 [Screenshot: The `ec2-stack.yaml` file saved in a text editor before upload]
![The `ec2-stack.yaml` file saved in a text editor before upload](screenshots/01-ec2-stack-template-file.png)

---

### <img src="https://img.shields.io/badge/Step%203-Create%20Stack%20via%20Console-9B59B6?style=for-the-badge" />

Time to deploy! Let's use the AWS Console first.

1. Open the **AWS Console** in your browser
2. Search for **CloudFormation** in the search bar and click on it
3. You should see the CloudFormation dashboard with **no stacks** listed (if this is your first time)
4. Click the orange **Create stack** button (top right)
5. Select **With new resources (standard)** from the dropdown

**Configure your stack:**

6. Under **Template source**, select **Upload a template file**
7. Click **Choose file** and select your `ec2-stack.yaml` file
8. Click **Next**

**Specify stack details:**

9. **Stack name**: Type `ravi-ec2-stack`
10. **Parameters**:
    - **InstanceType**: Leave as `t2.micro` (default)
    - **KeyPairName**: Select `first-key-pair` from the dropdown (your existing key pair)
    - **LatestAmiId**: Leave as default (it auto-fills)
11. Click **Next**

**Configure stack options:**

12. Leave everything as default
13. Click **Next**

**Review:**

14. Scroll to the bottom — no "acknowledge IAM" checkbox appears (this template creates no IAM resources)
15. Click **Create stack** (orange button, bottom right)

📸 [Screenshot: The CloudFormation create stack page with the template uploaded and parameters filled in]
![The CloudFormation create stack page with the template uploaded and parameters filled in](screenshots/02-cloudformation-create-stack-form.png)


---

### <img src="https://img.shields.io/badge/Step%204-Wait%20for%20Stack%20Creation-3498DB?style=for-the-badge" />

Now watch the magic happen! 🪄

1. You should be on the **Stack details** page for `ravi-ec2-stack`
2. Click the **Events** tab to watch the creation in real-time
3. You'll see events appearing like:
   - `WebServerSecurityGroup` → CREATE_IN_PROGRESS → CREATE_COMPLETE
   - `WebServerInstance` → CREATE_IN_PROGRESS → CREATE_COMPLETE
4. The whole process takes **2-3 minutes**

**Watch for these statuses:**
- `CREATE_IN_PROGRESS` — Being created right now
- `CREATE_COMPLETE` — Successfully created ✅
- `CREATE_FAILED` — Something went wrong (check the error message)

5. Wait until the **Stack status** shows **`CREATE_COMPLETE`** (in green) at the top of the page

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "If you see `CREATE_FAILED`, don't panic! Read the error message — CloudFormation gives you detailed reasons. Common issues: wrong key pair name, missing permissions, or resource limits."

📸 [Screenshot: CloudFormation Events showing the stack resources created with CREATE_COMPLETE status]
![CloudFormation Events showing the stack resources created with CREATE_COMPLETE status](screenshots/03-cloudformation-events-create-complete.png)

---

### <img src="https://img.shields.io/badge/Step%205-Verify%20Your%20Work-1ABC9C?style=for-the-badge" />

Time to check that everything was created correctly!

**Check the Outputs tab:**

1. Click the **Outputs** tab on the stack details page
2. You should see three outputs:
   - **InstanceId** — Something like `i-0abc123def456`
   - **PublicIP** — Something like `54.xx.xx.xx`
   - **WebsiteURL** — Something like `http://54.xx.xx.xx`

3. Click the **WebsiteURL** link — it should open in a new tab
4. You should see a page that says: **"Deployed by CloudFormation!"**
5. 🎉 Congratulations! Your CloudFormation template worked!

**Check EC2:**

6. Open a new browser tab
7. Search for **EC2** in the AWS Console
8. Click **Instances** in the left sidebar
9. You should see an instance named **`CloudFormation-WebServer`**
10. Check that its state is **Running**

**Compare with previous labs:**

- In Lab 01, you launched an EC2 instance by clicking around the console
- In this lab, you defined it in a YAML file and CloudFormation built it for you
- Same result, but the CloudFormation approach is **repeatable, shareable, and deletable**!

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "The `UserData` script in the template automatically installed Apache and created a web page. This is called 'bootstrapping' — your instance is ready to serve traffic the moment it launches!"

📸 [Screenshot: Browser showing the deployed page with the CloudFormation message]
![Browser showing the deployed page with the CloudFormation message](screenshots/04-cloudformation-website-deployed.png)

---

### <img src="https://img.shields.io/badge/Step%206-Update%20the%20Stack-E74C3C?style=for-the-badge" />

CloudFormation can update resources, but the important detail is this: **changing the EC2 instance's `UserData` is what triggers the instance to restart and rerun the bootstrap script**. This is why the original lab sometimes looked like it was "not working" — because the manual `systemctl` commands were not the same as an actual CloudFormation update.

> ✅ Correct mental model: `UserData` runs when an EC2 instance launches or is restarted after an update. It does not magically run again just because you typed `systemctl enable httpd` on a running machine.

1. Go back to your `ec2-stack.yaml` file in your text editor.
2. Change the HTML in the `UserData` script from:
   ```bash
   <h1>Deployed by CloudFormation!</h1>
   ```
   to:
   ```bash
   <h1>Updated by CloudFormation!</h1>
   ```
3. Save the file.

**Now update the stack in AWS:**

4. Go back to the CloudFormation console.
5. Select your stack `ravi-ec2-stack`.
6. Click the **Update** button (top right).
7. Select **Replace current template**.
8. Click **Next**.
9. Upload the updated `ec2-stack.yaml` file.
10. Click **Next**.
11. Click **Next** again.
12. Click **Update stack**.

**Watch the update:**

13. CloudFormation will show a change set preview first. Click **Submit** to apply it.
14. The EC2 instance will be restarted because the `UserData` content changed.
15. Wait until the status changes to `UPDATE_COMPLETE`.

**Verify the change:**

16. Click the **Outputs** tab.
17. Open the **WebsiteURL** again.
18. You should now see: **"Updated by CloudFormation!"**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "If you only run `systemctl enable httpd` manually, that helps with boot-time startup but does not actually re-run the CloudFormation bootstrap logic. The reliable pattern is: update the template, then update the stack."

📸 [Screenshot: Updated website showing "Updated by CloudFormation!" message]

---

### <img src="https://img.shields.io/badge/Step%207-Create%20Stack%20via%20CLI-2ECC71?style=for-the-badge" />

Want to feel like a real cloud engineer? Use the command line! 🖥️

Open your terminal (Command Prompt or PowerShell) and run:

```bash
aws cloudformation validate-template --template-body file://ec2-stack.yaml

aws cloudformation create-stack --stack-name ravi-ec2-stack-cli --template-body file://ec2-stack.yaml --parameters "ParameterKey=KeyPairName,ParameterValue=first-key-pair"
```

**What each flag means:**
- `validate-template` — Checks the YAML for syntax and schema errors before you deploy (run this first)
- `--stack-name` — Name for your new stack
- `--template-body` — Path to your YAML file (`file://` means local file)
- `--parameters` — Passes the KeyPairName value (InstanceType and LatestAmiId use their defaults)
- No `--capabilities` flag — only required for templates that create IAM resources (ours doesn't)

**Wait for creation:**

```bash
aws cloudformation wait stack-create-complete --stack-name ravi-ec2-stack-cli
```

This command waits until the stack is fully created (2-3 minutes).

**Check the stack:**

```bash
aws cloudformation describe-stacks --stack-name ravi-ec2-stack-cli
```

**Get the outputs:**

```bash
aws cloudformation describe-stacks --stack-name ravi-ec2-stack-cli --query "Stacks[0].Outputs"
```

You should see the InstanceId, PublicIP, and WebsiteURL!

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "The CLI is powerful for automation. Imagine running this in a script at 3 AM to deploy infrastructure automatically. That's the power of IaC!"

📸 [Screenshot: Terminal showing the CLI commands and their output]

---

## ✅ Validation Checklist

Before moving on, confirm all of these:

- [ ] `ec2-stack.yaml` file exists on your computer
- [ ] Stack `ravi-ec2-stack` shows status `CREATE_COMPLETE`
- [ ] Outputs tab shows InstanceId, PublicIP, and WebsiteURL
- [ ] WebsiteURL loads a page saying "Updated by CloudFormation!"
- [ ] EC2 console shows `CloudFormation-WebServer` instance running
- [ ] If you did the CLI bonus: stack `ravi-ec2-stack-cli` also exists and is complete

---

> **Achievement Unlocked:** IaC Pioneer! CloudFormation unlocked.

---

## 🧹 Cleanup (IMPORTANT!)

CloudFormation's **best feature**: deleting the stack deletes EVERYTHING it created! No manual cleanup needed.

**Delete the console stack:**

1. Go to **CloudFormation** → **Stacks**
2. Select `ravi-ec2-stack`
3. Click **Delete** (top right)
4. Confirm by typing `delete` in the confirmation box
5. Click **Delete stack**
6. Watch the events — resources are deleted in reverse order
7. Wait for the stack to disappear from the list (status: `DELETE_COMPLETE`)

**Delete the CLI stack (if you created it):**

```bash
aws cloudformation delete-stack --stack-name ravi-ec2-stack-cli
aws cloudformation wait stack-delete-complete --stack-name ravi-ec2-stack-cli
```

**Verify everything is gone:**

8. Go to **EC2** → **Instances**
9. Confirm `CloudFormation-WebServer` is terminated
10. Go to **CloudFormation** → confirm no stacks remain

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "This is why I love CloudFormation. One click and EVERYTHING is cleaned up. No orphaned resources, no surprise bills. Compare this to manually creating resources — you'd have to remember to delete each one individually!"

📸 [Screenshot: CloudFormation console showing no stacks remaining]

---

## 🧠 Memory Tips

Stick these in your brain and they'll never leave. 🧲

| 🧠 Memory Hook | Remember it like... |
|---|---|
| **Template = blueprint, Stack = house** | The YAML describes the plan; the **stack** is the actual resources built from it. 📐 |
| **Stack lifecycle: C-U-D** | **C**REATE → **U**PDATE → **D**ELETE. Say it like a chef: "cook, stir, dump." 🍳 |
| **Delete stack = magic cleanup** | One click and **every resource the stack made is gone**. No hunting through consoles! 🪄 |
| **Parameters = fill-in-the-blanks** | Make your template reusable — ask for key pair names, AMI IDs, sizes at deploy time. 🧩 |
| **CloudFormation is FREE** | You pay only for the resources it creates. The architect himself charges nothing. 🆓 |

> 🗣️ **Rithu:** *"The first time you delete a stack and watch 5 resources vanish at once, you'll understand why console-clicking feels primitive."

---

## 🎓 What You Learned

In this lab, you learned:

| Concept | What It Means |
|---------|---------------|
| **Infrastructure as Code (IaC)** | Define AWS resources in code files instead of clicking |
| **CloudFormation Template** | YAML/JSON file that describes your desired AWS resources |
| **Parameters** | User inputs that make templates reusable |
| **Resources** | The actual AWS resources CloudFormation creates |
| **Outputs** | Useful information displayed after stack creation |
| **Stack Lifecycle** | CREATE → UPDATE → DELETE |
| **Auto-teardown** | Deleting a stack removes ALL resources it created |
| **AWS CLI Integration** | Create stacks from the command line for automation |

---

## 🎮 Test Yourself! (No Peeking 👀)

**Q1:** What's the difference between a template and a stack?

<details><summary>👀 Show answer</summary>

**A:** The **template** is the YAML plan; the **stack** is the set of live resources CloudFormation created from that plan. 📐

</details>

**Q2:** How do you delete ALL the resources a CloudFormation stack created?

<details><summary>👀 Show answer</summary>

**A:** **Delete the stack.** CloudFormation tears down everything it created — instances, SGs, the lot. One click cleanup. 🪄

</details>

**Q3:** Why do companies prefer Infrastructure as Code over console clicks?

<details><summary>👀 Show answer</summary>

**A:** It's **reproducible** (same template = same infra), **reviewable** (code reviews!), **versionable** (git history), and **recoverable** (redeploy after disaster). 📜

</details>

### 🔥 Bonus Challenge

Add an **S3 bucket resource** to your template (search `AWS::S3::Bucket`), update the stack, and confirm the bucket appears in your account. Then **delete the stack** and watch the bucket vanish with it. You just did one-click create AND teardown. 🏗️

> 💪 **Rithu:** *"If you leave this lab still clicking the console for everything, re-read this section. Code is the way."

---

### 🆚 Pro Tip vs Noob Tip

| | Approach |
|---|---|
| **Noob Tip** | Rebuild infrastructure by hand after a disaster — hours of clicking |
| **Pro Tip** | IaC: redeploy the whole environment from a template in minutes. Version-controlled, reviewable |

---

## 🔗 What's Next?

You've mastered CloudFormation! In the next lab, we'll explore **CloudTrail** — AWS's audit logging service that records every action taken in your account.

**[Lab 22 — CloudTrail: Enable and Query →](../22%20-%20CloudTrail%20-%20Enable%20and%20Query/README.md)**

---

## ❓ Troubleshooting

<details>
<summary><strong>Stack creation fails with "The key pair does not exist"</strong></summary>

**Fix**: Go to EC2 → Key Pairs and verify your key pair name. Make sure you typed it exactly (case-sensitive).
</details>

<details>
<summary><strong>Stack creation fails with "AMI not found"</strong></summary>

**Fix**: The SSM parameter should auto-resolve. If it fails, manually replace the LatestAmiId parameter with a specific AMI ID for your region.
</details>

<details>
<summary><strong>Website shows "This site can't be reached"</strong></summary>

**Fix**: Check that the EC2 instance is running. Check the Security Group allows inbound HTTP (port 80). Wait 1-2 minutes for the UserData script to finish.
</details>

<details>
<summary><strong>"Permission denied" when creating the stack</strong></summary>

**Fix**: Make sure your IAM user/role has permissions for CloudFormation, EC2, and Security Groups. If using a root account, ensure MFA is enabled.
</details>

<details>
<summary><strong>Stack update fails</strong></summary>

**Fix**: Check the Events tab for the specific error. Some changes require resource replacement — CloudFormation will tell you which ones.
</details>

<details>
<summary><strong>Stack won't delete because of dependencies</strong></summary>

**Fix**: Check if you manually created resources that reference stack resources. Delete those first, then try again.
</details>

---

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "You just wrote your first CloudFormation template! From now on, every AWS resource you create, try to think: 'Can I template this?' The answer is almost always yes. Welcome to the world of Infrastructure as Code!"

---

<div align="center">

<img src="https://img.shields.io/badge/Lab%2021-Complete!-27AE60?style=for-the-badge&labelColor=232F3E" />

**🎉 Congratulations on completing Lab 21! 🎉**

</div>
