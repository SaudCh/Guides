# AWS Lambda Deployment Guide (Node.js)

This guide provides a comprehensive walkthrough for deploying Node.js applications to **AWS Lambda**. Whether you're deploying a single function or a full API (Express, Fastify), this guide covers the most efficient workflows using **Serverless Framework**, **AWS SAM**, and **GitHub Actions**.

## 📋 Table of Contents

- [Prerequisites](#-prerequisites)
- [Deployment Options](#-deployment-options)
- [Option 1: Serverless Framework (Recommended)](#-option-1-serverless-framework-recommended)
- [Option 2: AWS Serverless Application Model (SAM)](#-option-2-aws-serverless-application-model-sam)
- [Handling Express/Fastify Apps](#-handling-expressfastify-apps)
- [Environment Variables & Secrets](#-environment-variables--secrets)
- [Database Connections (RDS/MongoDB)](#-database-connections-rdsmongodb)
- [CI/CD with GitHub Actions](#-cicd-with-github-actions)
- [Monitoring & Troubleshooting](#-monitoring--troubleshooting)

---

## 🚀 Prerequisites

Before you begin, ensure you have:

- **AWS Account** with administrative access.
- **AWS CLI** installed and configured (`aws configure`).
- **Node.js** (LTS version recommended) and npm/yarn installed.
- **Serverless CLI** (optional but recommended): `npm install -g serverless`.

---

## 🏗️ Deployment Options

| Method                   | Pros                                                       | Best For                                              |
| :----------------------- | :--------------------------------------------------------- | :---------------------------------------------------- |
| **Serverless Framework** | Extremely easy, cloud-agnostic, huge plugin ecosystem.     | Rapid development, startups, cross-cloud flexibility. |
| **AWS SAM**              | Native AWS tool, uses CloudFormation, great local testing. | AWS-heavy environments, complex infrastructure.       |
| **AWS CDK**              | Infrastructure as Code using TypeScript/JavaScript.        | Developers who prefer code over YAML configuration.   |

---

## ⚡ Option 1: Serverless Framework (Recommended)

The Serverless Framework is the most popular way to deploy Node.js to Lambda.

### 1. Initialize Project

```bash
# Create a new service
serverless create --template aws-nodejs --path my-service
cd my-service
npm init -y
```

### 2. Configure `serverless.yml`

Create a `serverless.yml` in your root directory:

```yaml
service: my-node-app

provider:
  name: aws
  runtime: nodejs20.x
  region: us-east-1
  stage: ${opt:stage, 'dev'}
  memorySize: 128 # Default is 1024MB
  timeout: 10 # Seconds

functions:
  hello:
    handler: handler.hello
    events:
      - httpApi:
          path: /hello
          method: get

plugins:
  - serverless-offline # For local development
```

### 3. Deploy

```bash
# Deploy to AWS
serverless deploy

# Deploy a single function (faster for updates)
serverless deploy function -f hello
```

---

## 🛠️ Option 2: AWS Serverless Application Model (SAM)

SAM is AWS's official framework for building serverless applications.

### 1. Initialize Project

```bash
sam init
# Choose 'AWS Quick Start Templates' -> 'Hello World Example' -> 'nodejs20.x'
```

### 2. Build and Deploy

```bash
# Build the application
sam build

# Deploy (guided first time)
sam deploy --guided
```

---

## 🌐 Handling Express/Fastify Apps

You don't need to rewrite your entire Express app to use Lambda. Use a wrapper like `serverless-http`.

### 1. Install Wrapper

```bash
npm install serverless-http
```

### 2. Wrap your App (`app.js`)

```javascript
const express = require("express");
const serverless = require("serverless-http");
const app = express();

app.get("/health", (req, res) => {
  res.status(200).json({ status: "OK" });
});

module.exports.handler = serverless(app);
```

### 3. Update `serverless.yml`

```yaml
functions:
  api:
    handler: app.handler
    events:
      - httpApi: "*" # Catch-all for Express routing
```

---

## 🔑 Environment Variables & Secrets

### Using `serverless.yml`

```yaml
provider:
  environment:
    DB_URL: ${env:DB_URL}
    API_KEY: ${secrets:MY_SECRET_KEY} # If using SSM or Secrets Manager
```

### Best Practice: AWS Secrets Manager

For production apps, always store sensitive data in **AWS Secrets Manager** and fetch it at runtime or via the framework's integration.

---

## 🗄️ Database Connections (RDS/MongoDB)

Lambda functions are ephemeral, so managing standard database connections can be tricky.

### 1. Use Connection Pooling (Correctly)

Initialize your DB client **outside** the handler function to reuse it across requests.

```javascript
// Outside the handler (Cached)
const db = connectToDatabase();

exports.handler = async (event) => {
  // Use cached db connection
  return await db.query(...);
};
```

### 2. Use RDS Proxy

If using RDS (MySQL/PostgreSQL), use **AWS RDS Proxy** to manage connection pooling and prevent "Too many connections" errors.

---

## 🐙 CI/CD with GitHub Actions

Automate your deployments whenever you push to GitHub.

### 1. Add Secrets to GitHub

Go to **Settings -> Secrets -> Actions** and add:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

### 2. Create Workflow (`.github/workflows/deploy.yml`)

```yaml
name: Deploy to Lambda

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Serverless Deploy
        uses: serverless/github-action@v3.2
        with:
          args: deploy --stage prod
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## 📊 Monitoring & Troubleshooting

### 1. CloudWatch Logs

Every Lambda execution is logged to CloudWatch.

- **Log Groups:** `/aws/lambda/<function-name>`
- **Live Tail:** Use `serverless logs -f <function-name> -t` for real-time logs.

### 2. X-Ray Tracing

Enable X-Ray in `serverless.yml` to visualize request flow and identify bottlenecks:

```yaml
provider:
  tracing:
    lambda: true
    apiGateway: true
```

### 3. Cold Starts

If your function is slow on the first request:

- Use **Provisioned Concurrency** (costs extra).
- Keep the package size small (use `esbuild` or `webpack`).
- Increase memory (higher memory = faster CPU).

---

## 🎉 Summary Checklist

- [ ] Choose a framework (Serverless Framework is easiest).
- [ ] Wrap Express apps with `serverless-http`.
- [ ] Store secrets in AWS Secrets Manager.
- [ ] Cache DB connections outside the handler.
- [ ] Set up GitHub Actions for automated deployment.
- [ ] Monitor performance using CloudWatch and X-Ray.

---

_Need help? Refer to the [Official AWS Lambda Docs](https://docs.aws.amazon.com/lambda/) or the [Serverless Framework Documentation](https://www.serverless.com/framework/docs)._
