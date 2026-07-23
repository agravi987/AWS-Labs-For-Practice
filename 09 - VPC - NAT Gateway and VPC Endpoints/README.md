<div align="center">

<img src="https://img.shields.io/badge/Lab%2009-NAT%20Gateway%20%26%20VPC%20Endpoints-1ABC9C?style=for-the-badge&labelColor=232F3E" />

</div>

<div align="center">

<img src="https://img.shields.io/badge/Difficulty-Medium-F4D03F?style=flat-square" />
<img src="https://img.shields.io/badge/Time-~40min-3498DB?style=flat-square" />
<img src="https://img.shields.io/badge/Cost-~3%20USD-E74C3C?style=flat-square" />
<img src="https://img.shields.io/badge/Service-VPC%20%7C%20EC2-8E44AD?style=flat-square" />

</div>

> "Private subnets keep your instances hidden from the internet. NAT Gateways let them talk OUT without letting anyone talk IN. VPC Endpoints let them talk to AWS services without going through the internet at all!" — Rithu 🔒

---

## 🎯 Objective

In this lab, you'll extend your VPC from Lab 08 by adding a **private subnet**, a **NAT Gateway** (so private instances can access the internet), and a **VPC Endpoint** (so private instances can access S3 directly through the AWS network). This is how production networks are designed!

---

## 🧠 Prerequisites

- [x] Completed [Lab 08 — VPC: Build from Scratch](../08%20-%20VPC%20-%20Build%20from%20Scratch/README.md)
- [x] AWS account with console access
- [x] Familiarity with VPC, subnets, route tables, and security groups

---

## 💰 Cost Warning

> ⚠️ **NAT Gateway costs approximately $0.045/hour (~$32/month) plus data processing charges.** This is the most expensive resource in this lab. **DELETE THE NAT GATEWAY IMMEDIATELY after the lab!** We're only using it for a few minutes, so the actual cost will be pennies. But forget to delete it, and you'll get a surprise bill at the end of the month. **Do NOT skip cleanup!**

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│  VPC: ravi-custom-vpc (10.0.0.0/16)                              │
│                                                                  │
│  ┌──────────────────────────────┐  ┌──────────────────────────┐  │
│  │  PUBLIC SUBNET               │  │  PRIVATE SUBNET          │  │
│  │  10.0.1.0/24 | us-east-1a   │  │  10.0.2.0/24 | us-east-1a│  │
│  │                              │  │                          │  │
│  │  ┌───────────────────┐       │  │  ┌────────────────────┐  │  │
│  │  │ Bastion / Public   │       │  │  │ Private EC2        │  │  │
│  │  │ EC2                │       │  │  │ (no public IP)     │  │  │
│  │  └───────────────────┘       │  │  └─────────┬──────────┘  │  │
│  │                              │  │            │              │  │
│  │  ┌───────────────────┐       │  │  ┌─────────▼──────────┐  │  │
│  │  │ NAT Gateway        │◄─────┼──┼──│ Private RT          │  │  │
│  │  │ (Elastic IP)       │       │  │ │ 0.0.0.0/0 → NAT GW │  │  │
│  │  └───────────────────┘       │  │ │ 0.0.0.0/0 → S3 EP   │  │  │
│  │                              │  │ └────────────────────┘  │  │
│  │  ┌───────────────────┐       │  │                          │  │
│  │  │ Internet Gateway   │       │  │  ┌────────────────────┐  │  │
│  │  │ (ravi-igw)         │       │  │  │ S3 VPC Endpoint    │  │  │
│  │  └───────────────────┘       │  │  │ (com.amazonaws...s3)│  │  │
│  └──────────────────────────────┘  │  └────────────────────┘  │  │
│                                    └──────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Step-by-Step Instructions

### Step 0: Recreate or Reuse the VPC from Lab 08

> <img src="https://img.shields.io/badge/Step%200-Recreate%20or%20Reuse%20VPC-3498DB?style=for-the-badge" />

If you still have `ravi-custom-vpc` from Lab 08, use it! If you deleted it, recreate it quickly:

1. Go to **VPC** → **Create VPC**
2. **VPC only:**
   - Name: `ravi-custom-vpc`
   - IPv4 CIDR: `10.0.0.0/16`
3. Create the public subnet:
   - Name: `ravi-public-subnet-1a`
   - AZ: us-east-1a
   - CIDR: `10.0.1.0/24`
4. Create and attach Internet Gateway `ravi-igw`
5. Create route table `ravi-public-rt` with `0.0.0.0/0 → ravi-igw`
6. Associate with `ravi-public-subnet-1a`
7. Enable auto-assign public IP on the public subnet

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> This is the same setup from Lab 08. If you're recreating it, you can do it in about 5 minutes. If you still have it, skip to Step 1!

---

### Step 1: Create a Private Subnet

> <img src="https://img.shields.io/badge/Step%201-Create%20Private%20Subnet-2ECC71?style=for-the-badge" />

A private subnet has NO direct internet access — instances here can't be reached from the internet.

1. Go to **VPC** → **Subnets**
2. Click **Create subnet**
3. Configure:
   - **VPC:** `ravi-custom-vpc`
   - **Subnet name:** `ravi-private-subnet-1a`
   - **Availability Zone:** `us-east-1a`
   - **IPv4 CIDR block:** `10.0.2.0/24`

> 📸 [Screenshot: Private subnet configuration]

4. Click **Create subnet**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Why `10.0.2.0/24`? Because our VPC is `10.0.0.0/16` (65,536 IPs). The public subnet uses `10.0.1.0/24` (256 IPs). The private subnet gets `10.0.2.0/24` (another 256 IPs). Each subnet must have a non-overlapping CIDR range!

5. **Important:** Make sure auto-assign public IP is **DISABLED** on this subnet:
   - Click on `ravi-private-subnet-1a`
   - Click **Actions** → **Edit subnet settings**
   - Make sure **Enable auto-assign public IPv4 address** is UNCHECKED
   - Click **Save**

> 📸 [Screenshot: Auto-assign public IP DISABLED on private subnet]

---

### Step 2: Create the NAT Gateway

> <img src="https://img.shields.io/badge/Step%202-Create%20NAT%20Gateway-E74C3C?style=for-the-badge" />

The NAT Gateway lets instances in the private subnet access the internet (for updates, API calls, etc.) without being directly reachable from the internet.

1. Go to **VPC** → **NAT Gateways** (left sidebar)
2. Click **Create NAT Gateway**

#### Allocate an Elastic IP:

3. Click **Allocate Elastic IP** (or **Allocate new EIP**)
4. This assigns a static public IP address to the NAT Gateway

> 📸 [Screenshot: Allocate Elastic IP button]

#### Configure the NAT Gateway:

5. **Name:** `ravi-nat-gw`
6. **Subnet:** Select `ravi-public-subnet-1a`
   - ⚠️ **Important:** NAT Gateways MUST be in a PUBLIC subnet (one with an IGW route)!
7. **Connectivity type:** Public
8. **Elastic IP allocation ID:** Should be auto-filled from step 4
9. Click **Create NAT Gateway**

> 📸 [Screenshot: NAT Gateway creation form]

10. Wait for the NAT Gateway state to change to **Available** (this takes about 2-3 minutes)

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Why in the public subnet? Because the NAT Gateway needs to forward traffic to the internet via the Internet Gateway. It can only do that if it's in a subnet that has a route to the IGW. Think of the NAT Gateway as a receptionist — it sits in the lobby (public subnet) and relays messages to/from the internet on behalf of the private offices (private subnet)!

---

### Step 3: Create a Private Route Table

> <img src="https://img.shields.io/badge/Step%203-Create%20Private%20Route%20Table-F39C12?style=for-the-badge" />

We need a route table for the private subnet that routes internet traffic through the NAT Gateway.

1. Go to **VPC** → **Route Tables**
2. Click **Create route table**
3. Configure:
   - **Name:** `ravi-private-rt`
   - **VPC:** `ravi-custom-vpc`
4. Click **Create route table**

> 📸 [Screenshot: Private route table created]

5. Add a route to the internet via NAT:
   - Click on `ravi-private-rt`
   - **Routes** tab → **Edit routes**
   - **Add route:**
     - **Destination:** `0.0.0.0/0`
     - **Target:** Select **NAT Gateway** → choose `ravi-nat-gw`
   - Click **Save changes**

> 📸 [Screenshot: Route: 0.0.0.0/0 → ravi-nat-gw]

6. Associate with the private subnet:
   - **Subnet associations** tab → **Edit subnet associations**
   - Check `ravi-private-subnet-1a`
   - Click **Save**

> 📸 [Screenshot: Private subnet associated with private route table]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Notice the private route table has `0.0.0.0/0 → ravi-nat-gw` instead of `→ ravi-igw`. This means traffic goes through the NAT Gateway (outbound only) instead of directly to the internet (bidirectional). That's what makes the subnet "private"!

---

### Step 4: Create a Security Group for the Private EC2

> <img src="https://img.shields.io/badge/Step%204-Create%20Security%20Group-9B59B6?style=for-the-badge" />

We need a security group that allows SSH access from the public subnet.

1. Go to **VPC** → **Security Groups**
2. Click **Create security group**
3. Configure:
   - **Name:** `ravi-private-sg`
   - **Description:** `Private EC2 security group`
   - **VPC:** `ravi-custom-vpc`
4. **Inbound rules:**
   - Click **Add rule**
   - **Type:** SSH
   - **Port range:** 22
   - **Source:** Select **Custom** → type `10.0.1.0/24` (the public subnet CIDR)
   - 📸 [Screenshot: SSH rule from public subnet CIDR 10.0.1.0/24]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> We're allowing SSH only from the public subnet (10.0.1.0/24). This means you can SSH to the private EC2 from the public EC2 (bastion host) but NOT directly from the internet. This is a common security pattern!

5. **Outbound rules:** Allow all traffic (default)
6. Click **Create security group**

---

### Step 5: Launch EC2 in the Private Subnet

> <img src="https://img.shields.io/badge/Step%205-Launch%20Private%20EC2-1ABC9C?style=for-the-badge" />

1. Go to **EC2** → **Launch instance**
2. Configure:
   - **Name:** `private-ec2-test`
   - **AMI:** Amazon Linux 2023
   - **Instance type:** t2.micro
   - **Key pair:** Select your existing key pair
3. **Network settings** → **Edit:**
   - **VPC:** `ravi-custom-vpc`
   - **Subnet:** `ravi-private-subnet-1a`
   - **Auto-assign public IP:** Disabled
   - **Security group:** Select existing → `ravi-private-sg`
4. Click **Launch instance**
5. Wait for it to be **Running**

> 📸 [Screenshot: Private EC2 instance running — note NO public IP!]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Notice that the private EC2 has NO public IP address. You can't SSH to it directly from your laptop. That's by design! We'll access it through the public EC2 (bastion host).

---

### Step 6: Launch a Public EC2 (Bastion Host)

> <img src="https://img.shields.io/badge/Step%206-Launch%20Bastion%20Host-E67E22?style=for-the-badge" />

We need a "jump box" or "bastion host" to SSH into the private instance.

1. Go to **EC2** → **Launch instance**
2. Configure:
   - **Name:** `bastion-ec2`
   - **AMI:** Amazon Linux 2023
   - **Instance type:** t2.micro
   - **Key pair:** Select your existing key pair
3. **Network settings** → **Edit:**
   - **VPC:** `ravi-custom-vpc`
   - **Subnet:** `ravi-public-subnet-1a`
   - **Auto-assign public IP:** Enabled
   - **Security group:** Select existing → `ravi-vpc-sg` (from Lab 08 — allows SSH from My IP)
4. Click **Launch instance**
5. Wait for it to be **Running** and get a public IP

> 📸 [Screenshot: Bastion EC2 with public IP in public subnet]

---

### Step 7: SSH into Private EC2 via Bastion

> <img src="https://img.shields.io/badge/Step%207-SSH%20via%20Bastion-3498DB?style=for-the-badge" />

1. Copy the **public IP** of `bastion-ec2`
2. SSH into the bastion:

```bash
ssh -i "your-key.pem" ec2-user@<BASTION_PUBLIC_IP>
```

3. From the bastion, SSH into the private EC2:

```bash
ssh -i "your-key.pem" ec2-user@<PRIVATE_EC2_PRIVATE_IP>
```

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> You need to copy your `.pem` key file to the bastion first, or use SSH agent forwarding:
> ```bash
> ssh -A -i "your-key.pem" ec2-user@<BASTION_IP>
> ```
> The `-A` flag forwards your SSH agent so the bastion can use your key to connect to the private instance!

---

### Step 8: Verify Internet Access via NAT Gateway

> <img src="https://img.shields.io/badge/Step%208-Verify%20Internet%20Access-2ECC71?style=for-the-badge" />

On the PRIVATE EC2 (via bastion):

```bash
curl http://checkip.amazonaws.com
```

**Important:** The IP returned should be the **NAT Gateway's Elastic IP** — NOT the instance's IP (because the instance has no public IP!).

You can verify: go to the NAT Gateway in the VPC console and check its Elastic IP — it should match!

```bash
ping -c 4 8.8.8.8
```

This should work! The private EC2 can reach the internet through the NAT Gateway.

> 📸 [Screenshot: Private EC2 showing NAT Gateway's IP via curl]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> This is the magic of NAT Gateways! The private EC2 has no public IP, yet it can access the internet. The NAT Gateway acts as a proxy — it forwards outbound traffic and routes the responses back. But nobody from the internet can initiate connections TO the private EC2. It's a one-way door!

---

### Step 9: Create a VPC Endpoint for S3

> <img src="https://img.shields.io/badge/Step%209-Create%20VPC%20Endpoint-E74C3C?style=for-the-badge" />

VPC Endpoints let your private instances access AWS services (like S3) through the AWS private network — without going through the internet or the NAT Gateway. This is faster, more secure, and FREE!

1. Go to **VPC** → **Endpoints** (left sidebar)
2. Click **Create endpoint**
3. Configure:
   - **Name:** `ravi-s3-endpoint`
   - **Service:** Search for `s3` and select `com.amazonaws.us-east-1.s3` (Type: Gateway)
     - ⚠️ Make sure you pick the **Gateway** type, not Interface
   - **VPC:** `ravi-custom-vpc`
   - **Route tables:** Check `ravi-private-rt`
   - **Policy:** Full Access (or leave as default)
4. Click **Create endpoint**

> 📸 [Screenshot: VPC Endpoint creation with S3 gateway endpoint]

5. Wait for the endpoint to become **Available**

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Why use a VPC Endpoint for S3?
> - **FREE** — VPC Endpoints for S3 don't cost anything (Gateway type)
> - **Faster** — Traffic stays within the AWS network
> - **More secure** — No internet exposure
> - **No NAT Gateway charges** — S3 traffic doesn't go through the NAT, saving you data processing fees

---

### Step 10: Verify S3 Access via VPC Endpoint

> <img src="https://img.shields.io/badge/Step%2010-Verify%20S3%20Access-F39C12?style=for-the-badge" />

On the PRIVATE EC2 (via bastion), install the AWS CLI if not already present:

```bash
# Install AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Then test S3 access:

```bash
aws s3 ls
```

This should list your S3 buckets (or return empty if you have none). The important thing is that it WORKS — and it's going through the VPC Endpoint, NOT the NAT Gateway!

> 📸 [Screenshot: `aws s3 ls` working on private EC2 via VPC endpoint]

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> Without the VPC Endpoint, S3 traffic from the private EC2 would go: Private EC2 → NAT Gateway → Internet → S3. With the endpoint: Private EC2 → VPC Endpoint → S3. The second path is faster, cheaper, and more secure!

---

### Step 11: Verify Your Work

> <img src="https://img.shields.io/badge/Step%2011-Verify%20Your%20Work-9B59B6?style=for-the-badge" />

- [ ] Private subnet `ravi-private-subnet-1a` (10.0.2.0/24) exists with NO auto-assign public IP
- [ ] NAT Gateway `ravi-nat-gw` is available in the public subnet
- [ ] Private route table `ravi-private-rt` routes `0.0.0.0/0` to NAT Gateway
- [ ] Private EC2 has no public IP but can reach the internet (curl/ping work)
- [ ] Private EC2's outbound IP matches the NAT Gateway's Elastic IP
- [ ] VPC Endpoint `ravi-s3-endpoint` for S3 is available
- [ ] `aws s3 ls` works on private EC2 via the endpoint

---

## ✅ Validation Checklist

| # | Check | Status |
|---|-------|--------|
| 1 | Private subnet created with CIDR `10.0.2.0/24` | ☐ |
| 2 | NAT Gateway `ravi-nat-gw` available with Elastic IP | ☐ |
| 3 | Private route table routes through NAT | ☐ |
| 4 | Private EC2 has no public IP | ☐ |
| 5 | Private EC2 can reach internet via NAT | ☐ |
| 6 | VPC Endpoint for S3 created | ☐ |
| 7 | `aws s3 ls` works on private EC2 | ☐ |
| 8 | Bastion host allows SSH access to private EC2 | ☐ |

---

## 🧹 Cleanup (IMPORTANT!)

> ⚠️ **THE NAT GATEWAY COSTS ~$32/MONTH. DELETE IT FIRST! Do not forget!**

### Step 1: Delete the NAT Gateway (CRITICAL!)

> <img src="https://img.shields.io/badge/Step%201-Delete%20NAT%20Gateway-E74C3C?style=for-the-badge" />

1. Go to **VPC** → **NAT Gateways**
2. Select `ravi-nat-gw`
3. Click **Actions** → **Delete NAT gateway**
4. Type `ravi-nat-gw` to confirm
5. Click **Delete**

> 📸 [Screenshot: NAT Gateway deletion confirmed]

> ⚠️ **THIS IS THE MOST IMPORTANT STEP. Do not skip it!**

### Step 2: Release the Elastic IP

> <img src="https://img.shields.io/badge/Step%202-Release%20Elastic%20IP-F39C12?style=for-the-badge" />

1. Go to **VPC** → **Elastic IPs**
2. Select the Elastic IP used by the NAT Gateway
3. Click **Actions** → **Release Elastic IP addresses**
4. Confirm

### Step 3: Delete the VPC Endpoint

> <img src="https://img.shields.io/badge/Step%203-Delete%20VPC%20Endpoint-2ECC71?style=for-the-badge" />

1. Go to **VPC** → **Endpoints**
2. Select `ravi-s3-endpoint`
3. Click **Actions** → **Delete endpoints**
4. Confirm

### Step 4: Terminate Both EC2 Instances

> <img src="https://img.shields.io/badge/Step%204-Terminate%20EC2%20Instances-9B59B6?style=for-the-badge" />

1. Go to **EC2** → **Instances**
2. Select `bastion-ec2` → **Instance state** → **Terminate instance**
3. Select `private-ec2-test` → **Instance state** → **Terminate instance**

### Step 5: Delete Security Groups

> <img src="https://img.shields.io/badge/Step%205-Delete%20Security%20Groups-3498DB?style=for-the-badge" />

1. Go to **VPC** → **Security Groups**
2. Delete `ravi-private-sg`
3. Delete `ravi-vpc-sg` (if not needed for future labs)

### Step 6: Delete Route Tables

> <img src="https://img.shields.io/badge/Step%206-Delete%20Route%20Tables-E67E22?style=for-the-badge" />

1. Go to **VPC** → **Route Tables**
2. Disassociate and delete `ravi-private-rt`
3. Disassociate and delete `ravi-public-rt`

### Step 7: Detach and Delete Internet Gateway

> <img src="https://img.shields.io/badge/Step%207-Delete%20Internet%20Gateway-1ABC9C?style=for-the-badge" />

1. **Internet Gateways** → select `ravi-igw` → **Detach** → **Delete**

### Step 8: Delete Subnets

> <img src="https://img.shields.io/badge/Step%208-Delete%20Subnets-27AE60?style=for-the-badge" />

1. Delete `ravi-private-subnet-1a`
2. Delete `ravi-public-subnet-1a`

### Step 9: Delete the VPC

> <img src="https://img.shields.io/badge/Step%209-Delete%20VPC-C0392B?style=for-the-badge" />

1. **Your VPCs** → select `ravi-custom-vpc` → **Delete VPC** → type name → confirm

> 📸 [Screenshot: All resources deleted — clean console]

---

## 🎓 What You Learned

- **Private subnets** don't have direct internet access — instances there have no public IPs
- **NAT Gateways** allow outbound internet access for private instances without inbound exposure
- NAT Gateways sit in **public subnets** and route traffic through the **Internet Gateway**
- **VPC Endpoints** let private instances access AWS services through the AWS private network
- **Gateway VPC Endpoints** for S3 are free and don't require a NAT Gateway
- **Bastion hosts** (jump boxes) are used to SSH into private instances
- The difference between **outbound-only** (NAT) and **no connectivity** (no NAT, no endpoint)

---

## 🔗 What's Next?

You've built a proper VPC with public and private subnets! Now let's learn how to distribute traffic across multiple EC2 instances using an Application Load Balancer.

➡️ **[Lab 10 — ELB: Application Load Balancer](../10%20-%20ELB%20-%20Application%20Load%20Balancer/README.md)**

---

## ❓ Troubleshooting

<details>
<summary><strong>Click to expand Troubleshooting Table</strong></summary>

| Problem | Solution |
|---------|----------|
| Private EC2 can't reach internet | Check: (1) NAT Gateway is Available, (2) Private route table has `0.0.0.0/0 → NAT GW`, (3) Route table is associated with private subnet |
| `curl checkip.amazonaws.com` times out | NAT Gateway may still be initializing (wait 2-3 min). Also check the NAT's subnet has a route to the IGW |
| Can't SSH to private EC2 from bastion | Make sure: (1) Private SG allows SSH from public subnet CIDR `10.0.1.0/24`, (2) Bastion and private EC2 are in the same VPC, (3) You're using the correct key pair |
| `aws s3 ls` fails on private EC2 | Check: (1) VPC Endpoint is Available, (2) Endpoint is associated with `ravi-private-rt`, (3) AWS CLI is installed and configured |
| "Too many NAT Gateways" error | You may have hit the limit (5 per region). Delete unused NAT Gateways first |
| Can't delete NAT Gateway | Make sure no resources are actively using it. Wait a moment and try again |
| Elastic IP won't release | Disassociate it from any resource first (NAT Gateway should be deleted by now) |
| VPC Endpoint creation fails | Make sure you selected the **Gateway** type (not Interface) for S3 |

</details>

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
> The most expensive mistake in this lab is forgetting to delete the NAT Gateway. It costs $0.045/hour = $32.40/month. Set a timer on your phone if you need to! ⏰💰

---

<div align="center">

<img src="https://img.shields.io/badge/Lab%2009-Complete!-27AE60?style=for-the-badge&labelColor=232F3E" />

</div>