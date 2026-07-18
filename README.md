<div align="center">

# ☁️ AWS Hands-On Labs 🛠️

### *Where Theory Meets Practice — One Lab at a Time*

![AWS](https://img.shields.io/badge/Amazon%20AWS-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Hands-On](https://img.shields.io/badge/Type-Hands--On%20Labs-3498DB?style=for-the-badge&logo=terminal&logoColor=white)
![Labs](https://img.shields.io/badge/Labs-25-2ECC71?style=for-the-badge&logo=checkmarx&logoColor=white)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy%20%E2%86%92%20Hard-E74C3C?style=for-the-badge&logo=skilled&logoColor=white)
![Free](https://img.shields.io/badge/Estimated%20Cost-%3C%2415%20Total-9B59B6?style=for-the-badge&logo=dollar-sign&logoColor=white)

---

**👋 Hey there! I'm Rithu, and this repo is your hands-on playground for AWS.**

Whether you're just getting started or sharpening your skills, you're in the right place. Let's build things! 🚀

</div>

---

## 🎉 Welcome!

Hey Ravi! 👋

So you've been grinding through the theory — understanding what EC2 is, how S3 stores objects, why VPCs exist, and maybe even daydreaming about cloud architectures over your morning coffee ☕. That's awesome! But here's the thing about AWS:

> **You don't truly *know* it until you've broken it, fixed it, and deployed it yourself.** 💪

That's exactly why this repo exists. Think of me (Rithu) as your cloud buddy who's been down this road a few times. I've clicked the wrong button, forgotten to stop instances at 2 AM (💸 *bye bye, wallet*), and spent hours debugging security groups that "should have worked." 

You're going to learn from my mistakes and have way more fun doing it. Every lab here is designed to give you **real, hands-on experience** with real AWS services — no simulations, no shortcuts. You'll be working in your own AWS account, building real things, and by the end of all 25 labs? You'll be dangerous. 😎

So roll up those sleeves, grab your keyboard, and let's go build some cloud! ☁️🔨

---

## 📚 What Is This Repository?

**AWS Hands-On Labs** is a **practical, step-by-step companion** to the theory repository:

<div align="center">

### 👉 [Learn_AWS-Amazon-Web-Services](https://github.com/agravi987/Learn_AWS-Amazon-Web-Services) 👈

</div>

That theory repo covers **26 topics (00–25)** — from the fundamentals of Cloud Computing all the way to AWS Backup — with detailed notes, diagrams, and explanations.

**This repo** is where you get your hands dirty. 🧤

Each lab corresponds to a topic in the theory repo and walks you through **real, actionable exercises** in your own AWS account. You'll create resources, configure services, test scenarios, and (most importantly) clean up after yourself!

| Repository | Purpose |
|---|---|
| [Learn_AWS-Amazon-Web-Services](https://github.com/agravi987/Learn_AWS-Amazon-Web-Services) | 📖 Theory, concepts, notes & diagrams |
| **This Repo (AWS-Hands-On-Labs)** | 🔧 Hands-on labs, exercises & practice |

**The ideal workflow:**
1. Read the topic in the theory repo 📖
2. Come here and do the lab 🔨
3. Go back to theory and understand it deeper 🧠
4. Repeat! 🔄

---

## ✅ Prerequisites

Before you dive in, make sure you've got everything set up. We've put together a dedicated checklist for you.

### 👉 [PREREQUISITES.md](./PREREQUISITES.md) — Read This First! 👈

**Quick Summary of What You'll Need:**
- [ ] An AWS Account (with Free Tier access)
- [ ] AWS CLI installed and configured
- [ ] A decent internet connection
- [ ] A terminal / command line ready to go
- [ ] Basic comfort with the command line (don't worry, we'll guide you!)
- [ ] A billing alert set up (trust us on this one 😅)

Head over to [PREREQUISITES.md](./PREREQUISITES.md) for the full breakdown with links and step-by-step setup instructions.

---

## 🗺️ The Learning Path — All 25 Labs

Here's your roadmap. Each lab is designed to build on the last, but we know life happens — feel free to skip around if you need to. Just keep an eye on the "Prerequisites" column!

> 💡 **Difficulty Guide:** 🟢 Easy = Beginner-friendly, 🟡 Medium = Some experience helps, 🔴 Hard = Bring your A-game

| # | Lab Title | Difficulty | ⏱️ Time | 💰 Cost | Primary Service | Prerequisites |
|:-:|---|:-:|:-:|:-:|---|:-:|
| 01 | EC2 — Launch and Connect | 🟢 Easy | ~30 min | < $1 | EC2 | — |
| 02 | EC2 — Security Groups Deep Dive | 🟢 Easy | ~25 min | < $1 | EC2 / VPC | Lab 01 |
| 03 | EBS — Volumes and Snapshots | 🟢 Easy | ~30 min | < $1 | EBS | Lab 01 |
| 04 | AMI — Create and Clone | 🟢 Easy | ~20 min | < $1 | EC2 / AMI | Lab 01 |
| 05 | S3 — Static Website Hosting | 🟢 Easy | ~25 min | < $1 | S3 | — |
| 06 | S3 — Versioning and Lifecycle Policies | 🟢 Easy | ~20 min | < $1 | S3 | Lab 05 |
| 07 | S3 — Cross-Region Replication | 🟡 Medium | ~40 min | < $2 | S3 | Lab 05, 06 |
| 08 | VPC — Build from Scratch | 🟡 Medium | ~45 min | < $2 | VPC | — |
| 09 | VPC — NAT Gateway and VPC Endpoints | 🟡 Medium | ~40 min | ~$3 | VPC | Lab 08 |
| 10 | ELB — Application Load Balancer | 🟡 Medium | ~40 min | < $2 | EC2 / ELB | Lab 01 |
| 11 | ASG — Auto Scaling Group | 🟡 Medium | ~35 min | < $2 | EC2 / ASG | Lab 10 |
| 12 | Route 53 — DNS and Failover | 🟡 Medium | ~35 min | < $3 | Route 53 | Lab 10 |
| 13 | RDS — MySQL on AWS | 🟡 Medium | ~35 min | < $3 | RDS | Lab 01 |
| 14 | DynamoDB — CRUD Operations | 🟢 Easy | ~25 min | < $1 | DynamoDB | — |
| 15 | CloudWatch — Alarms and Dashboards | 🟢 Easy | ~25 min | < $1 | CloudWatch | Lab 01 |
| 16 | IAM — Users, Groups, Roles, Policies | 🟢 Easy | ~30 min | **Free** | IAM | — |
| 17 | SNS and SQS — Messaging | 🟢 Easy | ~25 min | < $1 | SNS / SQS | — |
| 18 | Lambda — S3 Triggered Function | 🟡 Medium | ~35 min | < $1 | Lambda / S3 | Lab 05 |
| 19 | Lambda — API Gateway REST API | 🟡 Medium | ~40 min | < $1 | Lambda / API Gateway | Lab 18 |
| 20 | ECS — Deploy NGINX on Fargate | 🔴 Hard | ~45 min | < $2 | ECS / Fargate | Lab 16 |
| 21 | CloudFormation — Deploy EC2 | 🟡 Medium | ~30 min | < $1 | CloudFormation | Lab 01 |
| 22 | CloudTrail — Enable and Query | 🟢 Easy | ~20 min | < $1 | CloudTrail | — |
| 23 | KMS — Encrypt S3 and EBS | 🟡 Medium | ~30 min | < $1 | KMS | Lab 03, 05 |
| 24 | AWS Backup — Multi-Service Backup | 🟡 Medium | ~35 min | < $2 | AWS Backup | Lab 03, 13 |
| 25 | **Capstone — Full Stack on AWS** | 🔴 Hard | ~90 min | < $5 | Multiple | All Previous Labs |

<div align="center">

**💰 Total Estimated Cost: Less than $15** (if you clean up after each lab!)

</div>

---

## 🧭 Recommended Learning Tracks

We've organized the labs into **thematic tracks** if you want to focus on specific areas. But we recommend the full sequential path for the best learning experience!

### 🏗️ Compute Track
| Labs | Focus |
|:-:|---|
| 01 → 02 → 04 → 10 → 11 | EC2 basics, networking, AMIs, load balancing, and auto scaling |

### 📦 Storage Track
| Labs | Focus |
|:-:|---|
| 03 → 05 → 06 → 07 | EBS volumes, S3 hosting, versioning, and cross-region replication |

### 🌐 Networking Track
| Labs | Focus |
|:-:|---|
| 02 → 08 → 09 → 12 | Security groups, VPC from scratch, NAT/Endpoints, DNS & failover |

### 🗄️ Database Track
| Labs | Focus |
|:-:|---|
| 13 → 14 | Relational (RDS/MySQL) and NoSQL (DynamoDB) |

### ⚡ Serverless Track
| Labs | Focus |
|:-:|---|
| 18 → 19 | Lambda triggers and API Gateway REST APIs |

### 🔐 Security & Compliance Track
| Labs | Focus |
|:-:|---|
| 16 → 22 → 23 | IAM, CloudTrail, and KMS encryption |

### 🐳 Containers Track
| Labs | Focus |
|:-:|---|
| 20 | ECS with Fargate |

### 🏢 Enterprise & DevOps Track
| Labs | Focus |
|:-:|---|
| 17 → 21 → 24 → 25 | Messaging, CloudFormation, Backup, and the Full Stack Capstone |

---

## 📖 How to Use These Labs

### The Golden Rules

**1. Follow the order (recommended)** 📋
> Labs 01 through 25 are designed to build on each other progressively. The prerequisites column tells you which labs to complete first. Following the sequence ensures you don't miss foundational skills.

**2. But it's okay to skip ahead** 🦘
> Need to learn something specific for a project or interview? Jump to the relevant lab! Just check the prerequisites and make sure you're comfortable with the concepts.

**3. Read before you click** 👀
> Each lab has a detailed step-by-step guide. Read the *entire* lab before starting so you understand the "why" behind each step — not just the "what."

**4. Take your time** 🐢
> There's no race. If a lab takes you twice the estimated time, that's totally fine. Understanding > speed.

**5. Break things (safely)** 🔨
> Once you've completed a lab, experiment! Change settings, try different options, see what happens when you do something unexpected. That's how you really learn.

**6. Document your journey** 📝
> Keep notes on what you learned, what confused you, and what you want to explore further. Future-you will thank present-you.

### Lab Structure

Each lab follows this format:

```
Lab XX - Title/
├── README.md          # Full lab guide with step-by-step instructions
├── screenshots/       # Reference screenshots (where applicable)
└── scripts/           # Any helper scripts (where applicable)
```

---

## 💸 Cost Management Tips — READ THIS!

Look, we've all been there — you forget to terminate an EC2 instance and wake up to a $200 bill. 😱 Let's make sure that never happens to you!

### 🛡️ Protect Your Wallet

**1. Use the Free Tier** 🆓
> Most of these labs are designed to work within the AWS Free Tier. Check [aws.amazon.com/free](https://aws.amazon.com/free/) to see what's included. Labs marked "Free" cost literally nothing.

**2. Set Up a Billing Alert** 🚨
> This is **non-negotiable**. Go to **AWS Billing Console → Budgets → Create a Budget** and set an alert at $1, $5, and $10. You'll get an email the moment your spending crosses the threshold.
>
> *Rithu learned this the hard way. Don't be like early Rithu.* 😅

**3. CLEAN UP AFTER EVERY LAB** 🧹
> This is the **#1 rule** of hands-on labs. When you're done, **terminate instances, delete buckets, remove snapshots, delete RDS instances, and tear down VPCs.** We include a cleanup section at the end of every single lab — **use it**.

**4. Check the AWS Billing Dashboard** 📊
> Periodically visit the [AWS Billing Dashboard](https://console.aws.amazon.com/billing/) to keep an eye on your charges. Don't just trust the alerts — verify!

**5. Use the Free Tier Only Dashboard** 📈
> The Free Tier page in your AWS console shows exactly how much of each free tier resource you've consumed. Check it regularly.

**6. Stop, Don't Just Terminate** ⏹️
> If you think you might continue a lab later, **stop** your EC2 instances instead of terminating them. Stopped instances don't incur compute charges (only minimal storage costs for EBS).

### 💡 Cost Estimates in This Repo

| Symbol | Meaning |
|:-:|---|
| < $1 | Pennies — barely registers on your bill |
| < $2 | A couple of pennies — still negligible |
| ~$3 | A few cents — totally fine for learning |
| < $5 | Still under a fancy coffee ☕ |
| **Free** | $0.00 — the best price! |

> ⚠️ **Disclaimer:** Cost estimates assume resources are cleaned up promptly. Leaving an EC2 t3.micro running for 24 hours costs about $0.10 — not bad for one instance, but they add up if you forget multiple! Always clean up!

---

## 🧹 CLEANUP REMINDER

<div align="center">

### ⚠️ **ALWAYS CLEAN UP YOUR RESOURCES AFTER EACH LAB!** ⚠️

**Every lab ends with a cleanup section. FOLLOW IT.**

Terminate EC2 instances, delete S3 buckets, remove RDS instances, release Elastic IPs,
delete VPCs, remove security groups, delete snapshots — **everything must go.**

**Your wallet will thank you.** 💳✨

*When in doubt, check the AWS Billing Dashboard. If you see unexpected charges,
terminate everything and submit a billing support ticket.*

</div>

---

## 🤝 Contributing

Found a bug in a lab? Have a suggestion for improvement? Want to add your own lab? We welcome contributions!

1. Fork this repository
2. Create a feature branch (`git checkout -b lab/improvement-name`)
3. Make your changes
4. Submit a Pull Request with a clear description

---

## 📬 Support

If you get stuck on a lab:

1. 📖 Re-read the lab instructions carefully
2. 🔍 Check the AWS documentation linked in each lab
3. 💬 Open a GitHub Issue with details about where you're stuck
4. 🌐 Ask in AWS communities — someone has definitely hit the same issue before!

---

## 🏆 Milestones

Track your progress and celebrate your wins!

- [ ] 🎯 Lab 01 — First EC2 instance launched! Welcome to the cloud!
- [ ] 🔒 Lab 02 — Security Groups mastered. You speak firewall now.
- [ ] 💾 Lab 03 — EBS & Snapshots. Your data is safe(ish).
- [ ] 🖼️ Lab 04 — AMIs created. Cloning machines like a pro.
- [ ] 🌐 Lab 05 — First website hosted on S3. You're a web host now!
- [ ] 🔄 Lab 06 — Versioning & Lifecycle. Your data manages itself.
- [ ] 🌍 Lab 07 — Cross-region replication. Thinking globally.
- [ ] 🏗️ Lab 08 — VPC built from scratch. Network architect vibes.
- [ ] 🚪 Lab 09 — NAT Gateways & Endpoints. Private networking done right.
- [ ] ⚖️ Lab 10 — Load balancer deployed. Traffic? Handled.
- [ ] 📈 Lab 11 — Auto Scaling. Your infra scales while you sleep.
- [ ] 🧭 Lab 12 — Route 53 DNS & Failover. You're a DNS wizard now.
- [ ] 🗄️ Lab 13 — RDS MySQL running. Databases in the cloud!
- [ ] ⚡ Lab 14 — DynamoDB CRUD. NoSQL champion.
- [ ] 👁️ Lab 15 — CloudWatch monitoring. You can see everything.
- [ ] 🔐 Lab 16 — IAM mastered. Security is no joke.
- [ ] 📨 Lab 17 — SNS & SQS. Messaging like a pro.
- [ ] ⚡ Lab 18 — Lambda + S3. Serverless is magic.
- [ ] 🌐 Lab 19 — API Gateway + Lambda. Building APIs? Easy.
- [ ] 🐳 Lab 20 — ECS Fargate. Containers without the headache.
- [ ] 📝 Lab 21 — CloudFormation. Infrastructure as Code unlocked.
- [ ] 📋 Lab 22 — CloudTrail. You see all. You know all.
- [ ] 🔑 Lab 23 — KMS encryption. Things are secure now.
- [ ] 💾 Lab 24 — AWS Backup. Everything is protected.
- [ ] 🏆 **Lab 25 — CAPSTONE COMPLETE! YOU ARE AN AWS BUILDER!** 🎊🎉

---

<div align="center">

### 🚀 Ready? Start with [Lab 01 — EC2 Launch and Connect](./Lab%2001%20-%20EC2%20Launch%20and%20Connect/)!

---

*Built with ☁️ and ❤️ by Rithu & Ravi*

*"The cloud is not magic. It's just someone else's computer. Now it's YOUR computer."* 🖥️

![Made with AWS](https://img.shields.io/badge/Made%20with-%E2%9D%A4%20for%20AWS-FF9900?style=for-the-badge)

</div>
