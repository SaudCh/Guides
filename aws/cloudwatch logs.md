# AWS CloudWatch Logs - Simple Setup

This guide explains how to easily send your application logs to **AWS CloudWatch Logs** for centralized logging and basic monitoring.

## 📋 Table of Contents

- [Prerequisites](#-prerequisites)
- [Step 1: Install CloudWatch Agent](#-step-1-install-cloudwatch-agent)
- [Step 2: Create IAM Role](#-step-2-create-iam-role)
- [Step 3: Configure Log Collection](#-step-3-configure-log-collection)
- [Step 4: Setup PM2 Logs](#-step-4-setup-pm2-logs)
- [Step 5: View Logs in CloudWatch](#-step-5-view-logs-in-cloudwatch)
- [Step 6: Basic Log Queries](#-step-6-basic-log-queries)
- [Troubleshooting](#-troubleshooting)

---

## 🚀 Prerequisites

- AWS Account with CloudWatch access
- EC2 instance with your application running
- PM2 process manager installed
- Basic knowledge of Linux commands

---

## 📊 Step 1: Install CloudWatch Agent

### Download and Install

```bash
# Download CloudWatch agent for Ubuntu
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb

# Install the agent
sudo dpkg -i amazon-cloudwatch-agent.deb

# Verify installation
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
```

### Check Installation

```bash
# Check if agent is installed
ls -la /opt/aws/amazon-cloudwatch-agent/

# Check agent version
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -v
```

---

## 🔑 Step 2: Create IAM Role

### Method 1: AWS Console (Recommended)

1. **Go to IAM Console** → **Roles** → **Create role**
2. **Select "EC2"** as the service
3. **Attach policies**:
   - `CloudWatchAgentServerPolicy`
   - `CloudWatchLogsFullAccess`
4. **Name the role**: `EC2-CloudWatch-Logs-Role`
5. **Create role**

### Method 2: AWS CLI

```bash
# Create IAM role
aws iam create-role \
  --role-name EC2-CloudWatch-Logs-Role \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
          "Service": "ec2.amazonaws.com"
        },
        "Action": "sts:AssumeRole"
      }
    ]
  }'

# Attach CloudWatch policies
aws iam attach-role-policy \
  --role-name EC2-CloudWatch-Logs-Role \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy

aws iam attach-role-policy \
  --role-name EC2-CloudWatch-Logs-Role \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchLogsFullAccess
```

### Attach Role to EC2 Instance

1. **Go to EC2 Console** → **Instances**
2. **Select your instance** → **Actions** → **Security** → **Modify IAM role**
3. **Select**: `EC2-CloudWatch-Logs-Role`
4. **Click "Update IAM role"**

---

## ⚙️ Step 3: Configure Log Collection

### Create Simple Configuration

Create `/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json`:

```json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/home/ubuntu/.pm2/logs/*.log",
            "log_group_name": "/aws/ec2/my-app",
            "log_stream_name": "{instance_id}-pm2"
          }
        ]
      }
    }
  }
}
```

### Start CloudWatch Agent

```bash
# Start the agent with configuration
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json \
  -s

# Check status
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
```

### Verify Agent is Running

```bash
# Check agent status
sudo systemctl status amazon-cloudwatch-agent

# View agent logs
sudo tail -f /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

---

## 🔄 Step 4: Setup PM2 Logs

### Configure PM2 for Better Logging

Update your PM2 ecosystem file (`ecosystem.config.js`):

```javascript
module.exports = {
  apps: [
    {
      name: "my-app",
      script: "index.js",
      instances: 1,
      autorestart: true,
      watch: false,
      max_memory_restart: "1G",
      env: {
        NODE_ENV: "production",
      },
      // Simple logging configuration
      log_type: "json",
      log_date_format: "YYYY-MM-DD HH:mm:ss Z",
      time: true,
      merge_logs: true,
    },
  ],
};
```

### Restart PM2 with New Configuration

```bash
# Restart PM2 with new config
pm2 restart ecosystem.config.js

# Check PM2 status
pm2 status

# View PM2 logs
pm2 logs
```

### Test Log Generation

```bash
# Generate some test logs
pm2 logs my-app --lines 10

# Check log files exist
ls -la /home/ubuntu/.pm2/logs/
```

---

## 📊 Step 5: View Logs in CloudWatch

### Create Log Group

```bash
# Create log group
aws logs create-log-group --log-group-name "/aws/ec2/my-app"

# Verify log group exists
aws logs describe-log-groups --log-group-name-prefix "/aws/ec2"
```

### View Logs in AWS Console

1. **Go to CloudWatch Console** → **Logs** → **Log groups**
2. **Click on** `/aws/ec2/my-app`
3. **Click on a log stream** to view logs
4. **Use the search bar** to filter logs

### View Logs via CLI

```bash
# List log streams
aws logs describe-log-streams --log-group-name "/aws/ec2/my-app"

# View recent log events
aws logs get-log-events \
  --log-group-name "/aws/ec2/my-app" \
  --log-stream-name "$(aws logs describe-log-streams --log-group-name "/aws/ec2/my-app" --query 'logStreams[0].logStreamName' --output text)"
```

---

## 🔍 Step 6: Basic Log Queries

### Simple Log Search

1. **Go to CloudWatch Console** → **Logs** → **Log groups**
2. **Select your log group** → **Search log group**
3. **Enter search terms**:
   - `ERROR` - Find error messages
   - `WARN` - Find warning messages
   - `INFO` - Find info messages

### Basic Log Insights Queries

1. **Go to CloudWatch Console** → **Logs** → **Log groups**
2. **Select your log group** → **Query logs with CloudWatch Insights**

#### Find Errors

```sql
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20
```

#### Find Recent Logs

```sql
fields @timestamp, @message
| sort @timestamp desc
| limit 50
```

#### Count Log Levels

```sql
fields @message
| filter @message like /INFO/ or @message like /ERROR/ or @message like /WARN/
| stats count() by @message
```

---

## 🐛 Troubleshooting

### Common Issues

#### Agent Not Starting

```bash
# Check agent status
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status

# Check agent logs
sudo tail -f /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log

# Restart agent
sudo systemctl restart amazon-cloudwatch-agent
```

#### Logs Not Appearing

```bash
# Check IAM permissions
aws sts get-caller-identity

# Verify log group exists
aws logs describe-log-groups --log-group-name "/aws/ec2/my-app"

# Check log files exist
ls -la /home/ubuntu/.pm2/logs/

# Test log file permissions
sudo tail -f /home/ubuntu/.pm2/logs/my-app-out.log
```

#### Permission Issues

```bash
# Check if role is attached
aws sts get-caller-identity

# Verify CloudWatch permissions
aws logs describe-log-groups
```

### Debug Commands

```bash
# Test CloudWatch connectivity
aws cloudwatch list-metrics --namespace "AWS/Logs"

# Check agent configuration
sudo cat /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json

# Verify log file permissions
ls -la /home/ubuntu/.pm2/logs/
```

### Quick Fixes

```bash
# Restart everything
sudo systemctl restart amazon-cloudwatch-agent
pm2 restart all

# Check status
pm2 status
sudo systemctl status amazon-cloudwatch-agent

# View recent logs
pm2 logs --lines 20
```

---

## ✅ Setup Complete!

Your logs are now being sent to CloudWatch! You can:

- ✅ **View logs** in the AWS Console
- ✅ **Search logs** by keywords
- ✅ **Query logs** with basic filters
- ✅ **Monitor** your application in real-time

### Next Steps

1. **Set up log retention** to manage costs
2. **Create log-based metrics** for monitoring
3. **Set up basic alerts** for errors
4. **Explore advanced features** in the [full CloudWatch guide](./cloudwatch.md)

### Log Retention

```bash
# Set log retention to 7 days (to save costs)
aws logs put-retention-policy --log-group-name "/aws/ec2/my-app" --retention-in-days 7
```

---

_Need help? Check the [AWS guides](./README.md) or refer to the [AWS CloudWatch Logs documentation](https://docs.aws.amazon.com/cloudwatch/latest/logs/)._
