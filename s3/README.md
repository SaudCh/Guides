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
5. Name the policy:  
