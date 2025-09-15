# AWS S3 Setup  

This guide explains how to configure an **AWS S3 bucket**, **IAM policy**, and **IAM user** for managing assets in the ServiceHub project.  

---

## 🚀 Prerequisites
- AWS account with access to **S3** and **IAM**.  
- Admin permissions to create buckets, policies, and users.  
- Node.js project with `.env` support (e.g., using `dotenv`).  

---

## 📦 Step 1: Create an S3 Bucket
1. Go to **S3** → **Create bucket**.  
2. Enter a Bucket name:  bucket-name (copy and save it)
3. Copy the Region and save it as well
4. Click **Create bucket**.  

---

## 🔑 Step 2: Create an IAM Policy
1. Go to **IAM** → **Policies** → **Create policy**.  
2. Choose **Service** → **S3**.  
3. Allow the following actions:  
- `s3:GetObject`  
- `s3:PutObject`  
- `s3:DeleteObject`  
4. Select **Specific resources** → Add your bucket ARN.  
5. Name the policy: `policy-name`

  
---

## 👤 Step 3: Create an IAM User
1. Go to **IAM** → **Users** → **Create user**.  
2. Enter a username.  
3. Attach the policy `policy-name`.  
4. After creating the user, open it and create an **Access Key**:  
- Choose **Other** as the use case.  
- Copy the **Access Key** and **Secret Key**.  

---

## ⚙️ Step 4: Configure Environment Variables
Add credentials to your project’s `.env`:  

```env
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_REGION=eu-west-2
AWS_S3_BUCKET=servicehub-assets

