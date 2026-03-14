# PM2 Logs with CloudWatch

This guide covers how to configure **PM2** for structured logging and stream those logs to **AWS CloudWatch Logs** for centralized monitoring, querying, and alerting.

> 💡 **New to CloudWatch?** See the [CloudWatch Logs simple setup guide](./cloudwatch%20logs.md) before following this guide.

## 📋 Table of Contents

- [Prerequisites](#-prerequisites)
- [Step 1: Configure PM2 Logging](#-step-1-configure-pm2-logging)
- [Step 2: Enable PM2 Log Rotation](#-step-2-enable-pm2-log-rotation)
- [Step 3: Install CloudWatch Agent](#-step-3-install-cloudwatch-agent)
- [Step 4: IAM Permissions](#-step-4-iam-permissions)
- [Step 5: Configure CloudWatch Agent for PM2](#-step-5-configure-cloudwatch-agent-for-pm2)
- [Step 6: Multi-App & Multi-Instance Setup](#-step-6-multi-app--multi-instance-setup)
- [Step 7: Query Logs with CloudWatch Insights](#-step-7-query-logs-with-cloudwatch-insights)
- [Step 8: Create Alarms for PM2 Errors](#-step-8-create-alarms-for-pm2-errors)
- [Troubleshooting](#-troubleshooting)

---

## 🚀 Prerequisites

- EC2 instance (Ubuntu) with PM2 installed and running
- AWS account with CloudWatch access
- IAM role attached to your EC2 instance (or AWS credentials configured)
- `amazon-cloudwatch-agent` installed (step 3 covers this)

---

## 📝 Step 1: Configure PM2 Logging

### Structured JSON Logging (Recommended)

Using JSON log format makes CloudWatch Insights queries much more powerful. Update your `ecosystem.config.js`:

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

      // Logging
      log_type: "json", // Emit JSON lines — parseable by CloudWatch Insights
      log_date_format: "YYYY-MM-DD HH:mm:ss Z",
      time: true,
      merge_logs: true, // Combine stdout + stderr into one file

      // Log file paths (defaults shown; customise if needed)
      out_file: "/home/ubuntu/.pm2/logs/my-app-out.log",
      error_file: "/home/ubuntu/.pm2/logs/my-app-error.log",
      pid_file: "/home/ubuntu/.pm2/pids/my-app.pid",
    },
  ],
};
```

### Apply the Configuration

```bash
# Reload PM2 with the updated config
pm2 reload ecosystem.config.js --update-env

# Verify logging is working
pm2 logs my-app --lines 20
```

### PM2 Log File Locations

```bash
# Default log directory
ls -lh ~/.pm2/logs/

# Files produced per app (when merge_logs: false)
# my-app-out.log    — stdout
# my-app-error.log  — stderr

# Single merged file (when merge_logs: true)
# my-app.log
```

---

## 🔄 Step 2: Enable PM2 Log Rotation

Log rotation prevents log files from filling up disk and keeps CloudWatch Agent healthy.

### Install pm2-logrotate

```bash
pm2 install pm2-logrotate
```

### Configure Rotation Settings

```bash
# Maximum size before rotating (e.g. 10 MB)
pm2 set pm2-logrotate:max_size 10M

# Number of rotated files to keep
pm2 set pm2-logrotate:retain 7

# Enable compression of rotated files
pm2 set pm2-logrotate:compress true

# Rotation schedule (daily at midnight)
pm2 set pm2-logrotate:rotateInterval "0 0 * * *"

# Append date to rotated file names
pm2 set pm2-logrotate:dateFormat "YYYY-MM-DD_HH-mm-ss"

# Verify settings
pm2 conf pm2-logrotate
```

> ⚠️ **Important**: CloudWatch Agent tails log files by tracking byte offsets. Log rotation is safe as long as new log events are written to the same path (which `pm2-logrotate` handles by default).

---

## 📊 Step 3: Install CloudWatch Agent

```bash
# Download agent for Ubuntu x86_64
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb

# Install
sudo dpkg -i amazon-cloudwatch-agent.deb

# Confirm binary is present
ls /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl
```

---

## 🔑 Step 4: IAM Permissions

The EC2 instance needs permission to publish logs. Attach the following managed policies to the instance's IAM role:

| Policy                        | Purpose                     |
| ----------------------------- | --------------------------- |
| `CloudWatchAgentServerPolicy` | Write metrics and logs      |
| `CloudWatchLogsFullAccess`    | Create log groups / streams |

### AWS Console

1. **IAM** → **Roles** → open the role attached to your EC2 instance
2. **Add permissions** → **Attach policies**
3. Search for and attach both policies above

### AWS CLI

```bash
# Replace EC2-Role-Name with your actual role name
aws iam attach-role-policy \
  --role-name EC2-Role-Name \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy

aws iam attach-role-policy \
  --role-name EC2-Role-Name \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchLogsFullAccess
```

---

## ⚙️ Step 5: Configure CloudWatch Agent for PM2

### Create the Agent Configuration

Create `/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json`:

```json
{
  "agent": {
    "run_as_user": "root"
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/home/ubuntu/.pm2/logs/my-app-out.log",
            "log_group_name": "/ec2/pm2/my-app",
            "log_stream_name": "{instance_id}/stdout",
            "log_format": "json/emf",
            "timezone": "UTC",
            "timestamp_format": "%Y-%m-%d %H:%M:%S"
          },
          {
            "file_path": "/home/ubuntu/.pm2/logs/my-app-error.log",
            "log_group_name": "/ec2/pm2/my-app",
            "log_stream_name": "{instance_id}/stderr",
            "log_format": "json/emf",
            "timezone": "UTC",
            "timestamp_format": "%Y-%m-%d %H:%M:%S"
          }
        ]
      }
    }
  }
}
```

> 💡 `{instance_id}` is replaced at runtime with the actual EC2 instance ID, which is handy for multi-instance setups.

### Start the Agent

```bash
# Load config and start
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json \
  -s

# Confirm it's running
sudo systemctl status amazon-cloudwatch-agent

# Enable auto-start on reboot
sudo systemctl enable amazon-cloudwatch-agent
```

### Verify Logs Are Arriving in CloudWatch

```bash
# Trigger a log line from your app, then check CloudWatch
aws logs get-log-events \
  --log-group-name /ec2/pm2/my-app \
  --log-stream-name "$(curl -s http://169.254.169.254/latest/meta-data/instance-id)/stdout" \
  --limit 10
```

---

## 🔀 Step 6: Multi-App & Multi-Instance Setup

### Multiple PM2 Apps

Add a `collect_list` entry per app (or use a wildcard path):

```json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/home/ubuntu/.pm2/logs/api-server-out.log",
            "log_group_name": "/ec2/pm2/api-server",
            "log_stream_name": "{instance_id}/stdout"
          },
          {
            "file_path": "/home/ubuntu/.pm2/logs/api-server-error.log",
            "log_group_name": "/ec2/pm2/api-server",
            "log_stream_name": "{instance_id}/stderr"
          },
          {
            "file_path": "/home/ubuntu/.pm2/logs/worker-out.log",
            "log_group_name": "/ec2/pm2/worker",
            "log_stream_name": "{instance_id}/stdout"
          },
          {
            "file_path": "/home/ubuntu/.pm2/logs/worker-error.log",
            "log_group_name": "/ec2/pm2/worker",
            "log_stream_name": "{instance_id}/stderr"
          }
        ]
      }
    }
  }
}
```

### PM2 Cluster Mode (Multiple Instances)

When running PM2 in cluster mode, each process writes to a numbered log file. Use a wildcard to capture all of them:

```javascript
// ecosystem.config.js
{
  name: "my-app",
  instances: "max",        // or a specific number
  exec_mode: "cluster",
  merge_logs: false,       // keep per-instance files so you can filter by instance
}
```

```json
{
  "file_path": "/home/ubuntu/.pm2/logs/my-app-out-*.log",
  "log_group_name": "/ec2/pm2/my-app",
  "log_stream_name": "{instance_id}/stdout"
}
```

### Auto-Discover New Log Files

To pick up log files created after the agent starts, set `auto_removal` and use a glob:

```json
{
  "file_path": "/home/ubuntu/.pm2/logs/*.log",
  "log_group_name": "/ec2/pm2/all",
  "log_stream_name": "{instance_id}/{hostname}"
}
```

---

## 🔍 Step 7: Query Logs with CloudWatch Insights

Navigate to **CloudWatch** → **Logs** → **Logs Insights** and select your log group (e.g. `/ec2/pm2/my-app`).

### View Recent Errors

```sql
fields @timestamp, @message
| filter @message like /error|Error|ERROR/
| sort @timestamp desc
| limit 50
```

### Parse JSON Log Fields

If your app emits JSON (e.g. using `winston` or `pino`), use `parse`:

```sql
fields @timestamp, @message
| parse @message '{"level":"*","message":"*"' as level, msg
| filter level = "error"
| sort @timestamp desc
| limit 50
```

### Count Errors Per Hour

```sql
fields @timestamp
| filter @message like /ERROR/
| stats count(*) as errorCount by bin(1h)
| sort @timestamp desc
```

### HTTP 5xx Response Rate

```sql
fields @timestamp, @message
| parse @message '"status":*,' as status
| filter status >= 500
| stats count(*) as count by bin(5m)
```

### Slowest Requests (if your app logs response time)

```sql
fields @timestamp, @message
| parse @message '"responseTime":*,' as responseTime
| sort responseTime desc
| limit 20
```

### PM2 Restart Events

```sql
fields @timestamp, @message
| filter @message like /restarted|restart/
| sort @timestamp desc
| limit 20
```

---

## 🚨 Step 8: Create Alarms for PM2 Errors

### Metric Filter — Count Error Log Lines

```bash
aws logs put-metric-filter \
  --log-group-name /ec2/pm2/my-app \
  --filter-name "PM2ErrorCount" \
  --filter-pattern "ERROR" \
  --metric-transformations \
      metricName=PM2Errors,metricNamespace=PM2/Logs,metricValue=1,defaultValue=0
```

### CloudWatch Alarm on the Metric

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "PM2-High-Error-Rate" \
  --alarm-description "Triggers when PM2 logs more than 10 errors in 5 minutes" \
  --namespace PM2/Logs \
  --metric-name PM2Errors \
  --statistic Sum \
  --period 300 \
  --threshold 10 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:YOUR_ACCOUNT_ID:your-sns-topic \
  --treat-missing-data notBreaching
```

### Metric Filter — PM2 Process Restarts

```bash
aws logs put-metric-filter \
  --log-group-name /ec2/pm2/my-app \
  --filter-name "PM2RestartCount" \
  --filter-pattern "restarted" \
  --metric-transformations \
      metricName=PM2Restarts,metricNamespace=PM2/Logs,metricValue=1,defaultValue=0
```

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "PM2-Unexpected-Restarts" \
  --alarm-description "Triggers when PM2 restarts the app more than 3 times in 5 minutes" \
  --namespace PM2/Logs \
  --metric-name PM2Restarts \
  --statistic Sum \
  --period 300 \
  --threshold 3 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:YOUR_ACCOUNT_ID:your-sns-topic \
  --treat-missing-data notBreaching
```

---

## 🛠️ Troubleshooting

### Logs Not Appearing in CloudWatch

```bash
# 1. Check agent status
sudo systemctl status amazon-cloudwatch-agent

# 2. Check agent log for errors
sudo tail -50 /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log

# 3. Verify the log file path exists and has content
ls -lh ~/.pm2/logs/
tail -20 ~/.pm2/logs/my-app-out.log

# 4. Restart the agent after config changes
sudo systemctl restart amazon-cloudwatch-agent
```

### Permission Denied Errors in Agent Log

```bash
# The agent runs as root but must be able to read PM2 log files
# Give read permission to the PM2 log directory
sudo chmod o+r /home/ubuntu/.pm2/logs/*.log

# Or add the ubuntu user's log path to the agent config explicitly
# and ensure the agent's run_as_user matches
```

### Duplicate Log Entries

This usually means the agent is configured to watch both the wildcard glob and a specific file. Audit your `collect_list` for overlapping paths.

### High Disk Usage from PM2 Logs

```bash
# Check log sizes
du -sh ~/.pm2/logs/*

# Flush PM2 logs manually
pm2 flush

# Verify pm2-logrotate is active
pm2 list
pm2 conf pm2-logrotate
```

### Agent Config Validation

```bash
# Validate config before applying
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json \
  --dry-run
```

---

## 📚 Related Guides

- [CloudWatch Logs — Simple Setup](./cloudwatch%20logs.md)
- [CloudWatch — Advanced Monitoring & Alerting](./cloudwatch.md)
- [EC2 Deployment with PM2](./ec2.md)
