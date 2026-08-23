---
layout: post
title: "AWS Route 53 Mastery: DNS Routing Policies, Domains, Traffic Management and Health Checks"
date: 2026-08-25 10:00:00 +0545
categories: [AWS, Networking]
tags: [aws, route53, dns, routing-policies, domain-management, health-checks, traffic-routing, devops]
---

# AWS Route 53 Mastery: DNS Routing Policies, Domains, Traffic Management and Health Checks

## Introduction

Your application is running perfectly in `us-east-1`. Users in Europe report slow page loads—latency is killing their experience. You manually set up a replica in `eu-west-1`, but now you have two separate URLs. European users still hit the US servers because DNS points everywhere to the same place. You need intelligent routing based on geography, but editing DNS records for millions of users isn't feasible.

**Amazon Route 53** is AWS's highly available and scalable Domain Name System (DNS) web service. But it's far more than just DNS—it's intelligent traffic routing, health checking with automatic failover, domain registration, and the glue that connects users to the closest, healthiest resources worldwide.

Understanding Route 53 means mastering routing policies (simple, weighted, latency-based, geolocation, failover), integrating health checks for automatic failover, managing domains, and building global architectures that route users intelligently.

In this guide, we'll master Route 53: registering domains, configuring hosted zones, implementing routing policies, enabling health checks, and building production architectures that serve users from the optimal location.

## What Is Amazon Route 53?

**Route 53** is a highly available and scalable DNS service that translates domain names to IP addresses.

### Core Functions

| Function | Description |
|----------|-------------|
| **Domain Registration** | Register and manage domains (.com, .org, etc.) |
| **DNS Hosting** | Host DNS records for your domains |
| **Traffic Routing** | Intelligent routing policies (latency, geo, weighted) |
| **Health Checking** | Monitor endpoints and failover automatically |
| **DNS Failover** | Automatic rerouting on health check failures |

### Why "Route 53"?

DNS operates on port 53—hence the name.

## DNS Basics

Before diving into Route 53, understand DNS fundamentals.

### DNS Record Types

| Record | Purpose | Example |
|--------|---------|---------|
| **A** | Map domain to IPv4 address | `myapp.com → 54.23.45.67` |
| **AAAA** | Map domain to IPv6 address | `myapp.com → 2001:0db8::1` |
| **CNAME** | Alias one domain to another | `www.myapp.com → myapp.com` |
| **Alias** | AWS-specific, map to AWS resources | `myapp.com → ALB` |
| **MX** | Mail exchange servers | `mail.myapp.com → mail-server` |
| **TXT** | Text records (verification, SPF) | `myapp.com → "v=spf1..."` |
| **NS** | Name servers | `myapp.com → ns-123.awsdns-45.com` |

### A vs CNAME vs Alias

| Type | Points To | Can Use at Root | Cost |
|------|-----------|-----------------|------|
| **A** | IP address | ✅ Yes | Queries charged |
| **CNAME** | Domain name | ❌ No | Queries charged |
| **Alias** | AWS resource (ALB, CloudFront) | ✅ Yes | Free for AWS resources |

**Best practice:** Use **Alias records** for AWS resources (free, supports root domain).

## Creating Hosted Zone

A **hosted zone** is a container for DNS records for a specific domain.

### Create Hosted Zone

```bash
# Create hosted zone
aws route53 create-hosted-zone \
  --name myapp.com \
  --caller-reference $(date +%s) \
  --hosted-zone-config Comment="Production domain"
```

**Output:**
```json
{
  "HostedZone": {
    "Id": "/hostedzone/Z1234567890ABC",
    "Name": "myapp.com.",
    "CallerReference": "1705334400",
    "Config": {
      "Comment": "Production domain"
    },
    "ResourceRecordSetCount": 2
  },
  "NameServers": [
    "ns-123.awsdns-45.com",
    "ns-456.awsdns-78.org",
    "ns-789.awsdns-01.net",
    "ns-012.awsdns-34.co.uk"
  ]
}
```

### Update Domain Registrar

Point your domain to Route 53 name servers:

1. **Copy name servers** from output
2. **Log in to domain registrar** (GoDaddy, Namecheap, etc.)
3. **Update name servers** to Route 53's NS records
4. **Wait for propagation** (up to 48 hours, typically 1-2 hours)

**Verify propagation:**
```bash
dig myapp.com NS +short
# Should return Route 53 name servers
```

## Creating DNS Records

### Simple A Record

```bash
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "myapp.com",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [{"Value": "54.23.45.67"}]
      }
    }]
  }'
```

### Alias Record (ALB)

```bash
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "myapp.com",
        "Type": "A",
        "AliasTarget": {
          "HostedZoneId": "Z35SXDOTRQ7X7K",
          "DNSName": "myapp-alb-123.us-east-1.elb.amazonaws.com",
          "EvaluateTargetHealth": true
        }
      }
    }]
  }'
```

**Key:** `EvaluateTargetHealth: true` enables automatic failover if ALB becomes unhealthy.

### CNAME Record

```bash
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "www.myapp.com",
        "Type": "CNAME",
        "TTL": 300,
        "ResourceRecords": [{"Value": "myapp.com"}]
      }
    }]
  }'
```

{% include inarticle-adsense.html %}

## Routing Policies

Route 53 offers sophisticated routing beyond basic DNS.

### 1. Simple Routing

**Use case:** Single resource serving all traffic.

```bash
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "myapp.com",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [{"Value": "54.23.45.67"}]
      }
    }]
  }'
```

### 2. Weighted Routing

**Use case:** A/B testing, gradual migrations, canary deployments.

```bash
# 90% traffic to old version
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "myapp.com",
        "Type": "A",
        "SetIdentifier": "Old-Version",
        "Weight": 90,
        "TTL": 60,
        "ResourceRecords": [{"Value": "54.23.45.67"}]
      }
    }]
  }'

# 10% traffic to new version
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "myapp.com",
        "Type": "A",
        "SetIdentifier": "New-Version",
        "Weight": 10,
        "TTL": 60,
        "ResourceRecords": [{"Value": "54.23.45.68"}]
      }
    }]
  }'
```

**Traffic distribution:** 90% → old, 10% → new. Gradually increase weight for new version.

### 3. Latency-Based Routing

**Use case:** Serve users from region with lowest latency.

```bash
# US region
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "myapp.com",
        "Type": "A",
        "SetIdentifier": "US-East",
        "Region": "us-east-1",
        "TTL": 60,
        "ResourceRecords": [{"Value": "54.23.45.67"}]
      }
    }]
  }'

# EU region
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "myapp.com",
        "Type": "A",
        "SetIdentifier": "EU-West",
        "Region": "eu-west-1",
        "TTL": 60,
        "ResourceRecords": [{"Value": "52.50.100.200"}]
      }
    }]
  }'
```

**Behavior:**
- User in New York → routed to `us-east-1`
- User in London → routed to `eu-west-1`
- Route 53 measures latency from user's resolver to each region

### 4. Geolocation Routing

**Use case:** Content localization, regulatory compliance, geographic restrictions.

```bash
# Default (rest of world)
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "myapp.com",
        "Type": "A",
        "SetIdentifier": "Default",
        "GeoLocation": {"ContinentCode": "*"},
        "TTL": 60,
        "ResourceRecords": [{"Value": "54.23.45.67"}]
      }
    }]
  }'

# Europe
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "myapp.com",
        "Type": "A",
        "SetIdentifier": "Europe",
        "GeoLocation": {"ContinentCode": "EU"},
        "TTL": 60,
        "ResourceRecords": [{"Value": "52.50.100.200"}]
      }
    }]
  }'

# Germany (more specific)
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "myapp.com",
        "Type": "A",
        "SetIdentifier": "Germany",
        "GeoLocation": {"CountryCode": "DE"},
        "TTL": 60,
        "ResourceRecords": [{"Value": "52.57.100.150"}]
      }
    }]
  }'
```

**Hierarchy:** Country > Continent > Default

### 5. Failover Routing

**Use case:** Active-passive disaster recovery.

```bash
# Primary (active)
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "myapp.com",
        "Type": "A",
        "SetIdentifier": "Primary",
        "Failover": "PRIMARY",
        "HealthCheckId": "abc-123-health-check",
        "TTL": 60,
        "ResourceRecords": [{"Value": "54.23.45.67"}]
      }
    }]
  }'

# Secondary (passive, failover)
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "myapp.com",
        "Type": "A",
        "SetIdentifier": "Secondary",
        "Failover": "SECONDARY",
        "TTL": 60,
        "ResourceRecords": [{"Value": "52.50.100.200"}]
      }
    }]
  }'
```

**Behavior:**
- Primary healthy → all traffic to primary
- Primary fails health check → automatic failover to secondary
- Primary recovers → traffic returns to primary

### 6. Geoproximity Routing

**Use case:** Route based on geographic location with bias adjustment.

**Example:** Prefer us-west-2 over us-east-1 for users in California.

### 7. Multi-Value Answer Routing

**Use case:** Return multiple healthy IPs (client-side load balancing).

```bash
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "myapp.com",
        "Type": "A",
        "SetIdentifier": "Server-1",
        "MultiValueAnswer": true,
        "HealthCheckId": "health-check-1",
        "TTL": 60,
        "ResourceRecords": [{"Value": "54.23.45.67"}]
      }
    }]
  }'
```

**Behavior:** Route 53 returns up to 8 healthy IPs. Client chooses randomly.

## Health Checks

Health checks monitor endpoints and trigger failover.

### Create Health Check

```bash
# HTTP health check
aws route53 create-health-check \
  --health-check-config '{
    "Type": "HTTPS",
    "ResourcePath": "/health",
    "FullyQualifiedDomainName": "myapp.com",
    "Port": 443,
    "RequestInterval": 30,
    "FailureThreshold": 3
  }' \
  --health-check-tags Key=Name,Value=myapp-health-check
```

**Parameters:**
- `RequestInterval`: 30 (standard) or 10 (fast) seconds
- `FailureThreshold`: Consecutive failures before marking unhealthy
- `Type`: HTTP, HTTPS, TCP, CALCULATED, CLOUDWATCH_METRIC

### Monitor Health Check Status

```bash
aws route53 get-health-check-status \
  --health-check-id abc-123-health-check
```

**Output:**
```json
{
  "HealthCheckObservations": [
    {
      "Region": "us-east-1",
      "StatusReport": {
        "Status": "Success",
        "CheckedTime": "2024-01-15T10:00:00Z"
      }
    },
    {
      "Region": "eu-west-1",
      "StatusReport": {
        "Status": "Success",
        "CheckedTime": "2024-01-15T10:00:00Z"
      }
    }
  ]
}
```

### Calculated Health Checks

Monitor multiple endpoints (e.g., database + app server):

```bash
aws route53 create-health-check \
  --health-check-config '{
    "Type": "CALCULATED",
    "ChildHealthChecks": [
      "health-check-id-1",
      "health-check-id-2",
      "health-check-id-3"
    ],
    "HealthThreshold": 2
  }'
```

**Behavior:** Health check passes if at least 2 of 3 children are healthy.

### CloudWatch Alarm Health Checks

Monitor based on CloudWatch metrics:

```bash
aws route53 create-health-check \
  --health-check-config '{
    "Type": "CLOUDWATCH_METRIC",
    "AlarmIdentifier": {
      "Region": "us-east-1",
      "Name": "HighErrorRate"
    },
    "InsufficientDataHealthStatus": "Healthy"
  }'
```

**Use case:** Failover when application error rate exceeds threshold.

## Traffic Flow

**Traffic Flow** is a visual editor for complex routing policies.

### Use Cases

- Combine multiple routing policies (latency + failover)
- Multi-level failover (primary → secondary → tertiary)
- A/B testing with health checks

**Traffic Flow is a paid feature** ($50/month per policy).

## Domain Registration

Register domains directly through Route 53.

```bash
aws route53domains register-domain \
  --domain-name myawesomeapp.com \
  --duration-in-years 1 \
  --admin-contact '{...}' \
  --registrant-contact '{...}' \
  --tech-contact '{...}' \
  --privacy-protect-admin-contact \
  --privacy-protect-registrant-contact \
  --privacy-protect-tech-contact \
  --auto-renew
```

**Pricing:** Varies by TLD (.com = $12/year, .io = $39/year).

### Transfer Domain to Route 53

```bash
aws route53domains transfer-domain \
  --domain-name existing-domain.com \
  --duration-in-years 1 \
  --auth-code TRANSFER_AUTH_CODE
```

## Real-World Architecture: Global Application

### Requirements

- Serve users from nearest region (lowest latency)
- Automatic failover if region goes down
- 99.99% availability

### Architecture

```
User Request → Route 53 (Latency-Based + Failover)
  │
  ├─→ us-east-1 (Primary)
  │    ├─ ALB (health checked)
  │    └─ Auto Scaling Group
  │
  ├─→ eu-west-1 (Primary)
  │    ├─ ALB (health checked)
  │    └─ Auto Scaling Group
  │
  └─→ ap-southeast-1 (Primary)
       ├─ ALB (health checked)
       └─ Auto Scaling Group
```

### Implementation

**1. Create health checks for each ALB:**

```bash
# US health check
aws route53 create-health-check \
  --health-check-config '{
    "Type": "HTTPS",
    "ResourcePath": "/health",
    "FullyQualifiedDomainName": "myapp-alb-us.elb.amazonaws.com",
    "Port": 443,
    "RequestInterval": 30,
    "FailureThreshold": 2
  }'

# EU health check (similar)
# AP health check (similar)
```

**2. Create latency-based records with health checks:**

```bash
# US record
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890ABC \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "myapp.com",
        "Type": "A",
        "SetIdentifier": "US-East-ALB",
        "Region": "us-east-1",
        "AliasTarget": {
          "HostedZoneId": "Z35SXDOTRQ7X7K",
          "DNSName": "myapp-alb-us.elb.amazonaws.com",
          "EvaluateTargetHealth": true
        },
        "HealthCheckId": "us-health-check-id"
      }
    }]
  }'

# EU record (similar, region: eu-west-1)
# AP record (similar, region: ap-southeast-1)
```

**Result:**
- User in New York → `us-east-1` (lowest latency)
- User in London → `eu-west-1` (lowest latency)
- If `us-east-1` fails health check → reroute to next-closest healthy region
- **Availability:** 99.99%+ (automatic multi-region failover)

## Monitoring and Troubleshooting

### Query Logging

Enable logging to see all DNS queries:

```bash
# Create CloudWatch log group
aws logs create-log-group --log-group-name /aws/route53/myapp.com

# Enable query logging
aws route53 create-query-logging-config \
  --hosted-zone-id Z1234567890ABC \
  --cloud-watch-logs-log-group-arn arn:aws:logs:us-east-1:123456789012:log-group:/aws/route53/myapp.com
```

**Log format:**
```
1.0 2024-01-15T10:00:00Z Z1234567890ABC myapp.com A NOERROR 1 192.0.2.1 - us-east-1
```

### DNS Propagation Check

```bash
# Check from multiple locations
dig @8.8.8.8 myapp.com  # Google DNS
dig @1.1.1.1 myapp.com  # Cloudflare DNS
dig myapp.com +trace    # Full DNS hierarchy
```

### TTL Considerations

**Low TTL (60s):**
- ✅ Fast updates (useful for testing, migrations)
- ❌ More DNS queries (higher cost, more load)

**High TTL (3600s):**
- ✅ Fewer DNS queries (lower cost, cached longer)
- ❌ Slow updates (DNS changes take 1 hour to propagate)

**Best practice:** Use 300s (5 minutes) as default balance.

## Cost Optimization

### Route 53 Pricing

- **Hosted zone:** $0.50/month
- **Standard queries:** $0.40 per million queries
- **Latency-based queries:** $0.60 per million queries
- **Geo queries:** $0.70 per million queries
- **Health checks:** $0.50/month (standard), $1.00/month (fast)

### Cost Reduction Strategies

1. **Use Alias records for AWS resources** (free queries)
2. **Increase TTL** where feasible (reduce query volume)
3. **Delete unused health checks**
4. **Consolidate hosted zones** (multi-domain setup)

## Security Best Practices

### 1. Enable DNSSEC

Protect against DNS spoofing:

```bash
aws route53 enable-hosted-zone-dnssec \
  --hosted-zone-id Z1234567890ABC
```

### 2. Use IAM Policies

Restrict who can modify DNS records:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "route53:GetHostedZone",
      "route53:ListResourceRecordSets"
    ],
    "Resource": "arn:aws:route53:::hostedzone/Z1234567890ABC"
  }]
}
```

### 3. Enable Query Logging

Audit all DNS queries for security analysis.

## Conclusion

AWS Route 53 transforms DNS from a static lookup service into an intelligent traffic management system. By mastering routing policies, health checks, and failover configurations, you architect globally distributed applications that route users to the optimal endpoint automatically.

The shift from static DNS to latency-based routing, from manual failover to automatic health checks, and from single-region fragility to multi-region resilience transforms DNS from configuration into intelligence.

Start simple: create a hosted zone, add A records, point your domain. Then evolve: implement latency-based routing, add health checks, build multi-region failover. Every iteration makes your architecture more resilient and your users' experience better.

Master Route 53, and you master global traffic routing.

## Suggested Reading

- [AWS Route 53 Official Documentation](https://docs.aws.amazon.com/route53/)
- [Route 53 Routing Policies](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html)
- [Route 53 Health Checks](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover.html)
- [DNS Best Practices](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/best-practices-dns.html)
- [DNSSEC in Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-configuring-dnssec.html)
- [Domain Registration Guide](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/domain-register.html)

{% include inarticle-adsense.html %}
