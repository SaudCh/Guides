# AWS CloudWatch - Advanced Monitoring & Alerting

This **advanced guide** explains how to set up comprehensive monitoring, logging, alerting, and performance optimization for your AWS infrastructure using **CloudWatch** with PM2, NGINX, and custom applications.

> 💡 **New to CloudWatch?** Start with the [simple CloudWatch Logs guide](./cloudwatch%20logs.md) for basic logging setup.

## 📋 Table of Contents

- [Prerequisites](#-prerequisites)
- [Step 1: Install CloudWatch Agent](#-step-1-install-cloudwatch-agent)
- [Step 2: Create IAM Role for CloudWatch](#-step-2-create-iam-role-for-cloudwatch)
- [Step 3: Configure CloudWatch Agent](#-step-3-configure-cloudwatch-agent)
- [Step 4: Setup Log Collection](#-step-4-setup-log-collection)
- [Step 5: Configure PM2 Logging](#-step-5-configure-pm2-logging)
- [Step 6: Create CloudWatch Dashboards](#-step-6-create-cloudwatch-dashboards)
- [Step 7: Setup CloudWatch Alarms](#-step-7-setup-cloudwatch-alarms)
- [Step 8: Configure SNS Notifications](#-step-8-configure-sns-notifications)
- [Step 9: Log Insights & Queries](#-step-9-log-insights--queries)
- [Step 10: Custom Metrics](#-step-10-custom-metrics)
- [Monitoring Scripts](#-monitoring-scripts)
- [Performance Optimization](#-performance-optimization)
- [Troubleshooting](#-troubleshooting)

---

## 🚀 Prerequisites

- AWS Account with CloudWatch access
- EC2 instance with your application running
- PM2 process manager installed
- NGINX web server (optional)
- **Basic CloudWatch Logs setup** (see [simple guide](./cloudwatch%20logs.md))
- Intermediate knowledge of AWS services and Linux commands
- Understanding of monitoring concepts and alerting

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

### Alternative Installation Methods

#### Using AWS CLI:

```bash
# Install AWS CLI (if not installed)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Download and install using AWS CLI
aws s3 cp s3://amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb /tmp/
sudo dpkg -i /tmp/amazon-cloudwatch-agent.deb
```

#### Using Package Manager:

```bash
# Add AWS repository
sudo apt update
sudo apt install -y wget
wget -qO - https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb -O /tmp/amazon-cloudwatch-agent.deb
sudo dpkg -i /tmp/amazon-cloudwatch-agent.deb
```

---

## 🔑 Step 2: Create IAM Role for CloudWatch

### Method 1: AWS Console

1. **Go to IAM Console** → **Roles** → **Create role**
2. **Select "EC2"** as the service that will use this role
3. **Attach policies**:
   - `CloudWatchAgentServerPolicy`
   - `CloudWatchLogsFullAccess`
4. **Name the role**: `EC2-CloudWatch-Role`
5. **Create role**

### Method 2: AWS CLI

```bash
# Create IAM role
aws iam create-role \
  --role-name EC2-CloudWatch-Role \
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
  --role-name EC2-CloudWatch-Role \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy

aws iam attach-role-policy \
  --role-name EC2-CloudWatch-Role \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchLogsFullAccess

# Create instance profile
aws iam create-instance-profile --instance-profile-name EC2-CloudWatch-InstanceProfile

# Add role to instance profile
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-CloudWatch-InstanceProfile \
  --role-name EC2-CloudWatch-Role
```

### Attach Role to EC2 Instance

1. **Go to EC2 Console** → **Instances**
2. **Select your instance** → **Actions** → **Security** → **Modify IAM role**
3. **Select**: `EC2-CloudWatch-Role`
4. **Click "Update IAM role"**

---

## ⚙️ Step 3: Configure CloudWatch Agent

### Create Configuration File

Create `/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json`:

```json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/home/ubuntu/.pm2/logs/*.log",
            "log_group_name": "/aws/ec2/pm2",
            "log_stream_name": "{instance_id}-pm2-{hostname}",
            "timezone": "UTC",
            "multi_line_start_pattern": "^\\d{4}-\\d{2}-\\d{2}",
            "timestamp_format": "%Y-%m-%d %H:%M:%S"
          },
          {
            "file_path": "/var/log/nginx/access.log",
            "log_group_name": "/aws/ec2/nginx",
            "log_stream_name": "{instance_id}-nginx-access-{hostname}",
            "timezone": "UTC"
          },
          {
            "file_path": "/var/log/nginx/error.log",
            "log_group_name": "/aws/ec2/nginx",
            "log_stream_name": "{instance_id}-nginx-error-{hostname}",
            "timezone": "UTC"
          },
          {
            "file_path": "/var/log/syslog",
            "log_group_name": "/aws/ec2/system",
            "log_stream_name": "{instance_id}-syslog-{hostname}",
            "timezone": "UTC"
          },
          {
            "file_path": "/var/log/auth.log",
            "log_group_name": "/aws/ec2/system",
            "log_stream_name": "{instance_id}-auth-{hostname}",
            "timezone": "UTC"
          }
        ]
      }
    }
  },
  "metrics": {
    "namespace": "CWAgent",
    "metrics_collected": {
      "cpu": {
        "measurement": [
          "cpu_usage_idle",
          "cpu_usage_iowait",
          "cpu_usage_user",
          "cpu_usage_system"
        ],
        "metrics_collection_interval": 60
      },
      "disk": {
        "measurement": ["used_percent"],
        "metrics_collection_interval": 60,
        "resources": ["*"]
      },
      "diskio": {
        "measurement": ["io_time"],
        "metrics_collection_interval": 60,
        "resources": ["*"]
      },
      "mem": {
        "measurement": ["mem_used_percent"],
        "metrics_collection_interval": 60
      },
      "netstat": {
        "measurement": ["tcp_established", "tcp_time_wait"],
        "metrics_collection_interval": 60
      },
      "swap": {
        "measurement": ["swap_used_percent"],
        "metrics_collection_interval": 60
      },
      "processes": {
        "measurement": ["running", "sleeping", "dead"],
        "metrics_collection_interval": 60
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

# View agent logs
sudo tail -f /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

### Verify Agent Status

```bash
# Check if agent is running
sudo systemctl status amazon-cloudwatch-agent

# Restart agent if needed
sudo systemctl restart amazon-cloudwatch-agent

# Enable auto-start on boot
sudo systemctl enable amazon-cloudwatch-agent
```

---

## 📝 Step 4: Setup Log Collection

### Create CloudWatch Log Groups

```bash
# Create log groups
aws logs create-log-group --log-group-name "/aws/ec2/pm2"
aws logs create-log-group --log-group-name "/aws/ec2/nginx"
aws logs create-log-group --log-group-name "/aws/ec2/system"

# Set retention policy (optional)
aws logs put-retention-policy --log-group-name "/aws/ec2/pm2" --retention-in-days 30
aws logs put-retention-policy --log-group-name "/aws/ec2/nginx" --retention-in-days 14
aws logs put-retention-policy --log-group-name "/aws/ec2/system" --retention-in-days 7
```

### Verify Log Collection

```bash
# Check if logs are being sent to CloudWatch
aws logs describe-log-streams --log-group-name "/aws/ec2/pm2"

# View recent log events
aws logs get-log-events \
  --log-group-name "/aws/ec2/pm2" \
  --log-stream-name "$(aws logs describe-log-streams --log-group-name "/aws/ec2/pm2" --query 'logStreams[0].logStreamName' --output text)"

# List all log streams
aws logs describe-log-streams --log-group-name "/aws/ec2/pm2" --order-by LastEventTime --descending
```

### Log Group Configuration

#### Set up Log Group Policies:

```bash
# Create resource policy for cross-account access (if needed)
aws logs put-resource-policy \
  --policy-name "CloudWatchLogsPolicy" \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
          "AWS": "arn:aws:iam::ACCOUNT-ID:root"
        },
        "Action": [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ],
        "Resource": "arn:aws:logs:REGION:ACCOUNT-ID:log-group:*"
      }
    ]
  }'
```

---

## 🔄 Step 5: Configure PM2 Logging

### Enhanced PM2 Configuration

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
      // Enhanced logging configuration
      log_type: "json",
      log_date_format: "YYYY-MM-DD HH:mm:ss Z",
      error_file: "/home/ubuntu/.pm2/logs/my-app-error.log",
      out_file: "/home/ubuntu/.pm2/logs/my-app-out.log",
      log_file: "/home/ubuntu/.pm2/logs/my-app-combined.log",
      time: true,
      merge_logs: true,
      // Log rotation
      log_rotate_max_size: "10M",
      log_rotate_retain: 5,
      // Structured logging
      log_format: "json",
      // Environment variables for logging
      env_production: {
        NODE_ENV: "production",
        LOG_LEVEL: "info",
        LOG_FORMAT: "json",
      },
    },
  ],
};
```

### Application Logging Setup

#### 1. Install Logging Dependencies

```bash
npm install winston winston-cloudwatch
```

#### 2. Configure Winston Logger (`logger.js`):

```javascript
const winston = require("winston");
const CloudWatchTransport = require("winston-cloudwatch");

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || "info",
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: "my-app" },
  transports: [
    // Console transport
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      ),
    }),
    // CloudWatch transport
    new CloudWatchTransport({
      logGroupName: "/aws/ec2/my-app",
      logStreamName: `${process.env.INSTANCE_ID || "unknown"}-my-app`,
      awsRegion: process.env.AWS_REGION || "us-east-1",
      retentionInDays: 30,
      jsonMessage: true,
    }),
  ],
});

module.exports = logger;
```

#### 3. Use Logger in Application:

```javascript
const logger = require("./logger");

// Info logging
logger.info("Application started", { port: 3000 });

// Error logging
logger.error("Database connection failed", { error: err.message });

// Performance logging
logger.info("Request processed", {
  method: "GET",
  url: "/api/users",
  responseTime: 150,
  statusCode: 200,
});
```

---

## 📊 Step 6: Create CloudWatch Dashboards

### Create Custom Dashboard

1. **Go to CloudWatch Console** → **Dashboards** → **Create dashboard**
2. **Name your dashboard**: `My-App-Monitoring`
3. **Add widgets**:

#### CPU Utilization Widget

```json
{
  "widgets": [
    {
      "type": "metric",
      "x": 0,
      "y": 0,
      "width": 12,
      "height": 6,
      "properties": {
        "metrics": [
          ["CWAgent", "cpu_usage_idle", "InstanceId", "i-1234567890abcdef0"],
          ["CWAgent", "cpu_usage_user", "InstanceId", "i-1234567890abcdef0"],
          ["CWAgent", "cpu_usage_system", "InstanceId", "i-1234567890abcdef0"]
        ],
        "view": "timeSeries",
        "stacked": false,
        "region": "us-east-1",
        "title": "CPU Utilization",
        "period": 300
      }
    }
  ]
}
```

#### Memory Usage Widget

```json
{
  "widgets": [
    {
      "type": "metric",
      "x": 12,
      "y": 0,
      "width": 12,
      "height": 6,
      "properties": {
        "metrics": [
          ["CWAgent", "mem_used_percent", "InstanceId", "i-1234567890abcdef0"]
        ],
        "view": "timeSeries",
        "stacked": false,
        "region": "us-east-1",
        "title": "Memory Usage",
        "period": 300,
        "yAxis": {
          "left": {
            "min": 0,
            "max": 100
          }
        }
      }
    }
  ]
}
```

### Dashboard Creation via CLI

```bash
# Create dashboard
aws cloudwatch put-dashboard \
  --dashboard-name "My-App-Monitoring" \
  --dashboard-body '{
    "widgets": [
      {
        "type": "metric",
        "x": 0,
        "y": 0,
        "width": 12,
        "height": 6,
        "properties": {
          "metrics": [
            ["CWAgent", "cpu_usage_idle", "InstanceId", "i-1234567890abcdef0"],
            ["CWAgent", "cpu_usage_user", "InstanceId", "i-1234567890abcdef0"]
          ],
          "view": "timeSeries",
          "stacked": false,
          "region": "us-east-1",
          "title": "CPU Utilization",
          "period": 300
        }
      }
    ]
  }'
```

---

## 🚨 Step 7: Setup CloudWatch Alarms

### High CPU Usage Alert

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "High-CPU-Utilization" \
  --alarm-description "Alert when CPU exceeds 80%" \
  --metric-name cpu_usage_user \
  --namespace CWAgent \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions "arn:aws:sns:us-east-1:123456789012:EC2-Alerts" \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0
```

### Memory Usage Alert

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "High-Memory-Utilization" \
  --alarm-description "Alert when memory exceeds 85%" \
  --metric-name mem_used_percent \
  --namespace CWAgent \
  --statistic Average \
  --period 300 \
  --threshold 85 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions "arn:aws:sns:us-east-1:123456789012:EC2-Alerts" \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0
```

### Disk Space Alert

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "Low-Disk-Space" \
  --alarm-description "Alert when disk space is below 20%" \
  --metric-name disk_used_percent \
  --namespace CWAgent \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions "arn:aws:sns:us-east-1:123456789012:EC2-Alerts" \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 Name=device,Value=/dev/xvda1
```

### Application Error Rate Alert

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "High-Error-Rate" \
  --alarm-description "Alert when error rate is high" \
  --metric-name ErrorCount \
  --namespace MyApp/Performance \
  --statistic Sum \
  --period 300 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions "arn:aws:sns:us-east-1:123456789012:EC2-Alerts"
```

---

## 📧 Step 8: Configure SNS Notifications

### Create SNS Topic

```bash
# Create SNS topic
aws sns create-topic --name "EC2-Alerts"

# Get topic ARN
aws sns list-topics --query 'Topics[?contains(TopicArn, `EC2-Alerts`)].TopicArn' --output text
```

### Subscribe to Notifications

```bash
# Email subscription
aws sns subscribe \
  --topic-arn "arn:aws:sns:us-east-1:123456789012:EC2-Alerts" \
  --protocol email \
  --notification-endpoint "your-email@example.com"

# SMS subscription
aws sns subscribe \
  --topic-arn "arn:aws:sns:us-east-1:123456789012:EC2-Alerts" \
  --protocol sms \
  --notification-endpoint "+1234567890"

# Slack webhook subscription
aws sns subscribe \
  --topic-arn "arn:aws:sns:us-east-1:123456789012:EC2-Alerts" \
  --protocol https \
  --notification-endpoint "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK"
```

### Test SNS Notifications

```bash
# Send test message
aws sns publish \
  --topic-arn "arn:aws:sns:us-east-1:123456789012:EC2-Alerts" \
  --message "This is a test message from CloudWatch" \
  --subject "CloudWatch Test Alert"
```

---

## 🔍 Step 9: Log Insights & Queries

### CloudWatch Logs Insights Queries

#### 1. Find Error Patterns

```sql
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 100
```

#### 2. Monitor Response Times

```sql
fields @timestamp, @message, response_time
| filter @message like /response_time/
| stats avg(response_time) by bin(5m)
```

#### 3. Track User Activity

```sql
fields @timestamp, @message
| filter @message like /GET/
| stats count() by bin(1h)
```

#### 4. Find Slow Queries

```sql
fields @timestamp, @message
| filter @message like /query/
| filter @message like /slow/
| sort @timestamp desc
| limit 50
```

#### 5. Monitor Authentication Failures

```sql
fields @timestamp, @message
| filter @message like /authentication/
| filter @message like /failed/
| stats count() by bin(1h)
```

### Create Saved Queries

```bash
# Create saved query
aws logs put-query-definition \
  --name "Error Analysis" \
  --query-string "fields @timestamp, @message | filter @message like /ERROR/ | sort @timestamp desc | limit 100" \
  --log-group-names "/aws/ec2/pm2"
```

---

## 📈 Step 10: Custom Metrics

### Send Custom Metrics from Application

```javascript
const AWS = require("aws-sdk");
const cloudwatch = new AWS.CloudWatch({ region: "us-east-1" });

// Send custom metric
const sendMetric = async (metricName, value, unit = "Count") => {
  const params = {
    Namespace: "MyApp/Performance",
    MetricData: [
      {
        MetricName: metricName,
        Value: value,
        Unit: unit,
        Timestamp: new Date(),
        Dimensions: [
          {
            Name: "InstanceId",
            Value: process.env.INSTANCE_ID || "unknown",
          },
          {
            Name: "Environment",
            Value: process.env.NODE_ENV || "production",
          },
        ],
      },
    ],
  };

  try {
    await cloudwatch.putMetricData(params).promise();
    console.log(`Metric ${metricName} sent: ${value}`);
  } catch (error) {
    console.error("Failed to send metric:", error);
  }
};

// Usage examples
sendMetric("ResponseTime", 150, "Milliseconds");
sendMetric("ActiveUsers", 25, "Count");
sendMetric("ErrorRate", 0.02, "Percent");
sendMetric("DatabaseConnections", 5, "Count");
```

### Custom Metric Collection Script

Create `metrics-collector.js`:

```javascript
const os = require("os");
const AWS = require("aws-sdk");

const cloudwatch = new AWS.CloudWatch({
  region: process.env.AWS_REGION || "us-east-1",
});

const collectSystemMetrics = async () => {
  const metrics = [];

  // Memory usage
  const totalMem = os.totalmem();
  const freeMem = os.freemem();
  const usedMemPercent = ((totalMem - freeMem) / totalMem) * 100;

  metrics.push({
    MetricName: "MemoryUsage",
    Value: usedMemPercent,
    Unit: "Percent",
  });

  // CPU load average
  const loadAvg = os.loadavg()[0];
  metrics.push({
    MetricName: "LoadAverage",
    Value: loadAvg,
    Unit: "Count",
  });

  // Network interfaces
  const networkInterfaces = os.networkInterfaces();
  let totalBytesReceived = 0;
  let totalBytesSent = 0;

  Object.keys(networkInterfaces).forEach((interfaceName) => {
    networkInterfaces[interfaceName].forEach((iface) => {
      if (!iface.internal) {
        totalBytesReceived += iface.bytesReceived || 0;
        totalBytesSent += iface.bytesSent || 0;
      }
    });
  });

  metrics.push({
    MetricName: "NetworkBytesReceived",
    Value: totalBytesReceived,
    Unit: "Bytes",
  });

  metrics.push({
    MetricName: "NetworkBytesSent",
    Value: totalBytesSent,
    Unit: "Bytes",
  });

  // Send metrics
  const params = {
    Namespace: "MyApp/System",
    MetricData: metrics.map((metric) => ({
      ...metric,
      Timestamp: new Date(),
      Dimensions: [
        {
          Name: "InstanceId",
          Value: process.env.INSTANCE_ID || "unknown",
        },
      ],
    })),
  };

  try {
    await cloudwatch.putMetricData(params).promise();
    console.log("System metrics sent successfully");
  } catch (error) {
    console.error("Failed to send system metrics:", error);
  }
};

// Run every 5 minutes
setInterval(collectSystemMetrics, 5 * 60 * 1000);

// Run immediately
collectSystemMetrics();
```

---

## 🔧 Monitoring Scripts

### Health Check Script (`health-check.js`)

```javascript
const http = require("http");
const fs = require("fs");

const checkHealth = async () => {
  try {
    // Check if PM2 processes are running
    const { exec } = require("child_process");

    exec("pm2 status --json", (error, stdout) => {
      if (error) {
        console.error("PM2 check failed:", error);
        process.exit(1);
      }

      const processes = JSON.parse(stdout);
      const runningProcesses = processes.filter(
        (p) => p.pm2_env.status === "online"
      );

      if (runningProcesses.length === 0) {
        console.error("No PM2 processes running");
        process.exit(1);
      }

      console.log(`${runningProcesses.length} PM2 processes running`);
    });

    // Check application endpoint
    const options = {
      hostname: "localhost",
      port: 3000,
      path: "/health",
      method: "GET",
      timeout: 5000,
    };

    const req = http.request(options, (res) => {
      if (res.statusCode === 200) {
        console.log("Application health check passed");
        process.exit(0);
      } else {
        console.error("Application health check failed:", res.statusCode);
        process.exit(1);
      }
    });

    req.on("error", (err) => {
      console.error("Health check request failed:", err);
      process.exit(1);
    });

    req.end();
  } catch (error) {
    console.error("Health check failed:", error);
    process.exit(1);
  }
};

checkHealth();
```

### Automated Monitoring Script (`monitor.sh`)

```bash
#!/bin/bash

# Health check script
HEALTH_CHECK_URL="http://localhost:3000/health"
LOG_FILE="/var/log/health-check.log"

# Function to log with timestamp
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> $LOG_FILE
}

# Check PM2 processes
pm2_status=$(pm2 status --json | jq -r '.[] | select(.pm2_env.status != "online") | .name')
if [ ! -z "$pm2_status" ]; then
    log "ERROR: PM2 processes not running: $pm2_status"
    pm2 restart all
    exit 1
fi

# Check application health
if ! curl -f -s $HEALTH_CHECK_URL > /dev/null; then
    log "ERROR: Application health check failed"
    pm2 restart all
    exit 1
fi

# Check disk space
disk_usage=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')
if [ $disk_usage -gt 80 ]; then
    log "WARNING: Disk usage is ${disk_usage}%"
fi

# Check memory usage
memory_usage=$(free | awk 'NR==2{printf "%.0f", $3*100/$2}')
if [ $memory_usage -gt 85 ]; then
    log "WARNING: Memory usage is ${memory_usage}%"
fi

log "Health check passed"
```

### Setup Cron Job for Monitoring

```bash
# Add to crontab
crontab -e

# Add this line to run every 5 minutes
*/5 * * * * /home/ubuntu/monitor.sh

# Make script executable
chmod +x /home/ubuntu/monitor.sh
```

---

## 🚀 Performance Optimization

### Log Aggregation and Analysis

#### 1. Centralized Logging Setup

```bash
# Install log aggregation tools
sudo apt install -y rsyslog

# Configure rsyslog for centralized logging
sudo nano /etc/rsyslog.conf
```

Add to `/etc/rsyslog.conf`:

```
# Send logs to CloudWatch
*.* @@logs.us-east-1.amazonaws.com:514
```

#### 2. Log Rotation Configuration

Create `/etc/logrotate.d/pm2`:

```
/home/ubuntu/.pm2/logs/*.log {
    daily
    missingok
    rotate 7
    compress
    delaycompress
    notifempty
    create 644 ubuntu ubuntu
    postrotate
        pm2 reloadLogs
    endscript
}
```

### CloudWatch Agent Optimization

#### 1. Reduce Log Volume

```json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/home/ubuntu/.pm2/logs/*error*.log",
            "log_group_name": "/aws/ec2/pm2/errors",
            "log_stream_name": "{instance_id}-errors-{hostname}",
            "timezone": "UTC"
          }
        ]
      }
    }
  }
}
```

#### 2. Optimize Metrics Collection

```json
{
  "metrics": {
    "namespace": "CWAgent",
    "metrics_collected": {
      "cpu": {
        "measurement": ["cpu_usage_user", "cpu_usage_system"],
        "metrics_collection_interval": 300
      },
      "mem": {
        "measurement": ["mem_used_percent"],
        "metrics_collection_interval": 300
      }
    }
  }
}
```

---

## 🐛 Troubleshooting

### Common Issues

#### CloudWatch Agent Not Starting

```bash
# Check agent status
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status

# Check agent logs
sudo tail -f /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log

# Restart agent
sudo systemctl restart amazon-cloudwatch-agent
```

#### Logs Not Appearing in CloudWatch

```bash
# Check IAM permissions
aws sts get-caller-identity

# Verify log groups exist
aws logs describe-log-groups --log-group-name-prefix "/aws/ec2"

# Check log streams
aws logs describe-log-streams --log-group-name "/aws/ec2/pm2"
```

#### High CloudWatch Costs

```bash
# Check log group sizes
aws logs describe-log-groups --query 'logGroups[*].{Name:logGroupName,Size:storedBytes}'

# Set retention policies
aws logs put-retention-policy --log-group-name "/aws/ec2/pm2" --retention-in-days 7

# Delete old log groups
aws logs delete-log-group --log-group-name "/aws/ec2/old-logs"
```

### Debug Commands

```bash
# Test CloudWatch connectivity
aws cloudwatch list-metrics --namespace "CWAgent"

# Check agent configuration
sudo cat /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json

# Verify log file permissions
ls -la /home/ubuntu/.pm2/logs/

# Test log streaming
sudo tail -f /home/ubuntu/.pm2/logs/my-app-error.log
```

### Performance Monitoring

```bash
# Check agent resource usage
ps aux | grep amazon-cloudwatch-agent

# Monitor log file sizes
du -sh /home/ubuntu/.pm2/logs/*

# Check disk I/O
iostat -x 1 5
```

---

## ✅ Setup Complete!

Your CloudWatch monitoring system is now fully configured with:

- ✅ **Comprehensive log collection** from PM2, NGINX, and system logs
- ✅ **Real-time metrics monitoring** (CPU, memory, disk, network)
- ✅ **Custom dashboards** for visualization
- ✅ **Automated alerting** with SNS notifications
- ✅ **Log analysis** with CloudWatch Insights
- ✅ **Custom metrics** from your applications
- ✅ **Health monitoring** scripts and automation
- ✅ **Performance optimization** and cost management

### Next Steps

1. **Set up additional alarms** for your specific use cases
2. **Create custom dashboards** for different teams
3. **Implement log-based metrics** for business insights
4. **Set up automated responses** to common issues
5. **Configure log archiving** for long-term storage

---

_Need help? Check the [AWS guides](./README.md) or refer to the [AWS CloudWatch documentation](https://docs.aws.amazon.com/cloudwatch/)._
