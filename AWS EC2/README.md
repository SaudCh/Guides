# AWS EC2 Deployment with GitHub Actions & PM2

This guide explains how to deploy a Node.js backend on **AWS EC2** with **GitHub Actions**, **PM2**, and **NGINX**.

## 📋 Table of Contents

- [Prerequisites](#-prerequisites)
- [Step 1: Launch EC2 Instance & Connect](#-step-1-launch-ec2-instance--connect)
- [Step 2: Install Node.js & PM2](#-step-2-install-nodejs--pm2)
- [Step 3: Add Environment Variables to GitHub](#-step-3-add-environment-variables-to-github)
- [Step 4: Setup Self-Hosted Runner](#-step-4-setup-self-hosted-runner)
- [Step 5: Configure GitHub Actions Workflow](#-step-5-configure-github-actions-workflow)
- [Step 6: Manage Application with PM2](#-step-6-manage-application-with-pm2)
- [Step 7: Configure AWS Security Group](#-step-7-configure-aws-security-group)
- [Step 8: Add Domain DNS Record](#-step-8-add-domain-dns-record)
- [Step 9: Install & Configure NGINX](#-step-9-install--configure-nginx)
- [Step 10: Setup SSL with Certbot](#-step-10-setup-ssl-with-certbot)
- [Step 11: Setup CloudWatch for PM2 Logs](#-step-11-setup-cloudwatch-for-pm2-logs)
- [Step 12: Finalize Workflow](#-step-12-finalize-workflow)
- [Monitoring & Alerting](#-monitoring--alerting)
- [Troubleshooting](#-troubleshooting)

---

## 🚀 Prerequisites

- AWS Account with EC2 access
- GitHub repository with your Node.js project
- Domain name (optional but recommended)
- Basic knowledge of Linux commands

---

## 🚀 Step 1: Launch EC2 Instance & Connect

1. **Log in to AWS** and navigate to the EC2 console
2. **Launch a new EC2 instance**:
   - Choose **Ubuntu Server 22.04 LTS** (or latest)
   - Select **t3.micro** (free tier) or appropriate instance type
   - Create or select a key pair for SSH access
   - Configure security group (we'll update this later)
3. **Create an Elastic IP** and associate it with the instance
4. **Connect to the EC2 instance** via SSH:

```bash
# Make your key file secure
chmod 400 "your-key.pem"

# Connect to your instance
ssh -i "your-key.pem" ubuntu@your-elastic-ip
```

5. **Navigate to your project folder** (example for GitHub Actions runner):

```bash
cd actions-runner/_work/Your-Backend/Your-Backend
```

---

## ⚙️ Step 2: Install Node.js & PM2

Run the following commands on your EC2 instance:

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Node.js 20.x
curl -sL https://deb.nodesource.com/setup_20.x -o /tmp/nodesource_setup.sh
sudo bash /tmp/nodesource_setup.sh
sudo apt install -y nodejs

# Verify installation
node -v
npm -v

# Install global packages
sudo npm install -g yarn pm2

# Verify PM2 installation
pm2 --version
```

---

## 🔑 Step 3: Add Environment Variables to GitHub

1. **Go to your GitHub repository** → **Settings** → **Secrets and variables** → **Actions**
2. **Click "New repository secret"**
3. **Create a secret named `PROD_ENV_FILE`**
4. **Add your production `.env` file contents** as the secret value

> ⚠️ **Important**: Never commit `.env` files to your repository. Use GitHub Secrets for sensitive data.

---

## 🖥️ Step 4: Setup Self-Hosted Runner

1. **In your GitHub repository** → **Settings** → **Actions** → **Runners**
2. **Click "New self-hosted runner"**
3. **Select "Linux" and "x64"** architecture
4. **Copy the provided commands** and run them on your EC2 instance:

```bash
# Download and configure the runner (replace with your actual commands)
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64-2.311.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz
echo "YOUR_TOKEN" | ./config.sh --url https://github.com/YOUR_USERNAME/YOUR_REPO --token "YOUR_TOKEN"

# Install and start the service
sudo ./svc.sh install
sudo ./svc.sh start

# Verify the runner is online in GitHub
```

---

## ⚡ Step 5: Configure GitHub Actions Workflow

1. **In your GitHub repository** → **Actions** → **New workflow** → **Node.js** → **Configure**
2. **Replace the default content** with the following workflow file (`.github/workflows/deploy.yml`):

```yaml
name: Deploy to EC2

on:
  push:
    branches: ["main"]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: self-hosted

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          submodules: "true"

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "yarn"
          cache-dependency-path: ./yarn.lock

      - name: Cache dependencies
        uses: actions/cache@v4
        with:
          path: |
            ./.yarn
            ./node_modules
          key: ${{ runner.os }}-yarn-${{ hashFiles('./yarn.lock') }}
          restore-keys: |
            ${{ runner.os }}-yarn-

      - name: Install dependencies
        run: yarn install --frozen-lockfile

      - name: Create environment file
        run: |
          echo "${{ secrets.PROD_ENV_FILE }}" > .env
          echo "Environment file created successfully"

      - name: Build application (if needed)
        run: |
          if [ -f "package.json" ] && grep -q "build" package.json; then
            yarn build
          else
            echo "No build script found, skipping..."
          fi

      - name: Restart application with PM2
        run: |
          pm2 restart all || pm2 start index.js --name "app"
          pm2 save
          pm2 status
```

---

## 🟢 Step 6: Manage Application with PM2

### Basic PM2 Commands

```bash
# Start your application
pm2 start index.js --name "my-app"

# Or if you have a package.json with start script
pm2 start npm --name "my-app" -- start

# View running processes
pm2 status

# View detailed information about an app
pm2 show my-app

# View logs
pm2 logs my-app
pm2 logs my-app --lines 100  # Show last 100 lines

# Restart application
pm2 restart my-app

# Stop application
pm2 stop my-app

# Delete application from PM2
pm2 delete my-app

# Clear all logs
pm2 flush
```

### PM2 Configuration

```bash
# Save current PM2 processes
pm2 save

# Setup PM2 to start on system boot
pm2 startup
# Follow the instructions provided by the command above

# Reload PM2 processes from saved configuration
pm2 resurrect
```

### PM2 Ecosystem File (Optional)

Create `ecosystem.config.js` in your project root:

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
    },
  ],
};
```

Then start with: `pm2 start ecosystem.config.js`

---

## 🌐 Step 7: Configure AWS Security Group

1. **Go to EC2 Console** → **Security Groups**
2. **Select your instance's security group**
3. **Edit Inbound Rules** and add the following:

| Type       | Protocol | Port Range | Source    | Description                      |
| ---------- | -------- | ---------- | --------- | -------------------------------- |
| HTTP       | TCP      | 80         | 0.0.0.0/0 | Web traffic                      |
| HTTPS      | TCP      | 443        | 0.0.0.0/0 | Secure web traffic               |
| SSH        | TCP      | 22         | Your IP   | SSH access                       |
| Custom TCP | TCP      | 3000       | 0.0.0.0/0 | Your app port (adjust as needed) |

> ⚠️ **Security Note**: Restrict SSH access to your IP address only for better security.

---

## 🌍 Step 8: Add Domain DNS Record

1. **Log in to your domain provider** (GoDaddy, Namecheap, etc.)
2. **Navigate to DNS management**
3. **Add an A record**:

   - **Name**: `api` (or subdomain of your choice)
   - **Type**: `A`
   - **Value**: Your Elastic IP address
   - **TTL**: `300` (5 minutes)

4. **Wait for DNS propagation** (can take up to 48 hours, usually much faster)
5. **Test your domain**: `ping api.yourdomain.com`

---

## ⚙️ Step 9: Install & Configure NGINX

### Install NGINX

```bash
sudo apt install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

### Configure NGINX

1. **Backup the default configuration**:

```bash
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.backup
```

2. **Edit the configuration**:

```bash
sudo nano /etc/nginx/sites-available/default
```

3. **Replace with your configuration** (adjust port as needed):

```nginx
server {
    listen 80;
    server_name api.yourdomain.com www.api.yourdomain.com;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;

    # Increase upload limit
    client_max_body_size 100M;

    location / {
        proxy_pass http://localhost:3000; # Adjust to your app port
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        proxy_read_timeout 86400;
    }
}
```

4. **Test and restart NGINX**:

```bash
# Test configuration
sudo nginx -t

# If test passes, restart NGINX
sudo systemctl restart nginx

# Check status
sudo systemctl status nginx
```

---

## 🔒 Step 10: Setup SSL with Certbot

### Install Certbot

```bash
sudo apt install snapd
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### Obtain SSL Certificate

```bash
# Get SSL certificate for your domain
sudo certbot --nginx -d api.yourdomain.com -d www.api.yourdomain.com

# Test automatic renewal
sudo certbot renew --dry-run
```

### Setup Auto-Renewal

```bash
# Add cron job for automatic renewal
sudo crontab -e

# Add this line to run renewal check twice daily
0 12 * * * /usr/bin/certbot renew --quiet
```

### Verify SSL

- Visit `https://api.yourdomain.com`
- Check SSL rating at [SSL Labs](https://www.ssllabs.com/ssltest/)

---

## 📊 Step 11: Setup Monitoring & Logging

For comprehensive monitoring, logging, and alerting, we recommend setting up **AWS CloudWatch**. This provides:

- **Centralized log collection** from PM2, NGINX, and system logs
- **Real-time metrics monitoring** (CPU, memory, disk, network)
- **Automated alerting** with SNS notifications
- **Custom dashboards** for visualization
- **Log analysis** with CloudWatch Insights

### Quick Setup

1. **Install CloudWatch Agent**:

```bash
wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
sudo dpkg -i amazon-cloudwatch-agent.deb
```

2. **Create IAM Role** with CloudWatch permissions
3. **Configure log collection** for PM2 and NGINX logs
4. **Set up alerts** for CPU, memory, and disk usage

### Enhanced PM2 Logging

Update your PM2 ecosystem file for better log management:

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
      // Enhanced logging
      log_type: "json",
      log_date_format: "YYYY-MM-DD HH:mm:ss Z",
      time: true,
      merge_logs: true,
      log_rotate_max_size: "10M",
      log_rotate_retain: 5,
    },
  ],
};
```

### Complete CloudWatch Setup

For detailed CloudWatch configuration including:

- ✅ **Complete agent setup** and configuration
- ✅ **Log group creation** and management
- ✅ **Dashboard creation** and customization
- ✅ **Alarm setup** with SNS notifications
- ✅ **Log Insights queries** for analysis
- ✅ **Custom metrics** from applications
- ✅ **Performance optimization** and troubleshooting

**See the dedicated [AWS CloudWatch Guide](../AWS%20CloudWatch/README.md) for comprehensive monitoring setup.**

---

## ✅ Step 12: Finalize Workflow

The GitHub Actions workflow is already configured to restart your application. The workflow will:

1. ✅ Checkout your code
2. ✅ Install dependencies
3. ✅ Create environment file from secrets
4. ✅ Build application (if needed)
5. ✅ Restart PM2 processes

### Manual Deployment

You can also manually deploy by running:

```bash
# Pull latest changes
git pull origin main

# Install/update dependencies
yarn install

# Restart application
    pm2 restart all
```

---

## 🎉 Deployment Complete!

Your Node.js app is now successfully deployed on AWS EC2 with:

- ✅ **GitHub Actions CI/CD pipeline**
- ✅ **PM2 process manager**
- ✅ **NGINX reverse proxy**
- ✅ **SSL certificate via Certbot**
- ✅ **Automatic deployments on push**
- ✅ **Process monitoring and logging**

## 🐛 Troubleshooting

### Common Issues

**PM2 not found**:

```bash
sudo npm install -g pm2
```

**NGINX not starting**:

```bash
sudo nginx -t  # Check configuration
sudo systemctl status nginx
```

**SSL certificate issues**:

```bash
sudo certbot certificates  # List certificates
sudo certbot renew --force-renewal  # Force renewal
```

**Application not accessible**:

- Check security group rules
- Verify NGINX configuration
- Check PM2 status: `pm2 status`
- View logs: `pm2 logs`

### Useful Commands

```bash
# View all logs
pm2 logs --lines 100

# Monitor resources
pm2 monit

# Restart everything
pm2 restart all

# Check NGINX status
sudo systemctl status nginx

# Test NGINX config
sudo nginx -t
```

---

## 📊 Monitoring & Alerting

For comprehensive monitoring and alerting, we recommend using **AWS CloudWatch**. This provides enterprise-grade monitoring capabilities including:

- **Real-time dashboards** for system and application metrics
- **Automated alerting** with email/SMS notifications
- **Log analysis** with powerful query capabilities
- **Custom metrics** from your applications
- **Performance monitoring** and optimization insights

### Quick Monitoring Setup

1. **Basic health monitoring**:

```bash
# Simple health check script
pm2 status
curl -f http://localhost:3000/health || echo "Health check failed"
```

2. **Resource monitoring**:

```bash
# Check system resources
top -bn1 | head -20
df -h
free -h
```

### Complete Monitoring Solution

For production environments, we strongly recommend setting up **AWS CloudWatch** for:

- ✅ **Centralized log collection** and analysis
- ✅ **Real-time metrics** and dashboards
- ✅ **Automated alerting** and notifications
- ✅ **Performance optimization** insights
- ✅ **Cost monitoring** and optimization
- ✅ **Security monitoring** and compliance

**See the dedicated [AWS CloudWatch Guide](../AWS%20CloudWatch/README.md) for complete monitoring setup including dashboards, alarms, log insights, and custom metrics.**

---

_Need help? Check the [main guides](../README.md) or create an issue in your repository._
