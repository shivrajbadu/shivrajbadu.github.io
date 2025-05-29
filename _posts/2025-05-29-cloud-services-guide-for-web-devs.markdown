---
layout: post
title: "The Complete Guide to AWS, GCP, and Azure for Web Developers"
date: 2025-05-29 2:30:00 +0545
categories: [cloud webdev devops aws gcp azure]
tags: [aws, gcp, azure, cloud, devops, web development, deployment]
---

A practical guide to the core services of AWS, GCP, and Azure with step-by-step instructions for developers building and deploying modern web applications.

# ☁️ The Complete Guide to AWS, GCP, and Azure for Web Developers

## 🚀 Introduction

Whether you're building your first full-stack app, deploying production services, or diving into the world of DevOps, understanding cloud platforms is a crucial part of modern web development.

This guide explores the **three major cloud platforms** — **Amazon Web Services (AWS)**, **Google Cloud Platform (GCP)**, and **Microsoft Azure** — and walks through essential services and how to work with them hands-on.

From hosting your app to managing storage, databases, and serverless functions, this guide gives you a practical roadmap to mastering cloud concepts that apply across projects, companies, and use cases.

---

## 🧭 What You'll Learn

| Topic                      | AWS               | GCP               | Azure               |
|---------------------------|--------------------|--------------------|----------------------|
| Virtual Machines          | EC2                | Compute Engine     | Virtual Machines     |
| Static File Storage       | S3                 | Cloud Storage      | Blob Storage         |
| Relational Databases      | RDS                | Cloud SQL          | Azure SQL Database   |
| Serverless Functions      | Lambda             | Cloud Functions    | Azure Functions      |
| App Hosting               | Elastic Beanstalk  | App Engine         | App Services         |
| DNS and Domain Routing    | Route 53           | Cloud DNS          | Azure DNS            |
| Identity & Access Control | IAM                | IAM                | Azure Active Directory |

---

## 🌐 Virtual Machines (Compute)

### 💡 Concept
Virtual Machines are the backbone of the cloud. They let you run any OS with full control, ideal for hosting backend services or custom environments.

### ✅ Launching a VM

#### 🔸 AWS EC2

\`\`\`bash
# Launch EC2 instance via Console > Choose Ubuntu
ssh -i "your-key.pem" ubuntu@<your-ec2-ip>

sudo apt update
sudo apt install nodejs npm
\`\`\`

#### 🔸 GCP Compute Engine

\`\`\`bash
# Launch VM from Console > Use browser-based SSH
sudo apt update
sudo apt install nodejs npm
\`\`\`

#### 🔸 Azure Virtual Machines

\`\`\`bash
# Create a VM via Azure Portal
ssh azureuser@<your-vm-ip>
sudo apt update
sudo apt install nodejs npm
\`\`\`

---

## 📁 Object Storage

### 💡 Concept
Object storage is used to store large files like images, PDFs, backups, or static website content.

### ✅ Uploading Files

#### 🔸 AWS S3

\`\`\`bash
aws s3 mb s3://my-bucket-name
aws s3 cp image.png s3://my-bucket-name
\`\`\`

#### 🔸 GCP Cloud Storage

\`\`\`bash
gsutil mb gs://my-bucket-name
gsutil cp image.png gs://my-bucket-name
\`\`\`

#### 🔸 Azure Blob Storage

\`\`\`bash
az storage blob upload \
  --account-name mystorage \
  --container-name mycontainer \
  --file image.png \
  --name image.png
\`\`\`

---

## 🧮 Relational Databases

### 💡 Concept
Relational databases store structured data using SQL. Cloud platforms offer managed databases with automatic backups and scalability.

### ✅ PostgreSQL Setup

#### 🔸 AWS RDS

- Create RDS (PostgreSQL) from Console
- Configure VPC/Security Group to allow port 5432

\`\`\`bash
psql -h your-rds-endpoint -U dbuser -d dbname
\`\`\`

#### 🔸 GCP Cloud SQL

\`\`\`bash
gcloud sql instances create my-postgres \
  --database-version=POSTGRES_13 --tier=db-f1-micro
\`\`\`

#### 🔸 Azure SQL Database

\`\`\`bash
sqlcmd -S server-name.database.windows.net -U username -P password
\`\`\`

---

## ⚡ Serverless Functions

### 💡 Concept
Serverless lets you run code without managing servers. Define a function, set triggers (like HTTP or storage events), and the cloud handles the rest.

### ✅ Deploying a Function

#### 🔸 AWS Lambda (Console)

- Create Lambda function
- Choose Node.js or Python or RoR
- Trigger via API Gateway or S3

#### 🔸 GCP Cloud Functions

\`\`\`bash
gcloud functions deploy helloWorld \
  --runtime nodejs18 \
  --trigger-http \
  --allow-unauthenticated
\`\`\`

#### 🔸 Azure Functions (CLI)

\`\`\`bash
func init MyFuncProj --worker-runtime node
cd MyFuncProj
func new --name HelloHttp --template "HTTP trigger"
func start
\`\`\`

---

## 🌍 Application Hosting

### 💡 Concept
Deploy and scale apps easily using managed hosting platforms. Ideal for web apps, APIs, or microservices.

### ✅ Hosting a Node.js App

#### 🔸 AWS Elastic Beanstalk

\`\`\`bash
eb init -p node.js my-app
eb create my-env
eb deploy
\`\`\`

#### 🔸 GCP App Engine

\`\`\`bash
gcloud app create
gcloud app deploy
\`\`\`

#### 🔸 Azure App Services

\`\`\`bash
az webapp up --name mywebapp123 \
  --runtime "NODE|18-lts" \
  --location "eastus"
\`\`\`

---

## 🔐 Identity & Access Management (IAM)

### 💡 Concept
IAM defines who can access what in your cloud. It includes users, roles, policies, and permissions.

| Cloud     | Features                        |
|-----------|----------------------------------|
| AWS IAM   | Fine-grained roles/policies      |
| GCP IAM   | Role-based access via principals |
| Azure AD  | Role assignments + AD groups     |

**Best Practices:**
- Use least privilege access
- Prefer roles over root/admin accounts
- Rotate keys regularly

---

## 🔧 Developer Tools to Install

| Tool      | Command (macOS)                         |
|-----------|------------------------------------------|
| AWS CLI   | \`brew install awscli\`                    |
| GCP SDK   | \`brew install --cask google-cloud-sdk\`   |
| Azure CLI | \`brew install azure-cli\`                 |
| Terraform | \`brew install terraform\`                 |
| Docker    | \`brew install --cask docker\`             |

---

## 🧠 Tips to Practice Cloud Skills

- ✅ Deploy a simple Node.js/Python/RoR app on a VM
- ✅ Move to serverless when comfortable
- ✅ Store logs and media in object storage
- ✅ Attach a cloud-managed SQL database
- ✅ Use IAM to test permission boundaries

---

## 📚 Summary

Cloud platforms are essential to every stage of web development — from prototype to production. Understanding core services across AWS, GCP, and Azure empowers developers to:

- Build and deploy scalable apps
- Work across different cloud providers
- Integrate DevOps best practices into workflows
---
