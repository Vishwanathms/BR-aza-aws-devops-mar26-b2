
# 🧪 Lab Manual: Setup SSM on Amazon Linux 2 EC2

---

## 🎯 Objective

Enable **AWS Systems Manager Session Manager** to securely access an EC2 instance **without SSH**.

---

## 🧰 Prerequisites

* EC2 instance (Amazon Linux 2) already running
* Internet access (via public IP OR NAT Gateway)
* IAM permissions to create roles
* Access to Amazon Web Services console

---

# 🪜 Step 1: Verify SSM Agent on EC2

Amazon Linux 2 usually has SSM Agent pre-installed.

### 🔹 Connect via SSH (one-time)

```bash
ssh ec2-user@<EC2-PUBLIC-IP>
```

### 🔹 Check SSM Agent status

```bash
sudo systemctl status amazon-ssm-agent
```

### ✅ Expected:

```
active (running)
```

---

### 🔹 If not installed / not running

```bash
sudo yum install -y amazon-ssm-agent
sudo systemctl start amazon-ssm-agent
sudo systemctl enable amazon-ssm-agent
```

---

# 🪪 Step 2: Create IAM Role for SSM

Go to IAM → Roles → Create Role

### 🔹 Configuration:

* Trusted entity: EC2
* Permissions:

Attach policy:

```
AmazonSSMManagedInstanceCore
```

👉 This policy allows SSM to:

* Connect to instance
* Run commands
* Collect logs

---

### 🔹 Role Name:

```
EC2-SSM-Role
```

---

# 🔗 Step 3: Attach IAM Role to EC2

1. Go to EC2 → Instances
2. Select your instance
3. Actions → Security → Modify IAM Role
4. Attach:

```
EC2-SSM-Role
```

---

# 🌐 Step 4: Ensure Network Connectivity

SSM requires access to AWS endpoints.

### Option 1 (Simplest)

✔ Instance has:

* Public IP
* Internet Gateway

---

### Option 2 (Private Instance)

✔ Use:

* NAT Gateway
  **OR**
* VPC Endpoints for:

  * `ssm`
  * `ec2messages`
  * `ssmmessages`

---

# 🔍 Step 5: Verify Instance in SSM

Go to:
👉 Systems Manager → Managed Instances

### ✅ Expected:

* Instance should appear as **Managed Instance**

---

# 💻 Step 6: Connect Using Session Manager

Go to:

* EC2 → Instance → Connect → Session Manager

Click:

```
Connect
```

### 🎉 You now have shell access WITHOUT SSH!

---

# 🧪 Step 7: Run Commands via SSM (Optional Lab)

Go to:

* Systems Manager → Run Command

### 🔹 Run:

```bash
echo "Hello from SSM" > /tmp/ssm-test.txt
```

### 🔹 Verify:

```bash
cat /tmp/ssm-test.txt
```

---

# 🔐 Step 8: (Optional) Remove SSH Access

To make environment secure:

* Remove port **22** from security group
* Use only SSM for access

---

# 🧯 Troubleshooting

### ❌ Instance not showing in SSM?

Check:

#### 1. IAM Role

* Must have:

```
AmazonSSMManagedInstanceCore
```

---

#### 2. Agent Running

```bash
sudo systemctl status amazon-ssm-agent
```

---

#### 3. Network

* Internet OR VPC endpoints required

---

#### 4. Check Logs

```bash
sudo cat /var/log/amazon/ssm/amazon-ssm-agent.log
```

---

# 🧱 Architecture Flow

```id="0z47ph"
SSM Console → SSM Service → EC2 SSM Agent → OS Shell
```

---

# 🚀 Bonus Lab Ideas (for your training)

* Patch management using SSM
* Automate software install using Run Command
* Use SSM Parameter Store for secrets
* Session logging to S3

---

# ✅ Final Outcome

You should now:

* Access EC2 without SSH
* Run remote commands
* Manage instance securely
