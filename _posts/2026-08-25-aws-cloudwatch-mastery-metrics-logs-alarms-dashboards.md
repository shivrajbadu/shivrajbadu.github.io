---
layout: post
title: "AWS CloudWatch Mastery: Metrics, Logs, Alarms, and Dashboards"
date: 2026-08-25 15:00:00 +0545
categories: [AWS, Monitoring]
tags: [aws, cloudwatch, monitoring, metrics, logs, alarms, dashboards, observability, devops]
---

# AWS CloudWatch Mastery: Metrics, Logs, Alarms, and Dashboards

## Introduction

Your production Rails application is down. Users report errors, but you don't know when it started, what caused it, or which component failed. You SSH into EC2 instances, grep log files, check CPU usage manually. By the time you identify the issue—a database connection pool exhaustion—you've lost revenue and customer trust. You were flying blind.

**Amazon CloudWatch** gives you eyes into your AWS infrastructure. It collects metrics (CPU, memory, disk, custom application metrics), aggregates logs (application logs, system logs, Lambda logs), triggers alarms (CPU > 80%, error rate > 5%), and visualizes everything in dashboards. You shift from reactive debugging to proactive monitoring, from manual log parsing to automated alerting, from guessing to knowing.

But CloudWatch isn't just "show me graphs." It's understanding metric namespaces and dimensions, creating custom metrics for business KPIs, writing Log Insights queries for log analytics, configuring composite alarms for complex conditions, building operational dashboards, and architecting observable systems where problems are detected before users notice.

In this guide, we'll master CloudWatch: collecting metrics, querying logs, creating alarms, building dashboards, and implementing production observability.

## What Is Amazon CloudWatch?

**CloudWatch** is a monitoring and observability service that collects and visualizes metrics, logs, and events from AWS resources and applications.

### Core CloudWatch Components

| Component | Description |
|-----------|-------------|
| **Metrics** | Time-series data (CPU, memory, custom metrics) |
| **Logs** | Centralized log aggregation and search |
| **Alarms** | Notifications based on metric thresholds |
| **Dashboards** | Visual monitoring with graphs and widgets |
| **Events (EventBridge)** | React to AWS resource changes |
| **Insights** | Query and analyze logs at scale |

### Why CloudWatch?

✅ **Visibility:** See what's happening across all AWS resources  
✅ **Alerting:** Get notified when metrics cross thresholds  
✅ **Troubleshooting:** Query logs to debug issues  
✅ **Optimization:** Identify resource waste and performance bottlenecks  
✅ **Automation:** Trigger actions (Auto Scaling, Lambda) based on metrics  

## CloudWatch Metrics

**Metrics:** Time-series data points with timestamp, namespace, dimensions, and value.

### AWS Service Metrics (Automatic)

AWS services automatically publish metrics to CloudWatch:

| Service | Key Metrics |
|---------|-------------|
| **EC2** | CPUUtilization, NetworkIn/Out, DiskReadOps/WriteOps |
| **RDS** | DatabaseConnections, ReadLatency, WriteLatency, FreeStorageSpace |
| **ELB** | RequestCount, TargetResponseTime, HealthyHostCount, HTTPCode_Target_4XX |
| **Lambda** | Invocations, Duration, Errors, Throttles |
| **S3** | BucketSizeBytes, NumberOfObjects (daily) |

### View Metrics (CLI)

```bash
# List available metrics
aws cloudwatch list-metrics \
  --namespace AWS/EC2

# Get CPU utilization
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --start-time 2024-01-15T00:00:00Z \
  --end-time 2024-01-15T23:59:59Z \
  --period 300 \
  --statistics Average,Maximum
```

**Response:**
```json
{
  "Datapoints": [
    {"Timestamp": "2024-01-15T00:00:00Z", "Average": 45.2, "Maximum": 68.1, "Unit": "Percent"},
    {"Timestamp": "2024-01-15T00:05:00Z", "Average": 52.7, "Maximum": 71.3, "Unit": "Percent"}
  ]
}
```

### Custom Metrics

Publish application-specific metrics (order count, API latency, business KPIs).

**Ruby example:**
```ruby
require 'aws-sdk-cloudwatch'

cloudwatch = Aws::CloudWatch::Client.new(region: 'us-east-1')

# Publish custom metric
cloudwatch.put_metric_data(
  namespace: 'MyApp/Orders',
  metric_data: [{
    metric_name: 'OrdersProcessed',
    value: 42,
    unit: 'Count',
    timestamp: Time.now,
    dimensions: [{
      name: 'Environment',
      value: 'production'
    }]
  }]
)
```

**CLI example:**
```bash
aws cloudwatch put-metric-data \
  --namespace MyApp/API \
  --metric-name ResponseTime \
  --value 234.5 \
  --unit Milliseconds \
  --dimensions Endpoint=/api/users,Method=GET
```

### High-Resolution Metrics

**Standard resolution:** 1-minute granularity (free)  
**High resolution:** 1-second granularity (+cost)

```ruby
cloudwatch.put_metric_data(
  namespace: 'MyApp/Performance',
  metric_data: [{
    metric_name: 'APILatency',
    value: 145.2,
    unit: 'Milliseconds',
    timestamp: Time.now,
    storage_resolution: 1  # 1-second resolution
  }]
)
```

**Use cases:** Real-time monitoring, sub-minute alerting.

### Metric Math

Combine metrics with expressions:

```bash
# Calculate error rate
aws cloudwatch get-metric-data \
  --metric-data-queries '[
    {
      "Id": "errors",
      "MetricStat": {
        "Metric": {
          "Namespace": "AWS/Lambda",
          "MetricName": "Errors",
          "Dimensions": [{"Name":"FunctionName","Value":"my-function"}]
        },
        "Period": 300,
        "Stat": "Sum"
      }
    },
    {
      "Id": "invocations",
      "MetricStat": {
        "Metric": {
          "Namespace": "AWS/Lambda",
          "MetricName": "Invocations",
          "Dimensions": [{"Name":"FunctionName","Value":"my-function"}]
        },
        "Period": 300,
        "Stat": "Sum"
      }
    },
    {
      "Id": "error_rate",
      "Expression": "errors / invocations * 100"
    }
  ]' \
  --start-time 2024-01-15T00:00:00Z \
  --end-time 2024-01-15T23:59:59Z
```

## CloudWatch Logs

**Logs:** Centralized storage and search for application and system logs.

### Log Structure

```
Log Group (e.g., /aws/lambda/my-function)
  └─ Log Stream (e.g., 2024/01/15/[$LATEST]abc123)
       └─ Log Events (individual log lines with timestamp)
```

### Creating Log Groups

```bash
aws logs create-log-group \
  --log-group-name /myapp/production
```

### Sending Logs (Application)

**Ruby with CloudWatch Logs SDK:**
```ruby
require 'aws-sdk-cloudwatchlogs'

logs = Aws::CloudWatchLogs::Client.new(region: 'us-east-1')

# Create log stream
logs.create_log_stream(
  log_group_name: '/myapp/production',
  log_stream_name: 'web-server-1'
)

# Send log events
logs.put_log_events(
  log_group_name: '/myapp/production',
  log_stream_name: 'web-server-1',
  log_events: [
    { timestamp: (Time.now.to_f * 1000).to_i, message: 'User login successful: user_id=123' },
    { timestamp: (Time.now.to_f * 1000).to_i, message: 'API request: GET /users duration=234ms' }
  ]
)
```

**Rails integration with lograge:**
```ruby
# Gemfile
gem 'lograge'
gem 'aws-sdk-cloudwatchlogs'

# config/environments/production.rb
config.lograge.enabled = true
config.lograge.formatter = Lograge::Formatters::Json.new

# Send to CloudWatch (custom logger)
cloudwatch_logger = CloudWatchLogger.new('/myapp/production', 'rails-app')
config.logger = ActiveSupport::Logger.new(cloudwatch_logger)
```

### CloudWatch Agent (EC2)

Install agent to send EC2 system and application logs:

```bash
# Install agent
sudo yum install amazon-cloudwatch-agent

# Configure agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard

# Start agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -s \
  -c file:/opt/aws/amazon-cloudwatch-agent/etc/config.json
```

**Collect custom logs:**
```json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [{
          "file_path": "/var/log/myapp/*.log",
          "log_group_name": "/myapp/production",
          "log_stream_name": "{instance_id}"
        }]
      }
    }
  }
}
```

### Viewing Logs

```bash
# List log groups
aws logs describe-log-groups

# Tail logs (like tail -f)
aws logs tail /aws/lambda/my-function --follow

# Get logs for time range
aws logs filter-log-events \
  --log-group-name /myapp/production \
  --start-time $(date -u -d '1 hour ago' +%s)000 \
  --end-time $(date -u +%s)000
```

{% include inarticle-adsense.html %}

## CloudWatch Logs Insights

**Logs Insights:** Query language for analyzing logs at scale.

### Query Examples

**1. Find errors:**
```sql
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 100
```

**2. Count errors by type:**
```sql
fields @message
| filter @message like /ERROR/
| parse @message /ERROR: (?<error_type>.*)/
| stats count() by error_type
| sort count desc
```

**3. API response times:**
```sql
fields @timestamp, duration
| filter method = "GET" and endpoint = "/api/users"
| stats avg(duration), max(duration), min(duration)
```

**4. P50, P90, P99 latency:**
```sql
fields duration
| filter method = "GET"
| stats percentile(duration, 50) as p50,
        percentile(duration, 90) as p90,
        percentile(duration, 99) as p99
```

**5. Errors over time (grouped by 5 minutes):**
```sql
fields @timestamp, @message
| filter @message like /ERROR/
| stats count() as error_count by bin(5m)
```

### Run Query (CLI)

```bash
aws logs start-query \
  --log-group-name /myapp/production \
  --start-time $(date -u -d '1 hour ago' +%s) \
  --end-time $(date -u +%s) \
  --query-string 'fields @timestamp, @message | filter @message like /ERROR/ | limit 20'

# Get query results
aws logs get-query-results --query-id <query-id>
```

### Save Queries

Save frequently-used queries in CloudWatch console for quick access.

## CloudWatch Alarms

**Alarms:** Get notified when metrics cross thresholds.

### Creating Alarms

**High CPU alarm:**
```bash
aws cloudwatch put-metric-alarm \
  --alarm-name high-cpu-instance-1 \
  --alarm-description "Alert when CPU exceeds 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --evaluation-periods 2 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts
```

**Behavior:**
- Check CPU every 5 minutes (period=300)
- Alarm if CPU > 80% for 2 consecutive periods (10 minutes total)
- Send SNS notification to `alerts` topic

**Lambda errors alarm:**
```bash
aws cloudwatch put-metric-alarm \
  --alarm-name lambda-high-errors \
  --metric-name Errors \
  --namespace AWS/Lambda \
  --statistic Sum \
  --period 60 \
  --evaluation-periods 1 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=FunctionName,Value=my-function \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:critical-alerts
```

### Alarm States

| State | Description |
|-------|-------------|
| **OK** | Metric within threshold |
| **ALARM** | Metric breached threshold |
| **INSUFFICIENT_DATA** | Not enough data to evaluate |

### Composite Alarms

Combine multiple alarms with AND/OR logic:

```bash
aws cloudwatch put-composite-alarm \
  --alarm-name system-unhealthy \
  --alarm-rule "ALARM(high-cpu-instance-1) AND ALARM(high-memory-instance-1)" \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:critical-alerts
```

**Use case:** Alert only when multiple conditions are true (avoid false positives).

### Anomaly Detection Alarms

Alarm on unusual patterns (ML-based):

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name anomaly-cpu \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --evaluation-periods 2 \
  --threshold-metric-id ad1 \
  --comparison-operator LessThanLowerOrGreaterThanUpperThreshold \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --metrics '[
    {
      "Id": "m1",
      "ReturnData": false,
      "MetricStat": {
        "Metric": {
          "Namespace": "AWS/EC2",
          "MetricName": "CPUUtilization",
          "Dimensions": [{"Name":"InstanceId","Value":"i-1234567890abcdef0"}]
        },
        "Period": 300,
        "Stat": "Average"
      }
    },
    {
      "Id": "ad1",
      "Expression": "ANOMALY_DETECTION_BAND(m1, 2)"
    }
  ]'
```

**Benefit:** Automatically learns normal patterns, alerts on deviations.

## CloudWatch Dashboards

**Dashboards:** Visualize metrics and logs in real-time.

### Creating Dashboard (CLI)

```bash
aws cloudwatch put-dashboard \
  --dashboard-name production-overview \
  --dashboard-body '{
    "widgets": [
      {
        "type": "metric",
        "properties": {
          "metrics": [
            ["AWS/EC2", "CPUUtilization", {"stat": "Average"}]
          ],
          "period": 300,
          "region": "us-east-1",
          "title": "EC2 CPU Utilization"
        }
      },
      {
        "type": "metric",
        "properties": {
          "metrics": [
            ["AWS/RDS", "DatabaseConnections"],
            [".", "ReadLatency"]
          ],
          "period": 300,
          "region": "us-east-1",
          "title": "RDS Metrics"
        }
      }
    ]
  }'
```

### Widget Types

| Widget | Description |
|--------|-------------|
| **Line graph** | Time-series metrics |
| **Number** | Single metric value |
| **Gauge** | Metric with min/max range |
| **Bar chart** | Compare metrics |
| **Pie chart** | Proportional data |
| **Logs table** | Recent log entries |

### Real-World Dashboard Example

**Production Rails application:**

1. **API Health:**
   - Request count (ALB RequestCount)
   - Response time (ALB TargetResponseTime)
   - Error rate (ALB HTTPCode_Target_5XX_Count)

2. **Application Performance:**
   - CPU utilization (EC2 CPUUtilization)
   - Memory usage (custom metric)
   - Active connections (custom metric)

3. **Database:**
   - Connections (RDS DatabaseConnections)
   - Read/write latency (RDS ReadLatency, WriteLatency)
   - Free storage space (RDS FreeStorageSpace)

4. **Background Jobs:**
   - Queue depth (custom SQS ApproximateNumberOfMessagesVisible)
   - Processing time (custom metric)
   - Failed jobs (custom metric)

5. **Business Metrics:**
   - Orders per minute (custom metric)
   - Revenue (custom metric)
   - Active users (custom metric)

## Real-World Monitoring Architecture

### Requirements

- Monitor EC2, RDS, Lambda, ALB
- Centralize all logs
- Alert on critical issues
- Dashboard for operations team
- Track custom business metrics

### Implementation

**1. Enable detailed monitoring:**
```bash
# EC2 detailed monitoring (1-minute resolution)
aws ec2 monitor-instances --instance-ids i-1234567890abcdef0

# RDS enhanced monitoring
aws rds modify-db-instance \
  --db-instance-identifier mydb \
  --monitoring-interval 60 \
  --monitoring-role-arn arn:aws:iam::123456789012:role/rds-monitoring-role
```

**2. Install CloudWatch agent on EC2:**
```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -s \
  -c file:/opt/aws/amazon-cloudwatch-agent/etc/config.json
```

**Config (custom metrics + logs):**
```json
{
  "metrics": {
    "namespace": "MyApp/Production",
    "metrics_collected": {
      "mem": {
        "measurement": [{"name": "mem_used_percent"}]
      },
      "disk": {
        "measurement": [{"name": "used_percent"}]
      }
    }
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [{
          "file_path": "/var/log/myapp/production.log",
          "log_group_name": "/myapp/production",
          "log_stream_name": "{instance_id}"
        }]
      }
    }
  }
}
```

**3. Send custom metrics from Rails:**
```ruby
# app/services/metrics_service.rb
class MetricsService
  def self.track_order(order)
    cloudwatch = Aws::CloudWatch::Client.new(region: 'us-east-1')
    
    cloudwatch.put_metric_data(
      namespace: 'MyApp/Business',
      metric_data: [{
        metric_name: 'OrdersCreated',
        value: 1,
        unit: 'Count',
        dimensions: [
          { name: 'Environment', value: Rails.env },
          { name: 'PaymentMethod', value: order.payment_method }
        ]
      }, {
        metric_name: 'Revenue',
        value: order.total.to_f,
        unit: 'None',
        dimensions: [{ name: 'Environment', value: Rails.env }]
      }]
    )
  end
end

# app/controllers/orders_controller.rb
def create
  order = Order.create!(order_params)
  MetricsService.track_order(order)
  render json: order
end
```

**4. Create alarms:**
```bash
# High CPU
aws cloudwatch put-metric-alarm \
  --alarm-name prod-high-cpu \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:devops-alerts

# High error rate (ALB 5xx)
aws cloudwatch put-metric-alarm \
  --alarm-name prod-high-errors \
  --metric-name HTTPCode_Target_5XX_Count \
  --namespace AWS/ApplicationELB \
  --statistic Sum \
  --period 60 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:critical-alerts

# Database storage low
aws cloudwatch put-metric-alarm \
  --alarm-name prod-db-storage-low \
  --metric-name FreeStorageSpace \
  --namespace AWS/RDS \
  --statistic Average \
  --period 300 \
  --threshold 10737418240 \
  --comparison-operator LessThanThreshold \
  --evaluation-periods 1 \
  --dimensions Name=DBInstanceIdentifier,Value=mydb \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:devops-alerts
```

**5. Create dashboard:**
```bash
aws cloudwatch put-dashboard \
  --dashboard-name production-health \
  --dashboard-body file://dashboard.json
```

**Result:**
- **Visibility:** Single dashboard shows all system health
- **Alerting:** Notified immediately of issues via SNS → Email/Slack
- **Troubleshooting:** Query logs with Insights to debug errors
- **Optimization:** Identify underutilized resources (low CPU, idle connections)

## CloudWatch Container Insights

Monitor ECS, EKS, Kubernetes clusters.

```bash
# Enable for ECS cluster
aws ecs update-cluster-settings \
  --cluster my-cluster \
  --settings name=containerInsights,value=enabled
```

**Automatic metrics:**
- CPU/memory per container
- Network traffic
- Task/pod counts
- Node metrics

## CloudWatch Pricing

| Component | Price |
|-----------|-------|
| **Standard metrics** | Free (EC2, RDS, etc., 5-minute resolution) |
| **Detailed metrics** | $0.30 per metric per month (1-minute resolution) |
| **Custom metrics** | $0.30 per metric per month |
| **Logs ingestion** | $0.50 per GB |
| **Logs storage** | $0.03 per GB per month |
| **Logs Insights queries** | $0.005 per GB scanned |
| **Alarms** | $0.10 per alarm per month |
| **Dashboards** | $3.00 per dashboard per month |

**Free tier:**
- 10 custom metrics
- 10 alarms
- 5 GB log ingestion
- 5 GB log storage

**Example (small production app):**
- 50 custom metrics: 50 × $0.30 = $15.00
- 10 GB logs/month: 10 × $0.50 = $5.00
- 20 alarms: 20 × $0.10 = $2.00
- 1 dashboard: $3.00
- **Total: ~$25/month**

## Best Practices

### 1. Use Structured Logging

```ruby
# Bad: Unstructured logs
Rails.logger.info "User 123 placed order 456"

# Good: Structured JSON
Rails.logger.info({
  event: 'order_created',
  user_id: 123,
  order_id: 456,
  total: 99.99,
  timestamp: Time.now.iso8601
}.to_json)
```

**Benefit:** Easier to query with Logs Insights.

### 2. Tag Resources for Cost Allocation

```bash
aws ec2 create-tags \
  --resources i-1234567890abcdef0 \
  --tags Key=Environment,Value=production Key=Team,Value=backend
```

**View costs by tag** in CloudWatch/Cost Explorer.

### 3. Set Alarm Actions

```bash
# Scale up Auto Scaling group when CPU high
aws cloudwatch put-metric-alarm \
  --alarm-name scale-up \
  --metric-name CPUUtilization \
  --threshold 70 \
  --alarm-actions arn:aws:autoscaling:us-east-1:123456789012:scalingPolicy:policy-id
```

### 4. Use Log Retention Policies

```bash
# Keep logs for 30 days (reduce cost)
aws logs put-retention-policy \
  --log-group-name /myapp/production \
  --retention-in-days 30
```

### 5. Create Runbooks

Link alarms to documentation:

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name high-cpu \
  --alarm-description "CPU > 80%. Runbook: https://wiki.company.com/runbooks/high-cpu" \
  ...
```

## Conclusion

AWS CloudWatch transforms infrastructure from opaque to observable. By mastering metrics collection, log aggregation, intelligent alarms, and operational dashboards, you shift from reactive firefighting to proactive monitoring, from blind debugging to data-driven troubleshooting, and from guessing to knowing.

The shift from manual log parsing to centralized log queries, from SSH-based debugging to real-time metrics, and from reactive alerts to predictive anomaly detection transforms operations from chaotic to controlled.

Start simple: enable detailed monitoring, create CPU alarms, build a basic dashboard. Then evolve: add custom business metrics, implement Logs Insights queries, create composite alarms, integrate with Auto Scaling. Every iteration makes your systems more observable and your operations more informed.

Master CloudWatch, and you master AWS observability.

## Suggested Reading

- [AWS CloudWatch Official Documentation](https://docs.aws.amazon.com/cloudwatch/)
- [CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/)
- [CloudWatch Logs Insights Query Syntax](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax.html)
- [CloudWatch Agent Configuration](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Agent-Configuration-File-Details.html)
- [CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)
- [Container Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ContainerInsights.html)

{% include inarticle-adsense.html %}
