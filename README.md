# 🚀 AWS Services In-Depth — By Vishwanath Gowda

Welcome to the **ultimate collection of AWS service articles** crafted for DevOps, Cloud Engineers, Architects, and Security professionals.  
This repo covers **real-time use cases**, **step-by-step guides**, and **hands-on examples** for AWS services like EC2, S3, Lambda, IAM, CloudTrail, and more.

---

## 📛 GitHub Badges

![GitHub Repo stars](https://img.shields.io/github/stars/vishwanathgowda/aws-articles?style=social)
![GitHub forks](https://img.shields.io/github/forks/vishwanathgowda/aws-articles?style=social)
![GitHub followers](https://img.shields.io/github/followers/vishwanathgowda?style=social)
![Articles Published](https://img.shields.io/badge/Articles-10+-green?logo=markdown)
[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

---

## ☁️ AWS Service Badges

![AWS](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazon-aws)
![EC2](https://img.shields.io/badge/EC2-Compute-yellow)
![S3](https://img.shields.io/badge/S3-Storage-blue)
![Lambda](https://img.shields.io/badge/Lambda-Serverless-ff69b4)
![IAM](https://img.shields.io/badge/IAM-Access%20Control-blueviolet)
![CloudTrail](https://img.shields.io/badge/CloudTrail-Audit-green)
![VPC](https://img.shields.io/badge/VPC-Networking-red)
![CloudWatch](https://img.shields.io/badge/CloudWatch-Monitoring-brightgreen)

---

## 📚 Table of Contents

> ⚡ Click on each service to dive into in-depth articles.

### 🔐 Identity & Access Management
- [IAM Roles vs IAM Users](IAM/iam-roles-vs-users.md)
- [IAM Policies – Inline vs Managed](IAM/iam-policies.md)

### 🖥️ EC2 (Elastic Compute Cloud)
- [Use EC2 Key Pair Across Regions](EC2/ec2-key-pairs.md)
- [EC2 vs Lambda – When to Use What](EC2/ec2-vs-lambda.md)

### 🪣 S3 (Simple Storage Service)
- [S3 Versioning & Lifecycle Rules](S3/s3-versioning-lifecycle.md)
- [S3 Bucket Policy vs IAM Policy](S3/s3-policy-comparison.md)

### 🧠 Lambda (Serverless)
- [Lambda Deep Dive with Hands-on](Lambda/lambda-basics.md)
- [Event Trigger Use Cases](Lambda/lambda-triggers.md)

### 📜 CloudTrail & Logging
- [AWS CloudTrail for Audit & Forensics](CloudTrail/audit-logging.md)

### 🌐 VPC & Networking
- [VPC Flow Logs Explained](VPC/vpc-flowlogs.md)
- [Private vs Public Subnet Use Cases](VPC/subnet-design.md)

---

## 📸 Architecture & Diagrams

You can find diagrams for each article in the [`/assets/diagrams`](assets/diagrams/) folder.  
All diagrams are created with [draw.io](https://draw.io) and free to use.

---

## 💡 Contribution Guide

Wanna share your AWS knowledge? Feel free to contribute!

```bash
# 1. Fork the repository
# 2. Clone your fork
git clone https://github.com/your-username/aws-articles.git

# 3. Create a branch
git checkout -b feature/new-topic

# 4. Add your .md file inside the right folder
# 5. Commit and Push
git commit -m "Add article on [your-topic]"
git push origin feature/new-topic

# 6. Create Pull Request 🚀


🔵🟢 Blue-Green Deployment using AWS ALB (Static Website) — README

📘 Project Title: Zero-Downtime Deployment with Blue-Green Strategy using ALB

This project demonstrates how to perform a Blue-Green deployment for static websites hosted on EC2 instances using AWS Application Load Balancer (ALB).

🎯 Objective

To deploy two versions of a static website on separate EC2 instances (Blue and Green) and use ALB Listener Rules to route traffic to either version without downtime.

🏗️ Architecture Overview

User
  │
  ▼
ALB (HTTP Listener)
  ├── Path /v1/* → Target Group: blue-tg → EC2-1 (Blue)
  └── Path /v2/* → Target Group: green-tg → EC2-2 (Green)

🧰 Requirements

AWS Account

2 EC2 Instances (Amazon Linux 2023)

ALB (Application Load Balancer)

Target Groups

Apache (httpd)

🧪 EC2 User Data Scripts

🔵 Blue EC2 Instance (Version 1)

#!/bin/bash
dnf update -y
dnf install -y httpd
systemctl enable httpd
systemctl start httpd

mkdir -p /var/www/html/v1
echo "Welcome to Vishwa App - Version 1 (Blue)" > /var/www/html/v1/index.html
echo "OK" > /var/www/html/index.html  # For health check

🟢 Green EC2 Instance (Version 2)

#!/bin/bash
dnf update -y
dnf install -y httpd
systemctl enable httpd
systemctl start httpd

mkdir -p /var/www/html/v2
echo "Welcome to Vishwa App - Version 2 (Green)" > /var/www/html/v2/index.html

# Optional: Prepare green for switch by duplicating v2 as v1
mkdir -p /var/www/html/v1
cp /var/www/html/v2/index.html /var/www/html/v1/index.html

echo "OK" > /var/www/html/index.html  # For health check

⚙️ Setup Steps

Step 1: Launch 2 EC2 Instances

EC2-1 → Blue

EC2-2 → Green

Use the above user data while launching

Attach Security Group with port 80 open to the world

Step 2: Create 2 Target Groups

blue-tg → Register EC2-1

green-tg → Register EC2-2

Protocol: HTTP, Port: 80

Health Check Path: /

Step 3: Create Application Load Balancer

Internet-facing ALB

Listener: HTTP (port 80)

Add dummy TG as default

Step 4: Configure Listener Rules

Go to ALB → Listener → Edit Rules:

Priority

Path

Target Group

1

/v1/*

blue-tg

2

/v2/*

green-tg

Step 5: Test URLs

http://<ALB-DNS>/v1/index.html → Should load Version 1 (Blue)

http://<ALB-DNS>/v2/index.html → Should load Version 2 (Green)

🔄 Switching to Green

Once v2 is tested and approved:

Go to ALB Listener → Edit /v1/* rule

Change "Forward to" from blue-tg → green-tg

💡 Now /v1/ points to Green EC2 → Live

🔁 Switch Back (Rollback to Blue)

If something goes wrong or you want to revert to the previous version:

Go to ALB → Listener → Edit Rules

Locate the rule for /v1/*

Change "Forward to" from green-tg → blue-tg

Save and apply the rule

✅ Now /v1/index.html will again point to the Blue EC2 (Version 1)

📌 Note: Keep the Blue EC2 instance running in standby so that rollback is instant.

✅ Best Practices

Always prepare v1 folder on Green instance before switching

Keep Blue EC2 running for rollback

Monitor ALB health checks and access logs

Remove /v2/ rule if not needed

🧪 Output Verification

URL

Response

/v1/index.html

Version 1 or 2 based on rule

/v2/index.html

Always Version 2

🏁 Final Note

This setup allows seamless deployment of new app versions without breaking the live application. It’s safe, reversible, and used in real-world DevOps pipelines.

Let’s deploy like a pro! 🚀

