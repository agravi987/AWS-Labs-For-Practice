# Lab 02 — EC2: Security Groups Deep Dive

![Difficulty: Easy](https://img.shields.io/badge/Difficulty-Easy-green)
![Time: ~25 min](https://img.shields.io/badge/Time-~25%20min-blue)
![Cost: <$1](https://img.shields.io/badge/Cost-%3C%241-lightgrey)
![Service: EC2/VPC](https://img.shields.io/badge/Service-EC2%2FVPC-orange)

> *"Security groups are the bouncers of your EC2 club. They decide who gets in (inbound) and what the party people can access (outbound). Tonight, YOU are the bouncer."* — Rithu

---

## 🎯 Objective

Become a security group expert by creating, modifying, and chaining security group rules on EC2 instances. You'll see firsthand how traffic flows in a live environment, then intentionally block traffic to understand how security groups enforce boundaries.

## 🧠 Prerequisites

- Completion of **[Lab 01 — EC2: Launch and Connect](../01%20-%20EC2%20-%20Launch%20and%20Connect/README.md)** — you need SSH basics
- Familiarity with Linux shell navigation

## 💰 Cost Warning

Still Free Tier eligible as long as you stick to t2.micro. Each security group is free. Even in a real environment, security groups themselves cost nothing beyond what AWS allocates. That said: **terminate unused instances**. AWS charges for compute time, not firewall rules.

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────┐
│                   VPC                            │
│                                                   │
│  ┌─────────────────────────┐  ┌─────────────┐    │
│  │   web-server-sg         │  │   app-sg     │    │
│  │   Inbound:              │  │   Inbound:   │    │
│  │   SSH (22) → My IP      │  │   HTTP (80)  │    │
│  │   HTTP (80) → 0.0.0.0/0 │  │   ← ref:    │    │
│  │   HTTPS (443) → 0.0.0.0 │  │   web-sg     │    │
│  │   Outbound: ALL          │  │              │    │
│  │                         │  │              │    │
│  │   t2.micro (httpd)      │  │   t2.micro   │    │
│  └─────────────────────────┘  └─────────────┘    │
│                                                   │
└──────────────────────────────────────────────────┘
```

## 🛠️ Step-by-Step Instructions

### Step 1: Create a New Security Group

Let's leave behind the auto-created groups from Lab 01.

1. EC2 Console → **Security Groups** under Network & Security in the left panel.
2. Click **Create security group** (blue button, top left).
3. Fill in:

   | Field | Value |
   |-------|-------|
   | Security group name | `web-server-sg` |
   | Description | `Security group for the web server lab` |
   | VPC | default VPC |

4. Add these **Inbound rules**:

   | Type | Protocol | Port | Source | Description |
   |------|----------|------|--------|-------------|
   | SSH | TCP | 22 | **My IP** | SSH from my secure fortress |
   | HTTP | TCP | 80 | **0.0.0.0/0** | Allow all web traffic |

   > 💡 **Rithu's Tip:** Notice how SSH is locked to YOUR IP and HTTP is open to the entire internet. That's intentional. You manage the server privately, your users browse the website publicly.

5. **Outbound rules** leave as default: **Allow all traffic** to 0.0.0.0/0.

6. Click **Create security group** at the bottom.

📸 [Screenshot: Security group creation page with web-server-sg showing SSH and HTTP inbound rules]

### Step 2: Launch an EC2 Instance with This Security Group

1. EC2 Console → **Instances** → **Launch instance**.
2. Name: `security-group-lab-instance`.

3. **AMI:** Amazon Linux 2023 (Free Tier).

4. **Instance type:** t2.micro.

5. **Key pair:** Select `first-key-pair` from Lab 01. If you deleted it, create a new one (follow Lab 01 Step 5).

6. **Network settings:**
   - VPC: default
   - Subnet: No preference (default)
   - **Auto-assign public IP:** Enable
   - **Firewall (security group):** Select **Select existing security group**
   - Check `web-server-sg`

7. **Storage:** Default 8 GiB gp2/gp3.

8. Click **Launch instance**.

### Step 3: Set Up the Web Server

Wait for 2/2 status checks. SSH into the instance:

```bash
ssh -i first-key-pair.pem ec2-user@<public-ip>
```

> 💡 **Rithu's Tip:** Forgot your key's location? Look in your Downloads folder. Future you keeps SSH keys organized. Present you? Well... at least it's somewhere.

Inside the instance:

```bash
sudo yum update -y
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
echo "<h1>Security Groups Lab - Ravi rocks!</h1>" | sudo tee /var/www/html/index.html
```

### Step 4: Verify HTTP Access

1. Open your browser.
2. Go to `http://<public-ip>` (not https).

📸 [Screenshot: Browser displaying "Security Groups Lab - Ravi rocks!"]

**Take a moment to celebrate.** 🎉

### Step 5: The SSH Restriction Test

The SSH rule is locked to **My IP**, which AWS determined when you created the rule.

**Test:** SSH into your instance from your current machine — it works. Now imagine you're at a coffee shop with a different IP. SSH would fail.

To simulate:

1. Temporarily change the SSH source for `web-server-sg`:
   - EC2 Console → Security Groups → `web-server-sg`, Inbound rules → **Edit inbound rules**.
   - Change SSH source to a random IP like `1.2.3.4/32`.
   - Click **Save rules**.
2. Go back to your terminal and press Enter. You'll notice nothing happens — SSH connections are stateful (already-established connections stay open).
3. Open a NEW terminal window and try SSH:

   ```bash
   ssh -i first-key-pair.pem ec2-user@<public-ip>
   ```

   It hangs and eventually times out. Connection never established. **Denied!** 🚫

4. **Change it back!** Set SSH source back to **My IP**, save rules, try SSH again.

### Step 6: Remove HTTP and Watch the Site Go Down

1. Security Groups → `web-server-sg` → Inbound → **Edit inbound rules**.
2. Click **Remove** (the trash icon) next to the HTTP rule.
3. Click **Save rules**.
4. Refresh your browser at `http://<public-ip>`.

📸 [Screenshot: Browser showing connection refused or timeout error]

The page won't load. Why? Because HTTP traffic can't reach port 80. AWS is silently dropping those packets.

> 💡 **Rithu's Tip:** Security groups are **stateless-aware** (they track connection state), so existing HTTP keep-alive connections might linger. Open a fresh Incognito/Private browser tab for a definitive test.

### Step 7: Re-add HTTP to Restore Access

1. Edit inbound rules for `web-server-sg`.
2. Click **Add rule**:
   - Type: **HTTP** | Source: **0.0.0.0//0** | Description: `HTTP restored`
3. **Save**.
4. Refresh the browser.

The site is back. Feels good to control traffic at will, right? 🔥

### Step 8: Add an HTTPS Rule (for illustration)

Even if you don't have an SSL cert right now, let's add the rule:

1. Edit inbound rules → **Add rule**.
   - Type: **HTTPS** | Source: **0.0.0.0/0** | Description: `HTTPS placeholder`
2. Save.

This rule accepts HTTPS traffic on port 443, which we'll need when we set up SSL in later labs.

### Step 9: Reference a Security Group Inside Another SG

This is where things get spicy. Security groups can reference OTHER security groups as a source, instead of IP ranges. This is gold for multi-tier apps.

1. Create a second SG:

   | Field | Value |
   |-------|-------|
   | Security group name | `app-sg` |
   | Description | `Backend app tier that trusts web-server-sg` |
   | VPC | default |

2. Add a single inbound rule:
   - Type: **HTTP**
   - Source: **Custom** → start typing `web-server-sg` in the source search box
   - Select `web-server-sg`

   Your source should display: `web-server-sg (sg-xxxxxxxx)`

3. Click **Create security group**.

📸 [Screenshot: app-sg inbound rule showing source as web-server-sg]

Now launch a SECOND EC2 instance:

1. Launch another t2.micro Amazon Linux 2023 instance named `app-instance`.
2. Network section: Select **app-sg** as the security group.
3. Launch it. SSH into both instances.

On `security-group-lab-instance`:

```bash
# Test connectivity from the web server to the app layer
curl http://<private-ip-of-app-instance>:80
```

Wait — the app instance doesn't have httpd running. Install and start it:

```bash
# Inside app-instance
sudo yum install -y httpd
sudo systemctl start httpd
echo "<h1>Backend App - Only reachable from web-server-sg</h1>" | sudo tee /var/www/html/index.html
```

Now curl again from `security-group-lab-instance`:

```bash
curl http://<private-ip-of-app-instance>
```

You should see the response! The `app-sg` is configured to trust traffic only if it comes from `web-server-sg`.

### Step 10: Verify Your Work

- [ ] `web-server-sg` has SSH from My IP, HTTP from anywhere, HTTPS from anywhere
- [ ] SSH works from your laptop
- [ ] Browser loads `http://<public-ip>` → sees the Security Groups Lab page
- [ ] Removing HTTP rule kills the site
- [ ] Re-adding HTTP restores the site
- [ ] `app-sg` references `web-server-sg` as source
- [ ] Web server can curl the app instance's HTTP endpoint via private IP

## ✅ Validation Checklist

- [ ] Security group `web-server-sg` created with SSH + HTTP inbound rules
- [ ] EC2 instance launched and serving custom index.html via httpd
- [ ] HTTP access verified from browser
- [ ] Temporary SSH restriction by changing source to 1.2.3.4/32
- [ ] HTTP rule removed → site goes down → verified
- [ ] HTTP rule re-added → site comes back → verified
- [ ] HTTPS rule added (placeholder)
- [ ] Security group `app-sg` created referencing `web-server-sg`
- [ ] Cross-SG HTTP access verified via curl on private IP

## 🧹 Cleanup (IMPORTANT!)

1. **Terminate both instances:**
   - `security-group-lab-instance` → Instance state → **Terminate** → Confirm.
   - `app-instance` → Instance state → **Terminate** → Confirm.

2. **Delete both security groups:**
   - EC2 Console → Security Groups.
   - Select `web-server-sg` → Actions → Delete security groups.
   - Select `app-sg` → Actions → Delete security groups.
   - **Note:** You might need to wait until the instances are fully terminated.

3. **Delete key pair** (if you created a new one).

## 🎓 What You Learned

| Concept | Takeaway |
|---------|----------|
| Inbound vs Outbound | Inbound controls incoming traffic; outbound controls what your resource reaches |
| Source IP filtering | SSH restricted to My IP keeps attackers out |
| Stateful firewalls | Security groups track connections; replies flow automatically |
| Dynamic rule changes | Rules apply immediately. No restart needed. |
| SG-to-SG referencing | Allows traffic between resources without exposing IPs |
| HTTP/HTTPS rules | Port 80 (HTTP) and 443 (HTTPS). Different rules, same drill |

## 🔗 What's Next?

Time to talk disks. EC2 without storage is like a laptop without a hard drive.

👉 **Proceed to Lab 03:** [EBS - Volumes and Snapshots](../03%20-%20EBS%20-%20Volumes%20and%20Snapshots/README.md)

We'll add extra storage volumes, format them, take snapshots, and restore from a backup. Yes, like cloud forensics but friendlier.

## ❓ Troubleshooting

| Problem | Likely Cause | Fix |
|---------|-------------|------|
| SSH times out after changing source IP | SSH rule source changed to a non-matching IP | Set source back to **My IP** immediately |
| Browser times out after removing HTTP | HTTP rule deleted; traffic blocked | Re-add HTTP (80) inbound rule |
| `app-instance` curl gets `Connection refused` | httpd not installed/running on app instance | Run `sudo systemctl status httpd`; install if needed |
| `operation not permitted` on security group deletion | Instances still running | Terminate associated instances first, wait 2–3 min |
| Multiple SGs on one instance and still blocked | Most restrictive rule applies | Check all assigned SGs for overlapping rules |
| SG-to-SG referencing doesn't work | Cross-account or cross-region referencing not allowed | Both SGs must be in the same VPC and region |

---

*Written with a click-click, save-save, test-test philosophy — Rithu*
