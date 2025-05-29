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

## Initial VM Deployment

For the VM deployment, you have several straightforward options:

- Traditional servers (DigitalOcean, Linode, AWS EC2)
- Platform-as-a-Service (Heroku, Railway, Render)
- Container platforms (Docker on any cloud provider)

The PaaS options are often easiest for getting started since they handle most infrastructure concerns automatically.

## Migration to Serverless

When you're ready to go serverless, you have a few architectural approaches:

### Function-based approach:

- Break your Rails app into smaller functions using AWS Lambda, Google Cloud Functions, or similar
- This requires significant refactoring since Rails is designed as a monolithic framework
- You'd typically extract specific endpoints or services into individual functions

### Container-based serverless:

- Use services like AWS Fargate, Google Cloud Run, or Azure Container Instances
- Package your Rails app in a container that scales to zero when not in use
- Minimal code changes required - mainly configuration adjustments
- Still gets you the serverless benefits of automatic scaling and pay-per-use

### Hybrid approach:

- Keep core Rails app on containers/VMs
- Extract specific heavy or intermittent workloads (image processing, report generation, etc.) into serverless functions
- Use message queues or webhooks to connect them

### Here's a practical example of extracting heavy workloads from a Rails app into serverless functions:

#### Current Rails Implementation

```ruby
# In your Rails app - products_controller.rb
class ProductsController < ApplicationController
  def create
    @product = Product.new(product_params)
    if @product.save
      # This blocks the request for 10-30 seconds
      ImageProcessorService.new(@product).process_images
      redirect_to @product
    end
  end
end

# Heavy service that blocks requests
class ImageProcessorService
  def process_images
    # Resize original image
    # Generate 5 different thumbnail sizes
    # Optimize for web
    # This takes 15-30 seconds per product
  end
end
```

#### After Serverless Migration

```ruby
# products_controller.rb - Now fast and responsive
class ProductsController < ApplicationController
  def create
    @product = Product.new(product_params)
    if @product.save
      # Queue the heavy work instead of doing it synchronously
      ImageProcessingJob.perform_later(@product.id)
      redirect_to @product, notice: "Product created! Images are being processed."
    end
  end
end

# Simple job that triggers serverless function
class ImageProcessingJob < ApplicationJob
  def perform(product_id)
    # Trigger AWS Lambda function
    lambda_client = Aws::Lambda::Client.new
    
    response = lambda_client.invoke({
      function_name: 'image-processor', # ← This name references the deployed function
      payload: { product_id: product_id }.to_json
    })

    puts "Lambda response: #{response.payload.read}"
  end
end
```

#### Serverless Function (AWS Lambda)

````ruby
# lambda_function.rb - Deployed as separate AWS Lambda
require 'aws-sdk-s3'
require 'mini_magick'

def lambda_handler(event:, context:)
  product_id = event['product_id']
  
  # Fetch product data from database
  product_data = fetch_product_from_api(product_id)
  
  # Download original image from S3
  original_image = download_image(product_data['image_url'])
  
  # Fetch from your Rails API
  ## rails_api_url = "https://your-rails-app.com/api/products/#{product_id}"
  
  # Process images (this heavy work now runs separately)
  thumbnails = generate_thumbnails(original_image)
  optimized_image = optimize_image(original_image)
  
  # Upload processed images back to S3
  upload_processed_images(product_id, thumbnails, optimized_image)
  
  # Update product record via API
  update_product_images(product_id, processed_image_urls)
  
  { statusCode: 200, body: 'Images processed successfully' }
end

def generate_thumbnails(image)
  sizes = [100, 200, 400, 800, 1200]
  sizes.map do |size|
    MiniMagick::Image.open(image).resize("#{size}x#{size}>")
  end
end
````

The lambda_function.rb file needs to be deployed as an AWS Lambda function with the name 'image-processor'.

### Step 1: Deploy Lambda Function

```bash
# You need to package and deploy lambda_function.rb to AWS Lambda
# This creates a function named 'image-processor' in AWS

# Using AWS CLI or deployment tools like:
aws lambda create-function \
  --function-name image-processor \
  --runtime ruby2.7 \
  --role arn:aws:iam::your-account:role/lambda-execution-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip
```

### Deployment Script (package.sh)

```bash
#!/bin/bash
# Install gems
bundle install --deployment

# Create deployment package
zip -r function.zip lambda_function.rb vendor/

# Deploy to AWS Lambda
aws lambda update-function-code \
  --function-name image-processor \
  --zip-file fileb://function.zip
```

### Step 2: The Connection

```ruby
# In Rails - ImageProcessingJob

class ImageProcessingJob < ApplicationJob
  def perform(product_id)
    lambda_client = Aws::Lambda::Client.new(region: 'us-east-1')
    
    # This calls the AWS Lambda function named 'image-processor'
    # The function name must match what you deployed to AWS
    response = lambda_client.invoke({
      function_name: 'image-processor',  # ← This name references the deployed function
      invocation_type: 'Event',         # Async execution
      payload: {
        product_id: product_id,
        callback_url: "#{ENV['RAILS_APP_URL']}/webhooks/image_complete"
       }.to_json
    })
    Rails.logger.info "Lambda invoked successfully: #{response.status_code}"
    
    puts "Lambda response: #{response.payload.read}"
  end
end
```

### Step 3: AWS Lambda Execution

When Rails calls `lambda_client.invoke()`, AWS Lambda:

1. Finds the function named 'image-processor'
2. Loads the deployed lambda_function.rb code
3. Calls the lambda_handler method with the payload
4. Executes all the image processing code
5. Returns the response back to Rails

#### Complete Example with Deployment
### File structure

serverless-functions/
├── image-processor/
│   ├── lambda_function.rb          # The handler code
│   ├── Gemfile                     # Dependencies
│   └── package.sh                  # Deployment script
└── rails-app/
    └── app/jobs/image_processing_job.rb  # Rails job

## The Flow in Action

1. User uploads product → Rails controller
2. Rails saves product → Database
3. Rails queues job → ImageProcessingJob.perform_later(product.id)
4. Job executes → Calls lambda_client.invoke(function_name: 'image-processor')
5. AWS receives call → Finds function named 'image-processor'
6. AWS executes → Runs lambda_handler method in deployed lambda_function.rb
7. Lambda processes → Downloads, resizes, uploads images
8. Lambda completes → Returns success response to Rails

The key is that 'image-processor' is the name you give the function when deploying to AWS Lambda, and that same name is used in Rails to reference and invoke it.

## 📚 Summary

Cloud platforms are essential to every stage of web development — from prototype to production. Understanding core services across AWS, GCP, and Azure empowers developers to:

- Build and deploy scalable apps
- Work across different cloud providers
- Integrate DevOps best practices into workflows
---
