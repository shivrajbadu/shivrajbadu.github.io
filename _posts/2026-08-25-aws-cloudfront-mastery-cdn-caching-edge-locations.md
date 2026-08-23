---
layout: post
title: "AWS CloudFront Mastery: CDN, Caching, and Edge Locations"
date: 2026-08-25 11:00:00 +0545
categories: [AWS, CDN]
tags: [aws, cloudfront, cdn, edge-locations, caching, performance-optimization, content-delivery]
---

# AWS CloudFront Mastery: CDN, Caching, and Edge Locations

## Introduction

Your Rails application serves users globally. Someone in Tokyo requests your homepage. The request travels 10,000 kilometers to your `us-east-1` server, waits for the response, and travels 10,000 kilometers back. Total latency: 300-500ms just for network round-trip, before your application even processes the request. Images, CSS, JavaScript—every asset repeats this journey. Page load times suffer. Users abandon slow sites.

**Amazon CloudFront** is AWS's Content Delivery Network (CDN) that caches content at 400+ edge locations worldwide. Instead of traveling to your origin server, users retrieve cached content from the nearest edge location—reducing latency from hundreds of milliseconds to single digits.

But CloudFront is more than just caching. It's intelligent cache invalidation, custom SSL certificates, Lambda@Edge for edge computing, origin failover, real-time logs, and the foundation for globally distributed applications.

In this guide, we'll master CloudFront: creating distributions, configuring caching behaviors, implementing SSL, optimizing performance, and building architectures that serve millions of users with single-digit latency worldwide.

## What Is Amazon CloudFront?

**CloudFront** is a fast CDN service that securely delivers data, videos, applications, and APIs globally with low latency and high transfer speeds.

### Key CloudFront Features

| Feature | Description |
|---------|-------------|
| **400+ Edge Locations** | Serve content from location nearest to users |
| **Origin Support** | S3, ALB, EC2, custom HTTP servers |
| **Caching** | Reduce origin load, improve performance |
| **SSL/TLS** | Free certificates via ACM |
| **Lambda@Edge** | Run code at edge locations |
| **Real-time Metrics** | Monitor performance and usage |
| **DDoS Protection** | AWS Shield Standard included |

### CloudFront vs Direct Origin

| Metric | Direct Origin (us-east-1) | CloudFront |
|--------|---------------------------|------------|
| **Tokyo user latency** | 300-500ms | 10-30ms |
| **Sydney user latency** | 250-400ms | 15-40ms |
| **London user latency** | 80-150ms | 5-20ms |
| **Origin load** | 100% of requests | 5-20% (cache hit ratio 80-95%) |
| **Bandwidth cost** | Full price | Lower (CloudFront cheaper than EC2 egress) |

## Core Concepts

### Origins

**Origin** is the source of your content.

| Origin Type | Use Case | Example |
|-------------|----------|---------|
| **S3 Bucket** | Static assets, media files | `my-assets.s3.amazonaws.com` |
| **ALB/ELB** | Dynamic content, APIs | `myapp-alb-123.us-east-1.elb.amazonaws.com` |
| **EC2** | Custom applications | `ec2-54-23-45-67.compute-1.amazonaws.com` |
| **Custom HTTP** | External origins | `api.example.com` |

### Edge Locations

**Edge locations** are data centers where CloudFront caches content.

- **400+ edge locations** worldwide
- **13 regional edge caches** for less popular content
- Content cached based on TTL (Time To Live)

### Cache Behaviors

**Cache behaviors** define how CloudFront handles requests.

- **Path patterns:** `/images/*`, `/api/*`, `*.jpg`
- **Origins:** Which origin to fetch from
- **TTL:** How long to cache
- **Query strings:** Include in cache key?
- **Headers:** Forward to origin?

## Creating a CloudFront Distribution

### Distribution for S3 Static Website

```bash
# Create distribution
aws cloudfront create-distribution \
  --distribution-config '{
    "CallerReference": "'$(date +%s)'",
    "Comment": "Static website distribution",
    "Enabled": true,
    "Origins": {
      "Quantity": 1,
      "Items": [{
        "Id": "S3-my-website",
        "DomainName": "my-website-bucket.s3.amazonaws.com",
        "S3OriginConfig": {
          "OriginAccessIdentity": ""
        }
      }]
    },
    "DefaultCacheBehavior": {
      "TargetOriginId": "S3-my-website",
      "ViewerProtocolPolicy": "redirect-to-https",
      "AllowedMethods": {
        "Quantity": 2,
        "Items": ["GET", "HEAD"]
      },
      "ForwardedValues": {
        "QueryString": false,
        "Cookies": {"Forward": "none"}
      },
      "MinTTL": 0,
      "DefaultTTL": 86400,
      "MaxTTL": 31536000,
      "Compress": true
    },
    "PriceClass": "PriceClass_100",
    "ViewerCertificate": {
      "CloudFrontDefaultCertificate": true
    }
  }'
```

**Output:**
```json
{
  "Distribution": {
    "Id": "E1234567890ABC",
    "DomainName": "d123abc456def.cloudfront.net",
    "Status": "InProgress"
  }
}
```

**Wait for deployment:**
```bash
aws cloudfront wait distribution-deployed \
  --id E1234567890ABC
```

### Distribution for Rails Application (ALB Origin)

```bash
aws cloudfront create-distribution \
  --distribution-config '{
    "CallerReference": "'$(date +%s)'",
    "Comment": "Rails app distribution",
    "Enabled": true,
    "Origins": {
      "Quantity": 1,
      "Items": [{
        "Id": "ALB-myapp",
        "DomainName": "myapp-alb-123.us-east-1.elb.amazonaws.com",
        "CustomOriginConfig": {
          "HTTPPort": 80,
          "HTTPSPort": 443,
          "OriginProtocolPolicy": "https-only",
          "OriginSslProtocols": {
            "Quantity": 1,
            "Items": ["TLSv1.2"]
          }
        }
      }]
    },
    "DefaultCacheBehavior": {
      "TargetOriginId": "ALB-myapp",
      "ViewerProtocolPolicy": "redirect-to-https",
      "AllowedMethods": {
        "Quantity": 7,
        "Items": ["GET", "HEAD", "OPTIONS", "PUT", "POST", "PATCH", "DELETE"]
      },
      "ForwardedValues": {
        "QueryString": true,
        "Cookies": {"Forward": "all"},
        "Headers": {
          "Quantity": 3,
          "Items": ["Host", "CloudFront-Forwarded-Proto", "CloudFront-Is-Mobile-Viewer"]
        }
      },
      "MinTTL": 0,
      "DefaultTTL": 0,
      "MaxTTL": 0,
      "Compress": true
    },
    "PriceClass": "PriceClass_All"
  }'
```

**Key differences:**
- `CustomOriginConfig`: For non-S3 origins
- `AllowedMethods`: Include POST, PUT, DELETE for APIs
- `ForwardedValues`: Pass cookies, headers, query strings
- `TTL`: 0 for dynamic content (no caching)

{% include inarticle-adsense.html %}

## Cache Behaviors

Configure different caching rules for different paths.

### Example: Separate Static and Dynamic Content

```bash
# Update distribution with multiple behaviors
aws cloudfront update-distribution \
  --id E1234567890ABC \
  --distribution-config '{
    "CacheBehaviors": {
      "Quantity": 2,
      "Items": [
        {
          "PathPattern": "/assets/*",
          "TargetOriginId": "S3-assets",
          "ViewerProtocolPolicy": "redirect-to-https",
          "ForwardedValues": {
            "QueryString": false,
            "Cookies": {"Forward": "none"}
          },
          "MinTTL": 0,
          "DefaultTTL": 31536000,
          "MaxTTL": 31536000,
          "Compress": true
        },
        {
          "PathPattern": "/api/*",
          "TargetOriginId": "ALB-myapp",
          "ViewerProtocolPolicy": "https-only",
          "ForwardedValues": {
            "QueryString": true,
            "Cookies": {"Forward": "all"},
            "Headers": {"Quantity": 1, "Items": ["Authorization"]}
          },
          "MinTTL": 0,
          "DefaultTTL": 0,
          "MaxTTL": 0
        }
      ]
    }
  }'
```

**Routing logic:**
- `/assets/*` → S3 origin, cache 1 year
- `/api/*` → ALB origin, no caching
- `/*` (default) → ALB origin, default caching

## Custom Domain and SSL

### Step 1: Request Certificate

```bash
# Request ACM certificate in us-east-1 (required for CloudFront)
aws acm request-certificate \
  --domain-name myapp.com \
  --subject-alternative-names www.myapp.com \
  --validation-method DNS \
  --region us-east-1
```

### Step 2: Validate Certificate

Add CNAME records to Route 53 for validation.

### Step 3: Associate with Distribution

```bash
aws cloudfront update-distribution \
  --id E1234567890ABC \
  --distribution-config '{
    "Aliases": {
      "Quantity": 2,
      "Items": ["myapp.com", "www.myapp.com"]
    },
    "ViewerCertificate": {
      "ACMCertificateArn": "arn:aws:acm:us-east-1:123456789012:certificate/abc-123",
      "SSLSupportMethod": "sni-only",
      "MinimumProtocolVersion": "TLSv1.2_2021"
    }
  }'
```

### Step 4: Create Route 53 Alias Record

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
          "HostedZoneId": "Z2FDTNDATAQYW2",
          "DNSName": "d123abc456def.cloudfront.net",
          "EvaluateTargetHealth": false
        }
      }
    }]
  }'
```

**Note:** `Z2FDTNDATAQYW2` is CloudFront's fixed hosted zone ID.

## Cache Optimization

### TTL Configuration

```bash
# Set cache-control headers in Rails
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  def set_cache_headers
    if request.path.start_with?('/assets')
      expires_in 1.year, public: true
    elsif request.path.start_with?('/api')
      expires_in 0, public: false
    else
      expires_in 5.minutes, public: true
    end
  end
end
```

**HTTP headers:**
```
Cache-Control: public, max-age=31536000  # 1 year
Cache-Control: no-cache, no-store, must-revalidate  # No cache
Cache-Control: public, max-age=300  # 5 minutes
```

### Query String Caching

**Option 1: Ignore query strings** (best for static assets)
```json
{
  "ForwardedValues": {
    "QueryString": false
  }
}
```

**Option 2: Cache based on specific query strings**
```json
{
  "ForwardedValues": {
    "QueryString": true,
    "QueryStringCacheKeys": {
      "Quantity": 2,
      "Items": ["version", "format"]
    }
  }
}
```

**Example:**
- `/image.jpg?version=2&format=webp` → Cached separately
- `/image.jpg?version=2&user=123` → `user` ignored, same cache as above

### Compression

Enable automatic gzip/brotli compression:

```json
{
  "Compress": true
}
```

**CloudFront automatically compresses:**
- HTML, CSS, JavaScript
- JSON, XML
- Text files

**Result:** 60-80% smaller files, faster downloads.

## Cache Invalidation

Remove cached content before TTL expires.

### Invalidate Specific Paths

```bash
# Invalidate single file
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/index.html"

# Invalidate directory
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/assets/*"

# Invalidate everything (expensive!)
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/*"
```

**Pricing:** First 1,000 invalidation paths/month free, $0.005 per path after.

### Versioned Assets (Better Than Invalidation)

Instead of invalidating, use versioned filenames:

```ruby
# Rails asset pipeline automatically versions
# app/assets/stylesheets/application.css → application-abc123.css

<link rel="stylesheet" href="<%= asset_path('application.css') %>">
# Outputs: /assets/application-abc123.css
```

**Benefit:** New version = new URL = no invalidation needed.

## Origin Failover

Configure backup origin for high availability.

```bash
aws cloudfront create-distribution \
  --distribution-config '{
    "Origins": {
      "Quantity": 2,
      "Items": [
        {
          "Id": "Primary-ALB",
          "DomainName": "myapp-us-alb.elb.amazonaws.com",
          "CustomOriginConfig": {...}
        },
        {
          "Id": "Secondary-ALB",
          "DomainName": "myapp-eu-alb.elb.amazonaws.com",
          "CustomOriginConfig": {...}
        }
      ]
    },
    "OriginGroups": {
      "Quantity": 1,
      "Items": [{
        "Id": "FailoverGroup",
        "FailoverCriteria": {
          "StatusCodes": {
            "Quantity": 3,
            "Items": [500, 502, 504]
          }
        },
        "Members": {
          "Quantity": 2,
          "Items": [
            {"OriginId": "Primary-ALB"},
            {"OriginId": "Secondary-ALB"}
          ]
        }
      }]
    },
    "DefaultCacheBehavior": {
      "TargetOriginId": "FailoverGroup",
      ...
    }
  }'
```

**Behavior:**
- Primary returns 500/502/504 → CloudFront tries secondary
- Both fail → CloudFront returns error to user
- Automatic failback when primary recovers

## Lambda@Edge

Run code at CloudFront edge locations.

### Use Cases

- **Authentication:** Check JWT tokens at edge
- **A/B testing:** Route to different origins
- **Image resizing:** Generate thumbnails on-the-fly
- **Header manipulation:** Add security headers
- **URL rewriting:** Clean URLs

### Example: Add Security Headers

```javascript
// lambda-edge-security-headers.js
exports.handler = async (event) => {
  const response = event.Records[0].cf.response;
  const headers = response.headers;

  headers['strict-transport-security'] = [{
    key: 'Strict-Transport-Security',
    value: 'max-age=31536000; includeSubDomains'
  }];

  headers['x-content-type-options'] = [{
    key: 'X-Content-Type-Options',
    value: 'nosniff'
  }];

  headers['x-frame-options'] = [{
    key: 'X-Frame-Options',
    value: 'DENY'
  }];

  return response;
};
```

**Deploy:**
```bash
# Create Lambda function in us-east-1 (required)
aws lambda create-function \
  --function-name security-headers \
  --runtime nodejs18.x \
  --handler index.handler \
  --zip-file fileb://function.zip \
  --role arn:aws:iam::123456789012:role/lambda-edge-role \
  --region us-east-1

# Publish version
aws lambda publish-version \
  --function-name security-headers \
  --region us-east-1

# Associate with CloudFront
aws cloudfront update-distribution \
  --id E1234567890ABC \
  --distribution-config '{
    "DefaultCacheBehavior": {
      "LambdaFunctionAssociations": {
        "Quantity": 1,
        "Items": [{
          "LambdaFunctionARN": "arn:aws:lambda:us-east-1:123456789012:function:security-headers:1",
          "EventType": "origin-response"
        }]
      }
    }
  }'
```

## Monitoring and Logging

### Enable Access Logs

```bash
# Create S3 bucket for logs
aws s3 mb s3://myapp-cloudfront-logs

# Enable logging
aws cloudfront update-distribution \
  --id E1234567890ABC \
  --distribution-config '{
    "Logging": {
      "Enabled": true,
      "Bucket": "myapp-cloudfront-logs.s3.amazonaws.com",
      "Prefix": "cloudfront/",
      "IncludeCookies": false
    }
  }'
```

### CloudWatch Metrics

```bash
# Cache hit rate
aws cloudwatch get-metric-statistics \
  --namespace AWS/CloudFront \
  --metric-name CacheHitRate \
  --dimensions Name=DistributionId,Value=E1234567890ABC \
  --start-time 2024-01-15T00:00:00Z \
  --end-time 2024-01-15T23:59:59Z \
  --period 3600 \
  --statistics Average

# Request count
aws cloudwatch get-metric-statistics \
  --metric-name Requests \
  ...
```

**Key metrics:**
- `CacheHitRate`: > 80% is good
- `OriginLatency`: Origin response time
- `4xxErrorRate`: Client errors
- `5xxErrorRate`: Origin/CloudFront errors

## Real-World Architecture: Global Application

### Requirements

- Serve static assets globally
- Dynamic API requests with low latency
- Automatic failover between regions
- SSL/TLS encryption

### Architecture

```
User → CloudFront
  │
  ├─→ /assets/* → S3 (cache 1 year)
  │
  ├─→ /api/* → Origin Group
  │    ├─ Primary: ALB us-east-1
  │    └─ Failover: ALB eu-west-1
  │
  └─→ /* → ALB us-east-1 (cache 5 min)
```

### Cost Savings

**Without CloudFront (all traffic to origin):**
- Data transfer: 10TB/month × $0.09/GB = $900
- Origin compute: Higher (handles all requests)
- **Total:** ~$1,200/month

**With CloudFront (80% cache hit):**
- CloudFront data transfer: 10TB × $0.085/GB = $850
- Origin data transfer: 2TB × $0.09/GB = $180
- Origin compute: Lower (handles 20% requests)
- **Total:** ~$900/month

**Savings:** ~$300/month + improved performance.

## Security Best Practices

### 1. Origin Access Identity (OAI)

Prevent direct S3 access, force CloudFront:

```bash
# Create OAI
aws cloudfront create-cloud-front-origin-access-identity \
  --cloud-front-origin-access-identity-config '{
    "CallerReference": "'$(date +%s)'",
    "Comment": "OAI for my-website"
  }'

# Update S3 bucket policy
aws s3api put-bucket-policy \
  --bucket my-website-bucket \
  --policy '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity E1234567890ABC"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-website-bucket/*"
    }]
  }'
```

### 2. Geographic Restrictions

```bash
aws cloudfront update-distribution \
  --id E1234567890ABC \
  --distribution-config '{
    "Restrictions": {
      "GeoRestriction": {
        "RestrictionType": "whitelist",
        "Quantity": 2,
        "Items": ["US", "CA"]
      }
    }
  }'
```

### 3. AWS WAF Integration

```bash
aws cloudfront update-distribution \
  --id E1234567890ABC \
  --distribution-config '{
    "WebACLId": "arn:aws:wafv2:us-east-1:123456789012:global/webacl/myapp-waf/abc-123"
  }'
```

## Conclusion

AWS CloudFront transforms content delivery from origin-centric to edge-centric architecture. By caching content at 400+ global edge locations, you serve users with single-digit latency regardless of their location or your origin's location.

The shift from direct origin access to edge caching, from single-origin fragility to multi-origin failover, and from slow global delivery to instant edge serving transforms user experience from acceptable to exceptional.

Start simple: create a distribution, point it at S3 or ALB, add custom domain with SSL. Then evolve: optimize cache behaviors, implement origin failover, add Lambda@Edge, monitor cache hit rates. Every iteration makes your application faster and more reliable globally.

Master CloudFront, and you master global content delivery.

## Suggested Reading

- [AWS CloudFront Official Documentation](https://docs.aws.amazon.com/cloudfront/)
- [CloudFront Developer Guide](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/)
- [Lambda@Edge Documentation](https://docs.aws.amazon.com/lambda/latest/dg/lambda-edge.html)
- [CloudFront Caching Best Practices](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/ConfiguringCaching.html)
- [CloudFront Security](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/security.html)
- [Origin Failover Documentation](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/high_availability_origin_failover.html)

{% include inarticle-adsense.html %}
