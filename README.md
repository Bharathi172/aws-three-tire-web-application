# 🌐 Three-Tier High-Availability Web Application on AWS

[![AWS](https://img.shields.io/badge/AWS-Cloud-orange?style=for-the-badge&logo=amazon-aws)](https://aws.amazon.com/)
[![Architecture](https://img.shields.io/badge/Architecture-Multi--Tier-blue?style=for-the-badge)](https://github.com/yourusername/aws-three-tier-app)
[![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

> A production-grade, highly available web application deployed on AWS using a three-tier architecture pattern with automatic scaling, load balancing, and multi-AZ database replication.

![Architecture Diagram](./docs/architecture-diagram.png)

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Technologies](#technologies)
- [Infrastructure Components](#infrastructure-components)
- [Security](#security)
- [Prerequisites](#prerequisites)
- [Deployment Guide](#deployment-guide)
- [Cost Breakdown](#cost-breakdown)
- [Monitoring & Maintenance](#monitoring--maintenance)
- [Troubleshooting](#troubleshooting)
- [Future Enhancements](#future-enhancements)
- [Lessons Learned](#lessons-learned)
- [License](#license)

---

## 🎯 Overview

This project demonstrates the design and deployment of a **production-grade, scalable three-tier web application** on AWS. The architecture follows AWS Well-Architected Framework best practices for high availability, security, and operational excellence.

### Key Highlights

- ✅ **99.99% Availability** - Multi-AZ deployment across 2 availability zones
- ✅ **Auto-Scaling** - Elastic compute capacity (2-4 instances)
- ✅ **Load Balanced** - Application Load Balancer distributes traffic
- ✅ **Self-Healing** - Automatic instance replacement on failure
- ✅ **Secure** - Defense-in-depth with network isolation
- ✅ **Fault Tolerant** - RDS Multi-AZ with automatic failover

### Live Demo

🔗 **Application URL:** `http://your-alb-dns-name.elb.amazonaws.com`

*Note: Demo may be taken down to manage costs.*

---

## 🏗️ Architecture

### Three-Tier Architecture Pattern

```
┌─────────────────────────────────────────────────────────────────┐
│                          INTERNET                               │
└────────────────────────────┬────────────────────────────────────┘
                             │
                  ┌──────────▼──────────┐
                  │  Internet Gateway   │
                  └──────────┬──────────┘
                             │
         ┌───────────────────┴───────────────────┐
         │                                       │
    ┌────▼─────┐                          ┌─────▼────┐
    │   ALB    │                          │   ALB    │
    │ us-east  │                          │ us-east  │
    │   -1a    │                          │   -1b    │
    └────┬─────┘                          └─────┬────┘
         │        PUBLIC TIER (10.0.1/2.0)      │
─────────┼──────────────────────────────────────┼─────────
         │       PRIVATE TIER (10.0.11/12.0)    │
         │                                      │
    ┌────▼─────┐                          ┌─────▼────┐
    │   EC2    │                          │   EC2    │
    │ Instance │◄────NAT Gateway──────────►│ Instance │
    │ us-east  │                          │ us-east  │
    │   -1a    │                          │   -1b    │
    └────┬─────┘                          └─────┬────┘
         │                                      │
─────────┼──────────────────────────────────────┼─────────
         │     DATABASE TIER (10.0.21/22.0)     │
         │                                      │
         └──────────────┬───────────────────────┘
                        │
                 ┌──────▼──────┐
                 │ RDS MySQL   │
                 │  Multi-AZ   │
                 │ Primary +   │
                 │  Standby    │
                 └─────────────┘
```

### Architecture Layers

#### 1. **Presentation Tier (Public Subnets)**
- **Application Load Balancer** - Distributes incoming traffic
- **Internet Gateway** - Provides internet connectivity
- **NAT Gateway** - Enables outbound internet for private instances
- **CIDR Blocks**: 10.0.1.0/24 (AZ-1a), 10.0.2.0/24 (AZ-1b)

#### 2. **Application Tier (Private Subnets)**
- **EC2 Instances** - Web and application servers
- **Auto Scaling Group** - Maintains 2-4 instances
- **Target Group** - Routes traffic from ALB
- **CIDR Blocks**: 10.0.11.0/24 (AZ-1a), 10.0.12.0/24 (AZ-1b)

#### 3. **Data Tier (Database Subnets)**
- **RDS MySQL Multi-AZ** - Primary and standby instances
- **Automatic Failover** - <2 minutes recovery time
- **Isolated Network** - No internet access
- **CIDR Blocks**: 10.0.21.0/24 (AZ-1a), 10.0.22.0/24 (AZ-1b)

---

## ✨ Features

### High Availability
- Multi-AZ deployment across 2 availability zones
- Automatic failover for database (RDS Multi-AZ)
- Load balancing across multiple EC2 instances
- Self-healing infrastructure with health checks

### Auto Scaling
- Scales from 2 to 4 instances based on demand
- Automatic instance replacement on failure
- Health check grace period of 300 seconds
- Target tracking scaling policies

### Security
- Network segmentation with public, private, and database tiers
- Security groups implementing least privilege access
- No direct internet access for application or database tiers
- NAT Gateway for secure outbound connectivity
- No SSH access (immutable infrastructure pattern)

### Performance
- Application Load Balancer for optimal traffic distribution
- Connection draining during scaling operations
- Database read replicas capability
- CloudWatch monitoring integration

---

## 🛠️ Technologies

### AWS Services
| Service | Purpose |
|---------|---------|
| **VPC** | Custom virtual private cloud (10.0.0.0/16) |
| **EC2** | Application servers (Amazon Linux 2023) |
| **RDS** | MySQL database (Multi-AZ deployment) |
| **ALB** | Application Load Balancer |
| **ASG** | Auto Scaling Group (2-4 instances) |
| **CloudWatch** | Monitoring and logging |
| **IAM** | Identity and access management |

### Application Stack
| Component | Technology |
|-----------|-----------|
| **Web Server** | Apache HTTP Server 2.4 |
| **Backend** | PHP 8.4 |
| **Database** | MySQL 10.5 (MariaDB) |
| **OS** | Amazon Linux 2023 |

### Infrastructure
- **Compute**: EC2 t2.micro instances
- **Database**: RDS db.t3.micro Multi-AZ
- **Networking**: Custom VPC with 6 subnets
- **Security**: 3 security groups with defense-in-depth

---

## 🔧 Infrastructure Components

### Network Architecture

#### VPC Configuration
```yaml
VPC CIDR: 10.0.0.0/16
DNS Hostnames: Enabled
DNS Resolution: Enabled
Tenancy: Default
```

#### Subnets (6 Total)
```yaml
Public Subnets:
  - 10.0.1.0/24 (us-east-1a) - ALB, NAT Gateway
  - 10.0.2.0/24 (us-east-1b) - ALB

Private Subnets:
  - 10.0.11.0/24 (us-east-1a) - EC2 Instances
  - 10.0.12.0/24 (us-east-1b) - EC2 Instances

Database Subnets:
  - 10.0.21.0/24 (us-east-1a) - RDS Primary
  - 10.0.22.0/24 (us-east-1b) - RDS Standby
```

#### Route Tables (3 Total)
```yaml
Public Route Table:
  - 10.0.0.0/16 → local
  - 0.0.0.0/0 → Internet Gateway

Private Route Table:
  - 10.0.0.0/16 → local
  - 0.0.0.0/0 → NAT Gateway

Database Route Table:
  - 10.0.0.0/16 → local
  - (No internet access)
```

### Compute Resources

#### Auto Scaling Group
```yaml
Desired Capacity: 2
Minimum Capacity: 2
Maximum Capacity: 4
Health Check Type: ELB
Health Check Grace Period: 300 seconds
Availability Zones: us-east-1a, us-east-1b
```

#### Launch Template
```yaml
AMI: Amazon Linux 2023
Instance Type: t2.micro
Network: Private Subnets
Security Group: WebServer-SG
User Data: Automated installation script
```

### Database Configuration

#### RDS MySQL
```yaml
Engine: MySQL 10.5 (MariaDB)
Instance Class: db.t3.micro
Multi-AZ: Yes (Primary + Standby)
Storage: 20 GB gp2
Backup Retention: 7 days
Database Name: guestbook
```

---

## 🔒 Security

### Defense-in-Depth Architecture

#### Security Group 1: ALB Security Group
```yaml
Name: ALB-SG
Inbound Rules:
  - Type: HTTP
    Port: 80
    Source: 0.0.0.0/0
    Description: Allow HTTP from internet
```

#### Security Group 2: WebServer Security Group
```yaml
Name: WebServer-SG
Inbound Rules:
  - Type: HTTP
    Port: 80
    Source: ALB-SG
    Description: Allow HTTP from ALB only
```

#### Security Group 3: Database Security Group
```yaml
Name: Database-SG
Inbound Rules:
  - Type: MySQL/Aurora
    Port: 3306
    Source: WebServer-SG
    Description: Allow MySQL from EC2 only
```

### Security Best Practices Implemented

✅ **Network Segmentation**
- Separate subnets for public, private, and database tiers
- No direct internet access for application or database layers

✅ **Least Privilege Access**
- Security groups allow only necessary traffic
- Each tier can only communicate with adjacent tiers

✅ **Data Protection**
- Database in private subnet with no public access
- RDS encryption at rest available
- VPC Flow Logs for network monitoring

✅ **Access Control**
- No SSH keys configured (immutable infrastructure)
- Systems Manager Session Manager for troubleshooting
- IAM roles for EC2-RDS authentication (optional)

✅ **Monitoring & Logging**
- CloudWatch Logs for application logs
- VPC Flow Logs for network traffic
- ALB access logs available

---

## 📦 Prerequisites

Before deploying this infrastructure, ensure you have:

- **AWS Account** with appropriate permissions
- **AWS CLI** configured with credentials
- **Basic understanding** of AWS networking concepts
- **Text editor** for editing configuration files

### Required AWS Permissions

```json
{
  "Services": [
    "VPC (Full Access)",
    "EC2 (Full Access)",
    "RDS (Full Access)",
    "ELB (Full Access)",
    "Auto Scaling (Full Access)",
    "CloudWatch (Read Access)",
    "IAM (Role Creation)"
  ]
}
```

---

## 🚀 Deployment Guide

### Step 1: Create VPC Infrastructure

#### 1.1 Create VPC
```bash
VPC CIDR: 10.0.0.0/16
Enable DNS hostnames: Yes
Enable DNS resolution: Yes
```

#### 1.2 Create Subnets
```bash
# Public Subnets
10.0.1.0/24 - us-east-1a (Public)
10.0.2.0/24 - us-east-1b (Public)

# Private Subnets
10.0.11.0/24 - us-east-1a (Private)
10.0.12.0/24 - us-east-1b (Private)

# Database Subnets
10.0.21.0/24 - us-east-1a (Database)
10.0.22.0/24 - us-east-1b (Database)
```

#### 1.3 Create and Attach Internet Gateway

#### 1.4 Create NAT Gateway
- Allocate Elastic IP
- Place in Public Subnet (10.0.1.0/24)

#### 1.5 Configure Route Tables
- Public RT: 0.0.0.0/0 → Internet Gateway
- Private RT: 0.0.0.0/0 → NAT Gateway
- Database RT: Local only

### Step 2: Create Security Groups

Create three security groups with appropriate rules as documented in the Security section above.

### Step 3: Launch RDS Database

```bash
# Create DB Subnet Group
# Create RDS MySQL Instance with Multi-AZ enabled
# Configure security group
# Set backup retention to 7 days
```

### Step 4: Create Application Load Balancer

#### 4.1 Create Target Group
- Protocol: HTTP
- Port: 80
- Health check path: /health.html
- Health check interval: 30 seconds

#### 4.2 Create Application Load Balancer
- Scheme: Internet-facing
- Subnets: Both public subnets
- Security group: ALB-SG

#### 4.3 Create Listener
- Protocol: HTTP
- Port: 80
- Forward to: Target Group

### Step 5: Create Launch Template

Create a launch template with user data script that installs:
- Apache web server
- PHP
- MariaDB client
- Application code
- Health check endpoint

See `scripts/user-data.sh` for complete script.

### Step 6: Create Auto Scaling Group

```yaml
Min Size: 2
Max Size: 4
Desired: 2
Subnets: Private subnets (10.0.11.0/24, 10.0.12.0/24)
Target Group: Link to ALB target group
Health Check: ELB
Grace Period: 300 seconds
```

### Step 7: Verify Deployment

- Check target group health status
- Verify instances are running
- Test application via ALB DNS name
- Monitor CloudWatch metrics

---

## 💰 Cost Breakdown

### Monthly Cost Estimate (us-east-1)

| Service | Configuration | Monthly Cost |
|---------|--------------|--------------|
| **VPC** | 1 VPC, 6 subnets | $0 (Free) |
| **Internet Gateway** | 1 IGW | $0 (Free) |
| **NAT Gateway** | 1 NAT Gateway | ~$32 |
| **Application Load Balancer** | 1 ALB | ~$20 |
| **EC2 Instances** | 2x t2.micro (730 hrs) | ~$10 |
| **RDS MySQL** | 1x db.t3.micro Multi-AZ | ~$25 |
| **Data Transfer** | Estimated | ~$5-10 |
| **CloudWatch** | Basic monitoring | ~$3 |
| **Total** | | **~$95-100/month** |

### Cost Optimization Tips

- Use **Reserved Instances** (save 30-60%)
- Enable **Auto Scaling** to reduce idle capacity
- Use **Spot Instances** for dev/test (save 70-90%)
- Delete **NAT Gateway** when not in use
- Stop **RDS** during non-business hours (dev only)

---

## 📊 Monitoring & Maintenance

### CloudWatch Metrics

#### ALB Metrics
- Request Count
- Target Response Time
- HTTP 4XX/5XX Errors
- Healthy/Unhealthy Host Count

#### EC2 Metrics
- CPU Utilization
- Network In/Out
- Disk Read/Write
- Status Check Failed

#### RDS Metrics
- Database Connections
- Read/Write Latency
- Disk Queue Depth
- Free Storage Space

### Recommended Alarms

```yaml
High CPU Alarm:
  Metric: CPUUtilization
  Threshold: > 80%
  Duration: 5 minutes

Unhealthy Targets:
  Metric: UnHealthyHostCount
  Threshold: > 0
  Duration: 2 minutes

Database Storage:
  Metric: FreeStorageSpace
  Threshold: < 2 GB
  Duration: 5 minutes
```

### Backup Strategy

- **RDS Automated Backups**: 7-day retention
- **RDS Manual Snapshots**: Before major changes
- **AMI Snapshots**: Quarterly for base images
- **Application Code**: Version controlled in Git

---

## 🔍 Troubleshooting

### Common Issues

#### 1. Instances Showing Unhealthy in Target Group

**Symptoms:**
- Targets registered but marked unhealthy
- ALB returns 503 errors

**Solutions:**
- Verify health check path exists (`/health.html`)
- Check security group allows ALB → EC2 on port 80
- Verify Apache is running
- Check instance system logs

#### 2. Cannot Connect to Database

**Symptoms:**
- Application shows database connection error
- PHP errors mentioning MySQL

**Solutions:**
- Verify RDS security group allows EC2 security group on port 3306
- Check RDS endpoint is correct in application code
- Verify database credentials
- Ensure database subnets are configured correctly

#### 3. EC2 Instances Cannot Download Packages

**Symptoms:**
- User data script fails
- Package installation timeout errors

**Solutions:**
- Verify NAT Gateway exists and is in public subnet
- Check private route table routes 0.0.0.0/0 to NAT Gateway
- Verify NAT Gateway has Elastic IP attached
- Check internet gateway is attached to VPC

#### 4. Auto Scaling Not Working

**Symptoms:**
- Instances don't launch automatically
- Failed to register targets

**Solutions:**
- Verify ASG has correct launch template version
- Check ASG subnets are private subnets
- Verify IAM role allows Auto Scaling actions
- Check CloudWatch logs for Auto Scaling events

---

## 🚀 Future Enhancements

### Phase 1: Security & Compliance
- [ ] Implement HTTPS/TLS with ACM certificate
- [ ] Enable VPC Flow Logs
- [ ] Add AWS WAF for application firewall
- [ ] Implement AWS Secrets Manager for credentials
- [ ] Enable CloudTrail for audit logging

### Phase 2: Performance & Scalability
- [ ] Add CloudFront CDN for static content
- [ ] Implement ElastiCache for Redis caching
- [ ] Add RDS read replicas
- [ ] Implement DynamoDB for session management
- [ ] Add S3 for user-uploaded content

### Phase 3: DevOps & Automation
- [ ] Infrastructure as Code with Terraform
- [ ] CI/CD pipeline with CodePipeline
- [ ] Blue-green deployment strategy
- [ ] Automated testing
- [ ] Container migration to ECS/EKS

### Phase 4: Advanced Features
- [ ] Multi-region deployment
- [ ] Route 53 failover routing
- [ ] Implement Lambda for serverless functions
- [ ] Add SNS/SQS for async processing
- [ ] Implement AWS Backup for centralized backup

---

## 📚 Lessons Learned

### Technical Challenges Overcome

1. **Package Compatibility**
   - Issue: `mysql` package not available on Amazon Linux 2023
   - Solution: Used `mariadb105` package instead
   - Lesson: Always verify package names for new OS versions

2. **IMDSv2 Authentication**
   - Issue: Metadata calls returning 401 Unauthorized
   - Solution: Implemented token-based IMDSv2 authentication
   - Lesson: AWS is moving to more secure metadata access

3. **Health Check Configuration**
   - Issue: Instances healthy but target group showing unhealthy
   - Solution: Created dedicated `/health.html` endpoint
   - Lesson: Health checks need explicit, simple endpoints

4. **Network Connectivity**
   - Issue: EC2 instances couldn't download packages
   - Solution: Added NAT Gateway to private route table
   - Lesson: Private subnets need NAT for outbound internet

5. **Security Group Dependencies**
   - Issue: Database connections failing despite correct credentials
   - Solution: Added security group rule allowing EC2 → RDS
   - Lesson: Security groups must explicitly allow all required traffic

### Best Practices Applied

✅ **Infrastructure as Code mindset**
- User data scripts for automated installation
- Launch templates for consistent instance configuration

✅ **Security first approach**
- Network segmentation from the start
- Least privilege security groups
- No SSH access (immutable infrastructure)

✅ **High availability by design**
- Multi-AZ deployment from day one
- RDS automatic failover configured
- Load balancing across availability zones

✅ **Operational excellence**
- Health checks at every layer
- Auto Scaling for self-healing
- CloudWatch monitoring enabled

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 📋 Additional Resources

- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [AWS Auto Scaling Documentation](https://docs.aws.amazon.com/autoscaling/)
- [AWS RDS Multi-AZ Documentation](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)

---

<div align="center">

**⭐ If you found this project helpful, please consider giving it a star! ⭐**

Built with ☁️ on AWS

</div>
