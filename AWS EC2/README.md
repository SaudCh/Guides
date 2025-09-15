# AWS EC2 Deployment with GitHub Actions & PM2

This guide explains how to deploy a Node.js backend on **AWS EC2** with **GitHub Actions**, **PM2**, and **NGINX**.

---

## 🚀 Step 1: Launch EC2 Instance & Connect
1. Log in to **AWS** and launch a new **EC2 instance**.
2. Create an **Elastic IP** and associate it with the instance.
3. Connect to the EC2 instance via SSH:

```bash
chmod 400 "your-key.pem"
ssh -i "your-key.pem" ubuntu@ec2-xx-xxx-xxx-xx.us-east-2.compute.amazonaws.com
```

4. Navigate to your project folder (example for GitHub Actions runner):
```bash
cd actions-runner/_work/Your-Backend/Your-Backend
```

---

## ⚙️ Step 2: Install Node.js & PM2
Run the following commands on your EC2 instance:

```bash
cd ~
curl -sL https://deb.nodesource.com/setup_20.x -o /tmp/nodesource_setup.sh
sudo bash /tmp/nodesource_setup.sh
sudo apt install -y nodejs
node -v

sudo npm install -g yarn
sudo npm install -g pm2
```

---

## 🔑 Step 3: Add Environment Variables to GitHub
1. Go to your **GitHub project** → **Settings** → **Secrets and variables** → **Actions**.
2. Create a new secret.
3. Add your `.env` file as a **repository secret**.

---

## 🖥️ Step 4: Setup Self-Hosted Runner
1. In your GitHub repo, go to **Settings** → **Actions** → **Runners**.
2. Add a **new self-hosted runner** and copy the commands.
3. Run them on your EC2 instance (except the last two). Then run:

```bash
sudo ./svc.sh install
sudo ./svc.sh start
```

---

## ⚡ Step 5: Configure GitHub Actions Workflow
1. Go to **Actions** → **New workflow** → **Node.js** → **Configure**.
2. Add the following workflow file (`.github/workflows/node.js.yml`):

```yaml
name: Node.js CI

on:
  push:
    branches: ["main"]

jobs:
  build:
    runs-on: self-hosted

    strategy:
      matrix:
        node-version: [22.x]

    steps:
      - name: Cache Yarn dependencies
        uses: actions/cache@v3
        with:
          path: |
            ./.yarn
            ./node_modules
          key: ${{ runner.os }}-yarn-${{ hashFiles('./yarn.lock') }}
          restore-keys: |
            ${{ runner.os }}-yarn-

      - uses: actions/checkout@v3
        with:
          submodules: "true"

      - name: List contents of the directory
        run: ls -la

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: "yarn"
          cache-dependency-path: ./yarn.lock

      - name: Install dependencies with Yarn
        working-directory: ./
        run: yarn install --frozen-lockfile

      - run: |
          touch .env
          echo "${{secrets.PROD_ENV_FILE}}" > .env

      - run: |
          pm2 restart all
```

---

## 🟢 Step 6: Manage Application with PM2
```bash
# Install PM2 (if not installed)
sudo npm install -g pm2

# Start app
pm2 start index

# Useful PM2 commands
pm2 show app
pm2 status
pm2 restart app
pm2 stop app
pm2 logs
pm2 flush

# Auto-start on reboot
pm2 startup ubuntu
```

---

## 🌐 Step 7: Configure AWS Security Group
Edit **Inbound Rules** in EC2 Security Group and allow:
- **HTTP** (TCP 80)
- **HTTPS** (TCP 443)
- **Custom TCP** (TCP 5001, both IPv4 & IPv6)

---

## 🌍 Step 8: Add Domain DNS Record
Point your domain (e.g., `api.yourdomain.com`) to your **Elastic IP** in your domain provider’s settings.

---

## ⚙️ Step 9: Install & Configure NGINX
```bash
sudo apt install -y nginx
sudo nano /etc/nginx/sites-available/default
```

Replace with your domain:

```nginx
server {
    server_name api.yourdomain.com www.api.yourdomain.com;

    location / {
        proxy_pass http://localhost:8001; # your app port
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Check and restart NGINX:
```bash
sudo nginx -t
sudo nginx -s reload
```

### Increase Upload Limit
Edit `nginx.conf`:
```bash
sudo nano /etc/nginx/nginx.conf
```
Add inside `http {}`:
```nginx
client_max_body_size 100M;
```

---

## 🔒 Step 9.1: Setup SSL with Certbot
```bash
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python3-certbot-nginx
sudo certbot --nginx -d api.yourdomain.com
```

Renew SSL (90 days):
```bash
sudo certbot renew --dry-run
```

---

## ✅ Step 10: Finalize Workflow
At the end of `node.js.yml`, ensure PM2 restarts automatically:

```yaml
- run: |
    pm2 restart all
```

---

## 🎉 Deployment Complete!
Your Node.js app is now deployed on AWS EC2 with:
- **GitHub Actions (CI/CD)**
- **PM2 process manager**
- **NGINX reverse proxy**
- **SSL certificate via Certbot**
