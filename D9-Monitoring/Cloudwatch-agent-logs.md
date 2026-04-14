
# 🧪 Lab Manual: Export Logs from EC2 (Amazon Linux 2) to CloudWatch

## 🎯 Objective

Stream system and application logs from an EC2 instance running **Amazon Linux 2** to **Amazon CloudWatch** using the CloudWatch Agent.

---

## 🧰 Prerequisites

* AWS account
* EC2 instance running **Amazon Linux 2** via Amazon EC2
* SSH access to EC2
* IAM role with `CloudWatchAgentServerPolicy`

---

## 🏗️ Architecture

```
EC2 (Amazon Linux 2)
        ↓
CloudWatch Agent
        ↓
CloudWatch Logs
```

---

## 🔐 Step 1: Attach IAM Role to EC2

1. Go to IAM Console
2. Create role:

   * Service: EC2
3. Attach policy:

   ```
   CloudWatchAgentServerPolicy
   ```
4. Attach role to EC2 instance

---

## 🖥️ Step 2: Connect to EC2

```bash
ssh ec2-user@<public-ip>
```

---

## 📦 Step 3: Install CloudWatch Agent (Amazon Linux 2)

```bash
sudo yum update -y
sudo yum install amazon-cloudwatch-agent -y
```

Verify installation:

```bash
rpm -qa | grep amazon-cloudwatch-agent
```

---

## ⚙️ Step 4: Create CloudWatch Agent Config

Instead of wizard (better for labs), create config manually:

```bash
sudo nano /opt/aws/amazon-cloudwatch-agent/bin/config.json
```

Paste:

```json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/messages",
            "log_group_name": "ec2-amazon-linux2-logs",
            "log_stream_name": "{instance_id}",
            "timezone": "UTC"
          },
          {
            "file_path": "/var/log/secure",
            "log_group_name": "ec2-security-logs",
            "log_stream_name": "{instance_id}",
            "timezone": "UTC"
          }
        ]
      }
    }
  }
}
```

---

## ▶️ Step 5: Start CloudWatch Agent

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
-a fetch-config \
-m ec2 \
-c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json \
-s
```

Enable at boot:

```bash
sudo systemctl enable amazon-cloudwatch-agent
```

Check status:

```bash
sudo systemctl status amazon-cloudwatch-agent
```

---

## 📊 Step 6: Verify Logs in CloudWatch

1. Open AWS Console

2. Go to **CloudWatch → Logs → Log groups**

3. Check:

   * `ec2-amazon-linux2-logs`
   * `ec2-security-logs`

4. Open log stream (instance ID)

---

## 🧪 Step 7: Test Log Streaming

Generate a test log:

```bash
logger "CloudWatch test log from Amazon Linux 2"
```

Wait ~10–20 seconds → verify in CloudWatch.

---

## 🛠️ Troubleshooting

### 🔍 Check agent logs

```bash
sudo tail -f /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

### ❌ Common Issues

| Issue                 | Cause            | Fix                   |
| --------------------- | ---------------- | --------------------- |
| No logs in CloudWatch | IAM role missing | Attach correct role   |
| Agent not running     | Service stopped  | Restart service       |
| Permission denied     | Wrong file path  | Check log file exists |
| No internet           | Private subnet   | Add NAT Gateway       |

---

## 🚀 Advanced Lab Extensions

### 1. Add Application Logs

Example:

```json
{
  "file_path": "/var/log/myapp.log",
  "log_group_name": "myapp-logs",
  "log_stream_name": "{instance_id}"
}
```

---

### 2. Enable Metrics Collection (Optional)

Add to config:

```json
{
  "metrics": {
    "metrics_collected": {
      "cpu": {
        "measurement": ["cpu_usage_idle", "cpu_usage_user"]
      }
    }
  }
}
```

---

## 📌 Key Takeaways

* CloudWatch Agent replaces old `awslogs`
* IAM role is mandatory
* Amazon Linux 2 logs mainly:

  * `/var/log/messages`
  * `/var/log/secure`
* Logs are near real-time in CloudWatch

---

## 🎓 Lab Checklist

* [ ] IAM role attached
* [ ] Agent installed
* [ ] Config file created
* [ ] Agent started
* [ ] Logs visible in CloudWatch
