---
layout: post
title: "AWS IAM Mastery: Users, Roles, Policies, and Best Practices"
date: 2026-08-24 09:00:00 +0545
categories: [AWS, Security]
tags: [aws, iam, security, access-control, authentication, authorization, devops, cloud-security]
---

# AWS IAM Mastery: Users, Roles, Policies, and Best Practices

## Introduction

You've just created your first AWS account. You have root access to everything—EC2 instances, S3 buckets, databases, billing. It feels powerful. Then reality hits: your development team needs access to deploy applications, your DevOps engineer needs to manage infrastructure, your data analyst needs read-only access to S3, and your auditor needs to review logs without changing anything.

How do you grant the right level of access to each person without sharing your root password? How do you let your application running on EC2 access S3 buckets without hardcoding credentials? How do you ensure a compromised key can't delete your entire production database?

**AWS Identity and Access Management (IAM)** is the answer. It's not just about creating users and passwords—it's the foundational security layer that controls who (or what) can do what in your AWS account. Understanding IAM isn't optional; it's the difference between a secure cloud infrastructure and a data breach waiting to happen.

In this guide, we'll master IAM from first principles: users, groups, roles, policies, permissions boundaries, and the security best practices that separate amateur AWS usage from production-grade infrastructure.

## What Is AWS IAM?

IAM is AWS's identity and access management service that enables you to:

✅ **Control who can authenticate** (sign in) to your AWS account  
✅ **Control what authenticated identities can do** (authorization)  
✅ **Manage access for humans** (users) and **applications** (roles)  
✅ **Enforce security policies** at scale across your organization  
✅ **Audit access** through detailed logs (CloudTrail integration)

### IAM Is Global

Unlike EC2 (region-specific) or S3 (bucket in specific regions), **IAM is a global service**. When you create an IAM user, role, or policy, it exists across all AWS regions simultaneously.

### IAM Is Free

AWS does not charge for IAM usage. You only pay for the AWS resources that IAM identities access (EC2, S3, RDS, etc.).

## Core IAM Concepts

### 1. IAM Users

An **IAM user** represents a person or service that interacts with AWS.

**When to create IAM users:**
- Individual developers who need AWS Console access
- CI/CD pipelines that deploy via CLI/SDK
- Third-party monitoring tools accessing your account

**Anatomy of an IAM user:**
```
User: john.doe@company.com
├── Username: john.doe
├── Password: (for AWS Console access)
├── Access Keys: (for CLI/SDK access)
│   ├── Access Key ID: AKIAIOSFODNN7EXAMPLE
│   └── Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
├── Permissions: (via attached policies)
└── MFA Device: (optional but recommended)
```

**Creating an IAM user via AWS CLI:**

```bash
# Create user
aws iam create-user --user-name john.doe

# Create login profile (console password)
aws iam create-login-profile \
  --user-name john.doe \
  --password 'TempPassword123!' \
  --password-reset-required

# Create access key (for CLI/SDK)
aws iam create-access-key --user-name john.doe
```

**Output:**
```json
{
  "AccessKey": {
    "UserName": "john.doe",
    "AccessKeyId": "AKIAIOSFODNN7EXAMPLE",
    "Status": "Active",
    "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
  }
}
```

⚠️ **Security note:** Store the `SecretAccessKey` securely—it's shown only once!

### 2. IAM Groups

**Groups** are collections of IAM users. Instead of attaching policies to each user individually, attach policies to groups.

**Example group structure:**
```
Groups:
├── Developers
│   ├── john.doe
│   ├── jane.smith
│   └── Policies: EC2FullAccess, S3ReadOnly
├── DevOps
│   ├── alice.admin
│   └── Policies: AdministratorAccess
└── DataAnalysts
    ├── bob.data
    └── Policies: S3ReadOnly, AthenaFullAccess
```

**Creating groups and adding users:**

```bash
# Create group
aws iam create-group --group-name Developers

# Add user to group
aws iam add-user-to-group \
  --user-name john.doe \
  --group-name Developers

# Attach policy to group
aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess
```

**Best practice:** Use groups to manage permissions, not individual user policies.

### 3. IAM Roles

**Roles** are identities with permissions, but unlike users, they're **assumed** by:
- AWS services (EC2, Lambda, ECS)
- Users from another AWS account
- Federated users (via SAML, OIDC)
- Applications running on EC2

**Key difference: Users vs Roles**

| Aspect | IAM User | IAM Role |
|--------|----------|----------|
| **Purpose** | Long-term identity for a person/service | Temporary identity assumed on-demand |
| **Credentials** | Permanent (password/access keys) | Temporary security credentials (rotated automatically) |
| **Use case** | Human accessing AWS Console/CLI | EC2 instance accessing S3, Lambda accessing DynamoDB |

**Why roles are more secure:**
- No hardcoded credentials in code
- Automatic credential rotation (typically every 15 minutes - 12 hours)
- Least privilege via temporary permissions

**Example: EC2 instance accessing S3**

❌ **Bad approach** (hardcoded credentials):
```ruby
# In your Rails app on EC2
Aws.config.update(
  credentials: Aws::Credentials.new(
    'AKIAIOSFODNN7EXAMPLE',
    'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY'
  )
)
```

Problems:
- Credentials in code (can be leaked via Git)
- Manual rotation required
- Broad permissions risk

✅ **Good approach** (IAM role):

1. Create IAM role with S3 access:
```bash
# Create trust policy (who can assume this role)
cat > trust-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Service": "ec2.amazonaws.com" },
    "Action": "sts:AssumeRole"
  }]
}
EOF

# Create role
aws iam create-role \
  --role-name EC2-S3-Access-Role \
  --assume-role-policy-document file://trust-policy.json

# Attach S3 policy
aws iam attach-role-policy \
  --role-name EC2-S3-Access-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create instance profile (wrapper for EC2)
aws iam create-instance-profile \
  --instance-profile-name EC2-S3-Access-Profile

# Add role to instance profile
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-S3-Access-Profile \
  --role-name EC2-S3-Access-Role
```

2. Attach role to EC2 instance:
```bash
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t3.micro \
  --iam-instance-profile Name=EC2-S3-Access-Profile
```

3. Rails app code (no credentials needed!):
```ruby
# AWS SDK automatically uses instance role credentials
s3 = Aws::S3::Client.new(region: 'us-east-1')
s3.list_buckets
```

The AWS SDK automatically:
- Detects it's running on EC2
- Fetches temporary credentials from instance metadata
- Rotates credentials before expiry

{% include inarticle-adsense.html %}

### 4. IAM Policies

**Policies** are JSON documents that define permissions. They specify:
- **What actions** are allowed/denied (`s3:GetObject`, `ec2:StartInstances`)
- **Which resources** the actions apply to (`arn:aws:s3:::my-bucket/*`)
- **Under what conditions** (IP restrictions, MFA required, time-based)

**Policy structure:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

**Policy types:**

| Type | Description | Example |
|------|-------------|---------|
| **AWS Managed** | Pre-built by AWS, cover common use cases | `AmazonS3ReadOnlyAccess` |
| **Customer Managed** | Custom policies you create | `MyApp-S3-Access` |
| **Inline** | Embedded directly in user/role/group | (Avoid—hard to manage) |

**Policy evaluation logic:**

```
Request → IAM evaluates all applicable policies
          ↓
      Explicit Deny? → ❌ DENY (overrides everything)
          ↓ No
      Explicit Allow? → ✅ ALLOW
          ↓ No
      ❌ DENY (default deny)
```

**Key principle:** **Explicit deny always wins.**

### 5. Policy Example: Least Privilege S3 Access

Let's create a policy that allows:
- Read/write access to `my-app-uploads/` prefix
- Read-only access to `my-app-public/` prefix
- No access to anything else

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::my-app-bucket",
      "Condition": {
        "StringLike": {
          "s3:prefix": [
            "uploads/*",
            "public/*"
          ]
        }
      }
    },
    {
      "Sid": "ReadWriteUploads",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::my-app-bucket/uploads/*"
    },
    {
      "Sid": "ReadOnlyPublic",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-app-bucket/public/*"
    }
  ]
}
```

**Create and attach the policy:**

```bash
# Save policy as custom-s3-policy.json
aws iam create-policy \
  --policy-name MyApp-S3-Limited-Access \
  --policy-document file://custom-s3-policy.json

# Attach to user
aws iam attach-user-policy \
  --user-name john.doe \
  --policy-arn arn:aws:iam::123456789012:policy/MyApp-S3-Limited-Access
```

## Policy Variables and Conditions

IAM policies support **dynamic variables** and **conditions** for fine-grained access control.

### Policy Variables

Use `${aws:username}` to create user-specific permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "s3:*",
    "Resource": "arn:aws:s3:::my-bucket/${aws:username}/*"
  }]
}
```

**Result:** Each user can only access their own folder in the bucket (`my-bucket/john.doe/`, `my-bucket/jane.smith/`).

### Condition Keys

Enforce security requirements:

**Require MFA for critical operations:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "ec2:TerminateInstances",
    "Resource": "*",
    "Condition": {
      "Bool": { "aws:MultiFactorAuthPresent": "true" }
    }
  }]
}
```

**Restrict access by IP address:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "s3:*",
    "Resource": "*",
    "Condition": {
      "IpAddress": {
        "aws:SourceIp": ["203.0.113.0/24", "198.51.100.0/24"]
      }
    }
  }]
}
```

**Time-based access (business hours only):**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "rds:*",
    "Resource": "*",
    "Condition": {
      "DateGreaterThan": {"aws:CurrentTime": "2024-01-01T09:00:00Z"},
      "DateLessThan": {"aws:CurrentTime": "2024-12-31T17:00:00Z"}
    }
  }]
}
```

## IAM Best Practices

### 1. Never Use Root Account

Your AWS root account has unrestricted access to **everything**, including billing.

**What to do:**
1. Enable MFA on root account
2. Create IAM admin user with `AdministratorAccess` policy
3. Lock root credentials in a secure vault
4. Use root account only for:
   - Changing account settings
   - Closing AWS account
   - Restoring IAM admin access if lost

### 2. Enable MFA Everywhere

Multi-factor authentication adds a second layer of security.

**Enable MFA via CLI:**
```bash
# Create virtual MFA device
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name john-doe-mfa \
  --outfile qr-code.png \
  --bootstrap-method QRCodePNG

# Scan QR code with Google Authenticator
# Get two consecutive codes, then enable MFA
aws iam enable-mfa-device \
  --user-name john.doe \
  --serial-number arn:aws:iam::123456789012:mfa/john-doe-mfa \
  --authentication-code-1 123456 \
  --authentication-code-2 789012
```

### 3. Use Roles for Applications

**Never hardcode AWS credentials** in application code or config files.

**For EC2/ECS/Lambda:** Use IAM roles (as shown earlier)  
**For local development:** Use AWS CLI profiles with named credentials

**Local development setup:**
```bash
# Configure named profile
aws configure --profile myapp-dev
AWS Access Key ID: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key: wJalrXUtnFEMI/...
Default region name: us-east-1

# Use in application
export AWS_PROFILE=myapp-dev
rails server
```

### 4. Apply Least Privilege

Grant only the permissions required for the task, nothing more.

❌ **Bad:** `AdministratorAccess` for everyone  
✅ **Good:** Custom policies with minimal required actions

**Example: Deploy-only role for CI/CD**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:PutObjectAcl"
      ],
      "Resource": "arn:aws:s3:::my-app-deployments/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ecs:UpdateService",
        "ecs:DescribeServices"
      ],
      "Resource": "arn:aws:ecs:us-east-1:123456789012:service/my-cluster/my-service"
    }
  ]
}
```

This role can **only** deploy to specific S3 bucket and ECS service—nothing else.

### 5. Rotate Credentials Regularly

**Access keys should be rotated** every 90 days.

**Check access key age:**
```bash
aws iam list-access-keys --user-name john.doe
```

**Rotate access keys:**
```bash
# Create new key
aws iam create-access-key --user-name john.doe

# Update applications to use new key
# Test thoroughly

# Delete old key (after confirming new key works)
aws iam delete-access-key \
  --user-name john.doe \
  --access-key-id AKIAIOSFODNN7EXAMPLE
```

### 6. Use IAM Access Analyzer

AWS IAM Access Analyzer identifies resources shared with external entities.

**Enable Access Analyzer:**
```bash
aws accessanalyzer create-analyzer \
  --analyzer-name my-account-analyzer \
  --type ACCOUNT
```

**Review findings:**
```bash
aws accessanalyzer list-findings \
  --analyzer-arn arn:aws:access-analyzer:us-east-1:123456789012:analyzer/my-account-analyzer
```

### 7. Monitor with CloudTrail

Every IAM action is logged in CloudTrail.

**Query recent IAM changes:**
```bash
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=CreateUser \
  --max-results 10
```

## Common IAM Patterns

### Pattern 1: Cross-Account Access

Allow users from Account A to access resources in Account B.

**In Account B (resource account), create role:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::111111111111:root"
    },
    "Action": "sts:AssumeRole"
  }]
}
```

**In Account A, allow users to assume the role:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "sts:AssumeRole",
    "Resource": "arn:aws:iam::222222222222:role/CrossAccountS3Access"
  }]
}
```

**Assume role from Account A:**
```bash
aws sts assume-role \
  --role-arn arn:aws:iam::222222222222:role/CrossAccountS3Access \
  --role-session-name my-session
```

### Pattern 2: Service Control Policies (SCPs)

In AWS Organizations, SCPs set permission boundaries for entire accounts.

**Example: Prevent EC2 termination in production account**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Action": "ec2:TerminateInstances",
    "Resource": "*"
  }]
}
```

**Apply to organization unit:**
```bash
aws organizations attach-policy \
  --policy-id p-12345678 \
  --target-id ou-prod-12345
```

## Troubleshooting IAM Issues

### Problem: "Access Denied" Error

**Diagnose with IAM Policy Simulator:**
```bash
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:user/john.doe \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::my-bucket/file.txt
```

**Common causes:**
1. Missing permission in policy
2. Explicit deny somewhere (check SCPs, permission boundaries)
3. Resource-based policy (S3 bucket policy) blocking access
4. Incorrect resource ARN in policy

### Problem: Role Cannot Be Assumed

**Check trust policy:**
```bash
aws iam get-role --role-name MyRole
```

**Verify `Principal` allows your service/account to assume it.**

### Problem: Policy Too Complex

**Use managed policies when possible:**
```bash
aws iam list-policies --scope AWS | grep S3
```

**Break large inline policies into multiple managed policies.**

## Real-World Example: Rails App on EC2

Let's design IAM architecture for a Rails application:

**Requirements:**
- EC2 instances need S3 access (user uploads)
- EC2 instances need RDS access (application database)
- Developers need SSH and deployment access
- CloudWatch logging enabled

**IAM architecture:**

```
1. EC2 Instance Role: Rails-App-EC2-Role
   Policies:
   - S3 access to uploads bucket
   - CloudWatch Logs write access
   - Systems Manager (for SSH via Session Manager)

2. RDS Security: Database security groups (not IAM)
   - SG allows traffic only from EC2 security group

3. Developer IAM Group: Rails-Developers
   Policies:
   - EC2 read/write (for deployments)
   - S3 read/write (uploads bucket only)
   - CloudWatch Logs read (for debugging)
   - Systems Manager StartSession (for SSH)

4. CI/CD Role: CircleCI-Deploy-Role
   Policies:
   - S3 PutObject (for assets)
   - EC2 DescribeInstances, StartInstances, StopInstances
   - Systems Manager SendCommand (for deployment scripts)
```

**Implementation:**

```bash
# 1. Create EC2 role
aws iam create-role --role-name Rails-App-EC2-Role \
  --assume-role-policy-document file://ec2-trust-policy.json

aws iam attach-role-policy --role-name Rails-App-EC2-Role \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy

aws iam put-role-policy --role-name Rails-App-EC2-Role \
  --policy-name S3-Uploads-Access \
  --policy-document file://s3-uploads-policy.json

# 2. Create developer group
aws iam create-group --group-name Rails-Developers
aws iam attach-group-policy --group-name Rails-Developers \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess
```

## Conclusion

AWS IAM is not just about creating users and assigning passwords—it's the foundational security layer that protects your entire cloud infrastructure. By mastering IAM users, groups, roles, and policies, you gain precise control over who can do what in your AWS environment.

The shift from hardcoded credentials to IAM roles, from overly permissive `AdministratorAccess` to least-privilege custom policies, and from password-only auth to MFA-enforced access transforms your AWS account from a security liability into a hardened fortress.

Start with the root account: lock it down, enable MFA, and never touch it again. Build IAM users and groups for your team. Create roles for your applications. Audit with Access Analyzer and CloudTrail. Iterate toward least privilege.

IAM is complex, but every hour invested in understanding it saves weeks of incident response down the road. Master IAM, and you master AWS security.

## Suggested Reading

- [AWS IAM Official Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/)
- [IAM Best Practices - AWS Security](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [AWS Security Token Service (STS) Documentation](https://docs.aws.amazon.com/STS/latest/APIReference/)
- [IAM Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html)
- [AWS Organizations and SCPs](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)
- [IAM Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)

{% include inarticle-adsense.html %}
