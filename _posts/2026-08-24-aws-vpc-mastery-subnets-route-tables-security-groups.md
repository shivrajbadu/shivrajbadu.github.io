---
layout: post
title: "AWS VPC Mastery: Subnets, Route Tables, and Security Groups"
date: 2026-08-24 10:00:00 +0545
categories: [AWS, Networking]
tags: [aws, vpc, networking, subnets, security-groups, route-tables, devops, cloud-infrastructure]
---

# AWS VPC Mastery: Subnets, Route Tables, and Security Groups

## Introduction

You've launched your first EC2 instance, and it's running beautifully. Then you realize: this server is exposed directly to the internet. Anyone can attempt to connect. There's no firewall, no network isolation, no private communication between your application server and database. Your infrastructure is essentially a house with no walls, no doors, just furniture sitting in an open field.

This is where **Amazon Virtual Private Cloud (VPC)** comes in. VPC is your private, isolated section of the AWS cloud where you define your own network topology—subnets, IP ranges, route tables, internet gateways, and security rules. It's the foundation of secure, scalable AWS architecture.

Understanding VPC isn't just about networking theory—it's about designing infrastructure where your web servers can talk to the internet, your application servers live in a protected layer, and your databases are completely isolated from external access. It's about building defense in depth, not just hoping your security groups are

 configured correctly.

In this guide, we'll build VPC mastery from the ground up: what VPCs are, how to design subnet architecture, how traffic flows through route tables, and how security groups and NACLs create layered defense.

## What Is AWS VPC?

**Amazon VPC** is a logically isolated virtual network in AWS where you launch resources like EC2, RDS, Lambda, and more. Think of it as your own private data center in the cloud, but with the flexibility and scalability of AWS.

### Key VPC Characteristics

| Feature | Description |
|---------|-------------|
| **Region-specific** | Each VPC exists in one AWS region (spans all AZs in that region) |
| **IP address range** | You define the CIDR block (e.g., `10.0.0.0/16`) |
| **Subnets** | Divide VPC into smaller networks, each in a specific AZ |
| **Isolation** | Resources in different VPCs cannot communicate (unless peered) |
| **Connectivity** | Control internet access, VPN connections, Direct Connect |

### Default VPC vs Custom VPC

When you create an AWS account, AWS automatically creates a **default VPC** in each region:

| Aspect | Default VPC | Custom VPC |
|--------|-------------|------------|
| **CIDR block** | `172.31.0.0/16` | You choose (e.g., `10.0.0.0/16`) |
| **Subnets** | One public subnet per AZ | You design subnet layout |
| **Internet Gateway** | Attached automatically | You attach manually |
| **Use case** | Quick testing, learning | Production environments |

**Best practice:** Use custom VPCs for production workloads to maintain full control.

## VPC Components Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    AWS Region (us-east-1)                   │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐ │
│  │               VPC (10.0.0.0/16)                       │ │
│  │                                                       │ │
│  │  ┌────────────────────┐  ┌────────────────────┐     │ │
│  │  │  Public Subnet     │  │  Private Subnet    │     │ │
│  │  │  10.0.1.0/24       │  │  10.0.2.0/24       │     │ │
│  │  │  AZ: us-east-1a    │  │  AZ: us-east-1a    │     │ │
│  │  │                    │  │                    │     │ │
│  │  │  ┌──────────┐      │  │  ┌──────────┐     │     │ │
│  │  │  │ Web      │      │  │  │ App      │     │     │ │
│  │  │  │ Server   │◄─────┼──┼─►│ Server   │     │     │ │
│  │  │  └──────────┘      │  │  └──────────┘     │     │ │
│  │  │       │            │  │       │           │     │ │
│  │  └───────┼────────────┘  └───────┼───────────┘     │ │
│  │          │                       │                 │ │
│  │     ┌────▼─────┐            ┌────▼─────┐          │ │
│  │     │ Internet │            │   NAT    │          │ │
│  │     │ Gateway  │            │ Gateway  │          │ │
│  │     └──────────┘            └──────────┘          │ │
│  │          │                                        │ │
│  └──────────┼────────────────────────────────────────┘ │
│             │                                          │
└─────────────┼──────────────────────────────────────────┘
              │
         ┌────▼─────┐
         │ Internet │
         └──────────┘
```

**Components:**
1. **VPC**: The container for everything (`10.0.0.0/16`)
2. **Subnets**: Smaller networks within VPC
3. **Internet Gateway**: Allows communication with the internet
4. **NAT Gateway**: Allows private subnets to access internet (outbound only)
5. **Route Tables**: Direct traffic between subnets and gateways
6. **Security Groups**: Stateful firewalls at instance level
7. **NACLs**: Stateless firewalls at subnet level

## Creating a Custom VPC

Let's build a production-ready VPC with public and private subnets.

### Step 1: Create the VPC

```bash
# Create VPC with 10.0.0.0/16 CIDR block
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=production-vpc}]'
```

**Output:**
```json
{
  "Vpc": {
    "VpcId": "vpc-0abc123def456789",
    "State": "available",
    "CidrBlock": "10.0.0.0/16",
    "DhcpOptionsId": "dopt-12345678",
    "InstanceTenancy": "default"
  }
}
```

**CIDR block breakdown:**
- `10.0.0.0/16` = 65,536 IP addresses
- Range: `10.0.0.0` to `10.0.255.255`
- AWS reserves 5 IPs per subnet (network, gateway, DNS, future, broadcast)

### Step 2: Enable DNS Hostnames

```bash
aws ec2 modify-vpc-attribute \
  --vpc-id vpc-0abc123def456789 \
  --enable-dns-hostnames
```

This allows EC2 instances to get public DNS names.

{% include inarticle-adsense.html %}

## Designing Subnets

Subnets divide your VPC into smaller networks. Each subnet:
- Exists in **one Availability Zone**
- Has a smaller CIDR block within the VPC CIDR
- Can be **public** (internet-accessible) or **private** (internal only)

### Subnet Design Strategy

For a highly available architecture, create subnets across **multiple AZs**:

```
VPC: 10.0.0.0/16 (65,536 IPs)

Public Subnets (internet-facing):
├── 10.0.1.0/24 (us-east-1a) → Web servers, load balancers
├── 10.0.2.0/24 (us-east-1b) → Web servers, load balancers

Private Subnets (application tier):
├── 10.0.11.0/24 (us-east-1a) → App servers
├── 10.0.12.0/24 (us-east-1b) → App servers

Private Subnets (database tier):
├── 10.0.21.0/24 (us-east-1a) → RDS, ElastiCache
└── 10.0.22.0/24 (us-east-1b) → RDS, ElastiCache
```

**Why /24 subnets?**
- `/24` = 256 IPs per subnet
- AWS reserves 5 IPs → 251 usable
- Enough for typical web/app layers
- Leaves room for growth

### Creating Subnets

```bash
# Public subnet in AZ-1a
aws ec2 create-subnet \
  --vpc-id vpc-0abc123def456789 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=public-subnet-1a}]'

# Public subnet in AZ-1b
aws ec2 create-subnet \
  --vpc-id vpc-0abc123def456789 \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1b \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=public-subnet-1b}]'

# Private subnet in AZ-1a
aws ec2 create-subnet \
  --vpc-id vpc-0abc123def456789 \
  --cidr-block 10.0.11.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=private-subnet-1a}]'
```

**Enable auto-assign public IP for public subnets:**
```bash
aws ec2 modify-subnet-attribute \
  --subnet-id subnet-0abc123 \
  --map-public-ip-on-launch
```

## Internet Gateway

An **Internet Gateway (IGW)** enables communication between your VPC and the internet.

```bash
# Create Internet Gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=production-igw}]'

# Attach to VPC
aws ec2 attach-internet-gateway \
  --vpc-id vpc-0abc123def456789 \
  --internet-gateway-id igw-0def456ghi789
```

**Key point:** Internet Gateway alone doesn't make subnets public—you need route table entries.

## Route Tables

**Route tables** determine where network traffic is directed.

### Concept: Public vs Private Subnets

The difference between public and private subnets is **the route table**:

| Subnet Type | Route to Internet | Route Entry |
|-------------|-------------------|-------------|
| **Public** | Via Internet Gateway | `0.0.0.0/0 → igw-xxxxx` |
| **Private** | Via NAT Gateway (or none) | `0.0.0.0/0 → nat-xxxxx` |

### Create Public Route Table

```bash
# Create route table
aws ec2 create-route-table \
  --vpc-id vpc-0abc123def456789 \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=public-route-table}]'

# Add route to Internet Gateway
aws ec2 create-route \
  --route-table-id rtb-0abc123 \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-0def456ghi789

# Associate with public subnets
aws ec2 associate-route-table \
  --route-table-id rtb-0abc123 \
  --subnet-id subnet-0abc123  # public-subnet-1a

aws ec2 associate-route-table \
  --route-table-id rtb-0abc123 \
  --subnet-id subnet-0def456  # public-subnet-1b
```

**Result:** Traffic from public subnets to `0.0.0.0/0` (any internet IP) goes via IGW.

### Create Private Route Table (with NAT)

Private subnets need **outbound** internet access (for OS updates, API calls) but shouldn't accept inbound connections from the internet.

**Solution: NAT Gateway** (Network Address Translation)

```bash
# Allocate Elastic IP for NAT Gateway
aws ec2 allocate-address --domain vpc

# Create NAT Gateway in public subnet
aws ec2 create-nat-gateway \
  --subnet-id subnet-0abc123 \  # Must be public subnet
  --allocation-id eipalloc-0xyz789

# Create private route table
aws ec2 create-route-table \
  --vpc-id vpc-0abc123def456789 \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=private-route-table}]'

# Add route to NAT Gateway
aws ec2 create-route \
  --route-table-id rtb-0private123 \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id nat-0abc123def

# Associate with private subnets
aws ec2 associate-route-table \
  --route-table-id rtb-0private123 \
  --subnet-id subnet-0priv123  # private-subnet-1a
```

**Traffic flow:**
```
Private instance → NAT Gateway (in public subnet) → Internet Gateway → Internet
```

**Important:** NAT Gateway is **one-way**—instances can initiate connections outbound, but internet cannot initiate connections inbound.

## Security Groups

**Security Groups** are stateful firewalls that control traffic at the **instance level**.

### Key Characteristics

| Feature | Behavior |
|---------|----------|
| **Stateful** | If you allow inbound traffic, response is automatically allowed |
| **Default** | All outbound allowed, all inbound denied |
| **Rules** | Only allow rules (no explicit deny) |
| **Scope** | Attached to ENI (Elastic Network Interface) of instances |

### Creating Security Groups

**Example: Web server security group**

```bash
# Create security group
aws ec2 create-security-group \
  --group-name web-server-sg \
  --description "Security group for web servers" \
  --vpc-id vpc-0abc123def456789

# Allow HTTP from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-0webserver123 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow HTTPS from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-0webserver123 \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Allow SSH from your office IP only
aws ec2 authorize-security-group-ingress \
  --group-id sg-0webserver123 \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.0/24
```

**Example: Application server security group**

```bash
aws ec2 create-security-group \
  --group-name app-server-sg \
  --description "Security group for app servers" \
  --vpc-id vpc-0abc123def456789

# Allow traffic from web server security group only
aws ec2 authorize-security-group-ingress \
  --group-id sg-0appserver123 \
  --protocol tcp \
  --port 3000 \
  --source-group sg-0webserver123
```

**Key insight:** Reference security groups in rules instead of IP addresses—dynamic and more secure.

## Network ACLs (NACLs)

**Network ACLs** are stateless firewalls at the **subnet level**.

### Security Groups vs NACLs

| Feature | Security Group | NACL |
|---------|----------------|------|
| **Level** | Instance (ENI) | Subnet |
| **State** | Stateful | Stateless |
| **Rules** | Allow only | Allow & Deny |
| **Evaluation** | All rules evaluated | Rules in numbered order |
| **Default** | Deny inbound, allow outbound | Allow all traffic |

### When to Use NACLs

- **Additional layer of defense** (defense in depth)
- **Block specific IP addresses** (DDoS mitigation)
- **Enforce subnet-level policies** (compliance requirements)

### Creating Custom NACL

```bash
# Create NACL
aws ec2 create-network-acl \
  --vpc-id vpc-0abc123def456789 \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=public-nacl}]'

# Allow inbound HTTP (rule 100)
aws ec2 create-network-acl-entry \
  --network-acl-id acl-0abc123 \
  --ingress \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow

# Allow inbound HTTPS (rule 110)
aws ec2 create-network-acl-entry \
  --network-acl-id acl-0abc123 \
  --ingress \
  --rule-number 110 \
  --protocol tcp \
  --port-range From=443,To=443 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow

# Allow ephemeral ports for return traffic (rule 120)
aws ec2 create-network-acl-entry \
  --network-acl-id acl-0abc123 \
  --ingress \
  --rule-number 120 \
  --protocol tcp \
  --port-range From=1024,To=65535 \
  --cidr-block 0.0.0.0/0 \
  --rule-action allow

# Deny specific IP (rule 50 - evaluated first)
aws ec2 create-network-acl-entry \
  --network-acl-id acl-0abc123 \
  --ingress \
  --rule-number 50 \
  --protocol -1 \
  --cidr-block 198.51.100.0/24 \
  --rule-action deny
```

**Important:** NACLs are stateless, so you must create both inbound and outbound rules for two-way communication.

## VPC Peering

**VPC Peering** connects two VPCs, allowing instances to communicate as if in the same network.

```bash
# Create peering connection
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-requester123 \
  --peer-vpc-id vpc-accepter456

# Accept peering connection (from accepter account)
aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id pcx-0abc123

# Add routes in both VPCs
aws ec2 create-route \
  --route-table-id rtb-requester \
  --destination-cidr-block 10.1.0.0/16 \  # Accepter VPC CIDR
  --vpc-peering-connection-id pcx-0abc123
```

**Use cases:**
- Connecting development and production VPCs
- Multi-account architecture
- Shared services VPC (logging, monitoring)

## Real-World Architecture: Three-Tier Web Application

Let's design a complete VPC for a Rails application with web, app, and database tiers.

### Architecture Requirements

- **High availability:** Resources in multiple AZs
- **Security:** Public web tier, private app and DB tiers
- **Scalability:** Load balancer in front of web servers
- **Compliance:** Database isolated with no internet access

### VPC Design

```
VPC: 10.0.0.0/16

Availability Zone 1a:
├── Public Subnet (10.0.1.0/24)
│   ├── NAT Gateway
│   └── Application Load Balancer
├── Private App Subnet (10.0.11.0/24)
│   └── EC2: Rails app servers
└── Private DB Subnet (10.0.21.0/24)
    └── RDS PostgreSQL (Multi-AZ primary)

Availability Zone 1b:
├── Public Subnet (10.0.2.0/24)
│   └── Application Load Balancer
├── Private App Subnet (10.0.12.0/24)
│   └── EC2: Rails app servers
└── Private DB Subnet (10.0.22.0/24)
    └── RDS PostgreSQL (Multi-AZ standby)
```

### Security Group Design

```bash
# Load Balancer SG
aws ec2 create-security-group --group-name alb-sg \
  --description "ALB security group"
# Allow 80/443 from 0.0.0.0/0

# App Server SG
aws ec2 create-security-group --group-name app-sg \
  --description "App server security group"
# Allow 3000 from alb-sg only

# Database SG
aws ec2 create-security-group --group-name db-sg \
  --description "Database security group"
# Allow 5432 from app-sg only
```

### Route Tables

**Public subnets:**
- Route: `0.0.0.0/0 → Internet Gateway`
- Purpose: ALB receives internet traffic

**Private app subnets:**
- Route: `0.0.0.0/0 → NAT Gateway`
- Purpose: App servers can download packages, call external APIs

**Private DB subnets:**
- Route: (none - VPC local only)
- Purpose: Database completely isolated

## Troubleshooting VPC Connectivity

### Problem: EC2 Instance Can't Access Internet

**Checklist:**
1. Is instance in a public subnet with IGW route?
2. Does instance have public IP or Elastic IP?
3. Is security group allowing outbound traffic?
4. Is NACL allowing traffic (both inbound and outbound)?

```bash
# Check route table
aws ec2 describe-route-tables --route-table-id rtb-xxx

# Check security group
aws ec2 describe-security-groups --group-id sg-xxx

# Check NACL
aws ec2 describe-network-acls --network-acl-id acl-xxx
```

### Problem: Cannot Connect to RDS from EC2

**Checklist:**
1. Are EC2 and RDS in the same VPC?
2. Is RDS security group allowing traffic from EC2 security group?
3. Is RDS in private subnet (no public access)?
4. Is DB endpoint correct in connection string?

```bash
# Test connectivity
nc -zv your-rds-endpoint.amazonaws.com 5432
```

## Cost Optimization

### NAT Gateway Costs

NAT Gateways are charged per hour + data processed:
- **Cost:** ~$0.045/hour + $0.045/GB processed
- **Monthly:** ~$33/month + data charges

**Alternative: NAT Instance** (for low-traffic environments)
- Use t3.micro EC2 (~$7.50/month)
- Configure as NAT with iptables
- Lower cost but less reliable (single point of failure)

### VPC Endpoints

For accessing AWS services (S3, DynamoDB) from private subnets, use **VPC Endpoints** instead of NAT Gateway to save data transfer costs.

```bash
# Create S3 VPC endpoint (Gateway type - free!)
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-0abc123 \
  --service-name com.amazonaws.us-east-1.s3 \
  --route-table-ids rtb-private123
```

**Cost savings:** Eliminates NAT Gateway data processing fees for S3 traffic.

## Conclusion

AWS VPC transforms the cloud from a shared, insecure environment into your private, controlled network infrastructure. By mastering subnets, route tables, and security groups, you architect multi-tier applications with defense in depth—public-facing web servers, protected application layers, and completely isolated databases.

The shift from default VPCs to custom-designed network topologies, from flat networks to multi-AZ high-availability architectures, and from overly permissive security groups to least-privilege firewall rules transforms your AWS infrastructure from fragile to resilient.

Start with a simple VPC: one public subnet, one private subnet, basic security groups. As you grow, add multi-AZ redundancy, VPC peering for microservices, VPC endpoints for cost savings. Every layer of network design is a layer of security, availability, and operational control.

Master VPC, and you master AWS networking.

## Suggested Reading

- [AWS VPC Official Documentation](https://docs.aws.amazon.com/vpc/)
- [VPC Security Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/security-best-practices.html)
- [Amazon VPC Peering Guide](https://docs.aws.amazon.com/vpc/latest/peering/)
- [VPC Endpoints Documentation](https://docs.aws.amazon.com/vpc/latest/privatelink/)
- [AWS Well-Architected Framework - Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/)
- [Network ACLs Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)

{% include inarticle-adsense.html %}
