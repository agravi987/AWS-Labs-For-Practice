# Launch and Connect to an EC2 Instance

> Launch and connect to an Amazon EC2 instance, then host a custom webpage.

**Category:** Cloud Computing · **Difficulty:** 2/5 · **Status:** not_started

## Milestones

- [ ] **Setup** — 1/2 steps complete
- [ ] **Build** — 0/3 steps complete
- [ ] **Verify & polish** — 0/2 steps complete

## Guide

### Step 1 — Launch an EC2 Instance

- **Objective:** Launch an EC2 instance from the AWS Management Console.
- **Instructions:**
  Go to the AWS Management Console, search for EC2, click on it, then click the 'Launch instance' button. Follow the wizard to launch an instance with the Amazon Linux 2023 AMI and a t2.micro instance type.
- **Verification:** Verify the instance is in a 'running' state and has passed 2/2 status checks.

- [x] Complete this step

### Step 2 — Create and Manage SSH Key Pairs

- **Objective:** Create a new SSH key pair for connecting to the EC2 instance.
- **Instructions:**
  During the instance launch process, create a new key pair named 'first-key-pair' and download the .pem file. For Windows users, convert the .pem file to a .ppk file using PuTTYgen.
- **Verification:** Verify you have the .pem (or .ppk for Windows) file and can use it to connect via SSH.

- [ ] Complete this step

### Step 3 — Connect to the EC2 Instance via SSH

- **Objective:** Establish an SSH connection to the EC2 instance.
- **Instructions:**
  Use the SSH client of your choice (built-in for Mac/Linux, PuTTY for Windows) with the .pem (or .ppk) file to connect to the instance. The username is 'ec2-user'.
- **Verification:** Verify you can execute commands on the instance, such as 'sudo dnf update -y'.

- [ ] Complete this step

### Step 4 — Install Apache Web Server

- **Objective:** Install and start the Apache web server on the EC2 instance.
- **Instructions:**
  Run 'sudo dnf install -y httpd' to install Apache, then 'sudo systemctl start httpd' to start it, and finally 'sudo systemctl enable httpd' to enable it to start automatically on boot.
- **Verification:** Verify Apache is running by checking its status with 'sudo systemctl status httpd'.

- [ ] Complete this step

### Step 5 — Create a Custom Web Page

- **Objective:** Create a simple custom webpage hosted on the EC2 instance.
- **Instructions:**
  Run 'echo "<h1>Hello from Ravi's first EC2 instance!</h1>" | sudo tee /var/www/html/index.html' to create a simple index.html file.
- **Verification:** Verify the webpage is accessible by navigating to 'http://<your-instance-public-ip>' in a web browser.

- [ ] Complete this step

### Step 6 — Verify Security Group Settings

- **Objective:** Ensure the security group allows inbound HTTP traffic.
- **Instructions:**
  Check the security group associated with your instance and verify it has an inbound rule for HTTP (port 80) from 0.0.0.0/0.
- **Verification:** If the rule is missing, add it and then verify your webpage is accessible from the internet.

- [ ] Complete this step

### Step 7 — Cleanup

- **Objective:** Properly terminate the EC2 instance and delete the key pair and security group.
- **Instructions:**
  Terminate the instance from the EC2 console, then delete the key pair and the security group used for this lab.
- **Verification:** Verify all resources created during this lab have been properly cleaned up.

- [ ] Complete this step
