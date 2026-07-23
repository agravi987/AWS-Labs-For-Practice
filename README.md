<div align="center">

<img src="https://img.shields.io/badge/AWS%20Hands-On%20Labs-FF9900?style=for-the-badge&labelColor=232F3E" />

# AWS Hands-On Labs

### Where Theory Meets Practice - One Lab at a Time

<img src="https://img.shields.io/badge/Amazon%20AWS-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white" />
<img src="https://img.shields.io/badge/Type-Hands--On%20Labs-3498DB?style=for-the-badge&logo=terminal&logoColor=white" />
<img src="https://img.shields.io/badge/Labs-25-2ECC71?style=for-the-badge&logo=checkmarx&logoColor=white" />
<img src="https://img.shields.io/badge/Difficulty-Easy%20to%20Hard-E74C3C?style=for-the-badge&logo=skilled&logoColor=white" />
<img src="https://img.shields.io/badge/Cost-Less%20Than%20%2415%20Total-9B59B6?style=for-the-badge&logo=dollar-sign&logoColor=white" />

---

**Hey there! I'm Rithu, and this repo is your hands-on playground for AWS.**

Whether you're just getting started or sharpening your skills, you're in the right place. Let's build things!

</div>

---

## Welcome!

**Hey Ravi!** So you've been grinding through the theory - understanding what EC2 is, how S3 stores objects, why VPCs exist, and maybe even daydreaming about cloud architectures over your morning coffee. That's awesome! But here's the thing about AWS:

> **You don't truly know it until you've broken it, fixed it, and deployed it yourself.**

That's exactly why this repo exists. Think of me (Rithu) as your cloud buddy who's been down this road a few times. I've clicked the wrong button, forgotten to stop instances at 2 AM, and spent hours debugging security groups that "should have worked."

You're going to learn from my mistakes and have way more fun doing it. Every lab here is designed to give you **real, hands-on experience** with real AWS services - no simulations, no shortcuts. You'll be working in your own AWS account, building real things, and by the end of all 25 labs? **You'll be dangerous.**

> *So roll up those sleeves, grab your keyboard, and let's go build some cloud!*

---

## What Is This Repository?

**AWS Hands-On Labs** is a **practical, step-by-step companion** to the theory repository:

<div align="center">

### [Learn_AWS-Amazon-Web-Services](https://github.com/agravi987/Learn_AWS-Amazon-Web-Services)

</div>

| | Repository | Purpose |
|---|---|---|
| [Learn_AWS](https://github.com/agravi987/Learn_AWS-Amazon-Web-Services) | Theory, concepts, notes & diagrams |
| **This Repo (Hands-On Labs)** | Hands-on labs, exercises & practice |

**The ideal workflow:**

```
  Read Theory  -->  Do the Lab  -->  Go Deeper  -->  Repeat!
```

---

## Prerequisites

Before you dive in, make sure you've got everything set up.

### [PREREQUISITES.md](./PREREQUISITES.md) - Read This First!

**Quick Checklist:**
- [ ] AWS Account (with Free Tier access)
- [ ] AWS CLI installed and configured
- [ ] Terminal / command line ready
- [ ] Basic command line comfort
- [ ] Billing alert set up (trust us)

Head over to [PREREQUISITES.md](./PREREQUISITES.md) for the full breakdown with links and step-by-step setup instructions.

---

## The Learning Path - All 25 Labs

Here's your roadmap. Each lab builds on the last, but feel free to skip around!

**Difficulty Guide:** Green = Beginner-friendly | Yellow = Some experience helps | Red = Bring your A-game

| # | Lab Title | Difficulty | Time | Cost | Primary Service | Prerequisites |
|:-:|---|:-:|:-:|:-:|---|:-:|
| 01 | EC2 - Launch and Connect | Green | ~30 min | < $1 | EC2 | - |
| 02 | EC2 - Security Groups Deep Dive | Green | ~25 min | < $1 | EC2 / VPC | Lab 01 |
| 03 | EBS - Volumes and Snapshots | Green | ~30 min | < $1 | EBS | Lab 01 |
| 04 | AMI - Create and Clone | Green | ~20 min | < $1 | EC2 / AMI | Lab 01 |
| 05 | S3 - Static Website Hosting | Green | ~25 min | < $1 | S3 | - |
| 06 | S3 - Versioning and Lifecycle Policies | Green | ~20 min | < $1 | S3 | Lab 05 |
| 07 | S3 - Cross-Region Replication | Yellow | ~40 min | < $2 | S3 | Lab 05, 06 |
| 08 | VPC - Build from Scratch | Yellow | ~45 min | < $2 | VPC | - |
| 09 | VPC - NAT Gateway and VPC Endpoints | Yellow | ~40 min | ~$3 | VPC | Lab 08 |
| 10 | ELB - Application Load Balancer | Yellow | ~40 min | < $2 | EC2 / ELB | Lab 01 |
| 11 | ASG - Auto Scaling Group | Yellow | ~35 min | < $2 | EC2 / ASG | Lab 10 |
| 12 | Route 53 - DNS and Failover | Yellow | ~35 min | < $3 | Route 53 | Lab 10 |
| 13 | RDS - MySQL on AWS | Yellow | ~35 min | < $3 | RDS | Lab 01 |
| 14 | DynamoDB - CRUD Operations | Green | ~25 min | < $1 | DynamoDB | - |
| 15 | CloudWatch - Alarms and Dashboards | Green | ~25 min | < $1 | CloudWatch | Lab 01 |
| 16 | IAM - Users, Groups, Roles, Policies | Green | ~30 min | **Free** | IAM | - |
| 17 | SNS and SQS - Messaging | Green | ~25 min | < $1 | SNS / SQS | - |
| 18 | Lambda - S3 Triggered Function | Yellow | ~35 min | < $1 | Lambda / S3 | Lab 05 |
| 19 | Lambda - API Gateway REST API | Yellow | ~40 min | < $1 | Lambda / API GW | Lab 18 |
| 20 | ECS - Deploy NGINX on Fargate | Red | ~45 min | < $2 | ECS / Fargate | Lab 16 |
| 21 | CloudFormation - Deploy EC2 | Yellow | ~30 min | < $1 | CloudFormation | Lab 01 |
| 22 | CloudTrail - Enable and Query | Green | ~20 min | < $1 | CloudTrail | - |
| 23 | KMS - Encrypt S3 and EBS | Yellow | ~30 min | < $1 | KMS | Lab 03, 05 |
| 24 | AWS Backup - Multi-Service Backup | Yellow | ~35 min | < $2 | AWS Backup | Lab 03, 13 |
| 25 | **Capstone - Full Stack on AWS** | Red | ~90 min | < $5 | Multiple | All Labs |

<div align="center">

**Total Estimated Cost: Less than $15** (if you clean up after each lab!)

</div>

---

## Recommended Learning Tracks

Organized by theme - but we recommend the full sequential path!

| Track | Labs | Focus |
|---|---|---|
| Compute | 01 -> 02 -> 04 -> 10 -> 11 | EC2 basics, networking, AMIs, load balancing, auto scaling |
| Storage | 03 -> 05 -> 06 -> 07 | EBS volumes, S3 hosting, versioning, cross-region replication |
| Networking | 02 -> 08 -> 09 -> 12 | Security groups, VPC from scratch, NAT/Endpoints, DNS & failover |
| Database | 13 -> 14 | Relational (RDS/MySQL) and NoSQL (DynamoDB) |
| Serverless | 18 -> 19 | Lambda triggers and API Gateway REST APIs |
| Security & Compliance | 16 -> 22 -> 23 | IAM, CloudTrail, and KMS encryption |
| Containers | 20 | ECS with Fargate |
| Enterprise & DevOps | 17 -> 21 -> 24 -> 25 | Messaging, CloudFormation, Backup, and the Capstone |

---

## How to Use These Labs

### The Golden Rules

| # | Rule | Details |
|:-:|---|---|
| 1 | **Follow the order** | Labs 01-25 build progressively. The prerequisites column tells you which labs to complete first. |
| 2 | **Skip ahead if needed** | Need something specific? Jump to the relevant lab! Just check prerequisites first. |
| 3 | **Read before you click** | Read the ENTIRE lab before starting - understand the "why" behind each step. |
| 4 | **Take your time** | No race. If a lab takes twice the estimated time, that's fine. Understanding > speed. |
| 5 | **Break things (safely)** | After completing a lab, experiment! That's how you really learn. |
| 6 | **Document your journey** | Keep notes on what you learned and what confused you. Future-you will thank present-you. |

### Lab Structure

```
Lab XX - Title/
README.md          # Full lab guide with step-by-step instructions
screenshots/       # Reference screenshots (where applicable)
scripts/           # Helper scripts (where applicable)
```

---

## Cost Management Tips - READ THIS!

> **WARNING: Protect Your Wallet!**

Look, we've all been there - you forget to terminate an EC2 instance and wake up to a $200 bill. Let's make sure that never happens to you!

### Protect Your Wallet

| # | Rule | Details |
|:-:|---|---|
| 1 | **Use the Free Tier** | Most labs work within Free Tier. Check [aws.amazon.com/free](https://aws.amazon.com/free/) |
| 2 | **Set Up Billing Alerts** | **Non-negotiable.** Budgets -> $1, $5, $10 alerts. Don't be like early Rithu. |
| 3 | **CLEAN UP AFTER EVERY LAB** | Terminate instances, delete buckets, remove snapshots - **everything must go** |
| 4 | **Check Billing Dashboard** | Visit [AWS Billing Dashboard](https://console.aws.amazon.com/billing/) periodically |
| 5 | **Use Free Tier Dashboard** | Shows exactly how much free tier resources you've consumed |
| 6 | **Stop, Don't Just Terminate** | If continuing later, **stop** instances (no compute charges, only minimal storage) |

### Cost Estimates

| Symbol | Meaning |
|:-:|---|
| < $1 | Pennies - barely registers on your bill |
| < $2 | A couple of pennies - still negligible |
| ~$3 | A few cents - totally fine for learning |
| < $5 | Still under a fancy coffee |
| **Free** | $0.00 - the best price! |

> **Disclaimer:** Estimates assume prompt cleanup. Leaving a t3.micro running 24h costs ~$0.10 - not bad for one, but they add up!

---

## CLEANUP REMINDER

<div align="center">

### ALWAYS CLEAN UP YOUR RESOURCES AFTER EACH LAB!

Every lab ends with a cleanup section. **FOLLOW IT.**

Terminate EC2 instances, delete S3 buckets, remove RDS instances, release Elastic IPs,
delete VPCs, remove security groups, delete snapshots - **everything must go.**

**Your wallet will thank you.**

</div>

---

## Contributing

Found a bug? Have a suggestion? Want to add your own lab? We welcome contributions!

1. Fork this repository
2. Create a feature branch (`git checkout -b lab/improvement-name`)
3. Make your changes
4. Submit a Pull Request with a clear description

---

## Support

If you get stuck on a lab:

1. Re-read the lab instructions carefully
2. Check the AWS documentation linked in each lab
3. Open a GitHub Issue with details about where you're stuck
4. Ask in AWS communities - someone has definitely hit the same issue before!

---

## Milestones

Track your progress and celebrate your wins!

- [ ] Lab 01 - First EC2 instance launched! Welcome to the cloud!
- [ ] Lab 02 - Security Groups mastered. You speak firewall now.
- [ ] Lab 03 - EBS & Snapshots. Your data is safe(ish).
- [ ] Lab 04 - AMIs created. Cloning machines like a pro.
- [ ] Lab 05 - First website hosted on S3. You're a web host now!
- [ ] Lab 06 - Versioning & Lifecycle. Your data manages itself.
- [ ] Lab 07 - Cross-region replication. Thinking globally.
- [ ] Lab 08 - VPC built from scratch. Network architect vibes.
- [ ] Lab 09 - NAT Gateways & Endpoints. Private networking done right.
- [ ] Lab 10 - Load balancer deployed. Traffic? Handled.
- [ ] Lab 11 - Auto Scaling. Your infra scales while you sleep.
- [ ] Lab 12 - Route 53 DNS & Failover. You're a DNS wizard now.
- [ ] Lab 13 - RDS MySQL running. Databases in the cloud!
- [ ] Lab 14 - DynamoDB CRUD. NoSQL champion.
- [ ] Lab 15 - CloudWatch monitoring. You can see everything.
- [ ] Lab 16 - IAM mastered. Security is no joke.
- [ ] Lab 17 - SNS & SQS. Messaging like a pro.
- [ ] Lab 18 - Lambda + S3. Serverless is magic.
- [ ] Lab 19 - API Gateway + Lambda. Building APIs? Easy.
- [ ] Lab 20 - ECS Fargate. Containers without the headache.
- [ ] Lab 21 - CloudFormation. Infrastructure as Code unlocked.
- [ ] Lab 22 - CloudTrail. You see all. You know all.
- [ ] Lab 23 - KMS encryption. Things are secure now.
- [ ] Lab 24 - AWS Backup. Everything is protected.
- [ ] **Lab 25 - CAPSTONE COMPLETE! YOU ARE AN AWS BUILDER!**

---

<div align="center">

### Ready? Start with [Lab 01 - EC2 Launch and Connect](./01%20-%20EC2%20-%20Launch%20and%20Connect/)!

---

*Built with cloud and love by Rithu & Ravi*

*"The cloud is not magic. It's just someone else's computer. Now it's YOUR computer."*

</div>
