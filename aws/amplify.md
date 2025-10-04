# AWS Amplify Hosting for React/Vite Apps

This guide explains how to deploy and host a **React/Vite application** on **AWS Amplify** with automatic CI/CD, custom domains, and environment management.

## 📋 Table of Contents

- [Prerequisites](#-prerequisites)
- [Step 1: Prepare Your React/Vite App](#-step-1-prepare-your-reactvite-app)
- [Step 2: Create AWS Amplify App](#-step-2-create-aws-amplify-app)
- [Step 3: Configure Build Settings](#-step-3-configure-build-settings)
- [Step 4: Set Up Environment Variables](#-step-4-set-up-environment-variables)
- [Step 5: Configure Custom Domain](#-step-5-configure-custom-domain)
- [Step 6: Set Up Branch Protection](#-step-6-set-up-branch-protection)
- [Step 7: Configure Redirects and Headers](#-step-7-configure-redirects-and-headers)
- [Step 8: Monitor and Debug](#-step-8-monitor-and-debug)
- [Advanced Configuration](#-advanced-configuration)
- [Troubleshooting](#-troubleshooting)

---

## 🚀 Prerequisites

- AWS Account with Amplify access
- GitHub repository with your React/Vite project
- Domain name (optional but recommended)
- Node.js and npm/yarn installed locally
- Basic knowledge of React and Vite

---

## 📱 Step 1: Prepare Your React/Vite App

### Create a New Vite React App (if starting fresh)

```bash
# Create new Vite React app
npm create vite@latest my-react-app -- --template react
cd my-react-app

# Install dependencies
npm install

# Install additional dependencies for production
npm install --save-dev @types/node
```

### Optimize Your App for Production

1. **Update `vite.config.js`**:

```js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import path from "path";

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
  build: {
    outDir: "dist",
    sourcemap: false,
    minify: "terser",
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ["react", "react-dom"],
        },
      },
    },
  },
  server: {
    port: 3000,
    host: true,
  },
});
```

2. **Create `.env.example`**:

```env
VITE_APP_TITLE=My React App
VITE_API_URL=https://api.example.com
VITE_APP_VERSION=1.0.0
```

3. **Update `package.json`** scripts:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext js,jsx --report-unused-disable-directives --max-warnings 0",
    "type-check": "tsc --noEmit"
  }
}
```

4. **Create `public/_redirects`** (for SPA routing):

```
/*    /index.html   200
```

---

## 🏗️ Step 2: Create AWS Amplify App

### Method 1: AWS Console

1. **Go to AWS Amplify Console**
2. **Click "New app" → "Host web app"**
3. **Connect your repository**:
   - Choose your Git provider (GitHub, GitLab, Bitbucket)
   - Select your repository
   - Choose the branch (usually `main` or `master`)
4. **Configure app details**:
   - **App name**: `my-react-app`
   - **Environment name**: `main`
   - **Build settings**: We'll configure this next

### Method 2: AWS CLI

```bash
# Install AWS CLI (if not installed)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Configure AWS CLI
aws configure

# Create Amplify app
aws amplify create-app \
  --name "my-react-app" \
  --description "React/Vite application" \
  --repository "https://github.com/username/my-react-app" \
  --platform "WEB"
```

---

## ⚙️ Step 3: Configure Build Settings

### Create `amplify.yml` in your project root:

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - echo "Installing dependencies..."
        - npm ci
    build:
      commands:
        - echo "Building the app..."
        - npm run build
  artifacts:
    baseDirectory: dist
    files:
      - "**/*"
  cache:
    paths:
      - node_modules/**/*
      - .npm/**/*
```

### Alternative: Configure in AWS Console

1. **Go to your Amplify app** → **Build settings**
2. **Edit build specification**:

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - echo "Installing dependencies..."
        - npm ci
        - echo "Installing Vite dependencies..."
        - npm install -g @vitejs/plugin-react
    build:
      commands:
        - echo "Building the app..."
        - npm run build
        - echo "Build completed successfully"
  artifacts:
    baseDirectory: dist
    files:
      - "**/*"
  cache:
    paths:
      - node_modules/**/*
      - .npm/**/*
```

---

## 🔑 Step 4: Set Up Environment Variables

### In AWS Amplify Console:

1. **Go to your app** → **Environment variables**
2. **Add the following variables**:

| Variable Name      | Value                     | Description  |
| ------------------ | ------------------------- | ------------ |
| `VITE_APP_TITLE`   | `My React App`            | App title    |
| `VITE_API_URL`     | `https://api.example.com` | API endpoint |
| `VITE_APP_VERSION` | `1.0.0`                   | App version  |
| `NODE_ENV`         | `production`              | Environment  |

### For Different Environments:

1. **Create branches** for different environments:

   - `main` → Production
   - `staging` → Staging
   - `develop` → Development

2. **Set different environment variables** for each branch

---

## 🌐 Step 5: Configure Custom Domain

### Add Custom Domain:

1. **Go to your app** → **Domain management**
2. **Add domain**:
   - Enter your domain: `myapp.com`
   - Choose subdomain: `www.myapp.com` (optional)
3. **Configure DNS**:
   - Add CNAME record pointing to Amplify domain
   - Wait for SSL certificate provisioning

### DNS Configuration:

| Type  | Name  | Value                        | TTL |
| ----- | ----- | ---------------------------- | --- |
| CNAME | `www` | `d1234567890.cloudfront.net` | 300 |
| A     | `@`   | `192.0.2.1`                  | 300 |

> Replace `d1234567890.cloudfront.net` with your actual Amplify domain

---

## 🛡️ Step 6: Set Up Branch Protection

### Enable Branch Protection:

1. **Go to your app** → **App settings** → **Access control**
2. **Enable branch protection**:
   - Require pull request reviews
   - Require status checks
   - Restrict pushes to protected branches

### Configure Webhooks:

1. **Go to your repository settings** → **Webhooks**
2. **Add webhook**:
   - **Payload URL**: `https://webhook.amplify.us-east-1.amazonaws.com/...`
   - **Content type**: `application/json`
   - **Events**: Push, Pull request

---

## 🔄 Step 7: Configure Redirects and Headers

### Create `public/_redirects`:

```
# SPA routing
/*    /index.html   200

# API redirects
/api/*    https://api.example.com/api/:splat    200

# Custom redirects
/old-page    /new-page    301
```

### Create `public/_headers`:

```
/*
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  X-XSS-Protection: 1; mode=block
  Referrer-Policy: strict-origin-when-cross-origin

/index.html
  Cache-Control: no-cache

/assets/*
  Cache-Control: public, max-age=31536000, immutable
```

---

## 📊 Step 8: Monitor and Debug

### View Build Logs:

1. **Go to your app** → **Build history**
2. **Click on any build** to view detailed logs
3. **Check for errors** in the build process

### Monitor Performance:

1. **Go to your app** → **Monitoring**
2. **View metrics**:
   - Build success rate
   - Deployment frequency
   - Performance metrics

### Debug Common Issues:

```bash
# Check build locally
npm run build

# Test production build
npm run preview

# Check for TypeScript errors
npm run type-check

# Lint your code
npm run lint
```

---

## 🚀 Advanced Configuration

### 1. Multiple Environment Setup

Create different Amplify apps for different environments:

```bash
# Production
aws amplify create-app --name "my-app-prod" --environment "production"

# Staging
aws amplify create-app --name "my-app-staging" --environment "staging"
```

### 2. Custom Build Commands

Update `amplify.yml` for complex builds:

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - echo "Installing dependencies..."
        - npm ci
        - echo "Running type check..."
        - npm run type-check
        - echo "Running linting..."
        - npm run lint
    build:
      commands:
        - echo "Building the app..."
        - npm run build
        - echo "Running post-build tests..."
        - npm run test:ci
  artifacts:
    baseDirectory: dist
    files:
      - "**/*"
  cache:
    paths:
      - node_modules/**/*
      - .npm/**/*
```

### 3. Environment-Specific Builds

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - echo "Installing dependencies..."
        - npm ci
        - echo "Setting up environment..."
        - if [ "$AWS_BRANCH" = "main" ]; then
          echo "Building for production...";
          export VITE_APP_ENV=production;
          elif [ "$AWS_BRANCH" = "staging" ]; then
          echo "Building for staging...";
          export VITE_APP_ENV=staging;
          else
          echo "Building for development...";
          export VITE_APP_ENV=development;
          fi
    build:
      commands:
        - echo "Building the app..."
        - npm run build
  artifacts:
    baseDirectory: dist
    files:
      - "**/*"
```

### 4. Performance Optimization

Add to `vite.config.js`:

```js
export default defineConfig({
  plugins: [react()],
  build: {
    outDir: "dist",
    sourcemap: false,
    minify: "terser",
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ["react", "react-dom"],
          router: ["react-router-dom"],
          ui: ["@mui/material", "@emotion/react"],
        },
      },
    },
    chunkSizeWarningLimit: 1000,
  },
  server: {
    port: 3000,
    host: true,
  },
});
```

---

## 🐛 Troubleshooting

### Common Issues

**Build Fails with "Command not found"**:

```bash
# Check if Node.js is available
node --version
npm --version

# Install specific Node.js version
nvm install 18
nvm use 18
```

**Environment Variables Not Working**:

- Check variable names start with `VITE_`
- Ensure variables are set in Amplify console
- Rebuild the app after adding variables

**Routing Issues (404 on refresh)**:

- Ensure `public/_redirects` file exists
- Check redirect rules are correct
- Verify SPA routing configuration

**Slow Build Times**:

- Enable caching in `amplify.yml`
- Use `npm ci` instead of `npm install`
- Optimize dependencies

### Debug Commands

```bash
# Check build locally
npm run build

# Test production build
npm run preview

# Check bundle size
npx vite-bundle-analyzer

# Check for unused dependencies
npx depcheck
```

### Useful Amplify CLI Commands

```bash
# Install Amplify CLI
npm install -g @aws-amplify/cli

# Initialize Amplify
amplify init

# Add hosting
amplify add hosting

# Deploy
amplify publish

# Check status
amplify status

# View logs
amplify console
```

---

## ✅ Deployment Complete!

Your React/Vite app is now successfully deployed on AWS Amplify with:

- ✅ **Automatic CI/CD** from your Git repository
- ✅ **Custom domain** with SSL certificate
- ✅ **Environment management** for different stages
- ✅ **Performance optimization** and caching
- ✅ **Branch protection** and security
- ✅ **Monitoring and debugging** tools
- ✅ **Scalable hosting** with global CDN

### Next Steps

1. **Set up monitoring** with CloudWatch
2. **Configure custom headers** for security
3. **Set up automated testing** in your CI/CD pipeline
4. **Implement feature flags** for gradual rollouts
5. **Add performance monitoring** with tools like Sentry

---

_Need help? Check the [AWS guides](./README.md) or refer to the [AWS Amplify documentation](https://docs.aws.amazon.com/amplify/)._
