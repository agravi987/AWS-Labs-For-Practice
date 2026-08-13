---
name: aws-labs-practice
description: Guide for working in this AWS hands-on lab repo (25 lab folders under AWS-Labs-For-Practice). Use whenever answering questions about or modifying this repo's lab guides, LAB.md files, READMEs, or AWS console/CLI steps. Use to keep answers aligned with current AWS documentation instead of stale model training data.
---

# AWS Labs Practice Repo

This repository is a hands-on AWS practice collection. Every folder is one lab, named `NN - <Topic> - <Title>`, containing:

- `README.md` — overview of the lab
- `LAB.md` — the step-by-step guide with Milestones, Objectives, Instructions, and Verification checks
- `screenshots/` — reference screenshots

## Lab map (by folder)

| Folder | Topic | Key services |
|---|---|---|
| 01 | EC2 - Launch and Connect | EC2, SSH key pairs, Amazon Linux 2023 |
| 02 | EC2 - Security Groups Deep Dive | Security groups, inbound/outbound rules |
| 03 | EBS - Volumes and Snapshots | EBS, snapshots, volumes |
| 04 | AMI - Create and Clone | AMIs, launch templates |
| 05 | S3 - Static Website Hosting | S3, static website hosting, bucket policies |
| 06 | S3 - Versioning and Lifecycle Policies | S3 versioning, lifecycle rules |
| 07 | S3 - Cross-Region Replication | S3 replication, versioning prerequisite |
| 08 | VPC - Build from Scratch | VPC, subnets, route tables, IGW |
| 09 | VPC - NAT Gateway and VPC Endpoints | NAT gateway, VPC endpoints, private subnets |
| 10 | ELB - Application Load Balancer | ALB, target groups, listeners |
| 11 | ASG - Auto Scaling Group | ASG, scaling policies, launch templates |
| 12 | Route 53 - DNS and Failover | Route 53, hosted zones, failover routing |
| 13 | RDS - MySQL on AWS | RDS MySQL, security groups, connection |
| 14 | DynamoDB - CRUD Operations | DynamoDB, tables, console/CLI CRUD |
| 15 | CloudWatch - Alarms and Dashboards | CloudWatch metrics, alarms, dashboards |
| 16 | IAM - Users, Groups, Roles, Policies | IAM identities, policies, MFA |
| 17 | SNS and SQS - Messaging | SNS topics, SQS queues, subscriptions |
| 18 | Lambda - S3 Triggered Function | Lambda, S3 event triggers, IAM roles |
| 19 | Lambda - API Gateway REST API | Lambda, API Gateway, REST endpoints |
| 20 | ECS - Deploy NGINX on Fargate | ECS, Fargate, ECR, task definitions |
| 21 | CloudFormation - Deploy EC2 | CloudFormation, stacks, templates |
| 22 | CloudTrail - Enable and Query | CloudTrail, trails, event history |
| 23 | KMS - Encrypt S3 and EBS | KMS keys, encryption for S3/EBS |
| 24 | AWS Backup - Multi-Service Backup | AWS Backup, backup plans, vaults |
| 25 | Capstone - Full Stack on AWS | Combination of prior labs end-to-end |

## Guidance for answers

AWS console UI and CLI behaviors change over time, and the model's training data goes stale. When working in this repo:

1. **Verify against current docs first.** Use the `aws-docs` MCP server (`search_documentation`, `read_documentation`) or fetch `https://docs.aws.amazon.com/...` pages before giving console paths, CLI commands, or service behavior. Cite the source.
2. **Use the AWS Agent Toolkit skills** in this repo's `.opencode/skills/` when they apply (e.g., `aws-iam`, `aws-networking`, `aws-compute`, `launching-ec2-instance-with-best-practices`).
3. **Keep lab instructions faithful.** When asked to update or write `LAB.md` content, preserve the existing Milestone/Objective/Instructions/Verification structure and only change what the user asks.
4. **Match the lab context.** Labs are written for the AWS Management Console (with CLI alternates where sensible), use the Free Tier-friendly resources mentioned (e.g., `t2.micro`, Amazon Linux 2023), and avoid inventing steps that add cost.
5. **Call out version drift.** If a current console flow differs from what a `LAB.md` describes, say so explicitly and propose the corrected path rather than silently rewriting.

## Troubleshooting triggers

- If a step references a console option that no longer exists, verify against current EC2/S3/VPC/IAM docs and suggest the updated equivalent.
- For connection issues (SSH, PuTTY `.ppk` conversion, security group rules), reference the EC2 user guide before answering.
