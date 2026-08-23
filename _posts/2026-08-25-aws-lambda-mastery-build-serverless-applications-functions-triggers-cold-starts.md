---
layout: post
title: "Build serverless Application with AWS Lambda: Functions, Triggers, and Cold Starts"
date: 2026-08-25 12:00:00 +0545
categories: [AWS, Serverless]
tags: [aws, lambda, serverless, functions, triggers, cold-starts, event-driven, devops]
---

# Build Serverless Application with AWS Lambda: Functions, Triggers, and Cold Starts

## Introduction

Your Rails application needs to resize uploaded images, send welcome emails, process payment webhooks, and generate nightly reports. Traditionally, you'd spin up EC2 instances running background workers—Sidekiq, Delayed Job, or cron jobs. These instances run 24/7, consuming resources even when idle, requiring maintenance, patching, and scaling configuration.

**AWS Lambda** eliminates the server entirely. You write code, upload it, and AWS runs it in response to events—an S3 upload, an API request, a scheduled time. You pay only for execution time (rounded to nearest millisecond), not idle time. No servers to manage, no scaling configuration, no OS patching.

But Lambda isn't just "no servers." It's understanding event sources, managing cold starts, configuring memory and timeouts, handling concurrency limits, integrating with other AWS services, and architecting serverless applications that scale automatically from zero to thousands of concurrent executions.

In this guide, we'll master Lambda: creating functions, configuring triggers, optimizing cold starts, managing permissions, and building production serverless architectures.

## What Is AWS Lambda?

**Lambda** is a serverless compute service that runs code in response to events without provisioning or managing servers.

### Key Lambda Characteristics

| Feature | Description |
|---------|-------------|
| **Event-Driven** | Code runs in response to triggers (S3, API Gateway, SQS) |
| **Auto-Scaling** | Scales from 0 to 10,000+ concurrent executions |
| **Pay-Per-Use** | Charged per request and compute time (100ms increments) |
| **No Servers** | AWS manages all infrastructure |
| **Supported Runtimes** | Python, Node.js, Ruby, Go, Java, .NET, Custom |

### Lambda vs EC2

| Aspect | EC2 | Lambda |
|--------|-----|--------|
| **Management** | You manage OS, scaling, patching | AWS manages everything |
| **Scaling** | Manual (Auto Scaling Groups) | Automatic (event-driven) |
| **Pricing** | Pay for uptime ($/hour) | Pay per request ($0.20 per 1M requests) |
| **Idle Cost** | Full cost even when idle | Zero cost when not running |
| **Cold Start** | None (always running) | Yes (0.5-3 seconds) |

## Creating Your First Lambda Function

### Simple Function (Python)

```python
# lambda_function.py
def lambda_handler(event, context):
    name = event.get('name', 'World')
    return {
        'statusCode': 200,
        'body': f'Hello, {name}!'
    }
```

**Create function:**
```bash
# Package code
zip function.zip lambda_function.py

# Create Lambda function
aws lambda create-function \
  --function-name hello-world \
  --runtime python3.11 \
  --role arn:aws:iam::123456789012:role/lambda-execution-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip \
  --timeout 30 \
  --memory-size 128
```

**Invoke function:**
```bash
aws lambda invoke \
  --function-name hello-world \
  --payload '{"name":"DevOps"}' \
  response.json

cat response.json
# {"statusCode": 200, "body": "Hello, DevOps!"}
```

### Ruby Function

```ruby
# lambda_function.rb
def lambda_handler(event:, context:)
  name = event['name'] || 'World'
  {
    statusCode: 200,
    body: "Hello, #{name}!"
  }
end
```

### Node.js Function

```javascript
// index.js
exports.handler = async (event) => {
  const name = event.name || 'World';
  return {
    statusCode: 200,
    body: `Hello, ${name}!`
  };
};
```

## Lambda Event Sources (Triggers)

Lambda functions respond to various event sources.

### 1. S3 Events

Trigger when objects are uploaded to S3.

```bash
# Grant S3 permission to invoke Lambda
aws lambda add-permission \
  --function-name process-image \
  --statement-id s3-trigger \
  --action lambda:InvokeFunction \
  --principal s3.amazonaws.com \
  --source-arn arn:aws:s3:::my-uploads-bucket

# Configure S3 bucket notification
aws s3api put-bucket-notification-configuration \
  --bucket my-uploads-bucket \
  --notification-configuration '{
    "LambdaFunctionConfigurations": [{
      "LambdaFunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:process-image",
      "Events": ["s3:ObjectCreated:*"],
      "Filter": {
        "Key": {
          "FilterRules": [{
            "Name": "prefix",
            "Value": "uploads/images/"
          }, {
            "Name": "suffix",
            "Value": ".jpg"
          }]
        }
      }
    }]
  }'
```

**Use case:** Image resizing, video transcoding, data processing.

### 2. API Gateway

Create HTTP APIs with Lambda backend.

```bash
# Create REST API
aws apigateway create-rest-api \
  --name my-api \
  --endpoint-configuration types=REGIONAL

# Create resource and method
# ... (complex, use AWS Console or SAM for simplicity)

# Or use HTTP API (simpler)
aws apigatewayv2 create-api \
  --name my-http-api \
  --protocol-type HTTP \
  --target arn:aws:lambda:us-east-1:123456789012:function:api-handler
```

**URL:** `https://abc123.execute-api.us-east-1.amazonaws.com/prod/users`

**Use case:** RESTful APIs, webhooks, microservices.

### 3. CloudWatch Events (EventBridge)

Schedule functions (cron jobs) or respond to AWS events.

```bash
# Create rule (run daily at 2 AM UTC)
aws events put-rule \
  --name daily-report \
  --schedule-expression "cron(0 2 * * ? *)"

# Add Lambda as target
aws events put-targets \
  --rule daily-report \
  --targets "Id=1,Arn=arn:aws:lambda:us-east-1:123456789012:function:generate-report"

# Grant permission
aws lambda add-permission \
  --function-name generate-report \
  --statement-id eventbridge-trigger \
  --action lambda:InvokeFunction \
  --principal events.amazonaws.com \
  --source-arn arn:aws:events:us-east-1:123456789012:rule/daily-report
```

**Use case:** Scheduled jobs, AWS resource monitoring, automated responses.

### 4. SQS Queue

Process messages from SQS queue.

```bash
# Create event source mapping
aws lambda create-event-source-mapping \
  --function-name process-queue-messages \
  --event-source-arn arn:aws:sqs:us-east-1:123456789012:my-queue \
  --batch-size 10 \
  --maximum-batching-window-in-seconds 5
```

**Use case:** Asynchronous processing, decoupled architectures.

### 5. DynamoDB Streams

React to database changes.

```bash
aws lambda create-event-source-mapping \
  --function-name process-db-changes \
  --event-source-arn arn:aws:dynamodb:us-east-1:123456789012:table/Users/stream/2024-01-15T00:00:00.000 \
  --starting-position LATEST
```

**Use case:** Data replication, audit trails, real-time analytics.

{% include inarticle-adsense.html %}

## Lambda Configuration

### Memory and CPU

Lambda allocates CPU proportionally to memory:

| Memory | vCPU | Use Case |
|--------|------|----------|
| 128 MB | 0.08 vCPU | Simple tasks |
| 512 MB | 0.33 vCPU | Light processing |
| 1024 MB | 0.67 vCPU | Standard workloads |
| 3008 MB | 2 vCPU | CPU-intensive |
| 10240 MB | 6 vCPU | Maximum power |

```bash
# Update memory (also increases CPU)
aws lambda update-function-configuration \
  --function-name my-function \
  --memory-size 1024
```

**Cost increases with memory**, but faster execution may reduce total cost.

### Timeout

Maximum execution time (1-900 seconds).

```bash
aws lambda update-function-configuration \
  --function-name my-function \
  --timeout 300  # 5 minutes
```

**Default:** 3 seconds  
**Maximum:** 15 minutes

### Environment Variables

```bash
aws lambda update-function-configuration \
  --function-name my-function \
  --environment Variables='{
    DATABASE_URL=postgresql://...,
    API_KEY=abc123,
    ENV=production
  }'
```

**Access in code:**
```python
import os
db_url = os.environ['DATABASE_URL']
```

### Concurrency Limits

**Account limit:** 1,000 concurrent executions (can request increase)

**Reserved concurrency:**
```bash
# Reserve 100 concurrent executions for this function
aws lambda put-function-concurrency \
  --function-name critical-function \
  --reserved-concurrent-executions 100
```

**Provisioned concurrency** (pre-warmed, eliminates cold starts):
```bash
aws lambda put-provisioned-concurrency-config \
  --function-name my-function \
  --provisioned-concurrent-executions 5 \
  --qualifier prod
```

## Cold Starts

**Cold start:** Delay when Lambda initializes new execution environment.

### Cold Start Timeline

```
Request arrives
  ↓ ~100-500ms: Download code
  ↓ ~100-500ms: Initialize runtime
  ↓ ~10-100ms: Run init code (import modules)
  ↓ Function executes
```

**Total cold start:** 500ms-3s depending on runtime and code size.

### Minimizing Cold Starts

**1. Keep deployment package small:**
```bash
# Bad: 50MB package
zip -r function.zip .

# Good: 5MB package (exclude tests, docs)
zip -r function.zip lambda_function.py requirements/
```

**2. Use lightweight runtimes:**
- ✅ Node.js, Python: Fast cold starts (~500ms)
- ⚠️ Java, .NET: Slower cold starts (~2-3s)

**3. Provisioned Concurrency:**
```bash
aws lambda put-provisioned-concurrency-config \
  --function-name latency-critical \
  --provisioned-concurrent-executions 10 \
  --qualifier prod
```

**Cost:** $0.015 per GB-hour (in addition to execution cost)

**4. Keep functions warm:**
```bash
# Create CloudWatch rule (every 5 minutes)
aws events put-rule \
  --name keep-warm \
  --schedule-expression "rate(5 minutes)"

aws events put-targets \
  --rule keep-warm \
  --targets "Id=1,Arn=arn:aws:lambda:...:function:my-function,Input={\"warmup\":true}"
```

**Function code:**
```python
def lambda_handler(event, context):
    if event.get('warmup'):
        return {'statusCode': 200, 'body': 'Warmed'}
    # Real logic here
```

## IAM Permissions

Lambda needs two types of permissions:

### 1. Execution Role (What Lambda Can Do)

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "s3:GetObject",
      "s3:PutObject",
      "dynamodb:PutItem",
      "logs:CreateLogGroup",
      "logs:CreateLogStream",
      "logs:PutLogEvents"
    ],
    "Resource": "*"
  }]
}
```

### 2. Resource-Based Policy (What Can Invoke Lambda)

```bash
# Allow S3 to invoke Lambda
aws lambda add-permission \
  --function-name my-function \
  --statement-id s3-invoke \
  --action lambda:InvokeFunction \
  --principal s3.amazonaws.com \
  --source-arn arn:aws:s3:::my-bucket
```

## Lambda Layers

**Layers** share code and dependencies across functions.

### Create Layer

```bash
# Create layer directory
mkdir -p layer/python
pip install requests -t layer/python/

# Zip layer
cd layer
zip -r ../layer.zip .

# Publish layer
aws lambda publish-layer-version \
  --layer-name common-dependencies \
  --zip-file fileb://../layer.zip \
  --compatible-runtimes python3.11
```

### Use Layer in Function

```bash
aws lambda update-function-configuration \
  --function-name my-function \
  --layers arn:aws:lambda:us-east-1:123456789012:layer:common-dependencies:1
```

**Benefits:**
- Reduce deployment package size
- Share code across functions
- Update dependencies independently

## Real-World Example: Image Processing Pipeline

### Architecture

```
User uploads image → S3
  ↓ Trigger
Lambda (resize)
  ↓ Save thumbnail
S3 (thumbnails/)
  ↓ Trigger
Lambda (optimize)
  ↓ Save optimized
S3 (optimized/)
```

### Implementation

**1. Resize function:**
```python
# resize_image.py
import boto3
from PIL import Image
import io

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Get S3 object info
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    # Download image
    response = s3.get_object(Bucket=bucket, Key=key)
    image_data = response['Body'].read()
    
    # Resize
    image = Image.open(io.BytesIO(image_data))
    image.thumbnail((200, 200))
    
    # Save thumbnail
    buffer = io.BytesIO()
    image.save(buffer, 'JPEG')
    buffer.seek(0)
    
    thumbnail_key = f"thumbnails/{key.split('/')[-1]}"
    s3.put_object(
        Bucket=bucket,
        Key=thumbnail_key,
        Body=buffer,
        ContentType='image/jpeg'
    )
    
    return {'statusCode': 200, 'message': 'Resized'}
```

**2. Package with PIL:**
```bash
mkdir package
pip install Pillow -t package/
cp resize_image.py package/
cd package
zip -r ../function.zip .
```

**3. Deploy:**
```bash
aws lambda create-function \
  --function-name resize-image \
  --runtime python3.11 \
  --role arn:aws:iam::123456789012:role/lambda-s3-role \
  --handler resize_image.lambda_handler \
  --zip-file fileb://function.zip \
  --timeout 60 \
  --memory-size 1024
```

## Monitoring and Debugging

### CloudWatch Logs

Lambda automatically logs to CloudWatch:

```python
import json

def lambda_handler(event, context):
    print(json.dumps(event))  # Appears in CloudWatch Logs
    print(f"Request ID: {context.request_id}")
    print(f"Memory limit: {context.memory_limit_in_mb} MB")
    
    return {'statusCode': 200}
```

**View logs:**
```bash
aws logs tail /aws/lambda/my-function --follow
```

### X-Ray Tracing

Enable distributed tracing:

```bash
aws lambda update-function-configuration \
  --function-name my-function \
  --tracing-config Mode=Active
```

**Code:**
```python
from aws_xray_sdk.core import xray_recorder

@xray_recorder.capture('process_data')
def process_data(data):
    # Function automatically traced
    return data
```

### CloudWatch Metrics

```bash
# Invocations
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Invocations \
  --dimensions Name=FunctionName,Value=my-function \
  --start-time 2024-01-15T00:00:00Z \
  --end-time 2024-01-15T23:59:59Z \
  --period 3600 \
  --statistics Sum

# Errors
aws cloudwatch get-metric-statistics \
  --metric-name Errors \
  ...

# Duration
aws cloudwatch get-metric-statistics \
  --metric-name Duration \
  --statistics Average \
  ...
```

## Lambda Pricing

### Cost Components

| Component | Price |
|-----------|-------|
| **Requests** | $0.20 per 1M requests |
| **Compute** | $0.0000166667 per GB-second |
| **Free Tier** | 1M requests + 400,000 GB-seconds/month |

### Example Calculation

**Function:**
- Memory: 512 MB (0.5 GB)
- Execution: 200ms (0.2 seconds)
- Requests: 10 million/month

**Cost:**
- Requests: (10M - 1M free) × $0.20 / 1M = $1.80
- Compute: 10M × 0.5 GB × 0.2s × $0.0000166667 = $16.67
- **Total: $18.47/month**

**Equivalent EC2 (t3.micro running 24/7): ~$7.50/month**

**Lambda is cost-effective when:**
- Sporadic usage (not 24/7)
- Event-driven workloads
- No server management overhead valued

## Best Practices

### 1. Separate Handler from Business Logic

```python
# Bad: Everything in handler
def lambda_handler(event, context):
    # 100 lines of logic here

# Good: Separate concerns
def process_order(order_data):
    # Business logic (testable!)
    return result

def lambda_handler(event, context):
    order = event['order']
    result = process_order(order)
    return {'statusCode': 200, 'body': result}
```

### 2. Use Environment Variables for Config

```python
import os
DB_HOST = os.environ['DB_HOST']
API_KEY = os.environ.get('API_KEY', 'default-key')
```

### 3. Handle Errors Gracefully

```python
def lambda_handler(event, context):
    try:
        result = process_data(event)
        return {'statusCode': 200, 'body': result}
    except ValueError as e:
        return {'statusCode': 400, 'body': str(e)}
    except Exception as e:
        print(f"Error: {e}")
        return {'statusCode': 500, 'body': 'Internal error'}
```

### 4. Initialize Outside Handler

```python
# Initialize once (reused across invocations)
import boto3
s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Use s3 client (already initialized)
    s3.get_object(...)
```

## Conclusion

AWS Lambda transforms compute from infrastructure management to pure code execution. By mastering event sources, optimizing cold starts, and architecting event-driven systems, you build applications that scale automatically, cost-efficiently, and require zero server management.

The shift from always-on servers to event-driven functions, from manual scaling to automatic concurrency, and from infrastructure overhead to pure business logic transforms how we build and deploy applications.

Start simple: create a function, trigger it from S3, process data. Then evolve: build API backends with API Gateway, implement event-driven architectures with SQS/SNS, optimize with layers and provisioned concurrency. Every iteration makes your architecture more scalable and your operations simpler.

Master Lambda, and you master serverless computing.

## Suggested Reading

- [AWS Lambda Official Documentation](https://docs.aws.amazon.com/lambda/)
- [Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/)
- [Lambda Best Practices](https://docs.aws.amazon.com/lambda/latest/operatorguide/best-practices.html)
- [Lambda Pricing](https://aws.amazon.com/lambda/pricing/)
- [AWS SAM (Serverless Application Model)](https://docs.aws.amazon.com/serverless-application-model/)
- [Lambda Powertools](https://awslabs.github.io/aws-lambda-powertools-python/)

{% include inarticle-adsense.html %}
