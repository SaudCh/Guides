# AWS EC2 Multi-Branch CI/CD Pipeline

This guide explains how to set up **GitHub Actions CI/CD pipelines** for multiple branches (Development and Staging) deploying to separate AWS EC2 instances with environment-specific configurations.

## 📋 Table of Contents

- [Prerequisites](#-prerequisites)
- [Step 1: Setup EC2 Instances](#-step-1-setup-ec2-instances)
- [Step 2: Configure GitHub Repository](#-step-2-configure-github-repository)
- [Step 3: Setup Self-Hosted Runners](#-step-3-setup-self-hosted-runners)
- [Step 4: Create Environment Variables](#-step-4-create-environment-variables)
- [Step 5: Configure GitHub Actions Workflows](#-step-5-configure-github-actions-workflows)
- [Step 6: Environment-Specific Configuration](#-step-6-environment-specific-configuration)
- [Step 7: Branch Protection Rules](#-step-7-branch-protection-rules)
- [Step 8: Testing the Pipeline](#-step-8-testing-the-pipeline)
- [Troubleshooting](#-troubleshooting)

---

## 🚀 Prerequisites

- AWS Account with EC2 access
- GitHub repository with multiple branches
- Two EC2 instances (or ability to create them)
- Basic knowledge of GitHub Actions and AWS EC2
- PM2 and NGINX installed on target instances

---

## 🖥️ Step 1: Setup EC2 Instances

### Create Development Environment

1. **Launch Development EC2 Instance**:

   - **Instance Type**: `t3.micro` (or appropriate for development)
   - **Name Tag**: `my-app-dev`
   - **Security Group**: Allow SSH (22), HTTP (80), HTTPS (443), and app port (3000)
   - **Key Pair**: Create or select existing key pair

2. **Install Dependencies**:

```bash
# Connect to development instance
ssh -i "your-key.pem" ubuntu@dev-instance-ip

# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js
curl -sL https://deb.nodesource.com/setup_20.x -o /tmp/nodesource_setup.sh
sudo bash /tmp/nodesource_setup.sh
sudo apt install -y nodejs

# Install PM2 and other tools
sudo npm install -g pm2 yarn
```

### Create Staging Environment

1. **Launch Staging EC2 Instance**:

   - **Instance Type**: `t3.small` (or appropriate for staging)
   - **Name Tag**: `my-app-staging`
   - **Security Group**: Same as development
   - **Key Pair**: Same key pair

2. **Install Dependencies** (same as development):

```bash
# Connect to staging instance
ssh -i "your-key.pem" ubuntu@staging-instance-ip

# Follow same installation steps as development
```

### Configure NGINX on Both Instances

```bash
# Install NGINX
sudo apt install -y nginx

# Create environment-specific NGINX configs
sudo nano /etc/nginx/sites-available/my-app-dev
sudo nano /etc/nginx/sites-available/my-app-staging
```

**Development NGINX Config** (`/etc/nginx/sites-available/my-app-dev`):

```nginx
server {
    listen 80;
    server_name dev.myapp.com;  # Replace with your dev domain

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**Staging NGINX Config** (`/etc/nginx/sites-available/my-app-staging`):

```nginx
server {
    listen 80;
    server_name staging.myapp.com;  # Replace with your staging domain

    location / {
        proxy_pass http://localhost:3001;  # Different port for staging
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable configurations:

```bash
# Enable sites
sudo ln -s /etc/nginx/sites-available/my-app-dev /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/my-app-staging /etc/nginx/sites-enabled/

# Remove default site
sudo rm /etc/nginx/sites-enabled/default

# Test and restart NGINX
sudo nginx -t
sudo systemctl restart nginx
```

---

## 🔧 Step 2: Configure GitHub Repository

### Repository Structure

Ensure your repository has the following structure:

```
my-app/
├── .github/
│   └── workflows/
│       ├── deploy-dev.yml
│       ├── deploy-staging.yml
│       └── deploy-production.yml
├── src/
├── package.json
├── ecosystem.config.js
├── .env.example
└── README.md
```

### Create PM2 Ecosystem Files

**Development Ecosystem** (`ecosystem.dev.config.js`):

```javascript
module.exports = {
  apps: [
    {
      name: "my-app-dev",
      script: "src/index.js",
      instances: 1,
      exec_mode: "fork",
      env: {
        NODE_ENV: "development",
        PORT: 3000,
        APP_ENV: "development",
      },
      log_file: "/home/ubuntu/.pm2/logs/my-app-dev.log",
      error_file: "/home/ubuntu/.pm2/logs/my-app-dev-error.log",
      out_file: "/home/ubuntu/.pm2/logs/my-app-dev-out.log",
      time: true,
    },
  ],
};
```

**Staging Ecosystem** (`ecosystem.staging.config.js`):

```javascript
module.exports = {
  apps: [
    {
      name: "my-app-staging",
      script: "src/index.js",
      instances: 1,
      exec_mode: "fork",
      env: {
        NODE_ENV: "staging",
        PORT: 3001,
        APP_ENV: "staging",
      },
      log_file: "/home/ubuntu/.pm2/logs/my-app-staging.log",
      error_file: "/home/ubuntu/.pm2/logs/my-app-staging-error.log",
      out_file: "/home/ubuntu/.pm2/logs/my-app-staging-out.log",
      time: true,
    },
  ],
};
```

---

## 🏃 Step 3: Setup Self-Hosted Runners

### Install Runner on Development Instance

```bash
# Connect to development instance
ssh -i "your-key.pem" ubuntu@dev-instance-ip

# Create runner directory
mkdir actions-runner && cd actions-runner

# Download runner (replace with your actual token and URL)
curl -o actions-runner-linux-x64-2.311.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz

# Extract installer
tar xzf actions-runner-linux-x64-2.311.0.tar.gz

# Configure runner for development
./config.sh --url https://github.com/YOUR_USERNAME/YOUR_REPO --token YOUR_DEV_TOKEN --name "dev-runner" --labels "development,ubuntu,self-hosted"

# Install and start service
sudo ./svc.sh install
sudo ./svc.sh start
```

### Install Runner on Staging Instance

```bash
# Connect to staging instance
ssh -i "your-key.pem" ubuntu@staging-instance-ip

# Follow same steps but with staging token
./config.sh --url https://github.com/YOUR_USERNAME/YOUR_REPO --token YOUR_STAGING_TOKEN --name "staging-runner" --labels "staging,ubuntu,self-hosted"

# Install and start service
sudo ./svc.sh install
sudo ./svc.sh start
```

---

## 🔑 Step 4: Create Environment Variables

### GitHub Repository Secrets

Go to your GitHub repository → **Settings** → **Secrets and variables** → **Actions**

#### Development Secrets:

| Secret Name    | Description              | Example Value                            |
| -------------- | ------------------------ | ---------------------------------------- |
| `DEV_EC2_HOST` | Development instance IP  | `1.2.3.4`                                |
| `DEV_EC2_USER` | SSH username             | `ubuntu`                                 |
| `DEV_SSH_KEY`  | Private SSH key          | `-----BEGIN OPENSSH PRIVATE KEY-----...` |
| `DEV_ENV_FILE` | Development .env content | `NODE_ENV=development\nPORT=3000...`     |

#### Staging Secrets:

| Secret Name        | Description          | Example Value                            |
| ------------------ | -------------------- | ---------------------------------------- |
| `STAGING_EC2_HOST` | Staging instance IP  | `5.6.7.8`                                |
| `STAGING_EC2_USER` | SSH username         | `ubuntu`                                 |
| `STAGING_SSH_KEY`  | Private SSH key      | `-----BEGIN OPENSSH PRIVATE KEY-----...` |
| `STAGING_ENV_FILE` | Staging .env content | `NODE_ENV=staging\nPORT=3001...`         |

### Create .env Files for Each Environment

**Development .env** (`.env.dev`):

```env
NODE_ENV=development
PORT=3000
APP_ENV=development
DATABASE_URL=postgresql://dev_user:dev_pass@localhost:5432/myapp_dev
API_BASE_URL=https://dev-api.myapp.com
LOG_LEVEL=debug
```

**Staging .env** (`.env.staging`):

```env
NODE_ENV=staging
PORT=3001
APP_ENV=staging
DATABASE_URL=postgresql://staging_user:staging_pass@localhost:5432/myapp_staging
API_BASE_URL=https://staging-api.myapp.com
LOG_LEVEL=info
```

---

## ⚙️ Step 5: Configure GitHub Actions Workflows

### Development Workflow (`.github/workflows/deploy-dev.yml`)

```yaml
name: Deploy to Development

on:
  push:
    branches: [development]
  pull_request:
    branches: [development]

jobs:
  deploy-dev:
    runs-on: self-hosted
    environment: development

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "yarn"

      - name: Install dependencies
        run: yarn install --frozen-lockfile

      - name: Run tests
        run: yarn test

      - name: Lint code
        run: yarn lint

      - name: Build application
        run: yarn build

      - name: Create environment file
        run: |
          echo "${{ secrets.DEV_ENV_FILE }}" > .env
          echo "Development environment file created"

      - name: Deploy to Development Server
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.DEV_EC2_HOST }}
          username: ${{ secrets.DEV_EC2_USER }}
          key: ${{ secrets.DEV_SSH_KEY }}
          script: |
            cd /home/ubuntu/my-app
            git pull origin development
            yarn install --production
            echo "${{ secrets.DEV_ENV_FILE }}" > .env
            pm2 restart ecosystem.dev.config.js
            pm2 save

      - name: Verify deployment
        run: |
          echo "Development deployment completed successfully"
          echo "Application should be available at: http://${{ secrets.DEV_EC2_HOST }}:3000"
```

### Staging Workflow (`.github/workflows/deploy-staging.yml`)

```yaml
name: Deploy to Staging

on:
  push:
    branches: [staging]
  workflow_dispatch:
    inputs:
      force_deploy:
        description: "Force deploy even if tests fail"
        required: false
        default: "false"

jobs:
  deploy-staging:
    runs-on: self-hosted
    environment: staging

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "yarn"

      - name: Install dependencies
        run: yarn install --frozen-lockfile

      - name: Run tests
        run: yarn test
        continue-on-error: ${{ github.event.inputs.force_deploy == 'true' }}

      - name: Run linting
        run: yarn lint
        continue-on-error: ${{ github.event.inputs.force_deploy == 'true' }}

      - name: Build application
        run: yarn build

      - name: Run security audit
        run: yarn audit --audit-level moderate
        continue-on-error: ${{ github.event.inputs.force_deploy == 'true' }}

      - name: Create environment file
        run: |
          echo "${{ secrets.STAGING_ENV_FILE }}" > .env
          echo "Staging environment file created"

      - name: Deploy to Staging Server
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.STAGING_EC2_HOST }}
          username: ${{ secrets.STAGING_EC2_USER }}
          key: ${{ secrets.STAGING_SSH_KEY }}
          script: |
            cd /home/ubuntu/my-app
            git pull origin staging
            yarn install --production
            echo "${{ secrets.STAGING_ENV_FILE }}" > .env
            pm2 restart ecosystem.staging.config.js
            pm2 save

      - name: Run smoke tests
        run: |
          echo "Running smoke tests against staging..."
          # Add your smoke test commands here
          curl -f http://${{ secrets.STAGING_EC2_HOST }}:3001/health || exit 1

      - name: Notify deployment
        if: success()
        run: |
          echo "Staging deployment completed successfully"
          echo "Application available at: http://${{ secrets.STAGING_EC2_HOST }}:3001"

      - name: Notify failure
        if: failure()
        run: |
          echo "Staging deployment failed"
          exit 1
```

### Production Workflow (`.github/workflows/deploy-production.yml`)

```yaml
name: Deploy to Production

on:
  push:
    tags:
      - "v*"
  workflow_dispatch:
    inputs:
      tag:
        description: "Tag to deploy"
        required: true
        default: "latest"

jobs:
  deploy-production:
    runs-on: self-hosted
    environment: production

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event.inputs.tag || github.ref }}

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "yarn"

      - name: Install dependencies
        run: yarn install --frozen-lockfile

      - name: Run comprehensive tests
        run: |
          yarn test
          yarn test:integration
          yarn test:e2e

      - name: Security audit
        run: yarn audit --audit-level high

      - name: Build application
        run: yarn build

      - name: Create production environment file
        run: |
          echo "${{ secrets.PROD_ENV_FILE }}" > .env
          echo "Production environment file created"

      - name: Deploy to Production Server
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.PROD_EC2_HOST }}
          username: ${{ secrets.PROD_EC2_USER }}
          key: ${{ secrets.PROD_SSH_KEY }}
          script: |
            cd /home/ubuntu/my-app
            git fetch --all
            git checkout ${{ github.event.inputs.tag || github.ref_name }}
            yarn install --production
            echo "${{ secrets.PROD_ENV_FILE }}" > .env
            pm2 restart ecosystem.production.config.js
            pm2 save

      - name: Health check
        run: |
          echo "Running production health checks..."
          sleep 30  # Wait for app to start
          curl -f http://${{ secrets.PROD_EC2_HOST }}:3000/health || exit 1

      - name: Create GitHub release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.event.inputs.tag || github.ref_name }}
          release_name: Release ${{ github.event.inputs.tag || github.ref_name }}
          body: |
            Production deployment completed successfully.
            - Environment: Production
            - Version: ${{ github.event.inputs.tag || github.ref_name }}
            - Deployed at: ${{ github.run_number }}
          draft: false
          prerelease: false
```

---

## 🔧 Step 6: Environment-Specific Configuration

### Application Configuration

Create environment-specific configuration files:

**Config Structure** (`src/config/index.js`):

```javascript
const config = {
  development: {
    port: process.env.PORT || 3000,
    database: {
      url: process.env.DATABASE_URL || "postgresql://localhost:5432/myapp_dev",
      pool: { min: 2, max: 5 },
    },
    redis: {
      host: "localhost",
      port: 6379,
      db: 0,
    },
    logging: {
      level: "debug",
      format: "combined",
    },
    cors: {
      origin: ["http://localhost:3000", "http://dev.myapp.com"],
      credentials: true,
    },
  },

  staging: {
    port: process.env.PORT || 3001,
    database: {
      url:
        process.env.DATABASE_URL || "postgresql://localhost:5432/myapp_staging",
      pool: { min: 5, max: 20 },
    },
    redis: {
      host: process.env.REDIS_HOST || "localhost",
      port: 6379,
      db: 1,
    },
    logging: {
      level: "info",
      format: "combined",
    },
    cors: {
      origin: ["http://staging.myapp.com"],
      credentials: true,
    },
  },

  production: {
    port: process.env.PORT || 3000,
    database: {
      url: process.env.DATABASE_URL,
      pool: { min: 10, max: 50 },
    },
    redis: {
      host: process.env.REDIS_HOST,
      port: 6379,
      db: 2,
    },
    logging: {
      level: "warn",
      format: "combined",
    },
    cors: {
      origin: ["https://myapp.com"],
      credentials: true,
    },
  },
};

const env = process.env.NODE_ENV || "development";
module.exports = config[env];
```

### Database Migrations

Add database migration scripts to your workflows:

```yaml
- name: Run database migrations
  run: |
    echo "Running database migrations for ${{ env.APP_ENV }}"
    yarn migrate:up
    yarn seed:run
```

### Health Check Endpoints

Ensure your application has health check endpoints:

```javascript
// src/routes/health.js
app.get("/health", (req, res) => {
  res.status(200).json({
    status: "healthy",
    environment: process.env.NODE_ENV,
    timestamp: new Date().toISOString(),
    version: process.env.npm_package_version,
    uptime: process.uptime(),
  });
});

app.get("/health/detailed", async (req, res) => {
  try {
    // Check database connection
    await db.raw("SELECT 1");

    // Check Redis connection
    await redis.ping();

    res.status(200).json({
      status: "healthy",
      environment: process.env.NODE_ENV,
      database: "connected",
      redis: "connected",
      timestamp: new Date().toISOString(),
    });
  } catch (error) {
    res.status(503).json({
      status: "unhealthy",
      error: error.message,
      timestamp: new Date().toISOString(),
    });
  }
});
```

---

## 🛡️ Step 7: Branch Protection Rules

### Setup Branch Protection

1. **Go to GitHub Repository** → **Settings** → **Branches**
2. **Add rule for `development` branch**:

   - ✅ Require a pull request before merging
   - ✅ Require status checks to pass before merging
   - ✅ Require branches to be up to date before merging
   - ✅ Restrict pushes that create files larger than 100 MB

3. **Add rule for `staging` branch**:

   - ✅ Require a pull request before merging
   - ✅ Require status checks to pass before merging
   - ✅ Require branches to be up to date before merging
   - ✅ Include administrators

4. **Add rule for `main` branch**:
   - ✅ Require a pull request before merging
   - ✅ Require status checks to pass before merging
   - ✅ Require branches to be up to date before merging
   - ✅ Include administrators
   - ✅ Require linear history

### Required Status Checks

Configure required status checks for each branch:

- **Development**: `deploy-dev`
- **Staging**: `deploy-staging`
- **Production**: `deploy-production`

---

## 🧪 Step 8: Testing the Pipeline

### Test Development Pipeline

```bash
# Create a feature branch
git checkout -b feature/new-feature

# Make some changes
echo "console.log('New feature added');" >> src/index.js

# Commit and push
git add .
git commit -m "Add new feature"
git push origin feature/new-feature

# Create pull request to development branch
# Merge the PR to trigger development deployment
```

### Test Staging Pipeline

```bash
# Merge development to staging
git checkout staging
git merge development
git push origin staging

# This should trigger staging deployment
```

### Test Production Pipeline

```bash
# Create a release tag
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0

# This should trigger production deployment
```

### Manual Deployment

You can also trigger manual deployments:

1. **Go to GitHub Actions** → **Deploy to Staging**
2. **Click "Run workflow"**
3. **Select branch and options**
4. **Click "Run workflow"**

---

## 🐛 Troubleshooting

### Common Issues

#### Runner Not Available

```bash
# Check runner status
sudo systemctl status actions.runner.*

# Restart runner
sudo systemctl restart actions.runner.*

# Check runner logs
sudo journalctl -u actions.runner.* -f
```

#### Deployment Fails

```bash
# Check PM2 status on target server
pm2 status
pm2 logs

# Check application logs
tail -f /home/ubuntu/.pm2/logs/my-app-*.log

# Check NGINX status
sudo systemctl status nginx
sudo nginx -t
```

#### Environment Variables Not Working

```bash
# Check environment file exists
ls -la .env

# Check environment variables
cat .env

# Restart application with new env
pm2 restart all
```

### Debug Commands

```bash
# Test SSH connection
ssh -i "your-key.pem" ubuntu@your-instance-ip "echo 'SSH working'"

# Test application health
curl -f http://your-instance-ip:3000/health

# Check disk space
df -h

# Check memory usage
free -h
```

### Workflow Debugging

Add debug steps to your workflows:

```yaml
- name: Debug information
  run: |
    echo "Branch: ${{ github.ref_name }}"
    echo "Environment: ${{ env.APP_ENV }}"
    echo "Commit: ${{ github.sha }}"
    echo "Runner: ${{ runner.os }}"
```

---

## ✅ Pipeline Complete!

Your multi-branch CI/CD pipeline is now set up with:

- ✅ **Development branch** → Development server
- ✅ **Staging branch** → Staging server
- ✅ **Production tags** → Production server
- ✅ **Environment-specific** configurations
- ✅ **Automated testing** and deployment
- ✅ **Branch protection** rules
- ✅ **Health checks** and monitoring
- ✅ **Manual deployment** options

### Workflow Summary

1. **Development**: Push to `development` branch → Auto-deploy to dev server
2. **Staging**: Merge to `staging` branch → Auto-deploy to staging server
3. **Production**: Create release tag → Auto-deploy to production server

### Next Steps

1. **Set up monitoring** with CloudWatch for each environment
2. **Configure SSL certificates** for staging and production
3. **Set up automated backups** for databases
4. **Implement blue-green deployments** for zero-downtime updates
5. **Add integration tests** to your workflows

---

_Need help? Check the [main guides](../README.md) or refer to the [GitHub Actions documentation](https://docs.github.com/en/actions)._
