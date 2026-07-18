# Lab 01 — EC2: Launch and SSH Connect

![Difficulty: Easy](https://img.shields.io/badge/Difficulty-Easy-green)
![Time: ~30 min](https://img.shields.io/badge/Time-~30%20min-blue)
![Cost: <$1](https://img.shields.io/badge/Cost-%3C%241-lightgrey)
![Service: EC2](https://img.shields.io/badge/Service-EC2-orange)

> *"Ravi, every cloud engineer's journey starts with launching a virtual server. It's like the 'Hello World' of AWS. Let's get your first EC2 instance up and running!"* — Rithu

---

## 🎯 Objective

Launch your first Amazon EC2 instance using the AWS Management Console, connect to it via SSH, install a web server, and verify it's running. By the end of this lab, you'll have successfully hosted a simple webpage in the cloud.

## 🧠 Prerequisites

- An **AWS account** with administrative access
- Basic familiarity with the AWS Management Console (navigating to services)
- No prior EC2 experience needed — this lab assumes you're opening the console for the first time!

## 💰 Cost Warning

All resources in this lab are **Free Tier eligible** (t2.micro). However, as our favorite greenhorn once said:
> *"I forgot to terminate my instance and got a surprise $15 AWS bill the next month."*

**Never forget to terminate.** Seriously. There's a Cleanup section at the bottom. USE IT.

## 🏗️ Architecture

```
┌─────────────────────────────────┐
│         Public Subnet           │
│  ┌─────────────────────────┐   │
│  │   t2.micro EC2          │   │
│  │   Amazon Linux 2023     │   │
│  │   Security Group:       │   │
│  │   SSH (22) → My IP      │   │
│  │                         │   │
│  │   httpd installed       │   │
│  │   index.html served     │   │
│  └─────────┬───────────────┘   │
│            │ Internet Gateway   │
│            ▼                    │
│    Browser: http://<public-ip>  │
└─────────────────────────────────┘
```

## 🛠️ Step-by-Step Instructions

### Step 1: Launch the EC2 Instance

1. Go to the **AWS Management Console** and search for **EC2** in the top search bar. Click it.
2. Click the orange **Launch instance** button (not the dropdown arrow, the actual button).

📸 [Screenshot: EC2 Dashboard showing the Launch instance button]

3. **Name your instance:** Enter `first-ec2-instance` in the "Name" field.

4. **Application and OS Images:**
   - Click "Amazon Linux" (it's usually the default).
   - Select **Amazon Linux 2023 AMI** (Free Tier eligible).

5. **Instance type:**
   - Select **t2.micro** (that little asterisk saying "Free tier eligible" is like winning the lottery).

6. **Key pair (login):**
   - Click **Create new key pair**.
   - Key pair name: `first-key-pair`
   - Key pair type: **RSA**
   - Private key file format: **.pem**
   - Click **Create key pair**. The `.pem` file will download automatically.
   - **SAVE THIS FILE somewhere safe — this is your door key!**

> 💡 **Rithu's Tip:** Move that `.pem` file to your `~/.ssh/` folder on Mac/Linux or a dedicated folder on Windows. Don't lose it. You can't SSH without it.

7. **Network settings:** Click **Edit**.
   - VPC: leave as default
   - Subnet: leave as default
   - **Auto-assign public IP:** Enable
   - **Firewall (security groups):** Select "Create security group"
   - Security group name: `launch-wizard-1`
   - Description: `launch-wizard-1 created for first-ec2-instance`
   - Click **Add security group rule**:
     - Type: **SSH**
     - Source: **My IP** (this is crucial — restricts access to YOUR IP only)
     - Description: `SSH from my IP`

8. **Configure storage:** Leave the default 8 GiB gp2/gp3 root volume.

9. Click **Launch instance** at the bottom.

10. You'll see a success screen. Click **View all instances**.

📸 [Screenshot: Instance list showing first-ec2-instance with 2/2 status checks]

### Step 2: Wait for the Instance to be Ready

- The instance will show **Pending** for a few seconds, then **Running**.
- Wait for **2/2 status checks** to pass (refresh every 30 seconds).

### Step 3: Find Your Public IP

1. Click on your instance `first-ec2-instance` in the list.
2. In the bottom pane, look at the **Details** tab.
3. Copy the **Public IPv4 address** — you'll need this.
4. Also copy the **Public IPv4 DNS** — it works too.

### Step 4: Connect via SSH

Choose your weapon below. Pick ONE based on your operating system.

---

#### Option A: Mac / Linux — Terminal

Open your terminal and run:

```bash
# 1. Set restrictive permissions on your key file
chmod 400 /path/to/your/first-key-pair.pem

# 2. SSH into the instance
ssh -i /path/to/your/first-key-pair.pem ec2-user@<YOUR-PUBLIC-IP>
```

Replace `<YOUR-PUBLIC-IP>` with the IP address you copied.

> 💡 **Rithu's Tip:** If you get `Permissions 0644 for 'first-key-pair.pem' are too open`, you forgot the `chmod 400` step. SSH is paranoid about loose keys — and it should be!

If you see:

```
The authenticity of host 'xx.xx.xx.xx' can't be established.
Are you sure you want to continue connecting (yes/no)?
```

Type `yes` and press Enter.

Welcome to your first EC2 instance! Your prompt should look like:

```
[ec2-user@ip-xxx-xxx-xxx-xxx ~]$
```

---

#### Option B: Windows — PuTTY

PuTTY uses `.ppk` files, not `.pem`. Here's how to convert:

1. **Download PuTTYgen** (comes with Putty installer)
2. Open PuTTYgen and click **Load**
3. Browse to and select your `first-key-pair.pem` file. PuTTYgen might say "PuTTY Key Generator needs to import this key." You might need to change the file filter to "All Files (*.*)" to see the `.pem`.
4. Click **OK**.
5. Click **Save private key**.
6. PuTTYgen will ask if you want to save without a passphrase — click **Yes**.
7. Name it `first-key-pair.ppk` and save.

Now connect with PuTTY:

1. Open PuTTY.
2. In **Host Name**: `ec2-user@<YOUR-PUBLIC-IP>`
3. In **Port**: `22`
4. Connection type: **SSH**
5. Go to **Connection > SSH > Auth** (or **Connection > SSH > Auth > Credentials** in newer versions).
6. Browse for the **Private key file** → select `first-key-pair.ppk`.
7. Click **Open** → **Accept** the fingerprint.

You'll be greeted with the `ec2-user` prompt. You made it! 🎉

---

#### Option C: AWS EC2 Instance Connect (Browser-Based)

This one's easy — no client config needed:

1. Select your instance in the EC2 console.
2. Click the **Connect** button up top.
3. Go to the **EC2 Instance Connect** tab.
4. Click **Connect**.
5. A new browser tab opens with a terminal. Magic!

> 💡 **Rithu's Tip:** EC2 Instance Connect is great for quick tasks, but falls flat when your instance loses public access or is in a private subnet. Learn the CLI methods — they'll save you someday!

### Step 5: Install a Web Server (httpd)

Now that you're inside your EC2 instance, let's install Apache:

```bash
sudo yum update -y
```

Wait for the updates to finish (might take a minute).

```bash
sudo yum install -y httpd
```

You'll see lots of text scroll by. Docker-like energy. That's the package manager doing its thing.

```bash
sudo systemctl start httpd
```

No errors? Great.

```bash
sudo systemctl enable httpd
```

This makes sure the web server starts automatically if the instance reboots.

Check that it's running:

```bash
sudo systemctl status httpd
```

You should see `● httpd.service - The Apache HTTP Server` with status `active (running)`.

### Step 6: Create a Custom Web Page

Let's put your personal touch on it:

```bash
echo "<h1>Hello from Ravi's first EC2 instance!</h1>" | sudo tee /var/www/html/index.html
```

> 💡 **Rithu's Tip:** The `tee` command writes to a file AND prints to the terminal. It's like a microphone and speaker at the same time. `sudo` is needed because `/var/www/html/` is owned by `root`.

### Step 7: Verify Your Work

1. Open your browser.
2. Go to: `http://<YOUR-PUBLIC-IP>`
3. **You should see:** a white page with:
   - **Hello from Ravi's first EC2 instance!**

📸 [Screenshot: Browser showing the "Hello from Ravi's first EC2 instance!" page]

> 💡 **Rithu's Tip:** Notice it's `http://` NOT `https://`. We haven't configured SSL yet. HTTP on port 80, HTTPS on 443. Patience, grasshopper.

**If nothing loads:**
- Check that your security group has an inbound rule for HTTP (port 80) — we only added SSH! Go add it now if you want the page to load.
- Wait... we didn't add an HTTP rule. Did we? Nope! But it still loaded because... hmm, actually it won't yet.
- **Let's fix this.** Go to the EC2 console → Security Groups → select your SG → Inbound rules → Edit inbound rules → Add rule:
  - Type: **HTTP**
  - Source: **0.0.0.0/0** (or **Anywhere-IPv4**)
  - Description: `HTTP from anywhere`
- Save, reload your browser. Magic.

## ✅ Validation Checklist

- [ ] EC2 instance launched from Amazon Linux 2023 AMI
- [ ] Key pair `first-key-pair` created and downloaded
- [ ] SSH connection established to the instance
- [ ] Apache httpd installed and running
- [ ] Custom index.html visible via browser at `http://<public-ip>`
- [ ] HTTP inbound rule added to security group

## 🧹 Cleanup (IMPORTANT!)

Don't skip this. Future Ravi will thank you.

1. **Terminate the instance:**
   - EC2 Console → Instances → select `first-ec2-instance`
   - Instance state → **Terminate** → Click **Terminate** in the modal.

2. **Delete the key pair:**
   - EC2 Console → Network & Security → **Key Pairs**
   - Select `first-key-pair` → Actions → **Delete** → Confirm.
   - Also delete the `.pem` / `.ppk` file from your computer.

3. **Clean up the security group:**
   - EC2 Console → Network & Security → **Security Groups**
   - The SG will show "vpc-xxx" association — you might need to wait for the instance termination to finish.
   - Once terminated, select the SG → Actions → **Delete security groups** → Confirm.

> 💡 **Rithu's Tip:** You can also just **Delete** the entire CloudFormation stack or AWS resource group if you created one. For now, manual cleanup is fine. Get in the muscle memory early!

## 🎓 What You Learned

| Concept | Takeaway |
|---------|---------|
| EC2 Launch | t2.micro is the Free Tier king |
| Key Pairs | RSA .pem for Mac/Linux, convert to .ppk for PuTTY |
| Security Groups | Stateful firewall. Allow only what's needed |
| SSH Methods | Mac/Linux CLI, Windows PuTTY, EC2 Instance Connect |
| Apache httpd | Install, start, enable for automatic boot |
| User Data | Not covered here, but remember: scripts run at launch! |

## 🔗 What's Next?

This was the warm-up! Next up, we'll play with firewalls.

👉 **Proceed to Lab 02:** [EC2 - Security Groups Deep Dive](../02%20-%20EC2%20-%20Security%20Groups%20Deep%20Dive/README.md)

We'll lock down traffic, open ports on-demand, and reference security groups within each other. It'll be like being a cloud bouncer. 🫡

## ❓ Troubleshooting

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| `Permission denied (publickey)` | Wrong key file or permissions | Ensure `chmod 400` on `.pem`, verify key name matches the instance |
| PuTTY error: `Disconnected: No supported authentication algorithms` | `.pem` used directly | Convert to `.ppk` with PuTTYgen |
| Website doesn't load in browser | Security group missing HTTP rule | Add inbound rule HTTP (80) from 0.0.0.0/0 |
| Connection timeout | Wrong IP or instance unreachable | Verify Public IPv4 address, check instance state is `Running` |
| `sudo: yum: command not found` | You might be on Amazon Linux 2023 using `dnf` | Try `sudo dnf install -y httpd` instead |
| Browser shows Apache default page | Custom index.html might have failed | Re-run the `echo | sudo tee` command |

---

*Written with ☕ and a lot of patience — Rithu*
