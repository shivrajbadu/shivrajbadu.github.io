---
layout: post
title: "AWS EC2 Mastery: Deploy and Scale Virtual Servers in the Cloud"
date: 2026-08-24 11:00:00 +0545
categories: [AWS, Compute]
tags: [aws, ec2, virtual-servers, cloud-computing, auto-scaling, devops, infrastructure]
---

# AWS EC2 Mastery: Deploy and Scale Virtual Servers in the Cloud

## Introduction

Your application is growing. What started as a simple Rails app on a single server now handles thousands of requests per minute during peak hours. Your current hosting can't keep up—page loads are slow, requests are timing out, and manual server provisioning takes days. You need infrastructure that scales with demand, deploys in minutes not days, and only charges you for what you use.

**Amazon Elastic Compute Cloud (EC2)** is AWS's virtual server service that revolutionizes how we think about compute resources. Instead of purchasing physical servers, racking them in data centers, and managing hardware lifecycles, you launch virtual machines in minutes, scale capacity up or down based on demand, and pay only for the hours you consume.

But EC2 isn't just about launching instances. It's about understanding instance types for your workload, choosing the right AMI, configuring auto-scaling for traffic spikes, optimizing costs with reserved instances and spot pricing, and building resilient architectures across availability zones.

In this guide, we'll master EC2 from the ground up: launching instances, understanding instance families, configuring storage, implementing auto-scaling, and building production-grade architectures that handle millions of requests.

## What Is Amazon EC2?

**EC2** provides resizable compute capacity in the cloud—virtual servers you can launch in minutes and terminate when no longer needed.

### Key EC2 Characteristics

| Feature | Description |
|---------|-------------|
| **Virtual Servers** | Launch instances from Amazon Machine Images (AMIs) |
| **Instance Types** | Choose CPU, memory, storage, network capacity |
| **Elastic** | Scale capacity up/down within minutes |
| **Pay-as-you-go** | Pay only for running instances (per second billing) |
| **Global** | Launch in any AWS region and availability zone |

### Why EC2 Over Traditional Hosting?

| Aspect | Traditional Servers | EC2 |
|--------|-------------------|-----|
| **Provisioning** | Days to weeks | Minutes |
| **Scaling** | Buy more hardware | Launch more instances |
| **Cost Model** | Fixed (CapEx) | Variable (OpEx) |
| **Maintenance** | Your responsibility | AWS manages hypervisor layer |
| **Global Reach** | Expensive | Launch in 30+ regions instantly |

## EC2 Instance Types

EC2 offers dozens of instance types optimized for different workloads.

### Instance Family Naming Convention

Example: `t3.medium`

```
t3.medium
│└─┴────────── Size (nano, micro, small, medium, large, xlarge, etc.)
└──────────── Generation (t3 = 3rd gen of t family)
└────────────── Family (t = burstable)
```

### Major Instance Families

| Family | Name | Use Case | vCPU:RAM Ratio |
|--------|------|----------|----------------|
| **T3/T3a** | Burstable | Web servers, dev environments | 1:2 (1 vCPU : 2GB RAM) |
| **M5/M6i** | General Purpose | Balanced workloads, app servers | 1:4 |
| **C5/C6i** | Compute Optimized | CPU-intensive, batch processing | 1:2 |
| **R5/R6i** | Memory Optimized | Databases, caching, analytics | 1:8 |
| **X1/X2** | Memory Intensive | SAP HANA, in-memory databases | 1:16+ |
| **P3/P4** | GPU Instances | Machine learning, video encoding | GPU-based |
| **G4** | Graphics | Gaming, graphics workloads | GPU-based |
| **I3/I4i** | Storage Optimized | NoSQL databases, data warehouses | NVMe SSD |

### Choosing the Right Instance Type

**Web application (Rails/Django):**
- Start: `t3.medium` (2 vCPU, 4GB RAM)
- Production: `m5.large` (2 vCPU, 8GB RAM)

**Database (PostgreSQL):**
- Development: `t3.medium`
- Production: `r5.xlarge` (4 vCPU, 32GB RAM)

**Background worker (Sidekiq/Celery):**
- `c5.large` (2 vCPU, 4GB RAM)

**Machine learning training:**
- `p3.2xlarge` (8 vCPU, 61GB RAM, Tesla V100 GPU)

## Launching Your First EC2 Instance

Let's launch a Rails application server.

### Prerequisites

1. AWS CLI configured
2. VPC with public subnet (from VPC guide)
3. Key pair for SSH access

### Step 1: Create Key Pair

```bash
# Create SSH key pair
aws ec2 create-key-pair \
  --key-name my-app-key \
  --query 'KeyMaterial' \
  --output text > my-app-key.pem

# Secure the key
chmod 400 my-app-key.pem
```

### Step 2: Create Security Group

```bash
# Create security group in your VPC
aws ec2 create-security-group \
  --group-name rails-app-sg \
  --description "Security group for Rails application" \
  --vpc-id vpc-0abc123def456789

# Allow SSH from your IP
aws ec2 authorize-security-group-ingress \
  --group-id sg-0abc123 \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.0/24  # Your office IP

# Allow HTTP from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-0abc123 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow HTTPS from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-0abc123 \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0
```

### Step 3: Launch Instance

```bash
# Launch Ubuntu 22.04 LTS instance
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \  # Ubuntu 22.04 AMI (region-specific)
  --instance-type t3.medium \
  --key-name my-app-key \
  --security-group-ids sg-0abc123 \
  --subnet-id subnet-0abc123 \  # Public subnet
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=rails-app-server}]' \
  --user-data file://user-data.sh
```

**user-data.sh** (bootstrap script):
```bash
#!/bin/bash
# Update packages
apt-get update
apt-get upgrade -y

# Install Ruby dependencies
apt-get install -y git curl libssl-dev libreadline-dev zlib1g-dev \
  autoconf bison build-essential libyaml-dev libreadline-dev \
  libncurses5-dev libffi-dev libgdbm-dev

# Install rbenv
git clone https://github.com/rbenv/rbenv.git /home/ubuntu/.rbenv
git clone https://github.com/rbenv/ruby-build.git /home/ubuntu/.rbenv/plugins/ruby-build

# Install Ruby
/home/ubuntu/.rbenv/bin/rbenv install 3.2.2
/home/ubuntu/.rbenv/bin/rbenv global 3.2.2

# Install Node.js (for asset pipeline)
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt-get install -y nodejs

# Install Yarn
npm install -g yarn

echo "Bootstrap complete!" > /home/ubuntu/bootstrap.log
```

**Output:**
```json
{
  "Instances": [{
    "InstanceId": "i-0abc123def456789",
    "InstanceType": "t3.medium",
    "State": {"Name": "pending"},
    "PublicDnsName": "",
    "PrivateIpAddress": "10.0.1.50"
  }]
}
```

### Step 4: Wait for Instance to Run

```bash
# Wait for instance to be running
aws ec2 wait instance-running --instance-ids i-0abc123def456789

# Get public IP
aws ec2 describe-instances \
  --instance-ids i-0abc123def456789 \
  --query 'Reservations[0].Instances[0].PublicIpAddress' \
  --output text
```

### Step 5: Connect via SSH

```bash
ssh -i my-app-key.pem ubuntu@<public-ip>

# Verify bootstrap
cat /home/ubuntu/bootstrap.log
```

{% include inarticle-adsense.html %}

## Amazon Machine Images (AMIs)

An **AMI** is a template containing the OS, application server, and applications needed to launch an instance.

### AMI Types

| Type | Description | Use Case |
|------|-------------|----------|
| **AWS Official** | Amazon Linux, Ubuntu, Windows | Standard OS deployments |
| **Marketplace** | Pre-configured (LAMP, WordPress, etc.) | Quick setup |
| **Community** | User-contributed | Specific configurations |
| **Custom** | Your own images | Repeatable deployments |

### Creating a Custom AMI

After configuring your instance with application dependencies:

```bash
# Stop instance (optional but recommended)
aws ec2 stop-instances --instance-ids i-0abc123def456789

# Create AMI
aws ec2 create-image \
  --instance-id i-0abc123def456789 \
  --name "rails-app-v1.0" \
  --description "Rails 7.1 with Ruby 3.2.2, PostgreSQL client, Redis"

# Wait for AMI creation
aws ec2 wait image-available --image-ids ami-0newimage123
```

**Benefits:**
- Launch identical instances in seconds
- Version your infrastructure
- Disaster recovery (restore from known-good state)
- Multi-region deployment (copy AMI to other regions)

### Launching from Custom AMI

```bash
aws ec2 run-instances \
  --image-id ami-0newimage123 \
  --instance-type t3.medium \
  --key-name my-app-key \
  --security-group-ids sg-0abc123 \
  --subnet-id subnet-0abc123
```

**Result:** Instance boots with all dependencies pre-installed.

## EC2 Storage Options

EC2 instances can attach multiple storage types.

### Elastic Block Store (EBS)

**EBS** provides persistent block storage that persists independently of instance lifecycle.

| Volume Type | Use Case | IOPS | Throughput | Cost |
|-------------|----------|------|------------|------|
| **gp3** | General purpose SSD | 3,000-16,000 | 125-1,000 MB/s | $0.08/GB-month |
| **io2** | High-performance SSD | Up to 64,000 | Up to 1,000 MB/s | $0.125/GB-month + IOPS cost |
| **st1** | Throughput-optimized HDD | N/A | Up to 500 MB/s | $0.045/GB-month |
| **sc1** | Cold HDD (archives) | N/A | Up to 250 MB/s | $0.015/GB-month |

**Attach EBS volume:**
```bash
# Create 100GB gp3 volume
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 100 \
  --volume-type gp3 \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=app-data}]'

# Attach to instance
aws ec2 attach-volume \
  --volume-id vol-0abc123 \
  --instance-id i-0abc123 \
  --device /dev/sdf

# On instance: Format and mount
sudo mkfs -t ext4 /dev/sdf
sudo mkdir /data
sudo mount /dev/sdf /data
```

### Instance Store (Ephemeral Storage)

**Instance store** provides temporary storage physically attached to the host.

**Characteristics:**
- ✅ **High performance:** Low latency, high IOPS
- ✅ **No cost:** Included with instance type
- ❌ **Ephemeral:** Data lost on stop/terminate/hardware failure

**Use cases:**
- Temporary files
- Caches
- Buffers
- Stateless applications

## Auto Scaling

**Auto Scaling** automatically adjusts EC2 capacity based on demand.

### Components

1. **Launch Template:** Defines instance configuration
2. **Auto Scaling Group:** Manages instance fleet
3. **Scaling Policies:** Rules for scaling up/down

### Step 1: Create Launch Template

```bash
aws ec2 create-launch-template \
  --launch-template-name rails-app-template \
  --version-description "Rails app v1.0" \
  --launch-template-data '{
    "ImageId": "ami-0abc123",
    "InstanceType": "t3.medium",
    "KeyName": "my-app-key",
    "SecurityGroupIds": ["sg-0abc123"],
    "IamInstanceProfile": {"Name": "EC2-S3-Access-Profile"},
    "UserData": "IyEvYmluL2Jhc2gKL29wdC9hcHAvc3RhcnQuc2g="
  }'
```

### Step 2: Create Auto Scaling Group

```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name rails-app-asg \
  --launch-template LaunchTemplateName=rails-app-template,Version='$Latest' \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 3 \
  --vpc-zone-identifier "subnet-0abc123,subnet-0def456" \
  --health-check-type ELB \
  --health-check-grace-period 300 \
  --tags Key=Name,Value=rails-app-server,PropagateAtLaunch=true
```

**Parameters:**
- `min-size`: Minimum instances (always running)
- `max-size`: Maximum instances (cap for scaling)
- `desired-capacity`: Initial instance count

### Step 3: Configure Scaling Policies

**Target Tracking Scaling** (recommended):
```bash
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name rails-app-asg \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "TargetValue": 70.0
  }'
```

**Result:** Auto Scaling maintains average CPU utilization at 70%
- If CPU > 70%: Launch instances
- If CPU < 70%: Terminate instances

**Step Scaling** (more control):
```bash
# Scale up when CPU > 80%
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name rails-app-asg \
  --policy-name scale-up-policy \
  --scaling-adjustment 2 \
  --adjustment-type ChangeInCapacity

# Create CloudWatch alarm
aws cloudwatch put-metric-alarm \
  --alarm-name cpu-high \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --period 120 \
  --statistic Average \
  --threshold 80.0 \
  --alarm-actions arn:aws:autoscaling:region:account:scalingPolicy:policy-id
```

## Elastic Load Balancer Integration

Integrate Auto Scaling with Application Load Balancer for traffic distribution:

```bash
# Attach target group to Auto Scaling Group
aws autoscaling attach-load-balancer-target-groups \
  --auto-scaling-group-name rails-app-asg \
  --target-group-arns arn:aws:elasticloadbalancing:region:account:targetgroup/my-targets/abc123
```

**Traffic flow:**
```
Internet → ALB → Target Group → Auto Scaling Group (2-10 instances)
```

## EC2 Pricing Models

### 1. On-Demand Instances

**Pay per second** for instances you launch.

- **Cost:** `t3.medium` = $0.0416/hour (~$30/month)
- **Use case:** Unpredictable workloads, testing
- **No commitment:** Launch and terminate anytime

### 2. Reserved Instances

**Commit to 1-3 years** for significant discounts.

| Term | Payment | Discount |
|------|---------|----------|
| 1 year, No upfront | Monthly | ~30% |
| 1 year, All upfront | Upfront | ~40% |
| 3 years, All upfront | Upfront | ~60% |

**Example:** `t3.medium` reserved for 3 years = $0.0166/hour (~$12/month)

**Use case:** Steady-state workloads (databases, always-on app servers)

### 3. Spot Instances

**Bid on unused EC2 capacity** for up to 90% discount.

- **Cost:** `t3.medium` spot = $0.0125/hour (~$9/month)
- **Catch:** AWS can terminate with 2-minute notice
- **Use case:** Batch processing, fault-tolerant workloads, CI/CD workers

**Launch spot instance:**
```bash
aws ec2 request-spot-instances \
  --spot-price "0.05" \
  --instance-count 3 \
  --launch-specification file://spot-spec.json
```

### 4. Savings Plans

**Flexible pricing model** committing to consistent usage ($/hour) for 1-3 years.

- **Discount:** Up to 72% vs on-demand
- **Flexibility:** Applies across instance family, size, region, OS

**Use case:** Flexible workloads that change instance types frequently

## Real-World Architecture: High-Availability Rails Application

### Requirements

- Handle 10,000 requests/minute
- Zero downtime deployments
- Auto-scale during traffic spikes
- Multi-AZ for high availability

### Architecture

```
Internet
  │
  ▼
Application Load Balancer (us-east-1a, us-east-1b)
  │
  ├─────────────┬─────────────┐
  │             │             │
  ▼             ▼             ▼
Auto Scaling Group (2-10 instances)
├── t3.medium (us-east-1a)
├── t3.medium (us-east-1a)
├── t3.medium (us-east-1b)
└── t3.medium (us-east-1b)
       │
       ▼
RDS PostgreSQL (Multi-AZ)
ElastiCache Redis (Cluster Mode)
```

### Implementation

**1. Launch Template with IAM role:**
```bash
aws ec2 create-launch-template \
  --launch-template-name rails-production \
  --launch-template-data '{
    "ImageId": "ami-rails-app",
    "InstanceType": "t3.medium",
    "IamInstanceProfile": {"Name": "EC2-Production-Role"},
    "SecurityGroupIds": ["sg-app-servers"],
    "UserData": "base64-encoded-startup-script"
  }'
```

**2. Auto Scaling Group (multi-AZ):**
```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name production-asg \
  --launch-template LaunchTemplateName=rails-production \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 4 \
  --vpc-zone-identifier "subnet-1a,subnet-1b" \
  --target-group-arns arn:aws:elasticloadbalancing:...:targetgroup/rails-tg \
  --health-check-type ELB \
  --health-check-grace-period 300
```

**3. Scaling policy (target tracking):**
```bash
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name production-asg \
  --policy-name request-count-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ALBRequestCountPerTarget",
      "ResourceLabel": "app/my-alb/abc123/targetgroup/rails-tg/def456"
    },
    "TargetValue": 1000.0
  }'
```

**Result:** System automatically scales to maintain ~1000 requests/target.

## Monitoring and Troubleshooting

### CloudWatch Metrics

Monitor key EC2 metrics:

```bash
# Get CPU utilization
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-0abc123 \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-01T23:59:59Z \
  --period 3600 \
  --statistics Average
```

**Key metrics:**
- `CPUUtilization`: CPU usage percentage
- `NetworkIn/Out`: Network traffic
- `DiskReadOps/WriteOps`: Disk I/O
- `StatusCheckFailed`: Instance/system health

### Instance Metadata

Access instance information from within the instance:

```bash
# Get instance ID
curl http://169.254.169.254/latest/meta-data/instance-id

# Get IAM role credentials
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/EC2-Role

# Get user data
curl http://169.254.169.254/latest/user-data
```

### SSH via Systems Manager

Avoid opening port 22 by using AWS Systems Manager Session Manager:

```bash
# Install SSM agent (included in Amazon Linux, Ubuntu 16.04+)

# Start session (no key pair needed!)
aws ssm start-session --target i-0abc123def456789
```

**Benefits:**
- No inbound firewall rules
- Centralized audit logging
- No key management

## Security Best Practices

### 1. Use IAM Roles, Not Access Keys

❌ **Bad:** Hardcoded credentials on EC2
```bash
export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/...
```

✅ **Good:** IAM instance profile
```bash
aws ec2 run-instances \
  --iam-instance-profile Name=EC2-S3-Access-Profile \
  ...
```

### 2. Restrict Security Groups

❌ **Bad:** Open to world
```bash
# Allow SSH from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-abc123 \
  --protocol tcp --port 22 --cidr 0.0.0.0/0
```

✅ **Good:** Restrict to known IPs
```bash
# Allow SSH from office only
aws ec2 authorize-security-group-ingress \
  --group-id sg-abc123 \
  --protocol tcp --port 22 --cidr 203.0.113.0/24
```

### 3. Enable EBS Encryption

```bash
# Enable default EBS encryption
aws ec2 enable-ebs-encryption-by-default --region us-east-1

# Launch with encrypted root volume
aws ec2 run-instances \
  --block-device-mappings '[{
    "DeviceName":"/dev/xvda",
    "Ebs":{"Encrypted":true,"VolumeSize":20}
  }]' \
  ...
```

### 4. Regular AMI Updates

```bash
# Automate AMI creation with Lambda
# Monthly AMI builds with latest security patches
```

## Conclusion

AWS EC2 transforms computing from a capital-intensive, slow-to-provision fixed resource into an on-demand, elastic utility. By mastering instance types, AMIs, auto-scaling, and pricing models, you architect infrastructure that scales with your business, not ahead of it or behind it.

The shift from manual server provisioning to launch templates, from fixed capacity to auto-scaling groups, and from guessing capacity needs to metrics-driven scaling transforms infrastructure from a constraint into an enabler.

Start simple: launch a single instance, SSH in, deploy your application. Then evolve: create custom AMIs, implement auto-scaling, integrate with load balancers, optimize costs with reserved instances. Every iteration makes your infrastructure more resilient, more scalable, and more cost-effective.

Master EC2, and you master cloud computing.

## Suggested Reading

- [AWS EC2 Official Documentation](https://docs.aws.amazon.com/ec2/)
- [EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
- [Auto Scaling Documentation](https://docs.aws.amazon.com/autoscaling/)
- [EC2 Pricing](https://aws.amazon.com/ec2/pricing/)
- [AMI Best Practices](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)
- [EC2 Systems Manager](https://docs.aws.amazon.com/systems-manager/)

{% include inarticle-adsense.html %}
