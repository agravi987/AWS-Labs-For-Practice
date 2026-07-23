<div align="center">

# AWS Hands-On Labs

<img src="https://img.shields.io/badge/Prerequisites-Setup%20Guide-2ECC71?style=for-the-badge&labelColor=232F3E" />

**"Hey Ravi, before we start building awesome stuff on AWS, let's make sure your toolbox is ready!"** — Rithu

</div>

---

## Table of Contents

1. [Welcome & Why Prerequisites Matter](#1--welcome--why-prerequisites-matter)
2. [Creating an AWS Account](#2--creating-an-aws-account)
3. [Enabling MFA on Root Account](#3--enabling-mfa-on-root-account)
4. [Creating an IAM Admin User](#4--creating-an-iam-admin-user)
5. [Setting Up Billing Alerts (AWS Budgets)](#5--setting-up-billing-alerts-aws-budgets)
6. [Installing AWS CLI](#6--installing-aws-cli)
7. [Configuring AWS CLI](#7--configuring-aws-cli)
8. [Installing a Code Editor (VS Code)](#8--installing-a-code-editor-vs-code)
9. [SSH Key Pair Setup](#9--ssh-key-pair-setup)
10. [Understanding Free Tier Limits](#10--understanding-free-tier-limits)
11. [Ready to Go!](#11--ready-to-go)

---

## 1. Welcome & Why Prerequisites Matter

<img src="https://img.shields.io/badge/Section%201-Welcome-2ECC71?style=for-the-badge" />

**Rithu:** *"Hey Ravi! Welcome to our AWS Hands-On Labs!"*

**Ravi:** *"Thanks Rithu! I'm excited! Can we just jump straight into deploying servers?"*

**Rithu:** *"I love the enthusiasm, but hold on — imagine trying to cook a gourmet meal without turning on the stove or chopping your veggies first. That's exactly what diving into AWS without setting up your environment is like. We need to:*

- *Make sure you have a proper AWS account (not using root for daily work — that's a big no-no!)*
- *Set up security so your account doesn't get hacked*
- *Make sure you don't accidentally spend your rent money on AWS 🫣*
- *Install the tools we'll need*

*Trust me, spending 30 minutes here will save you hours of headaches later. Ready? Let's do this!"*

> <img src="https://img.shields.io/badge/Warning-Important-E74C3C?style=flat-square" />
>
> **Bottom Line:** These prerequisites are non-negotiable. Every single lab in this repository assumes you've completed every section on this page. If you skip steps, you *will* get stuck later — and nobody wants that!

---

## 2. Creating an AWS Account

<img src="https://img.shields.io/badge/Section%202-AWS%20Account-3498DB?style=for-the-badge" />

**Rithu:** *"This is Step Zero, Ravi. No AWS account = no cloud magic."*

### Step-by-Step Instructions

<details>
<summary><strong>Step 1: Open the AWS Website</strong></summary>

1. Open your web browser (Chrome, Firefox, Edge — any will work).
2. Type **https://aws.amazon.com** in the address bar and press Enter.

📸 [Screenshot: AWS homepage showing the "Sign In to the Console" and "Create an AWS Account" buttons in the top-right corner]

</details>

<details>
<summary><strong>Step 2: Start the Sign-Up Process</strong></summary>

1. In the top-right corner of the AWS homepage, click the orange **"Create an AWS Account"** button.

📸 [Screenshot: AWS homepage with an arrow pointing to the "Create an AWS Account" button]

</details>

<details>
<summary><strong>Step 3: Enter Your Email and Account Details</strong></summary>

1. **Root user email address:** Enter a valid email address (this becomes your root account email — use one you actually check!).
2. **AWS account name:** Enter a name for your account (e.g., `ravi-lab-account` or whatever you like).
3. Click **"Continue"**.

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
>
> *"The email you enter here becomes your root user login. Write it down somewhere safe — you'll need it forever."*

📸 [Screenshot: Sign-up form showing email address and AWS account name fields]

</details>

<details>
<summary><strong>Step 4: Choose Your Account Type</strong></summary>

1. Select **"Personal"** (this is the free tier–eligible option for individuals).
   - *Note: If you're setting this up for a company, you'd choose "Professional" — but for our labs, "Personal" is perfect.*
2. Click **"Continue"**.

📸 [Screenshot: Account type selection with "Personal" radio button selected]

</details>

<details>
<summary><strong>Step 5: Enter Your Contact Information</strong></summary>

1. Fill in your details:
   - **Full Name**
   - **Phone Number** (you'll need to verify this)
   - **Country/Region**
   - **Address**
2. Read and accept the **AWS Customer Agreement** and **Privacy Notice** by checking the boxes.
3. Click **"Create Account and Continue"**.

📸 [Screenshot: Contact information form with all fields filled in and checkboxes checked]

</details>

<details>
<summary><strong>Step 6: Enter Payment Information (Credit/Debit Card)</strong></summary>

1. Enter your **credit card** or **debit card** information.
2. AWS will place a **temporary $1.00 hold** on your card to verify it's valid. This is NOT a charge — it will be released within 3-5 business days.

> <img src="https://img.shields.io/badge/Warning-Important-E74C3C?style=flat-square" />
>
> **IMPORTANT WARNING — Read This Twice!**
>
> Rithu says: *"Yes, Ravi, you MUST have a credit or debit card. There is no way around this. AWS will not charge you anything on the Free Tier as long as you stay within limits — but they need a card on file. If you don't have one, ask a family member or friend to help. And PLEASE set up billing alerts after this (Section 5)!"*

> <img src="https://img.shields.io/badge/Tip-Pro%20Tip-FFC300?style=flat-square" />
>
> If you have a credit card with a low limit, use that one. It's an extra layer of protection against accidental overspending.

📸 [Screenshot: Payment information form with credit card fields (blurred/sensitive data)]

</details>

<details>
<summary><strong>Step 7: Verify Your Identity</strong></summary>

1. AWS will ask you to verify your **phone number**.
2. Choose **"Text message"** or **"Voice call"**.
3. Enter the verification code you receive.
4. Click **"Verify code"**.

📸 [Screenshot: Phone verification screen with text message option selected]

</details>

<details>
<summary><strong>Step 8: Choose a Support Plan</strong></summary>

1. You'll be presented with four support plan options:
   - **Basic** — **FREE** ✅ (Choose this one for the labs!)
   - Developer — $29/month
   - Business — $100/month
   - Enterprise — $15,000+/year

2. Select **"Basic"** and click **"Complete Sign Up"**.

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
>
> *"Basic is totally fine for learning. You get access to the AWS Support Forums and documentation. If you ever need real help, you can upgrade later — but for these labs, Basic is all you need."*

📸 [Screenshot: Support plan selection page with "Basic" plan highlighted]

</details>

<details>
<summary><strong>Step 9: Confirmation</strong></summary>

1. You should see a confirmation page. AWS may take a few minutes to fully activate your account.
2. **Check your email** — you'll receive a welcome email from AWS confirming your account is ready.

> 🎉 **Ravi:** *"I got the email! My account is live!"*
>
> **Rithu:** *"Awesome! But don't go celebrating just yet — we've got some very important security setup to do next!"*

📸 [Screenshot: AWS account activation confirmation page]

</details>

---

## 3. Enabling MFA on Root Account

<img src="https://img.shields.io/badge/Section%203-MFA%20Setup-E74C3C?style=for-the-badge" />

**Rithu:** *"This is the SINGLE most important security step, Ravi. I've seen people skip this and regret it. MFA on your root account is like putting a deadbolt on your front door."*

**Ravi:** *"Okay okay, what's MFA?"*

**Rithu:** *"Multi-Factor Authentication. It means even if someone steals your password, they STILL can't get in without your phone. Let's set it up."*

### What You'll Need

- A smartphone with one of these apps installed:
  - **Google Authenticator** (recommended) — [iOS](https://apps.apple.com/app/google-authenticator/id388497605) | [Android](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2)
  - **Authy** (also great, supports cloud backup) — [iOS](https://apps.apple.com/app/authy/id494168027) | [Android](https://play.google.com/store/apps/details?id=com.authy.authy)

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
>
> *"Install one of these apps on your phone NOW before continuing. I prefer Google Authenticator because it's simple and it's made by Google. Authy is nice too because it backs up your codes to the cloud — so if you lose your phone, you don't lose access to everything."*

### Step-by-Step Instructions

<details>
<summary><strong>Step 1: Sign In as Root</strong></summary>

1. Go to **https://aws.amazon.com**.
2. Click **"Sign In to the Console"** in the top-right.
3. Under **"Root user"**, enter the email address you used to create your AWS account.
4. Enter your password.
5. Click **"Sign In"**.

> <img src="https://img.shields.io/badge/Warning-Important-E74C3C?style=flat-square" />
>
> **Make sure you select "Root user"** — NOT "IAM user". We're setting up security on the root account first.

📸 [Screenshot: AWS sign-in page with "Root user" selected and email field visible]

</details>

<details>
<summary><strong>Step 2: Navigate to IAM Security Credentials</strong></summary>

1. In the AWS Management Console, click the **search bar** at the top and type **"IAM"**.
2. Click **"IAM"** from the results (it'll say "Identity and Access Management" underneath).
3. In the left-hand navigation panel, click **"Security credentials"**.

📸 [Screenshot: IAM dashboard with "Security credentials" tab highlighted in the left sidebar]

</details>

<details>
<summary><strong>Step 3: Assign an MFA Device</strong></summary>

1. On the Security credentials page, scroll down to the **"Multi-factor authentication (MFA)"** section.
2. Click the **"Assign MFA device"** button.

📸 [Screenshot: MFA section showing the "Assign MFA device" button]

</details>

<details>
<summary><strong>Step 4: Configure Your MFA Device</strong></summary>

1. **Device name:** Enter a friendly name like `root-mfa` or `ravi-phone`.
2. **MFA device type:** Select **"Authenticator app"** (this is the recommended option).
3. Click **"Next"**.

📸 [Screenshot: MFA device configuration page with "Authenticator app" selected]

</details>

<details>
<summary><strong>Step 5: Set Up the Authenticator App</strong></summary>

1. You'll see a **QR code** on screen.
2. **On your phone:**
   - Open **Google Authenticator** (or Authy).
   - Tap the **"+"** button (or "Add Account").
   - Select **"Scan a QR code"**.
   - Point your camera at the QR code on your screen.
   - The app will generate a 6-digit code that refreshes every 30 seconds.

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
>
> *"If you can't scan the QR code, most authenticator apps have a 'Enter a setup key' option where you can manually type in the secret key shown below the QR code."*

📸 [Screenshot: QR code display page with setup key visible (blurred)]

</details>

<details>
<summary><strong>Step 6: Enter Verification Codes</strong></summary>

1. Back on the AWS page, you'll see two input fields:
   - **MFA code 1:** Enter the current 6-digit code from your authenticator app.
   - **Wait for the code to refresh** (it changes every 30 seconds).
   - **MFA code 2:** Enter the NEW 6-digit code after it refreshes.
2. Click **"Assign MFA"**.

> <img src="https://img.shields.io/badge/Warning-Important-E74C3C?style=flat-square" />
>
> **Important:** You must enter two CONSECUTIVE codes from the same device. Wait for the first code to expire and a new one to appear, then enter the new one in the second field.

📸 [Screenshot: Two MFA code input fields with codes entered]

</details>

<details>
<summary><strong>Step 7: Confirm Success</strong></summary>

1. You should see a success message confirming MFA has been assigned.
2. Your root account now requires both your password AND your authenticator app to sign in.

> 🎉 **Ravi:** *"That was easy! My root account is now super secure!"*
>
> **Rithu:** *"Great job! But here's the thing — you should almost NEVER sign in as root again. Next, we'll create an IAM admin user for daily use."*

📸 [Screenshot: Success message confirming MFA assignment]

</details>

---

## 4. Creating an IAM Admin User

<img src="https://img.shields.io/badge/Section%204-IAM%20User-9B59B6?style=for-the-badge" />

**Rithu:** *"Okay Ravi, this is rule number one of AWS: NEVER use your root account for day-to-day work. Root has unlimited power — and with great power comes great responsibility... and great risk."*

**Ravi:** *"So what do I use instead?"*

**Rithu:** *"An IAM user! Think of it like this: your root account is the master key to your entire house. An IAM user is a regular key that opens most doors but not the safe. We'll create an admin user that has most of the permissions you need, but it's still safer than using root."*

### Why Not Use Root?

<table>
  <thead>
    <tr>
      <th>Root Account</th>
      <th>IAM User</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Has <strong>unlimited</strong> access to everything</td>
      <td>Has permissions you <strong>explicitly</strong> assign</td>
    </tr>
    <tr>
      <td>If compromised, attacker owns your whole AWS</td>
      <td>If compromised, damage is limited</td>
    </tr>
    <tr>
      <td>No audit trail for who did what</td>
      <td>You can track exactly who did what</td>
    </tr>
    <tr>
      <td>AWS <strong>recommends</strong> disabling root for daily use</td>
      <td>AWS <strong>recommends</strong> using IAM users</td>
    </tr>
  </tbody>
</table>

### Step-by-Step Instructions

> <img src="https://img.shields.io/badge/Warning-Important-E74C3C?style=flat-square" />
>
> **Make sure you're signed in as root for these steps.**

<details>
<summary><strong>Step 1: Navigate to IAM Users</strong></summary>

1. In the AWS Console, go to **IAM** (search for "IAM" in the top search bar).
2. In the left-hand navigation panel, click **"Users"**.
3. Click the orange **"Create user"** button.

📸 [Screenshot: IAM Users page with "Create user" button highlighted]

</details>

<details>
<summary><strong>Step 2: Set User Details</strong></summary>

1. **User name:** Enter `admin-user`.
2. Under **"Provide user access to the AWS Management Console"**:
   - Select **"I want to create an IAM user"**.
   - Check **"User must create a password at next sign-in"** (this lets you set a password for console access).
3. Click **"Next"**.

📸 [Screenshot: User creation form with username "admin-user" entered and console access options selected]

</details>

<details>
<summary><strong>Step 3: Set Permissions</strong></summary>

1. Select **"Attach policies directly"**.
2. In the search box, type **"AdministratorAccess"**.
3. Check the box next to **"AdministratorAccess"** policy.
4. Click **"Next"**.

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
>
> *"AdministratorAccess gives this user almost everything except billing and a few root-only actions. It's perfect for learning. In a real production environment, you'd follow the principle of least privilege — but for labs, admin access is fine."*

📸 [Screenshot: Permissions page with "AdministratorAccess" policy checked in the search results]

</details>

<details>
<summary><strong>Step 4: Review and Create</strong></summary>

1. Review the details:
   - User name: `admin-user`
   - Console access: ✅ Enabled
   - Password: ✅ Custom (you'll set it)
   - Permissions: AdministratorAccess
2. Click **"Create user"**.

📸 [Screenshot: Review page showing user details before creation]

</details>

<details>
<summary><strong>Step 5: Save Your Credentials (CRITICAL!)</strong></summary>

1. You'll see a success page with your user details.
2. **SAVE THE FOLLOWING INFORMATION IMMEDIATELY** — you may not see the secret access key again!

   - **Console sign-in URL:** (something like `https://123456789012.signin.aws.amazon.com/console`)
   - **Username:** `admin-user`
   - **Password:** (the one you set)

3. For **Programmatic access**, you'll also need:
   - **Access Key ID**
   - **Secret Access Key**

> <img src="https://img.shields.io/badge/Warning-Important-E74C3C?style=flat-square" />
>
> **CRITICAL: Click "Download .csv" to save your credentials!** The Secret Access Key is ONLY shown once. If you lose it, you'll have to create a new one. Store this file somewhere safe — like a password manager or encrypted storage. NOT on your desktop in plain text!

📸 [Screenshot: Success page with credential download button and access keys visible (blurred)]

</details>

<details>
<summary><strong>Step 6: Sign Out of Root</strong></summary>

1. Click your account name in the top-right corner of the AWS Console.
2. Click **"Sign Out"**.

📸 [Screenshot: AWS console dropdown with "Sign Out" option highlighted]

</details>

<details>
<summary><strong>Step 7: Sign In as IAM User</strong></summary>

1. Go to **https://aws.amazon.com**.
2. Click **"Sign In to the Console"**.
3. **Important:** You may need to sign in differently now:
   - If your account has an alias, enter the alias.
   - Or use your **12-digit AWS Account ID** (the one from your root account).
   - Enter **Username:** `admin-user`
   - Enter the **password** you set.
4. Click **"Sign in"**.

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
>
> *"Bookmark the IAM user sign-in URL! It looks like: `https://YOUR_ACCOUNT_ID.signin.aws.amazon.com/console` — this makes signing in much easier."*

📸 [Screenshot: AWS sign-in page with IAM user fields visible and account ID filled in]

</details>

<details>
<summary><strong>Step 8: Verify It Works</strong></summary>

1. Once signed in, click your username in the top-right corner.
2. You should see **"IAM user: admin-user"** (or similar), NOT "Root user".
3. Try navigating to a few services (EC2, S3) to make sure you have access.

> 🎉 **Ravi:** *"I'm in as admin-user! No more root for me!"*
>
> **Rithu:** *"That's the spirit! Your root account is now locked away safely with MFA. From here on, you'll use this admin-user for all our labs."*

📸 [Screenshot: AWS console showing "IAM user: admin-user" in the top-right corner]

</details>

---

## 5. Setting Up Billing Alerts (AWS Budgets)

<img src="https://img.shields.io/badge/Section%205-Budgets%20%26%20Billing-F39C12?style=for-the-badge" />

**Rithu:** *"Ravi, this is the section that will save you from crying at the end of the month. Trust me — I've been there."*

**Ravi:** *"You've gotten surprise bills?!"*

**Rithu:** *"Let's just say I once left an EC2 instance running over a weekend and learned a very expensive lesson. Setting up a billing budget takes 5 minutes and can prevent a $500+ surprise. Let's do it NOW."*

### Why This Is Critical

- AWS bills by usage — there's no "auto-shutoff" if you exceed free tier.
- It's **very easy** to accidentally exceed free tier limits.
- A **$1 budget** with an 80% alert will email you BEFORE you spend real money.
- **This is not optional. Do not skip this step.**

### Step-by-Step Instructions

<details>
<summary><strong>Step 1: Navigate to AWS Budgets</strong></summary>

1. Make sure you're signed in as your **IAM admin user**.
2. In the AWS Console search bar, type **"Budgets"**.
3. Click **"AWS Budgets"** from the results.

📸 [Screenshot: AWS Console search results showing "Budgets" highlighted]

</details>

<details>
<summary><strong>Step 2: Create a New Budget</strong></summary>

1. On the AWS Budgets page, click the orange **"Create budget"** button.

📸 [Screenshot: AWS Budgets dashboard with "Create budget" button highlighted]

</details>

<details>
<summary><strong>Step 3: Choose Budget Type</strong></summary>

1. Select **"Cost budget"** (this tracks your actual spending).
2. Click **"Next"**.

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
>
> *"There are different budget types — cost, usage, and reservation. For our purposes, cost is what we want. It tracks your actual dollar spending."*

📸 [Screenshot: Budget type selection with "Cost budget" selected]

</details>

<details>
<summary><strong>Step 4: Set Budget Details</strong></summary>

1. **Budget name:** Enter `Monthly Lab Budget`.
2. **Budget amount:** Enter `1.00` (yes, just one dollar).
3. **Budget period:** Select **"Monthly"**.
4. Click **"Next"**.

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
>
> *"$1 is very conservative. You can set this higher if you want — maybe $5 or $10 — but starting at $1 means you'll get alerted super early. The goal is to get notified BEFORE you spend too much."*

📸 [Screenshot: Budget details form with $1.00 amount and Monthly period selected]

</details>

<details>
<summary><strong>Step 5: Set Alert Threshold</strong></summary>

1. Under **"Alert conditions"**, set the threshold:
   - **Alert at:** `80` **% of budget**.
   - This means you'll get an alert when you've spent $0.80 (80% of your $1 budget).
2. **Notification options:**
   - Check **"Email"**.
   - Enter your **email address** in the field.
   - You can add additional email addresses if you want (e.g., a friend or mentor).
3. Click **"Next"**.

📸 [Screenshot: Alert threshold set to 80% with email notification field filled in]

</details>

<details>
<summary><strong>Step 6: Review and Create</strong></summary>

1. Review your budget:
   - Name: Monthly Lab Budget
   - Amount: $1.00/monthly
   - Alert: 80% → email notification
2. Click **"Create budget"**.

📸 [Screenshot: Budget review page showing all configured details]

</details>

<details>
<summary><strong>Step 7: Confirm Budget Is Active</strong></summary>

1. You should see your new budget listed on the Budgets dashboard.
2. The **Status** should show as **"Active"**.

> <img src="https://img.shields.io/badge/Info-Good%20to%20Know-3498DB?style=flat-square" />
>
> **Bonus Tip:** You can also set up **AWS Billing Alerts** in CloudWatch for even more granular notifications, but the Budgets approach is simpler and sufficient for beginners.

> 🎉 **Ravi:** *"Done! Now I won't accidentally spend my grocery money on AWS!"*
>
> **Rithu:** *"That's the goal! Now let's move on to the tools we'll need for our labs."*

📸 [Screenshot: Budgets dashboard showing the "Monthly Lab Budget" with Active status]

</details>

---

## 6. Installing AWS CLI

<img src="https://img.shields.io/badge/Section%206-AWS%20CLI-1ABC9C?style=for-the-badge" />

**Rithu:** *"The AWS Command Line Interface — or CLI — lets you manage AWS services from your terminal instead of the web console. It's essential for automation and many of our labs."*

**Ravi:** *"I've never used a command line before..."*

**Rithu:** *"Don't worry! I'll walk you through it step by step."*

### Installation Instructions

---

### 🪟 Windows

#### Option A: MSI Installer (Recommended)

1. Open your web browser and go to:
   **https://awscli.amazonaws.com/AWSCLIV2.msi**
   The download should start automatically.

2. Once downloaded, **double-click the MSI file** to run the installer.

3. Follow the installation wizard:
   - Click **"Next"** on the welcome screen.
   - Accept the license agreement → Click **"Next"**.
   - Keep the default installation path → Click **"Next"**.
   - Click **"Install"**.
   - Click **"Finish"** when complete.

4. **Verify the installation:**
   - Open **Command Prompt** (press `Win + R`, type `cmd`, press Enter).
   - Type the following command and press Enter:

   ```
   aws --version
   ```

   - You should see output like:
   ```
   aws-cli/2.x.x Python/3.x.x Windows/10 botocore/2.x.x
   ```

> <img src="https://img.shields.io/badge/Warning-Important-E74C3C?style=flat-square" />
>
> **If you get `'aws' is not recognized as an internal or external command`:**
> Close your current Command Prompt window and open a new one. The installer adds AWS to your PATH, but you need a fresh terminal for it to take effect.

📸 [Screenshot: AWS CLI MSI installer download page]

📸 [Screenshot: Command prompt showing "aws --version" output]

---

#### Option B: Using the AWS CLI MSI Installer URL

If the direct download doesn't work:
1. Go to **https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html**
2. Click the **"AWS CLI v2"** download link for Windows.
3. Follow the same installation steps as Option A.

---

### 🍎 macOS

#### Option A: PKG Installer (Recommended)

1. Download the installer from:
   **https://awscli.amazonaws.com/AWSCLIV2.pkg**

2. **Double-click** the downloaded `.pkg` file.
3. Follow the installation wizard:
   - Click **"Continue"**.
   - Agree to the license → Click **"Agree"**.
   - Click **"Install"**.
   - Enter your Mac password when prompted.
   - Click **"Install Software"**.
   - Click **"Close"** when done.

4. **Verify the installation:**
   - Open **Terminal** (press `Cmd + Space`, type "Terminal", press Enter).
   - Type:

   ```
   aws --version
   ```

   - You should see output like:
   ```
   aws-cli/2.x.x Python/3.x.x macOS/xx.x ...
   ```

---

#### Option B: Using Homebrew

If you have [Homebrew](https://brew.sh/) installed:

```bash
brew install awscli
```

Then verify with:

```bash
aws --version
```

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
>
> *"If you don't have Homebrew and don't want to install it, just use the PKG installer above — it works perfectly!"*

---

### 🐧 Linux

#### Option A: Using pip (Recommended)

1. Make sure you have Python 3 installed:

   ```bash
   python3 --version
   ```

   If Python is not installed, install it first:
   - **Ubuntu/Debian:** `sudo apt update && sudo apt install python3-pip -y`
   - **Amazon Linux/RHEL/CentOS:** `sudo yum install python3-pip -y`

2. Install AWS CLI using pip:

   ```bash
   pip3 install awscli --break-system-packages
   ```

   > If you get a "break-system-packages" error on newer Python versions, use:
   > ```bash
   > pip3 install awscli --user
   > ```
   > Then add `~/.local/bin` to your PATH:
   > ```bash
   > echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
   > source ~/.bashrc
   > ```

3. **Verify the installation:**

   ```bash
   aws --version
   ```

---

#### Option B: Using the Bundled Installer

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

---

#### Option C: Using Snap (Ubuntu)

```bash
sudo snap install aws-cli --classic
aws --version
```

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
>
> *"On Linux, I recommend the bundled installer (Option B) or pip (Option A) for the cleanest install. The snap version sometimes lags behind in updates."*

📸 [Screenshot: Terminal showing successful AWS CLI installation output]

---

## 7. Configuring AWS CLI

<img src="https://img.shields.io/badge/Section%207-CLI%20Config-3498DB?style=for-the-badge" />

**Rithu:** *"Installing the CLI is only half the battle — now we need to tell it WHO you are and WHERE you want to connect."*

**Ravi:** *"Like logging in?"*

**Rithu:** *"Exactly! But instead of typing your username and password every time, the CLI uses access keys. Let's set it up."*

### Prerequisites

Before configuring, you need your **Access Key ID** and **Secret Access Key** from the admin user you created in Section 4. If you downloaded the `.csv` file, open it now.

> <img src="https://img.shields.io/badge/Warning-Important-E74C3C?style=flat-square" />
>
> **Don't have your access keys?** Here's how to create new ones:
> 1. Sign in to the AWS Console as your **admin-user**.
> 2. Click your username in the top-right → **"Security credentials"**.
> 3. Scroll to **"Access keys"** section.
> 4. Click **"Create access key"**.
> 5. Select **"Command Line Interface (CLI)"** as the use case.
> 6. Check the confirmation box.
> 7. Click **"Create access key"**.
> 8. **Copy and save both the Access Key ID and Secret Access Key** — the Secret is only shown once!

📸 [Screenshot: IAM Security credentials page showing "Create access key" button]

---

### Step-by-Step Configuration

<details>
<summary><strong>Step 1: Open Your Terminal</strong></summary>

- **Windows:** Open Command Prompt or PowerShell.
- **macOS:** Open Terminal.
- **Linux:** Open your terminal emulator.

</details>

<details>
<summary><strong>Step 2: Run the Configure Command</strong></summary>

Type the following and press Enter:

```bash
aws configure
```

The CLI will prompt you for four pieces of information:

</details>

<details>
<summary><strong>Step 3: Enter Your Credentials</strong></summary>

**Prompt 1:** `AWS Access Key ID [None]:`
- Paste your **Access Key ID** (looks like `AKIAIOSFODNN7EXAMPLE`).
- Press Enter.

**Prompt 2:** `AWS Secret Access Key [None]:`
- Paste your **Secret Access Key** (looks like `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`).
- Press Enter.
- *Note: Your cursor won't move as you type — this is normal for password-style inputs. The characters are being entered, just not displayed.*

**Prompt 3:** `Default region name [None]:`
- Type `us-east-1` (this is the US East — N. Virginia region, which has the most services and is cheapest).
- Press Enter.

**Prompt 4:** `Default output format [None]:`
- Type `json`
- Press Enter.

</details>

<details>
<summary><strong>Step 4: Verify Configuration</strong></summary>

Type the following and press Enter:

```bash
aws sts get-caller-identity
```

You should see output like:

```json
{
    "UserId": "AIDAEXAMPLEID",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/admin-user"
}
```

> <img src="https://img.shields.io/badge/Info-Good%20to%20Know-3498DB?style=flat-square" />
>
> **If you see this output, you're all set!** The `Arn` field shows `user/admin-user`, confirming you're connected as your IAM user.

> <img src="https://img.shields.io/badge/Warning-Important-E74C3C?style=flat-square" />
>
> **If you get an error:**
> - `Unable to locate credentials` — Re-run `aws configure` and make sure your keys are correct.
> - `InvalidClientTokenId` — Your access key is wrong or has been deleted. Create a new one.
> - `SignatureDoesNotMatch` — Your secret key is wrong. Re-run `aws configure`.

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
>
> *"Where are these credentials stored? On Windows, they're in `C:\Users\YOUR_NAME\.aws\credentials`. On Mac/Linux, they're in `~/.aws/credentials`. You can edit these files directly if needed, but `aws configure` is easier."*

📸 [Screenshot: Terminal showing "aws sts get-caller-identity" output with user ARN visible]

</details>

---

## 8. Installing a Code Editor (VS Code)

<img src="https://img.shields.io/badge/Section%208-VS%20Code-2ECC71?style=for-the-badge" />

**Rithu:** *"We'll be writing some scripts and config files, Ravi. Having a good code editor makes life SO much easier."*

**Ravi:** *"Can't I just use Notepad?"*

**Rithu:** *"You CAN... but VS Code is free, has syntax highlighting, auto-complete, terminal built in, and AWS extensions. It's a no-brainer."*

### Installation

1. Go to **https://code.visualstudio.com/**.
2. Click the big blue **"Download for Windows"** (or Mac/Linux) button.
3. Run the installer and follow the prompts (defaults are fine).
4. Launch VS Code when installed.

### Optional: AWS Extensions

1. Open VS Code.
2. Click the **Extensions** icon in the left sidebar (looks like 4 squares).
3. Search for **"AWS Toolkit"**.
4. Click **"Install"** on the **AWS Toolkit** extension.
5. This gives you AWS Explorer, credential management, and more right inside VS Code.

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
>
> *"This is optional for our labs — we'll mostly use the terminal and AWS Console. But if you plan to write Lambda functions or Infrastructure as Code (IaC) later, VS Code + AWS Toolkit is chef's kiss."*

📸 [Screenshot: VS Code with the AWS Toolkit extension visible in the Extensions sidebar]

---

## 9. SSH Key Pair Setup

<img src="https://img.shields.io/badge/Section%209-SSH%20Keys-E74C3C?style=for-the-badge" />

**Rithu:** *"Ravi, for several labs we'll be connecting to EC2 instances — virtual servers running in the cloud. To do that securely, we need an SSH key pair."*

**Ravi:** *"What's an SSH key?"*

**Rithu:** *"Instead of a password, we use a special file (like a digital key) to prove who we are. Think of it like a house key — only someone with the key can get in. We generate two keys: a PUBLIC key (which we upload to AWS) and a PRIVATE key (which stays on your computer and you NEVER share)."*

### 🪟 Windows Setup

#### Step 1: Install PuTTY

PuTTY is the most common SSH client on Windows.

1. Go to **https://www.putty.org/**.
2. Click **"Download PuTTY"**.
3. Download the **MSI installer** (`putty-64bit-X.XX-installer.msi`).
4. Run the installer and follow the prompts (defaults are fine).

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
>
> *"This installs both PuTTY (SSH client) and PuTTYgen (key generator). You need both!"*

---

#### Step 2: Generate an SSH Key with PuTTYgen

1. Open **PuTTYgen** from your Start Menu.
2. In the PuTTYgen window:
   - Under **"Parameters"** at the bottom, make sure:
     - **Type of key to generate:** `RSA`
     - **Number of bits:** `2048`
3. Click the **"Generate"** button.
4. **Move your mouse randomly** over the blank area in the window — this generates randomness for the key.
5. Once the key is generated, you'll see your public key displayed.

📸 [Screenshot: PuTTYgen window showing the generated public key]

---

#### Step 3: Save Your Keys

1. **Save the private key:**
   - Click **"Save private key"**.
   - A warning will pop up asking if you want to save without a passphrase. For learning purposes, click **"Yes"**.
   - Choose a location and filename, e.g., `C:\Users\YOUR_NAME\Documents\aws-lab-key.ppk`
   - Click **"Save"**.

   > <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
   >
   > *"The .ppk file is PuTTY's format. Keep this file safe — it's like your house key! Anyone with this file can access your EC2 instances."*

2. **Copy the public key:**
   - The public key is displayed at the top of the PuTTYgen window (starts with `ssh-rsa AAAA...`).
   - **Select and copy this entire string.** You'll need it when creating an EC2 key pair in AWS.

📸 [Screenshot: PuTTYgen with "Save private key" button highlighted and public key visible]

---

#### Step 4: Convert .pem to .ppk (If Needed)

If you already have a `.pem` key file (e.g., downloaded from AWS):

1. Open **PuTTYgen**.
2. Click **"Load"**.
3. Change the file type filter to **"All Files (*.*)"**.
4. Select your `.pem` file.
5. Click **"Open"** — PuTTYgen will import the key.
6. Click **"Save private key"** → Save as `.ppk`.

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
>
> *"PuTTY uses .ppk format, while AWS and most Linux tools use .pem format. PuTTYgen lets you convert between them. Got it?"*

📸 [Screenshot: PuTTYgen "Load" dialog showing .pem file import]

---

### 🍎 macOS / 🐧 Linux Setup

#### Step 1: Generate an SSH Key

1. Open **Terminal**.
2. Run the following command:

   ```bash
   ssh-keygen -t rsa -b 2048 -f ~/.ssh/aws-lab-key -C "aws-lab-key"
   ```

3. You'll be prompted:
   - `Enter passphrase (empty for no passphrase):` → Press **Enter** (no passphrase for simplicity).
   - `Enter same passphrase again:` → Press **Enter** again.

4. You now have two files:
   - `~/.ssh/aws-lab-key` (private key — KEEP SECRET)
   - `~/.ssh/aws-lab-key.pub` (public key — this goes to AWS)

---

#### Step 2: Set Correct Permissions

This is CRITICAL on Mac/Linux — SSH will refuse to work with incorrect permissions.

```bash
chmod 400 ~/.ssh/aws-lab-key
```

> <img src="https://img.shields.io/badge/Warning-Important-E74C3C?style=flat-square" />
>
> **Do not skip this step!** If your private key file is readable by others, SSH will give you errors or warnings.

---

#### Step 3: View Your Public Key

```bash
cat ~/.ssh/aws-lab-key.pub
```

Copy this output — you'll need it when creating an EC2 key pair.

📸 [Screenshot: Terminal showing ssh-keygen output and the generated public key]

---

### Using Your Keys with AWS EC2

When creating an EC2 instance in our labs:

1. During instance creation, you'll be prompted to **select or create a key pair**.
2. Choose **"Create a new key pair"**.
3. Give it a name (e.g., `aws-lab-key`).
4. **For PuTTYgen-generated keys:** Upload your `.pub` key (you can copy-paste the public key text).
5. **For ssh-keygen keys:** Upload the `.pub` file.
6. AWS will provide the private key for download (`.pem` format).

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Tip-FFC300?style=flat-square" />
>
> *"When connecting with PuTTY, you'll need to convert the .pem to .ppk (see Step 4 above). When connecting from Mac/Linux, use the .pem directly. We'll cover the exact connection commands in the labs!"*

---

### Summary of Key Formats

<table>
  <thead>
    <tr>
      <th>Platform</th>
      <th>Private Key Format</th>
      <th>Public Key Format</th>
      <th>SSH Client</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Windows</td>
      <td><code>.ppk</code></td>
      <td><code>ssh-rsa AAAA...</code> (text)</td>
      <td>PuTTY</td>
    </tr>
    <tr>
      <td>Mac/Linux</td>
      <td><code>.pem</code></td>
      <td><code>.pub</code> file</td>
      <td>Built-in <code>ssh</code> command</td>
    </tr>
    <tr>
      <td>AWS Console</td>
      <td><code>.pem</code> (downloaded)</td>
      <td>Imported into EC2</td>
      <td>Any</td>
    </tr>
  </tbody>
</table>

> 🎉 **Ravi:** *"OK so my private key stays on my computer, the public key goes to AWS, and that's how they know it's me?"*
>
> **Rithu:** *"Nailed it! 🎯"*

📸 [Screenshot: Summary showing key files in file explorer with private key highlighted as "DO NOT SHARE"]

---

## 10. Understanding Free Tier Limits

<img src="https://img.shields.io/badge/Section%2010-Free%20Tier-F39C12?style=for-the-badge" />

**Rithu:** *"Okay Ravi, this is where a LOT of beginners get burned — literally, by their credit card statements. AWS has a generous Free Tier, but it has very specific limits. You NEED to know these."*

**Ravi:** *"I thought everything was free for the first year?"*

**Rithu:** *"That's the common misconception! There are THREE types of free tiers, and they work very differently:"*

### The Three Types of Free Tier

<table>
  <thead>
    <tr>
      <th>Type</th>
      <th>What It Means</th>
      <th>Duration</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Always Free</strong></td>
      <td>Free forever, regardless of when you signed up</td>
      <td>Indefinite</td>
    </tr>
    <tr>
      <td><strong>12 Months Free</strong></td>
      <td>Free for 12 months after you create your AWS account</td>
      <td>12 months from sign-up</td>
    </tr>
    <tr>
      <td><strong>Free Trials</strong></td>
      <td>One-time free usage for specific services</td>
      <td>Varies (1-6 months)</td>
    </tr>
  </tbody>
</table>

> <img src="https://img.shields.io/badge/Warning-Important-E74C3C?style=flat-square" />
>
> **Critical:** The **12-month free tier starts from the day you create your account** — not from when you start using a service. If you created your account 6 months ago, you only have 6 months of free tier left for those services!

---

### Free Tier Limits by Service

#### Compute

<table>
  <thead>
    <tr>
      <th>Service</th>
      <th>Free Tier Limit</th>
      <th>Duration</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>EC2</strong></td>
      <td>750 hours/month of <code>t2.micro</code> or <code>t3.micro</code></td>
      <td>12 months</td>
      <td>Linux or Windows, in specific regions only</td>
    </tr>
    <tr>
      <td><strong>Lambda</strong></td>
      <td>1M requests/month + 400,000 GB-seconds of compute</td>
      <td>Always Free</td>
      <td>Serverless — no servers to manage</td>
    </tr>
    <tr>
      <td><strong>ECS</strong></td>
      <td>750 hours/month of <code>t2.micro</code> Fargate</td>
      <td>12 months</td>
      <td>Container orchestration</td>
    </tr>
    <tr>
      <td><strong>Lightsail</strong></td>
      <td>750 hours/month of various plans</td>
      <td>3 months</td>
      <td>Simplified VPS — great for beginners</td>
    </tr>
  </tbody>
</table>

---

#### Storage

<table>
  <thead>
    <tr>
      <th>Service</th>
      <th>Free Tier Limit</th>
      <th>Duration</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>S3</strong></td>
      <td>5 GB Standard storage</td>
      <td>Always Free</td>
      <td>Object storage — great for backups and static files</td>
    </tr>
    <tr>
      <td><strong>EBS</strong></td>
      <td>30 GB of gp2/gp3 or Magnetic</td>
      <td>12 months</td>
      <td>Block storage for EC2</td>
    </tr>
    <tr>
      <td><strong>EFS</strong></td>
      <td>5 GB of Standard storage</td>
      <td>12 months</td>
      <td>File storage for EC2</td>
    </tr>
  </tbody>
</table>

---

#### Database

<table>
  <thead>
    <tr>
      <th>Service</th>
      <th>Free Tier Limit</th>
      <th>Duration</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>RDS</strong></td>
      <td>750 hours/month of <code>db.t2.micro</code> (MySQL, PostgreSQL, MariaDB, Oracle BYOL)</td>
      <td>12 months</td>
      <td>20 GB of storage included</td>
    </tr>
    <tr>
      <td><strong>DynamoDB</strong></td>
      <td>25 GB storage + 25 WCU + 25 RCU</td>
      <td>Always Free</td>
      <td>NoSQL database</td>
    </tr>
    <tr>
      <td><strong>ElastiCache</strong></td>
      <td>750 hours/month of <code>cache.t2.micro</code> (Redis or Memcached)</td>
      <td>12 months</td>
      <td>In-memory caching</td>
    </tr>
  </tbody>
</table>

---

#### Networking & Content Delivery

<table>
  <thead>
    <tr>
      <th>Service</th>
      <th>Free Tier Limit</th>
      <th>Duration</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>VPC</strong></td>
      <td>No additional charges</td>
      <td>Always Free</td>
      <td>Virtual networking — always free to create</td>
    </tr>
    <tr>
      <td><strong>CloudFront</strong></td>
      <td>1 TB data transfer out/month</td>
      <td>12 months</td>
      <td>Content Delivery Network</td>
    </tr>
    <tr>
      <td><strong>Route 53</strong></td>
      <td>1 million queries/month</td>
      <td>Always Free</td>
      <td>DNS service</td>
    </tr>
    <tr>
      <td><strong>Data Transfer</strong></td>
      <td>100 GB outbound/month</td>
      <td>12 months</td>
      <td>Outbound = from AWS to internet</td>
    </tr>
  </tbody>
</table>

---

#### Developer Tools & Management

<table>
  <thead>
    <tr>
      <th>Service</th>
      <th>Free Tier Limit</th>
      <th>Duration</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>CloudWatch</strong></td>
      <td>10 custom metrics + 10 alarms + 1M API requests</td>
      <td>Always Free</td>
      <td>Monitoring and alerting</td>
    </tr>
    <tr>
      <td><strong>CloudFormation</strong></td>
      <td>No additional charges</td>
      <td>Always Free</td>
      <td>Infrastructure as Code — you pay only for resources created</td>
    </tr>
    <tr>
      <td><strong>CodeBuild</strong></td>
      <td>100 build minutes/month</td>
      <td>Always Free</td>
      <td>CI/CD builds</td>
    </tr>
    <tr>
      <td><strong>CodeCommit</strong></td>
      <td>5 users + 5 GB storage</td>
      <td>Always Free</td>
      <td>Private Git repositories</td>
    </tr>
    <tr>
      <td><strong>CodeDeploy</strong></td>
      <td>100 deployments/month</td>
      <td>Always Free</td>
      <td>Automated deployments</td>
    </tr>
  </tbody>
</table>

---

#### AI / ML & Analytics

<table>
  <thead>
    <tr>
      <th>Service</th>
      <th>Free Tier Limit</th>
      <th>Duration</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>SageMaker</strong></td>
      <td>250 hours of <code>ml.t2.medium</code></td>
      <td>2 months</td>
      <td>Machine learning platform</td>
    </tr>
    <tr>
      <td><strong>Comprehend</strong></td>
      <td>50K units of text/month</td>
      <td>12 months</td>
      <td>NLP service</td>
    </tr>
    <tr>
      <td><strong>Rekognition</strong></td>
      <td>5K images/month for 12 months</td>
      <td>12 months</td>
      <td>Image analysis</td>
    </tr>
  </tbody>
</table>

---

### 🚨 Common Free Tier Mistakes (DON'T BE RAVI!)

**Rithu:** *"Let me tell you about the mistakes I see beginners make all the time, Ravi:"*

<details>
<summary><strong>1. Leaving EC2 instances running 24/7</strong></summary>

- 750 hours/month = ~31 days of ONE instance
- If you run 2 instances for 15 days = 720 hours (you're fine!)
- But running 2 instances for a full month = 1,440 hours (**OVER THE LIMIT!**)
- **Always stop/terminate instances when not in use!**

</details>

<details>
<summary><strong>2. Using the wrong instance type</strong></summary>

- Only `t2.micro` and `t3.micro` are free tier eligible
- Running a `t3.large` will cost money even within the 12-month window!

</details>

<details>
<summary><strong>3. Storing too much in S3</strong></summary>

- 5 GB is free — sounds small, and it IS small
- Video files, large datasets, and backups add up fast

</details>

<details>
<summary><strong>4. Forgetting about data transfer</strong></summary>

- AWS charges for data going OUT to the internet
- 100 GB/month outbound is free — but uploading is always free

</details>

<details>
<summary><strong>5. Not stopping RDS instances</strong></summary>

- RDS charges by uptime — stop them when not in use
- A `db.t2.micro` running 24/7 for a month = ~720 hours (within 750 limit)
- But forget about it for 2 months = surprise bill!

</details>

> <img src="https://img.shields.io/badge/Tip-Rithu's%20Golden%20Rule-FFC300?style=flat-square" />
>
> *"If you're done using something, STOP it or TERMINATE it. You can always start it again later. AWS charges by the second for most services — so unused resources = wasted money."*

📸 [Screenshot: AWS Cost Explorer dashboard showing a sample monthly breakdown]

---

### How to Check What's Still Free

1. Go to **https://aws.amazon.com/free/**.
2. Sign in with your AWS account.
3. You'll see a personalized view showing:
   - Which free tier services you've used
   - How much of your allocation is remaining
   - Alerts for services approaching their limits

> <img src="https://img.shields.io/badge/Info-Good%20to%20Know-3498DB?style=flat-square" />
>
> **Pro Tip:** Bookmark this page. Check it weekly until you're comfortable with your spending patterns.

📸 [Screenshot: AWS Free Tier page showing usage tracking dashboard]

---

## 11. Ready to Go!

<img src="https://img.shields.io/badge/Section%2011-Ready!-2ECC71?style=for-the-badge" />

**Ravi:** *"Okay Rithu, I've done EVERYTHING. Let me recap:"*

**Rithu:** *"Go for it!"*

**Ravi:**

- ✅ Created an AWS account
- ✅ Enabled MFA on root account
- ✅ Created an IAM admin user
- ✅ Set up a billing budget with email alerts
- ✅ Installed and configured AWS CLI
- ✅ Installed VS Code
- ✅ Generated SSH key pairs
- ✅ Understand free tier limits

**Rithu:** *"Look at you go! 🎉 You're officially ready to start building on AWS!"*

---

### Your Checklist Before Proceeding

Before you start Lab 01, make sure you can:

- [ ] Sign in to the AWS Console as your IAM admin-user
- [ ] Run `aws --version` and see a version number
- [ ] Run `aws sts get-caller-identity` and see your user ARN
- [ ] Confirm your billing budget is active in AWS Budgets
- [ ] Have your SSH key files saved in a safe location
- [ ] Have VS Code installed (or any code editor you prefer)

---

### 🚀 What's Next?

Now that you're all set up, head over to **[Lab 01](./Lab-01/)** to start your first hands-on exercise!

**Rithu:** *"Remember, Ravi — every AWS expert started exactly where you are right now. The difference is they kept going. Let's build something awesome!"*

**Ravi:** *"Let's DO this! 🚀"*

---

> 📚 **Need help?** If you get stuck on any of these prerequisites, reach out before proceeding to the labs. It's better to fix issues now than to debug lab failures later due to misconfigured prerequisites.
>
> 🔗 **Useful links:**
> - [AWS Free Tier page](https://aws.amazon.com/free/)
> - [AWS CLI documentation](https://docs.aws.amazon.com/cli/latest/userguide/getting-started.html)
> - [IAM best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
> - [AWS Budgets documentation](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets.html)

---

<div align="center">

<img src="https://img.shields.io/badge/Setup%20Complete!-You're%20Ready%20to%20Build-2ECC71?style=for-the-badge&labelColor=232F3E" />

**Congratulations! You've completed all the prerequisites.**

*Your AWS environment is secure, your tools are installed, and you're ready to start building on AWS.*

**Next Step →** [Lab 01: Your First AWS Hands-On Exercise](./Lab-01/)

---

<sub>Made with care for the AWS Hands-On Labs</sub>

</div>
