# 🪣 S3 Versioning & Lifecycle Management – In-Depth Guide

> Author: **Vishwanath Gowda**  
> Category: `AWS / S3`  
> Tags: `s3`, `versioning`, `lifecycle`, `storage classes`, `cost optimization`, `data retention`

---

## 🔍 What is S3 Versioning?

**S3 Versioning** allows you to preserve, retrieve, and restore every version of every object stored in a bucket.  
Even if a file is deleted or overwritten, the previous versions are retained.

---

### ✅ Why Use S3 Versioning?

| Feature                 | Description                                                                 |
|------------------------|-----------------------------------------------------------------------------|
| 🛡️ Data Protection      | Prevents accidental deletion or overwrites                                  |
| 🔄 File Recovery        | Retrieve older versions of objects                                           |
| 🔐 Compliance Support   | Useful for legal and audit requirements                                     |
| 💸 Cost Implication     | Versioning increases storage cost if not paired with lifecycle rules         |

---

## 🛠️ How to Enable Versioning

### Using AWS Console:
1. Go to **S3 → Your Bucket**
2. Click **Properties**
3. Scroll to **Bucket Versioning**
4. Click **Enable**

### Using AWS CLI:
```bash
aws s3api put-bucket-versioning \
  --bucket my-versioned-bucket \
  --versioning-configuration Status=Enabled
