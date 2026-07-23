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

## 🎯 Objective

By the end of this lab, you will:
- Understand what Infrastructure as Code (IaC) means and why it matters
- Write a CloudFormation template in YAML to deploy an EC2 instance with a web server
- Create, update, and delete a CloudFormation stack using the console and CLI
- Experience the magic of one-click infrastructure teardown

CloudFormation is **FREE** — you only pay for the AWS resources it creates. Think of CloudFormation as your personal AWS architect that never forgets, never makes typos, and works while you sleep. ☁️

---

## 🧠 Prerequisites

- [ ] Completed Lab 20 (EC2)
- [ ] An EC2 key pair exists (from previous labs)
- [ ] AWS Console access with appropriate permissions
- [ ] Basic familiarity with the AWS Console

---

## 💰 Cost Warning

CloudFormation itself costs **$0**. You are only charged for the resources it creates:

| Resource | Cost |
|----------|------|
| t2.micro EC2 instance | ~$0.0116/hr (Free Tier eligible) |
| Data transfer (minimal) | ~$0.00 |

Estimated total lab cost: **< $1** if cleaned up within 1 hour.

> ⚠️ **IMPORTANT**: Delete your stack before leaving! CloudFormation stacks don't auto-delete. Leaving a t2.micro running for 24 hours costs ~$0.28. Always clean up!

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

---

## 🛠️ Step-by-Step Instructions

### <img src="https://img.shields.io/badge/Step%201-Understand%20CloudFormation-FF6B6B?style=for-the-badge" />

Before writing any code, let's understand what we're doing.

**Infrastructure as Code (IaC)** means defining your AWS resources in a text file (YAML or JSON) instead of clicking around the console. Think of it like a recipe:

- **Console clicking** = cooking without a recipe (works, but messy and hard to repeat)
- **CloudFormation** = following a precise recipe (repeatable, shareable, version-controlled)

**Why should you care?**
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
Description: CloudFormation Lab - Launch an EC2 Instance

Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
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
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0

  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyPairName
      ImageId: !Ref LatestAmiId
      SecurityGroups:
        - !Ref WebServerSecurityGroup
      Tags:
        - Key: Name
          Value: CloudFormation-WebServer
      UserData:
        Fn::Base64: |
          #!/bin/bash
          yum install -y httpd
          systemctl start httpd
          echo "<h1>Deployed by CloudFormation!</h1>" > /var/www/html/index.html

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

📸 [Screenshot: Screenshot of your ec2-stack.yaml file in your text editor]

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

14. Scroll down and check **"I acknowledge that AWS CloudFormation might create IAM resources with custom names"**
15. Click **Create stack** (orange button, bottom right)

📸 [Screenshot: The "Create stack" page showing the template uploaded and parameters filled in]

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

📸 [Screenshot: Events tab showing all resources created with CREATE_COMPLETE status]

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

- In Lab 04, you launched an EC2 instance by clicking around the console
- In this lab, you defined it in a YAML file and CloudFormation built it for you
- Same result, but the CloudFormation approach is **repeatable, shareable, and deletable**!

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "The `UserData` script in the template automatically installed Apache and created a web page. This is called 'bootstrapping' — your instance is ready to serve traffic the moment it launches!"

📸 [Screenshot: Browser showing "Deployed by CloudFormation!" and the EC2 console showing the running instance]

---

### <img src="https://img.shields.io/badge/Step%206-Update%20the%20Stack-E74C3C?style=for-the-badge" />

CloudFormation doesn't just create — it also manages updates! Let's change the web page.

1. Go back to your `ec2-stack.yaml` file in your text editor
2. Find this line:
   ```yaml
   echo "<h1>Deployed by CloudFormation!</h1>" > /var/www/html/index.html
   ```
3. Change it to:
   ```yaml
   echo "<h1>Updated by CloudFormation!</h1>" > /var/www/html/index.html
   ```
4. **Save** the file

**Now update the stack in AWS:**

5. Go back to the CloudFormation console
6. Select your stack `ravi-ec2-stack`
7. Click the **Update** button (top right)
8. Select **Replace current template**
9. Click **Next**
10. Upload your updated `ec2-stack.yaml` file
11. Click **Next** (parameters stay the same)
12. Click **Next** again
13. Check the acknowledgment checkbox
14. Click **Update stack**

**Watch the update:**

15. Click the **Events** tab
16. You'll see CloudFormation update the stack — it may **replace** the EC2 instance (delete old, create new)
17. Wait for `UPDATE_COMPLETE` status

**Verify the change:**

18. Go to the **Outputs** tab
19. Click the **WebsiteURL** link
20. You should now see: **"Updated by CloudFormation!"**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> "CloudFormation is smart — it figures out which resources need to be replaced vs updated in place. Some changes require replacement (like changing an AMI), while others can be updated in place (like changing security group rules)."

📸 [Screenshot: Updated website showing "Updated by CloudFormation!" message]

---

### <img src="https://img.shields.io/badge/Step%207-Create%20Stack%20via%20CLI-2ECC71?style=for-the-badge" />

Want to feel like a real cloud engineer? Use the command line! 🖥️

Open your terminal (Command Prompt or PowerShell) and run:

```bash
aws cloudformation create-stack \
  --stack-name ravi-ec2-stack-cli \
  --template-body file://ec2-stack.yaml \
  --parameters ParameterKey=KeyPairName,ParameterValue=first-key-pair \
  --capabilities CAPABILITY_IAM
```

**What each flag means:**
- `--stack-name` — Name for your new stack
- `--template-body` — Path to your YAML file (`file://` means local file)
- `--parameters` — Passing the KeyPairName parameter (InstanceType and LatestAmiId use defaults)
- `--capabilities` — Acknowledges IAM resource creation (needed even though our template doesn't create IAM)

**Wait for creation:**

```bash
aws cloudformation wait stack-create-complete --stack-name ravi-ec2-stack-cli
```

This command waits until the stack is fully created (may take 2-3 minutes).

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
