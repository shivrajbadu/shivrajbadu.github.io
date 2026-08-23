---
layout: post
title: "AWS ELB Mastery: ALB, NLB, Gateway LB for Distribute Traffic for High Availability and Fault Tolerance"
date: 2026-08-25 09:00:00 +0545
categories: [AWS, Networking]
tags: [aws, elb, load-balancer, alb, nlb, high-availability, fault-tolerance, devops]
---

# AWS ELB Mastery: ALB, NLB, Gateway LB for High Availability and Fault Tolerance

## Introduction

Your Rails application runs on a single EC2 instance. Traffic is growing. One day, the instance fails—maybe a hardware issue, maybe a deployment gone wrong. Your entire application is down. Users see errors. Revenue stops. You frantically spin up a new instance, update DNS, and wait 5 minutes for propagation. By the time you're back online, you've lost users and credibility.

**Elastic Load Balancing (ELB)** eliminates single points of failure by distributing traffic across multiple healthy instances. But it's more than just round-robin traffic distribution—it's intelligent health checking, automatic failover, SSL termination, WebSocket support, and integration with Auto Scaling for elastic capacity.

But choosing the right load balancer isn't obvious. Application Load Balancer (ALB) for HTTP/HTTPS? Network Load Balancer (NLB) for ultra-low latency? Gateway Load Balancer for traffic inspection? Each has distinct use cases, capabilities, and cost profiles.

In this guide, we'll master ELB: understanding load balancer types, configuring target groups, implementing health checks, enabling SSL/TLS, integrating with Auto Scaling, and building production architectures that serve millions of requests with zero downtime.

## What Is Elastic Load Balancing?

**ELB** automatically distributes incoming traffic across multiple targets (EC2, containers, IP addresses, Lambda functions).

### ELB Types

| Type | Layer | Use Case | Protocol Support |
|------|-------|----------|------------------|
| **Application Load Balancer (ALB)** | Layer 7 (Application) | Web applications, microservices, HTTP(S) | HTTP, HTTPS, gRPC |
| **Network Load Balancer (NLB)** | Layer 4 (Transport) | Ultra-low latency, TCP/UDP traffic | TCP, UDP, TLS |
| **Gateway Load Balancer (GWLB)** | Layer 3 (Network) | Firewalls, intrusion detection | IP packets |
| **Classic Load Balancer (CLB)** | Legacy | Deprecated (use ALB/NLB instead) | HTTP, HTTPS, TCP |

### Why Load Balancers?

✅ **High Availability:** Distribute traffic across multiple AZs  
✅ **Fault Tolerance:** Automatic failover to healthy instances  
✅ **Scalability:** Add/remove instances without downtime  
✅ **SSL Termination:** Handle TLS encryption at load balancer  
✅ **Health Checking:** Route traffic only to healthy targets  
✅ **Session Stickiness:** Route users to same instance  

## Application Load Balancer (ALB)

**ALB** operates at Layer 7 (HTTP/HTTPS) with advanced routing capabilities.

### ALB Features

- **Content-based routing:** Route by URL path, hostname, headers
- **WebSocket support:** Full-duplex communication
- **HTTP/2 and gRPC:** Modern protocols
- **Lambda targets:** Serverless backends
- **Fixed response:** Return static responses (maintenance mode)
- **Redirect rules:** HTTP → HTTPS redirects

### Create ALB

```bash
# Create ALB
aws elbv2 create-load-balancer \
  --name myapp-alb \
  --subnets subnet-0abc123 subnet-0def456 \  # Public subnets, multi-AZ
  --security-groups sg-0alb123 \
  --scheme internet-facing \
  --type application \
  --ip-address-type ipv4 \
  --tags Key=Environment,Value=production
```

**Output:**
```json
{
  "LoadBalancers": [{
    "LoadBalancerArn": "arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/myapp-alb/abc123",
    "DNSName": "myapp-alb-123456789.us-east-1.elb.amazonaws.com"
  }]
}
```

### Create Target Group

Target group defines backend instances:

```bash
aws elbv2 create-target-group \
  --name myapp-targets \
  --protocol HTTP \
  --port 3000 \
  --vpc-id vpc-0abc123 \
  --health-check-protocol HTTP \
  --health-check-path /health \
  --health-check-interval-seconds 30 \
  --health-check-timeout-seconds 5 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 3 \
  --target-type instance  # or 'ip' for containers, 'lambda' for functions
```

### Register Targets

```bash
# Register EC2 instances
aws elbv2 register-targets \
  --target-group-arn arn:aws:elasticloadbalancing:...:targetgroup/myapp-targets/abc123 \
  --targets Id=i-0abc123 Id=i-0def456
```

### Create Listener

Listener defines how traffic is routed:

```bash
# HTTP listener (port 80)
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:...:loadbalancer/app/myapp-alb/abc123 \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:...:targetgroup/myapp-targets/abc123
```

**Traffic flow:**
```
User → ALB (port 80) → Target Group → EC2 instances (port 3000)
```

### Path-Based Routing

Route traffic based on URL path:

```bash
# Create target groups for different services
aws elbv2 create-target-group --name api-targets --protocol HTTP --port 3000 ...
aws elbv2 create-target-group --name frontend-targets --protocol HTTP --port 8080 ...

# Create rules
aws elbv2 create-rule \
  --listener-arn arn:aws:elasticloadbalancing:...:listener/app/myapp-alb/abc123 \
  --priority 1 \
  --conditions Field=path-pattern,Values='/api/*' \
  --actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:...:targetgroup/api-targets/abc123

aws elbv2 create-rule \
  --listener-arn arn:aws:elasticloadbalancing:...:listener/app/myapp-alb/abc123 \
  --priority 2 \
  --conditions Field=path-pattern,Values='/*' \
  --actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:...:targetgroup/frontend-targets/abc123
```

**Routing behavior:**
- `/api/users` → API servers (port 3000)
- `/dashboard` → Frontend servers (port 8080)

### Host-Based Routing

Route by hostname (multi-tenant):

```bash
aws elbv2 create-rule \
  --listener-arn arn:... \
  --priority 1 \
  --conditions Field=host-header,Values='admin.myapp.com' \
  --actions Type=forward,TargetGroupArn=arn:...:targetgroup/admin-targets/abc123

aws elbv2 create-rule \
  --listener-arn arn:... \
  --priority 2 \
  --conditions Field=host-header,Values='api.myapp.com' \
  --actions Type=forward,TargetGroupArn=arn:...:targetgroup/api-targets/abc123
```

### SSL/TLS Termination

Handle HTTPS at load balancer:

```bash
# Request ACM certificate
aws acm request-certificate \
  --domain-name myapp.com \
  --subject-alternative-names www.myapp.com \
  --validation-method DNS

# Create HTTPS listener
aws elbv2 create-listener \
  --load-balancer-arn arn:... \
  --protocol HTTPS \
  --port 443 \
  --certificates CertificateArn=arn:aws:acm:us-east-1:123456789012:certificate/abc123 \
  --ssl-policy ELBSecurityPolicy-TLS-1-2-2017-01 \
  --default-actions Type=forward,TargetGroupArn=arn:...:targetgroup/myapp-targets/abc123

# Redirect HTTP to HTTPS
aws elbv2 create-listener \
  --load-balancer-arn arn:... \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=redirect,RedirectConfig='{Protocol=HTTPS,Port=443,StatusCode=HTTP_301}'
```

**Security policies:**
- `ELBSecurityPolicy-TLS-1-2-2017-01`: TLS 1.2+ only (recommended)
- `ELBSecurityPolicy-2016-08`: TLS 1.0+ (legacy support)

{% include inarticle-adsense.html %}

## Network Load Balancer (NLB)

**NLB** operates at Layer 4 (TCP/UDP) for extreme performance.

### When to Use NLB

✅ **Use NLB when:**
- Ultra-low latency required (microseconds)
- Millions of requests per second
- Static IP addresses needed (Elastic IP)
- Preserve client IP address
- TCP/UDP protocols (non-HTTP)

❌ **Use ALB when:**
- HTTP/HTTPS routing logic needed
- Content-based routing
- WebSocket support
- Lambda targets

### NLB Performance Comparison

| Metric | ALB | NLB |
|--------|-----|-----|
| **Latency** | ~10ms | < 1ms |
| **Connections/sec** | Thousands | Millions |
| **Protocol** | HTTP/HTTPS/gRPC | TCP/UDP/TLS |
| **Static IP** | No | Yes (Elastic IP) |

### Create NLB

```bash
# Allocate Elastic IPs for NLB
aws ec2 allocate-address --domain vpc  # Do this for each subnet

# Create NLB
aws elbv2 create-load-balancer \
  --name myapp-nlb \
  --type network \
  --scheme internet-facing \
  --subnet-mappings SubnetId=subnet-0abc123,AllocationId=eipalloc-0xyz SubnetId=subnet-0def456,AllocationId=eipalloc-0uvw \
  --tags Key=Environment,Value=production
```

### NLB Target Group

```bash
aws elbv2 create-target-group \
  --name myapp-nlb-targets \
  --protocol TCP \
  --port 3000 \
  --vpc-id vpc-0abc123 \
  --health-check-protocol TCP \
  --health-check-interval-seconds 10 \
  --target-type instance
```

### NLB Use Case: Game Server

```
Game Clients (UDP)
  ↓
NLB (UDP:7777)
  ↓
Target Group
  ├─→ Game Server 1 (us-east-1a)
  ├─→ Game Server 2 (us-east-1a)
  └─→ Game Server 3 (us-east-1b)
```

```bash
# Create UDP listener
aws elbv2 create-listener \
  --load-balancer-arn arn:... \
  --protocol UDP \
  --port 7777 \
  --default-actions Type=forward,TargetGroupArn=arn:...:targetgroup/game-servers/abc123
```

## Gateway Load Balancer (GWLB)

**GWLB** enables deployment of third-party virtual appliances (firewalls, IDS/IPS).

### Architecture

```
Internet
  ↓
GWLB
  ├─→ Firewall Appliance 1
  ├─→ Firewall Appliance 2
  └─→ Firewall Appliance 3
  ↓
Application (after inspection)
```

### Use Cases

- **Network firewalls:** Palo Alto, Fortinet, Check Point
- **Intrusion detection:** Suricata, Snort
- **Deep packet inspection:** Custom appliances

**Most applications don't need GWLB**—focus on ALB/NLB for web applications.

## Health Checks

Health checks determine if targets are healthy.

### ALB Health Check Configuration

```bash
aws elbv2 modify-target-group \
  --target-group-arn arn:... \
  --health-check-protocol HTTP \
  --health-check-path /health \
  --health-check-interval-seconds 30 \
  --health-check-timeout-seconds 5 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 3 \
  --matcher HttpCode=200
```

**Parameters:**
- `health-check-interval`: Frequency (10-300 seconds)
- `healthy-threshold`: Consecutive successes to mark healthy
- `unhealthy-threshold`: Consecutive failures to mark unhealthy

### Rails Health Endpoint

```ruby
# config/routes.rb
get '/health', to: 'health#check'

# app/controllers/health_controller.rb
class HealthController < ApplicationController
  def check
    # Check database
    ActiveRecord::Base.connection.execute('SELECT 1')
    
    # Check Redis
    Redis.current.ping
    
    render json: { status: 'ok' }, status: :ok
  rescue => e
    render json: { status: 'error', message: e.message }, status: :service_unavailable
  end
end
```

**Health check logic:**
```
ALB sends request → /health endpoint → 200 OK → Healthy
ALB sends request → /health endpoint → 503 Error → Unhealthy after 3 failures
```

### View Target Health

```bash
aws elbv2 describe-target-health \
  --target-group-arn arn:...:targetgroup/myapp-targets/abc123
```

**Output:**
```json
{
  "TargetHealthDescriptions": [
    {
      "Target": {"Id": "i-0abc123", "Port": 3000},
      "HealthCheckPort": "3000",
      "TargetHealth": {"State": "healthy"}
    },
    {
      "Target": {"Id": "i-0def456", "Port": 3000},
      "TargetHealth": {
        "State": "unhealthy",
        "Reason": "Target.Timeout",
        "Description": "Connection to target timed out"
      }
    }
  ]
}
```

## Session Stickiness

**Stickiness** routes user to same target for session duration.

### Enable Stickiness (Cookie-based)

```bash
aws elbv2 modify-target-group-attributes \
  --target-group-arn arn:... \
  --attributes \
    Key=stickiness.enabled,Value=true \
    Key=stickiness.type,Value=lb_cookie \
    Key=stickiness.lb_cookie.duration_seconds,Value=86400  # 24 hours
```

**How it works:**
1. User makes first request
2. ALB sets cookie: `AWSALB=<target_id>`
3. Subsequent requests with cookie route to same target
4. Cookie expires after duration

**When to use:**
- Session data stored in memory (not recommended—use Redis instead)
- Stateful applications
- Testing/debugging

**When to avoid:**
- Stateless applications (better for scaling)
- Using external session store (Redis, database)

## Auto Scaling Integration

ALB/NLB integrate automatically with Auto Scaling Groups.

### Attach ALB to Auto Scaling Group

```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name myapp-asg \
  --launch-template LaunchTemplateName=myapp-template \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 3 \
  --target-group-arns arn:...:targetgroup/myapp-targets/abc123 \
  --vpc-zone-identifier "subnet-0abc123,subnet-0def456" \
  --health-check-type ELB \
  --health-check-grace-period 300
```

**Key parameter:** `health-check-type ELB`
- Auto Scaling uses ELB health checks (not just EC2 status)
- Instances failing health checks are terminated and replaced

**Traffic flow:**
```
User → ALB → Target Group
                ↓
        Auto Scaling Group (2-10 instances)
          ├─→ Instance 1 (healthy)
          ├─→ Instance 2 (healthy)
          ├─→ Instance 3 (unhealthy) ← Terminated by ASG
          └─→ Instance 4 (healthy) ← Launched by ASG
```

## Cross-Zone Load Balancing

Distribute traffic evenly across all targets in all enabled AZs.

### Without Cross-Zone Load Balancing

```
AZ-1a: 2 instances → Receive 50% of traffic (25% each)
AZ-1b: 4 instances → Receive 50% of traffic (12.5% each)
```

**Problem:** Uneven distribution (25% vs 12.5% per instance).

### With Cross-Zone Load Balancing

```
AZ-1a: 2 instances → Receive 33.3% each
AZ-1b: 4 instances → Receive 16.7% each
Total: 6 instances, evenly distributed
```

### Enable Cross-Zone

**ALB:** Enabled by default (free)  
**NLB:** Disabled by default (charges for cross-AZ data transfer)

```bash
# Enable for NLB
aws elbv2 modify-load-balancer-attributes \
  --load-balancer-arn arn:... \
  --attributes Key=load_balancing.cross_zone.enabled,Value=true
```

## Access Logs

Enable access logs for troubleshooting and analytics.

```bash
# Create S3 bucket for logs
aws s3 mb s3://myapp-alb-logs

# Enable access logs
aws elbv2 modify-load-balancer-attributes \
  --load-balancer-arn arn:... \
  --attributes \
    Key=access_logs.s3.enabled,Value=true \
    Key=access_logs.s3.bucket,Value=myapp-alb-logs \
    Key=access_logs.s3.prefix,Value=production-alb
```

**Log format:**
```
type time elb client:port target:port request_processing_time target_processing_time response_processing_time elb_status_code target_status_code received_bytes sent_bytes "request" "user_agent" ssl_cipher ssl_protocol target_group_arn "trace_id"
```

**Analysis with Athena:**
```sql
CREATE EXTERNAL TABLE alb_logs (
  type string,
  time string,
  elb string,
  client_port string,
  target_port string,
  request_processing_time double,
  target_processing_time double,
  response_processing_time double,
  elb_status_code string,
  target_status_code string,
  received_bytes bigint,
  sent_bytes bigint,
  request string
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
WITH SERDEPROPERTIES (
  'serialization.format' = '1',
  'input.regex' = '([^ ]*) ([^ ]*) ([^ ]*) ([^ ]*):([0-9]*) ([^ ]*)[:-]([0-9]*) ([-.0-9]*) ([-.0-9]*) ([-.0-9]*) (|[-0-9]*) (-|[-0-9]*) ([-0-9]*) ([-0-9]*) \"([^ ]*) ([^ ]*) (- |[^ ]*)\" \"([^\"]*)\".*'
)
LOCATION 's3://myapp-alb-logs/production-alb/';

-- Query slowest requests
SELECT request, target_processing_time
FROM alb_logs
WHERE target_processing_time > 1.0
ORDER BY target_processing_time DESC
LIMIT 100;
```

## Real-World Architecture: High-Traffic Rails Application

### Requirements

- Handle 100,000 requests/minute
- Zero downtime deployments
- Multi-region for global users
- SSL termination
- Auto-scale based on traffic

### Architecture

```
Route 53 (DNS)
  ├─→ us-east-1: ALB
  │    ├─→ Target Group
  │    │    └─→ Auto Scaling Group (2-20 instances)
  │    └─→ CloudWatch Alarms → Auto Scaling Policies
  │
  └─→ eu-west-1: ALB
       ├─→ Target Group
       │    └─→ Auto Scaling Group (2-15 instances)
       └─→ CloudWatch Alarms → Auto Scaling Policies
```

### Implementation

**1. Create ALB with HTTPS:**
```bash
aws elbv2 create-load-balancer \
  --name production-alb \
  --subnets subnet-public-1a subnet-public-1b \
  --security-groups sg-alb \
  --type application \
  --tags Key=Environment,Value=production

aws elbv2 create-listener \
  --load-balancer-arn arn:... \
  --protocol HTTPS --port 443 \
  --certificates CertificateArn=arn:aws:acm:...:certificate/abc123 \
  --default-actions Type=forward,TargetGroupArn=arn:...:targetgroup/prod-targets/abc123
```

**2. Create target group with health checks:**
```bash
aws elbv2 create-target-group \
  --name prod-targets \
  --protocol HTTP --port 3000 \
  --vpc-id vpc-0abc123 \
  --health-check-path /health \
  --health-check-interval-seconds 15 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 2
```

**3. Attach to Auto Scaling:**
```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name prod-asg \
  --launch-template LaunchTemplateName=rails-app \
  --min-size 2 --max-size 20 --desired-capacity 4 \
  --target-group-arns arn:...:targetgroup/prod-targets/abc123 \
  --vpc-zone-identifier "subnet-0abc123,subnet-0def456" \
  --health-check-type ELB \
  --health-check-grace-period 300

# Target tracking scaling
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name prod-asg \
  --policy-name request-count-scaling \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ALBRequestCountPerTarget",
      "ResourceLabel": "app/production-alb/abc123/targetgroup/prod-targets/def456"
    },
    "TargetValue": 1000.0
  }'
```

**Result:**
- **Availability:** 99.99% (multi-AZ)
- **Scalability:** 2-20 instances based on traffic
- **SSL:** Terminated at ALB
- **Zero-downtime deployments:** Rolling updates via ASG

## Monitoring and Troubleshooting

### Key CloudWatch Metrics

```bash
# Request count
aws cloudwatch get-metric-statistics \
  --namespace AWS/ApplicationELB \
  --metric-name RequestCount \
  --dimensions Name=LoadBalancer,Value=app/myapp-alb/abc123 \
  --start-time 2024-01-15T00:00:00Z \
  --end-time 2024-01-15T23:59:59Z \
  --period 300 \
  --statistics Sum

# Target response time
aws cloudwatch get-metric-statistics \
  --metric-name TargetResponseTime \
  --statistics Average \
  ...

# Unhealthy host count
aws cloudwatch get-metric-statistics \
  --metric-name UnHealthyHostCount \
  --statistics Maximum \
  ...
```

**Critical metrics:**
- `TargetResponseTime`: > 1s indicates slow backends
- `UnHealthyHostCount`: > 0 indicates failing instances
- `HTTPCode_Target_5XX_Count`: Backend errors
- `HTTPCode_ELB_5XX_Count`: Load balancer errors

### Common Issues

**Problem: All targets unhealthy**

**Checklist:**
1. Security group allows ALB → targets on health check port
2. Health check path returns 200 OK
3. Instances are running
4. Application is listening on correct port

**Problem: High latency**

**Causes:**
- Slow database queries
- Insufficient instance size
- Not enough instances (enable auto-scaling)
- Inefficient code (N+1 queries)

**Problem: 502 Bad Gateway**

**Causes:**
- Target closed connection before sending response
- Target sent malformed response
- Target took too long (idle timeout)

**Fix:** Increase idle timeout
```bash
aws elbv2 modify-load-balancer-attributes \
  --load-balancer-arn arn:... \
  --attributes Key=idle_timeout.timeout_seconds,Value=120
```

## Security Best Practices

### 1. Use Security Groups

```bash
# ALB security group
aws ec2 create-security-group \
  --group-name alb-sg \
  --description "ALB security group"

# Allow HTTPS from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-alb \
  --protocol tcp --port 443 --cidr 0.0.0.0/0

# Target security group
aws ec2 create-security-group \
  --group-name app-sg \
  --description "App servers"

# Allow traffic from ALB only
aws ec2 authorize-security-group-ingress \
  --group-id sg-app \
  --protocol tcp --port 3000 --source-group sg-alb
```

### 2. Use WAF

Attach AWS WAF to ALB for protection against:
- SQL injection
- Cross-site scripting (XSS)
- DDoS attacks
- Rate limiting

```bash
aws wafv2 associate-web-acl \
  --web-acl-arn arn:aws:wafv2:...:webacl/myapp-waf/abc123 \
  --resource-arn arn:aws:elasticloadbalancing:...:loadbalancer/app/myapp-alb/abc123
```

## Conclusion

AWS Elastic Load Balancing transforms single-instance fragility into distributed, fault-tolerant architecture. By mastering ALB for HTTP routing, NLB for ultra-low latency, health checks for automatic failover, and Auto Scaling integration, you architect applications that scale elastically and recover automatically from failures.

The shift from single servers to load-balanced fleets, from manual failover to automatic health checks, and from fixed capacity to elastic scaling transforms infrastructure from brittle to resilient.

Start simple: create an ALB, attach two instances, enable health checks. Then evolve: add SSL termination, implement path-based routing, integrate with Auto Scaling, enable access logs. Every iteration makes your architecture more reliable and your operations simpler.

Master ELB, and you master high availability.

## Suggested Reading

- [AWS ELB Official Documentation](https://docs.aws.amazon.com/elasticloadbalancing/)
- [Application Load Balancer Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/)
- [Network Load Balancer Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/)
- [ELB Best Practices](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/best-practices.html)
- [Target Groups Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html)
- [Health Checks Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html)

{% include inarticle-adsense.html %}
