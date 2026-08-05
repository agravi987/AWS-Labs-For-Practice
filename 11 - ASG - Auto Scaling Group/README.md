<div align="center">

<img src="https://img.shields.io/badge/Lab%2011-Auto%20Scaling%20Group-F39C12?style=for-the-badge&labelColor=232F3E" />

</div>

<div align="center">

<img src="https://img.shields.io/badge/Difficulty-Medium-F4D03F?style=flat-square" />
<img src="https://img.shields.io/badge/Time-~35min-3498DB?style=flat-square" />
<img src="https://img.shields.io/badge/Cost-%3C%242-2ECC71?style=flat-square" />
<img src="https://img.shields.io/badge/Service-EC2%2FASG-E67E22?style=flat-square" />

</div>

> "Ravi, imagine never having to manually launch an EC2 instance again. Auto Scaling Groups are like having a robot intern who watches your servers and spins up new ones when things get busy. Let's build one!" — Rithu

---

<details>
<summary><b>🎭 Ravi & Rithu's Coffee Break Chat</b></summary>

**Ravi:** "So the ASG just... hires more servers when it's busy?"

**Rithu:** "Exactly. And fires them when things calm down. All by itself. While you sleep."

**Ravi:** "What if a server randomly dies?"

**Rithu:** "The ASG notices and launches a replacement. It's like a plant-watering robot that also buys a new plant when one wilts."

**Ravi:** "How do I tell it what a 'server' looks like?"

**Rithu:** "With a Launch Template — the recipe card. AMI, instance type, security group, user data. The ASG just follows the recipe." 📋

</details>

---

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

> **What is this, really?** An Auto Scaling Group is a **robot operations manager** that keeps exactly the number of servers you want — no more, no less. Give it a recipe (Launch Template) and a target (desired count), and it launches servers when demand spikes (scale out) and retires them when it drops (scale in). Even better: if a server dies, it quietly launches a replacement. 🤖
>
> 🌍 **Why you should care:** This is how apps stay up during Black Friday and don't waste money in July. Automatic scaling is the difference between a hobby project and a production system.

---

## 🎯 Objective

In this lab, you will:

- Create a **Launch Template** that defines what each EC2 instance looks like
- Build an **Auto Scaling Group (ASG)** that automatically maintains your desired number of instances
- Configure a **Target Tracking Scaling Policy** based on CPU utilization
- **Stress test** your instances to watch the ASG scale out and scale in automatically
- Understand how AWS keeps your application available without you lifting a finger

---

## 🧠 Prerequisites

Before you start, make sure you have:

- ✅ Completed **Lab 10** (Load Balancer basics)
- ✅ An AWS account with Free Tier access
- ✅ Your **first-key-pair** key pair downloaded and ready
- ✅ Basic comfort with the EC2 console
- ✅ A cup of coffee (optional but recommended ☕)

---

## 💰 Cost Warning

| Resource | Cost |
|----------|------|
| t2.micro instances (Free Tier) | 750 hrs/month free |
| ASG itself | Free — you only pay for the EC2 instances it launches |
| **Estimated total** | **< $2 for this lab** |

> ⚠️ **Rithu says:** ASG is FREE — it's the instances it launches that cost money. Since we're using t2.micro (Free Tier eligible), you're totally safe as long as you clean up! But always watch your instance count. The max is set to 4, so worst case you're paying for a few extra t2.micro instances for a short time.

---

## 🏗️ Architecture

```
                    ┌─────────────────────────────┐
                    │      Auto Scaling Group      │
                    │         "ravi-asg"           │
                    │                              │
                    │   Desired: 2  Min: 1  Max: 4 │
                    └──────────┬──────────────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                 ▼
     ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
     │  EC2 (AZ-a)  │ │  EC2 (AZ-b)  │ │  EC2 (AZ-a)  │
     │  Amazon Linux │ │  Amazon Linux │ │  (Scale Out) │
     │  + Apache     │ │  + Apache     │ │              │
     └──────────────┘ └──────────────┘ └──────────────┘

     Scaling Policy: Target Tracking — CPU > 50% → Add instances
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Create a Launch Template

> <img src="https://img.shields.io/badge/Step%201-Create%20Launch%20Template-3498DB?style=for-the-badge" />

A Launch Template is like a recipe card — it tells the ASG exactly what kind of EC2 instance to create every time it needs a new one.

1. Go to the **EC2 Console** → left sidebar → click **Launch Templates**
2. Click the orange **Create launch template** button
3. Fill in the details:

| Field | Value |
|-------|-------|
| Launch template name | `ravi-web-template` |
| Template version description | "Web server template for ASG lab" |
| Auto Scaling guidance | ✅ Check this box |

4. Under **Amazon machine image (AMI)**, select:
   - Amazon Linux 2023 AMI (Free Tier eligible)
5. **Instance type:** `t2.micro`
6. **Key pair:** Select `first-key-pair` (the one you created in earlier labs)
7. Under **Network settings** → **Security groups:**
   - Click **Create security group**
   - Security group name: `asg-sg`
   - Description: "Security group for Auto Scaling Group lab"
   - **Inbound rules:**
     - Type: HTTP | Port: 80 | Source: Anywhere (`0.0.0.0/0`)
     - Type: SSH | Port: 22 | Source: **My IP**
   - Click **Create security group**

8. Under **Advanced details** → **User data**, paste this script:

```bash
#!/bin/bash
yum install -y httpd
systemctl start httpd
echo "<h1>Hello from Auto Scaling Group!</h1><p>Instance: $(hostname)</p>" > /var/www/html/index.html
```

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> This user data script installs Apache and creates a simple webpage that shows which instance is serving you. When the ASG launches new instances, each one runs this script automatically. Magic! ✨

9. Click **Create launch template**

📸 **[Screenshot: Launch template creation page with all fields filled in]**

---

### Step 2: Create the Auto Scaling Group

> <img src="https://img.shields.io/badge/Step%202-Create%20Auto%20Scaling%20Group-2ECC71?style=for-the-badge" />

Now the fun part — let's build the ASG!

1. In the EC2 Console → left sidebar → click **Auto Scaling Groups**
2. Click the orange **Create Auto Scaling group** button

**Name and launch template:**

| Field | Value |
|-------|-------|
| Auto Scaling group name | `ravi-asg` |
| Launch template | Select `ravi-web-template` |
| Version | Default (latest) |

3. Click **Next**

**Choose instance launch options:**

| Field | Value |
|-------|-------|
| VPC | Default VPC |
| Subnets | Select **us-east-1a** AND **us-east-1b** (click both checkboxes!) |

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Selecting 2 Availability Zones is super important! It means if one AZ goes down, your app still runs in the other. This is the cloud equivalent of "don't put all your eggs in one basket." 🧺

4. Click **Next**

**Configure advanced options:**

| Setting | Value |
|---------|-------|
| Load balancing | None (we'll keep it simple) |
| Health check type | EC2 |
| Health check grace period | 300 seconds |

5. Click **Next**

**Configure group size and scaling:**

| Setting | Value |
|---------|-------|
| Desired capacity | **2** |
| Minimum capacity | **1** |
| Maximum capacity | **4** |

6. Under **Scaling policies**, select: **Target tracking scaling policy**

| Setting | Value |
|---------|-------|
| Policy name | `ravi-cpu-target-tracking` |
| Metric type | Average CPU utilization |
| Target value | **50** |

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> A target value of 50% means: "Hey AWS, keep the average CPU across all my instances around 50%. If it goes above that, add more instances. If it drops below, remove some." It's like a thermostat for your servers! 🌡️

7. Click **Next**
8. **Add notifications:** Skip (click Next)
9. **Add tags:** Optional — add a tag `Name` = `ravi-asg-instance`
10. Click **Create Auto Scaling group**

📸 **[Screenshot: ASG creation page showing group sizes and scaling policy]**

---

### Step 3: Wait for Instances to Launch

> <img src="https://img.shields.io/badge/Step%203-Wait%20for%20Instances-E74C3C?style=for-the-badge" />

1. Go to **EC2 Console** → **Instances**
2. You should see **2 new instances** launching! They'll have names containing "asg"
3. Wait for both to reach **Running** state and pass **2/2 status checks**

> ⏱️ This usually takes 2-3 minutes. Grab a snack! 🍿

4. Go to **EC2 Console** → **Target Groups** (under Load Balancing)
   - If your ASG created a target group automatically, click on it
   - Check the **Targets** tab — you should see both instances registered

📸 **[Screenshot: EC2 instances page showing 2 ASG-managed instances running]**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Notice that ASG automatically launched exactly 2 instances (your desired capacity) across 2 different AZs. You didn't have to manually do anything. This is the power of automation!

---

### Step 4: Verify Your Instances

> <img src="https://img.shields.io/badge/Step%204-Verify%20Instances-F39C12?style=for-the-badge" />

Let's make sure everything is working:

1. Copy the **Public IP** of one of the ASG instances
2. Open your browser and navigate to: `http://<PUBLIC_IP>`
3. You should see:

```
Hello from Auto Scaling Group!
Instance: ip-172-31-xx-xx.ec2.internal
```

4. Refresh the page a few times — you might see a different hostname, confirming that traffic is reaching different instances (if you had a load balancer)!

5. Try the other instance's public IP too — both should show the same page

📸 **[Screenshot: Browser showing the "Hello from Auto Scaling Group!" page]**

---

### Step 5: Test Scaling — Let's Break Things! 🔥

> <img src="https://img.shields.io/badge/Step%205-Test%20Scaling-9B59B6?style=for-the-badge" />

Now comes the exciting part — let's force the ASG to scale out by stressing our instances!

1. Open your terminal and SSH into **one** of the ASG instances:

```bash
ssh -i "first-key-pair.pem" ec2-user@<PUBLIC_IP>
```

2. Install the stress tool:

```bash
sudo yum install -y stress
```

3. Start stressing the CPU! We'll max out all available CPUs for 5 minutes:

```bash
stress --cpu 4 --timeout 300s
```

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> The `stress` tool pushes your CPU to 100%. Since our target tracking policy is set at 50%, ASG should detect the high CPU and start spinning up new instances. Science in action! 🔬

4. Now, keep an eye on the **ASG Activity** tab:

   - Go to **EC2** → **Auto Scaling Groups** → click `ravi-asg`
   - Click the **Activity** tab
   - After a few minutes, you should see a new activity entry:
     - **Status:** Successful
     - **Description:** "Launching a new EC2 instance"
   - This means ASG detected high CPU and launched a NEW instance! 🎉

5. Go to **EC2** → **Instances** — you should now see **3 instances** (or even 4 if both original instances are stressed)

6. Go to **CloudWatch** → **Metrics** → **EC2** → **Per-Instance Metrics** → **CPUUtilization**
   - Select both original instances
   - Watch the CPU spike above 80%!

📸 **[Screenshot: CloudWatch CPU metrics showing the spike above 50%]**

7. Wait for the stress command to finish (5 minutes), or press `Ctrl+C` in the SSH session to stop it early

8. After CPU drops back below 50% for a while, ASG will **scale in** — it will terminate the extra instance automatically!

> ⏱️ Scale-in can take 5-10 minutes after the stress ends. Be patient!

---

### Step 6: Verify Your Work

> <img src="https://img.shields.io/badge/Step%206-Verify%20Your%20Work-1ABC9C?style=for-the-badge" />

Let's confirm everything worked as expected:

**Check ASG Activity History:**

1. Go to **EC2** → **Auto Scaling Groups** → click `ravi-asg`
2. Click the **Activity** tab
3. You should see entries like:
   - "Launching a new EC2 instance: In service"
   - "Terminating an EC2 instance: In service" (if scale-in happened)

**Check Instance Count:**

1. Go to **EC2** → **Instances**
2. Filter by instance state: Running
3. Count the ASG instances — should be back to 2 (or still at 3-4 if scale-in hasn't happened yet)

**Check ASG Status:**

1. Go to **Auto Scaling Groups** → click `ravi-asg`
2. Under **Group details:**
   - Desired capacity: 2
   - Current instances: Should be 2 (or transitioning)

📸 **[Screenshot: ASG Activity tab showing scale-out and scale-in events]**

**🎉 Congratulations, Ravi!** You just witnessed auto scaling in action. The ASG saw your CPU spike, launched new instances to handle the load, and then cleaned up when things calmed down. AWS at its finest!

---

## ✅ Validation Checklist

- [ ] Launch Template `ravi-web-template` exists with correct AMI, instance type, and user data
- [ ] Security group `asg-sg` has HTTP (80) open to the world and SSH (22) from My IP
- [ ] Auto Scaling Group `ravi-asg` is running with 2 instances across 2 AZs
- [ ] Both instances are passing health checks
- [ ] Web page is accessible via browser showing "Hello from Auto Scaling Group!"
- [ ] After stress test, ASG scaled out (more instances launched)
- [ ] After stress stopped, ASG scaled in (extra instance terminated)
- [ ] ASG Activity tab shows successful scaling events

---

## 🧹 Cleanup (IMPORTANT!)

**ASG can keep launching instances if misconfigured. Let's clean up everything!**

1. **Delete the Auto Scaling Group:**
   - Go to **EC2** → **Auto Scaling Groups**
   - Select `ravi-asg`
   - Click **Delete**
   - In the confirmation dialog, type `delete`
   - Click **Delete**

   > ⏱️ Wait for all instances to terminate before proceeding

2. **Delete the Launch Template:**
   - Go to **EC2** → **Launch Templates**
   - Select `ravi-web-template`
   - Click **Actions** → **Delete launch template**
   - Confirm deletion

3. **Delete the Security Group:**
   - Go to **EC2** → **Security Groups**
   - Find `asg-sg`
   - Click **Actions** → **Delete security group**
   - Confirm deletion

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Always clean up ASGs first! If you delete the security group while the ASG is still running, the ASG might try to launch new instances and fail, leaving you in a weird state. Order matters! 🔑

📸 **[Screenshot: All resources deleted — empty Launch Templates and ASG pages]**

---

## 🧠 Memory Tips

Stick these in your brain and they'll never leave. 🧲

| 🧠 Memory Hook | Remember it like... |
|---|---|
| **Launch Template = recipe** | The full server spec (AMI, type, SG, user data). The ASG bakes from this recipe every time. 📋 |
| **Desired count = target number** | How many servers you want alive *right now*. The ASG's obsession. 🎯 |
| **Scale OUT vs scale IN** | **Out** = add servers (CPU too hot). **In** = remove servers (CPU too cold). 🌡️ |
| **Target tracking = cruise control** | "Keep CPU ~50%" and the ASG adjusts the fleet automatically — like cruise control on a highway. 🚗 |
| **Self-healing** | A server dies → ASG replaces it. The robot intern never sleeps. 🤖 |

> 🗣️ **Rithu:** *"ASG = Always Self-regulating Growth. If you remember one acronym today, make it that one."

---

## 🎓 What You Learned

| Concept | What You Now Know |
|---------|-------------------|
| **Launch Templates** | How to define a reusable EC2 configuration with AMI, instance type, security groups, and user data |
| **Auto Scaling Groups** | How ASG maintains your desired instance count across multiple AZs |
| **Target Tracking** | How to set a target metric (like CPU) and let ASG handle scaling automatically |
| **Scale Out** | ASG launches new instances when metrics exceed the target |
| **Scale In** | ASG terminates extra instances when metrics drop below the target |
| **Multi-AZ Deployment** | Why and how to spread instances across Availability Zones |

---

## 🎮 Test Yourself! (No Peeking 👀)

**Q1:** What is a Launch Template, in one sentence?

<details><summary>👀 Show answer</summary>

**A:** A **reusable recipe** for instances — AMI, instance type, security groups, key pair, and user data. The ASG uses it for every server it launches. 📋

</details>

**Q2:** CPU utilization stays above your target for 10 minutes. What does the ASG do?

<details><summary>👀 Show answer</summary>

**A:** **Scale out** — launch new instances to spread the load and bring CPU back toward the target. 🆕

</details>

**Q3:** You manually terminate one of the ASG's instances. What happens next?

<details><summary>👀 Show answer</summary>

**A:** The ASG notices the count dropped below desired and **launches a replacement** automatically. It's self-healing by design. 🤖

</details>

### 🔥 Bonus Challenge

Install `stress` on one instance and run it to peg the CPU above the target. Watch the ASG **scale out** in real time (watch the instance count climb!). Then stop the stress test and watch it **scale back in**. You just directed a server orchestra. 🎼

> 💪 **Rithu:** *"Watching your first scale-out event live is a core memory. Do it, don't skip it."

---

### 🆚 Pro Tip vs Noob Tip

| | Approach |
|---|---|
| **Noob Tip** | Launch one big instance and pray it's enough |
| **Pro Tip** | ASG + target tracking: scale out on demand, scale in when quiet — pay only for what you need |

---

## 🔗 What's Next?

You've mastered auto scaling! Time to learn how to direct traffic to your instances using **DNS**:

➡️ **Lab 12 — Route 53: DNS and Failover** — Learn how DNS works and how to set up automatic failover when an instance goes down!

---

## ❓ Troubleshooting

<details>
<summary><strong>Click to expand Troubleshooting Section</strong></summary>

### My ASG didn't launch any instances

- Check that your Launch Template is valid and the AMI exists
- Make sure the subnets you selected have available IP addresses
- Check the ASG **Activity** tab for error messages
- Verify your instance quota hasn't been reached (check Service Quotas in the console)

### Both instances show the same hostname

- That's expected! Each instance will show its own hostname, but you need to open each IP in a separate browser tab to see the difference
- If you have a Load Balancer, refresh the page and watch the hostname change

### The stress test didn't trigger scaling

- Wait at least 5 minutes — target tracking needs time to evaluate the metric
- Check that the stress is actually running: `top` or `htop` in the SSH session
- Check CloudWatch metrics to verify CPU is actually above 50%
- The ASG needs CPU to stay above 50% for the full evaluation period (usually 5 minutes)

### My security group won't delete

- Make sure no instances are still using it
- Wait for the ASG to fully terminate all instances first
- Check if there are any other resources (like ENIs) still attached

### Error: "No capacity" when ASG tries to launch

- You may have hit your EC2 instance limit
- Check your limits at: **Service Quotas** → **EC2** → **Running On-Demand Standard instances**
- Request a limit increase if needed (but for t2.micro Free Tier, you should be fine)

</details>

> 🎉 **Amazing work, Ravi!** Auto Scaling Groups are one of the most powerful features in AWS. You now know how to build self-healing, automatically-scaling infrastructure. In the next lab, we'll explore Route 53 and DNS failover — another critical piece of building resilient applications! 🚀

---

<div align="center">

<img src="https://img.shields.io/badge/Lab%2011-Complete!-27AE60?style=for-the-badge&labelColor=232F3E" />

</div>
