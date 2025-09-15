# ServiceHub Assets – AWS S3 Setup  

This guide explains how to configure an **AWS S3 bucket**, **IAM policy**, and **IAM user** for managing assets in the ServiceHub project.  

---

## 🚀 Prerequisites
- AWS account with access to **S3** and **IAM**.  
- Admin permissions to create buckets, policies, and users.  
- Node.js project with `.env` support (e.g., using dotenv).  

---

## 📦 Step 1: Create an S3 Bucket
1. Go to **S3** → **Create bucket**.
3. Enter the Bucket name:  
   ```
   bucket-name
   ```  
4. Region:  
   ```
   region
   ```  
5. Click **Create bucket**.  

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
   ```
   policy-name
   ```  

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
AWS_S3_BUCKET=bucket-name
```

---

## 🔐 Step 5: Update Bucket Permissions
1. Go to **S3** → select your bucket (`bucket-name`).  
2. Open the **Permissions** tab.  
3. Under **Block public access (bucket settings)** → click **Edit** → **Uncheck all options** → Save changes.  
4. Scroll to **Bucket policy** → click **Edit** → paste the following policy:  

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "s3:GetObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::bucket-name/*"
    }
  ]
}
```

> Replace `arn:aws:s3:::bucket-name` with the actual bucket ARN (visible in your bucket’s **Properties** tab).  

---

## 📤 Step 6: Example Usage in Node.js
Install AWS SDK v3:  
```bash
npm install @aws-sdk/client-s3
```

### Upload File
```js
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";
import fs from "fs";

const s3 = new S3Client({
  region: process.env.AWS_REGION,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
  },
});

export const uploadFile = async (filePath, fileName) => {
  const fileContent = fs.readFileSync(filePath);

  const command = new PutObjectCommand({
    Bucket: process.env.AWS_S3_BUCKET,
    Key: fileName,
    Body: fileContent,
    ContentType: "application/octet-stream",
  });

  await s3.send(command);
  console.log(`${fileName} uploaded successfully ✅`);
};
```

Usage:  
```js
uploadFile("./local-file.png", "uploads/file.png");
```

---

## 📥 Download File
```js
import { GetObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";

export const downloadFile = async (fileName) => {
  const command = new GetObjectCommand({
    Bucket: process.env.AWS_S3_BUCKET,
    Key: fileName,
  });

  const url = await getSignedUrl(s3, command, { expiresIn: 3600 });
  console.log(`Download file from: ${url}`);
};
```

---

## 🗑️ Delete File
```js
import { DeleteObjectCommand } from "@aws-sdk/client-s3";

export const deleteFile = async (fileName) => {
  const command = new DeleteObjectCommand({
    Bucket: process.env.AWS_S3_BUCKET,
    Key: fileName,
  });

  await s3.send(command);
  console.log(`${fileName} deleted successfully ❌`);
};
```

---

## ✅ Done!
You now have:  
- An **S3 bucket** (`servicehub-assets`)  
- An **IAM user** with access keys  
- A working setup to **upload, download, and delete files** from S3 in Node.js 
