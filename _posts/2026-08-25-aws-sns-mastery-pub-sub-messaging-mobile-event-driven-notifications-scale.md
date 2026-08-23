---
layout: post
title: "AWS SNS Mastery: Pub/Sub Messaging, Mobile and Event Driven Notifications at Scale"
date: 2026-08-25 14:00:00 +0545
categories: [AWS, Messaging]
tags: [aws, sns, pub-sub, notifications, event-driven, messaging, mobile-push, devops]
---

# AWS SNS Mastery: Pub/Sub Messaging, Mobile and Event Driven Notifications at Scale

## Introduction

Your application needs to notify users when orders complete. You send emails directly from your Rails controller. Then you add SMS notifications—another service call. Then mobile push notifications—another integration. Then you want to trigger Lambda functions and log to S3 when orders complete. Your controller becomes a tangled mess of notification logic, each destination tightly coupled, each failure cascading.

**Amazon Simple Notification Service (SNS)** implements the publish-subscribe pattern. Publishers send messages to topics. Subscribers (email, SMS, mobile push, Lambda, SQS, HTTP endpoints) receive messages independently. One publish, multiple deliveries. Add subscribers without changing publishers. One service fails, others continue. Your application becomes loosely coupled, extensible, and resilient.

But SNS isn't just "send notifications." It's understanding topics and subscriptions, message filtering for targeted delivery, fanout patterns with SQS, mobile platform endpoints (iOS, Android), message attributes, delivery retries, and architecting event-driven systems where services react to events without direct coupling.

In this guide, we'll master SNS: creating topics, managing subscriptions, filtering messages, integrating with Lambda and SQS, implementing fanout patterns, and building production event-driven architectures.

## What Is Amazon SNS?

**SNS** is a fully managed pub/sub messaging service for application-to-application (A2A) and application-to-person (A2P) communication.

### Key SNS Characteristics

| Feature | Description |
|---------|-------------|
| **Pub/Sub Model** | Publishers send to topics, subscribers receive independently |
| **Multiple Protocols** | Email, SMS, HTTP/HTTPS, Lambda, SQS, Mobile Push |
| **Message Filtering** | Subscribers receive only relevant messages |
| **Fanout Pattern** | One message delivered to multiple subscribers |
| **High Throughput** | Millions of messages per second |

### SNS vs SQS

| Feature | SNS (Pub/Sub) | SQS (Queue) |
|---------|---------------|-------------|
| **Pattern** | Push (one-to-many) | Pull (one-to-one) |
| **Delivery** | Immediate (push to subscribers) | Polled by consumers |
| **Subscribers** | Multiple (fanout) | Single consumer per message |
| **Use Case** | Broadcast notifications, events | Task queues, buffering |

**Often used together:** SNS → SQS fanout for reliable delivery.

## Creating Topics

### Standard Topic

```bash
aws sns create-topic --name order-notifications

# Output:
{
  "TopicArn": "arn:aws:sns:us-east-1:123456789012:order-notifications"
}
```

### FIFO Topic

```bash
aws sns create-topic \
  --name order-notifications.fifo \
  --attributes FifoTopic=true,ContentBasedDeduplication=true
```

**FIFO topics:**
- Strict ordering
- Exactly-once delivery
- Must subscribe FIFO SQS queues (not email, SMS, etc.)
- Limited throughput (300 msgs/sec, 10,000 with batching)

**Use Standard topics** for most use cases.

## Publishing Messages

### Simple Publish

```bash
aws sns publish \
  --topic-arn arn:aws:sns:us-east-1:123456789012:order-notifications \
  --message "Order #123 has been shipped!"
```

**All subscribers receive this message.**

### Publish with Subject (Email)

```bash
aws sns publish \
  --topic-arn arn:aws:sns:us-east-1:123456789012:order-notifications \
  --subject "Order Shipped" \
  --message "Your order #123 has been shipped and will arrive in 2-3 days."
```

**Subject appears in email subject line.**

### Ruby Example

```ruby
require 'aws-sdk-sns'

sns = Aws::SNS::Client.new(region: 'us-east-1')

sns.publish(
  topic_arn: 'arn:aws:sns:us-east-1:123456789012:order-notifications',
  message: 'Order #123 has been shipped!',
  subject: 'Order Shipped'
)
```

### JSON Message with Multiple Formats

```bash
aws sns publish \
  --topic-arn arn:aws:sns:us-east-1:123456789012:order-notifications \
  --message-structure json \
  --message '{
    "default": "Order #123 shipped",
    "email": "Your order #123 has been shipped. Track it here: https://...",
    "sms": "Order #123 shipped. Track: https://short.link",
    "lambda": "{\"orderId\":123,\"status\":\"shipped\"}"
  }'
```

**Benefit:** Customize message per protocol.

## Subscriptions

### Email Subscription

```bash
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:order-notifications \
  --protocol email \
  --notification-endpoint admin@example.com
```

**User must confirm subscription via email link.**

### SMS Subscription

```bash
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:order-notifications \
  --protocol sms \
  --notification-endpoint +1234567890
```

### Lambda Subscription

```bash
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:order-notifications \
  --protocol lambda \
  --notification-endpoint arn:aws:lambda:us-east-1:123456789012:function:process-order-event
```

**Lambda function triggered automatically when message published.**

### SQS Subscription (Fanout Pattern)

```bash
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:order-notifications \
  --protocol sqs \
  --notification-endpoint arn:aws:sqs:us-east-1:123456789012:order-processing-queue
```

**Grant SNS permission to send to SQS:**
```bash
aws sqs set-queue-attributes \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789012/order-processing-queue \
  --attributes '{
    "Policy": "{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"sns.amazonaws.com\"},\"Action\":\"sqs:SendMessage\",\"Resource\":\"arn:aws:sqs:us-east-1:123456789012:order-processing-queue\",\"Condition\":{\"ArnEquals\":{\"aws:SourceArn\":\"arn:aws:sns:us-east-1:123456789012:order-notifications\"}}}]}"
  }'
```

### HTTP/HTTPS Subscription

```bash
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:order-notifications \
  --protocol https \
  --notification-endpoint https://api.myapp.com/webhooks/orders
```

**Your endpoint receives POST requests:**
```json
{
  "Type": "Notification",
  "MessageId": "abc-123",
  "TopicArn": "arn:aws:sns:us-east-1:123456789012:order-notifications",
  "Message": "Order #123 shipped",
  "Timestamp": "2024-01-15T10:30:00.000Z",
  "SignatureVersion": "1",
  "Signature": "...",
  "UnsubscribeURL": "https://..."
}
```

**Verify signature** to ensure authenticity.

## Message Filtering

Subscribers receive only messages matching their filter policy.

### Example: Order Events with Status

**Publish with attributes:**
```ruby
sns.publish(
  topic_arn: 'arn:aws:sns:us-east-1:123456789012:order-notifications',
  message: JSON.generate({ order_id: 123, status: 'shipped' }),
  message_attributes: {
    'event_type' => { data_type: 'String', string_value: 'order_shipped' },
    'priority' => { data_type: 'String', string_value: 'high' }
  }
)
```

**Subscribe with filter:**
```bash
# Only receive "order_shipped" events
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:order-notifications \
  --protocol email \
  --notification-endpoint shipping@example.com \
  --attributes '{
    "FilterPolicy": "{\"event_type\":[\"order_shipped\"]}"
  }'

# Only receive high-priority events
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:order-notifications \
  --protocol sms \
  --notification-endpoint +1234567890 \
  --attributes '{
    "FilterPolicy": "{\"priority\":[\"high\"]}"
  }'
```

### Filter Policy Operators

```json
{
  "event_type": ["order_shipped", "order_cancelled"],
  "price": [{"numeric": [">=", 100]}],
  "region": [{"anything-but": ["us-west-1"]}]
}
```

**Operators:**
- Exact match: `["value1", "value2"]`
- Numeric: `[{"numeric": [">", 100]}]`
- Prefix: `[{"prefix": "order_"}]`
- Anything-but: `[{"anything-but": ["value"]}]`

{% include inarticle-adsense.html %}

## SNS + SQS Fanout Pattern

**Problem:** One event, multiple processing pipelines.

**Solution:** SNS topic → Multiple SQS queues.

### Architecture

```
Order Placed
  ↓ Publish to SNS Topic
  ├─→ SQS (email-queue) → Email Worker
  ├─→ SQS (inventory-queue) → Inventory Worker
  ├─→ SQS (analytics-queue) → Analytics Worker
  └─→ Lambda (trigger directly)
```

### Implementation

**1. Create SNS topic:**
```bash
aws sns create-topic --name order-events
```

**2. Create SQS queues:**
```bash
aws sqs create-queue --queue-name email-queue
aws sqs create-queue --queue-name inventory-queue
aws sqs create-queue --queue-name analytics-queue
```

**3. Subscribe queues to topic:**
```bash
# Email queue
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:order-events \
  --protocol sqs \
  --notification-endpoint arn:aws:sqs:us-east-1:123456789012:email-queue

# Inventory queue
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:order-events \
  --protocol sqs \
  --notification-endpoint arn:aws:sqs:us-east-1:123456789012:inventory-queue

# Analytics queue (with filter: only completed orders)
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:order-events \
  --protocol sqs \
  --notification-endpoint arn:aws:sqs:us-east-1:123456789012:analytics-queue \
  --attributes '{
    "FilterPolicy": "{\"status\":[\"completed\"]}"
  }'
```

**4. Publish event:**
```ruby
sns.publish(
  topic_arn: 'arn:aws:sns:us-east-1:123456789012:order-events',
  message: JSON.generate({ order_id: 123, user_id: 456, total: 99.99 }),
  message_attributes: {
    'status' => { data_type: 'String', string_value: 'completed' }
  }
)
```

**Result:**
- Email queue receives message → Worker sends confirmation email
- Inventory queue receives message → Worker updates stock
- Analytics queue receives message (status=completed) → Worker logs analytics
- **All happen independently and asynchronously**

**Benefits:**
- **Decoupled:** Add/remove subscribers without changing publisher
- **Reliable:** SQS buffers messages (survives consumer failures)
- **Scalable:** Each queue scales independently
- **Filtered:** Subscribers receive only relevant events

## Mobile Push Notifications

SNS supports push notifications to iOS (APNS), Android (FCM), and other platforms.

### Setup iOS Push (APNS)

**1. Create platform application:**
```bash
aws sns create-platform-application \
  --name MyApp-iOS \
  --platform APNS \
  --attributes PlatformCredential=<APNS_PRIVATE_KEY>,PlatformPrincipal=<APNS_CERTIFICATE>
```

**2. Register device endpoint:**
```bash
aws sns create-platform-endpoint \
  --platform-application-arn arn:aws:sns:us-east-1:123456789012:app/APNS/MyApp-iOS \
  --token <DEVICE_TOKEN>

# Returns endpoint ARN
```

**3. Send push notification:**
```bash
aws sns publish \
  --target-arn arn:aws:sns:us-east-1:123456789012:endpoint/APNS/MyApp-iOS/abc-123 \
  --message '{"APNS":"{\"aps\":{\"alert\":\"Your order has shipped!\",\"badge\":1,\"sound\":\"default\"}}"}'
```

### Setup Android Push (FCM)

```bash
aws sns create-platform-application \
  --name MyApp-Android \
  --platform GCM \
  --attributes PlatformCredential=<FCM_SERVER_KEY>

aws sns create-platform-endpoint \
  --platform-application-arn arn:aws:sns:us-east-1:123456789012:app/GCM/MyApp-Android \
  --token <DEVICE_TOKEN>

aws sns publish \
  --target-arn arn:aws:sns:us-east-1:123456789012:endpoint/GCM/MyApp-Android/xyz-456 \
  --message '{"GCM":"{\"notification\":{\"title\":\"Order Shipped\",\"body\":\"Your order #123 is on the way!\"}}"}'
```

### Publish to Topic (All Platforms)

```bash
# Create topic for all users
aws sns create-topic --name app-notifications

# Subscribe iOS and Android endpoints to topic
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:app-notifications \
  --protocol application \
  --notification-endpoint arn:aws:sns:us-east-1:123456789012:endpoint/APNS/MyApp-iOS/abc-123

# Publish once, deliver to all
aws sns publish \
  --topic-arn arn:aws:sns:us-east-1:123456789012:app-notifications \
  --message "New feature available!"
```

## Real-World Architecture: Event-Driven Order System

### Requirements

- Notify customers (email, SMS, push)
- Update inventory
- Log analytics
- Trigger downstream workflows
- Decouple services

### Architecture

```
Rails App
  │
  └─→ Publish to SNS (order-events)
       │
       ├─→ Email (direct subscription)
       ├─→ SMS (direct subscription)
       ├─→ Mobile Push (direct subscription)
       ├─→ SQS (inventory-queue) → Inventory Worker
       ├─→ SQS (analytics-queue) → Analytics Worker
       └─→ Lambda (process-order-event)
```

### Implementation

**1. Create topic:**
```bash
aws sns create-topic --name order-events
```

**2. Rails publisher:**
```ruby
# app/services/order_event_service.rb
class OrderEventService
  def self.publish(event_type, order)
    sns = Aws::SNS::Client.new(region: 'us-east-1')
    
    message = {
      event_type: event_type,
      order_id: order.id,
      user_id: order.user_id,
      total: order.total.to_f,
      items: order.items.map { |i| { id: i.id, name: i.name, quantity: i.quantity } }
    }
    
    sns.publish(
      topic_arn: ENV['ORDER_EVENTS_TOPIC_ARN'],
      message: JSON.generate(message),
      subject: "Order #{event_type.humanize}",
      message_attributes: {
        'event_type' => { data_type: 'String', string_value: event_type },
        'user_email' => { data_type: 'String', string_value: order.user.email },
        'priority' => { data_type: 'String', string_value: order.priority }
      }
    )
  end
end

# app/controllers/orders_controller.rb
class OrdersController < ApplicationController
  def create
    order = Order.create!(order_params)
    
    # Publish event (non-blocking, asynchronous)
    OrderEventService.publish('order_created', order)
    
    render json: { order_id: order.id }, status: :created
  end
  
  def ship
    order = Order.find(params[:id])
    order.update!(status: 'shipped')
    
    OrderEventService.publish('order_shipped', order)
    
    render json: { status: 'shipped' }
  end
end
```

**3. Subscriptions:**

**Email (customer notification):**
```bash
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:order-events \
  --protocol email \
  --notification-endpoint notifications@myapp.com \
  --attributes '{
    "FilterPolicy": "{\"event_type\":[\"order_created\",\"order_shipped\"]}"
  }'
```

**SMS (high-priority orders):**
```bash
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:order-events \
  --protocol sms \
  --notification-endpoint +1234567890 \
  --attributes '{
    "FilterPolicy": "{\"priority\":[\"high\"]}"
  }'
```

**SQS (inventory updates):**
```bash
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:order-events \
  --protocol sqs \
  --notification-endpoint arn:aws:sqs:us-east-1:123456789012:inventory-queue
```

**Lambda (real-time processing):**
```bash
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:order-events \
  --protocol lambda \
  --notification-endpoint arn:aws:lambda:us-east-1:123456789012:function:process-order-event
```

**Result:**
- **Decoupled:** Services don't know about each other
- **Extensible:** Add subscribers without changing Rails app
- **Resilient:** One subscriber fails, others continue
- **Targeted:** Filter policies deliver only relevant events

## Delivery Retries and DLQ

SNS retries failed deliveries automatically.

### Retry Policy

| Protocol | Retry Attempts | Retry Duration |
|----------|----------------|----------------|
| HTTP/HTTPS | 3 (immediate), 2 (1s apart), 10 (exponential backoff), 38 (20s apart) | Up to 1 hour |
| Lambda | 2 retries | None (Lambda handles retries) |
| SQS | Unlimited | Until message retention expires |

### Dead-Letter Queue (DLQ)

For HTTP/HTTPS, Lambda subscriptions:

```bash
# Create DLQ
aws sqs create-queue --queue-name sns-failed-deliveries

# Configure subscription with DLQ
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:order-events \
  --protocol https \
  --notification-endpoint https://api.myapp.com/webhook \
  --attributes '{
    "RedrivePolicy": "{\"deadLetterTargetArn\":\"arn:aws:sqs:us-east-1:123456789012:sns-failed-deliveries\"}"
  }'
```

**Failed messages** (after all retries) → Sent to DLQ for investigation.

## Monitoring and Metrics

### CloudWatch Metrics

```bash
# Number of messages published
aws cloudwatch get-metric-statistics \
  --namespace AWS/SNS \
  --metric-name NumberOfMessagesPublished \
  --dimensions Name=TopicName,Value=order-events \
  --start-time 2024-01-15T00:00:00Z \
  --end-time 2024-01-15T23:59:59Z \
  --period 3600 \
  --statistics Sum

# Number of notifications delivered
aws cloudwatch get-metric-statistics \
  --metric-name NumberOfNotificationsDelivered \
  ...

# Number of notifications failed
aws cloudwatch get-metric-statistics \
  --metric-name NumberOfNotificationsFailed \
  ...
```

### CloudWatch Alarms

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name sns-delivery-failures \
  --metric-name NumberOfNotificationsFailed \
  --namespace AWS/SNS \
  --dimensions Name=TopicName,Value=order-events \
  --statistic Sum \
  --period 300 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1
```

## Security Best Practices

### 1. IAM Policies

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["sns:Publish"],
    "Resource": "arn:aws:sns:us-east-1:123456789012:order-events"
  }]
}
```

### 2. Topic Access Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::123456789012:role/OrderServiceRole"
    },
    "Action": "sns:Publish",
    "Resource": "arn:aws:sns:us-east-1:123456789012:order-events"
  }]
}
```

### 3. Verify HTTP Signatures

```ruby
require 'openssl'
require 'base64'

def verify_sns_signature(message_json)
  parsed = JSON.parse(message_json)
  
  # Get signing certificate
  cert_url = parsed['SigningCertURL']
  cert_pem = Net::HTTP.get(URI(cert_url))
  cert = OpenSSL::X509::Certificate.new(cert_pem)
  
  # Build signature string
  string_to_sign = ""
  string_to_sign += "Message\n#{parsed['Message']}\n"
  string_to_sign += "MessageId\n#{parsed['MessageId']}\n"
  # ... (add all fields per AWS documentation)
  
  # Verify signature
  signature = Base64.decode64(parsed['Signature'])
  cert.public_key.verify(OpenSSL::Digest::SHA1.new, signature, string_to_sign)
end
```

## SNS Pricing

| Component | Price |
|-----------|-------|
| **Publishes** | $0.50 per million (first 1M free) |
| **HTTP/HTTPS delivery** | $0.60 per million |
| **Email delivery** | $2.00 per 100,000 |
| **SMS delivery** | $0.00645 per message (US) |
| **Mobile push** | Free |
| **SQS/Lambda delivery** | Free (SQS/Lambda charges apply) |

**Example:** 1 million events/month → 1M SQS + 100K emails
- Publishes: Free (within 1M free tier)
- SQS delivery: Free
- Email: 100K × $2 / 100K = **$2.00/month**

## Best Practices

### 1. Use Message Attributes for Filtering

```ruby
# Enable targeted delivery
sns.publish(
  topic_arn: topic_arn,
  message: message,
  message_attributes: {
    'event_type' => { data_type: 'String', string_value: 'order_shipped' },
    'region' => { data_type: 'String', string_value: 'us-east-1' }
  }
)
```

### 2. SNS + SQS for Reliability

```
SNS (fast fanout) → SQS (durable buffering) → Worker (reliable processing)
```

### 3. Implement Idempotency

Messages may be delivered more than once (at-least-once delivery).

```ruby
def process_order_event(event_data)
  # Check if already processed
  return if OrderEvent.exists?(event_id: event_data['MessageId'])
  
  # Process
  process_order(event_data['order_id'])
  
  # Record as processed
  OrderEvent.create!(event_id: event_data['MessageId'])
end
```

### 4. Monitor Failed Deliveries

Set up alarms for `NumberOfNotificationsFailed` metric.

## Conclusion

AWS SNS transforms tightly-coupled direct calls into loosely-coupled publish-subscribe architectures. By mastering topics and subscriptions, message filtering, fanout patterns with SQS, and mobile push notifications, you architect event-driven systems where services react to events independently, failures are isolated, and new subscribers integrate without changing publishers.

The shift from direct service calls to event publishing, from synchronous coupling to asynchronous fanout, and from cascading failures to independent processing transforms applications from fragile monoliths to resilient distributed systems.

Start simple: create a topic, publish events, subscribe email. Then evolve: add SQS fanout for reliability, implement filtering for targeted delivery, integrate mobile push, trigger Lambda functions. Every iteration makes your architecture more decoupled and extensible.

Master SNS, and you master event-driven messaging.

## Suggested Reading

- [AWS SNS Official Documentation](https://docs.aws.amazon.com/sns/)
- [SNS Developer Guide](https://docs.aws.amazon.com/sns/latest/dg/)
- [Message Filtering](https://docs.aws.amazon.com/sns/latest/dg/sns-message-filtering.html)
- [SNS + SQS Fanout](https://docs.aws.amazon.com/sns/latest/dg/sns-sqs-as-subscriber.html)
- [Mobile Push Notifications](https://docs.aws.amazon.com/sns/latest/dg/sns-mobile-application-as-subscriber.html)
- [SNS Best Practices](https://docs.aws.amazon.com/sns/latest/dg/sns-best-practices.html)

{% include inarticle-adsense.html %}
