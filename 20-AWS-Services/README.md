# AWS Services & Architecture Mastery

## Overview

A comprehensive guide to Amazon Web Services, focusing on core infrastructure, architecture patterns, networking, security, and cost optimization. Designed for Full Stack Developers and System Architects.

## Study Checklist

### Fundamentals & Architecture
- [ ] **Global Infrastructure:** Regions, Availability Zones (AZs), Edge Locations
- [ ] **Models:** IaaS vs PaaS vs SaaS
- [ ] **Well-Architected Framework:** 6 Pillars
- [ ] **Hybrid & Multi-Cloud:** Direct Connect, Storage Gateway, VPN

### Compute & Networking
- [ ] **EC2:** Instance families, Pricing models (Spot, RI, Savings Plans)
- [ ] **VPC:** Subnets, Route Tables, IGW, NAT Gateway, NACLs vs SGs
- [ ] **Load Balancing:** ALB (Layer 7) vs NLB (Layer 4)
- [ ] **Connectivity:** VPC Peering vs Transit Gateway

### Storage & Databases
- [ ] **S3:** Storage Classes (Standard, IA, Glacier), Lifecycle Policies, Consistency
- [ ] **Block/File:** EBS vs Instance Store vs EFS
- [ ] **Databases:** RDS (SQL) vs DynamoDB (NoSQL) vs Aurora

### Security & Identity
- [ ] **IAM:** Users, Groups, Roles, Policies
- [ ] **Data Protection:** KMS (Encryption), WAF (Firewall), Shield (DDoS)
- [ ] **Access Control:** Bucket Policies vs IAM Policies

### Integration & Management
- [ ] **Messaging:** SQS (Queue) vs SNS (Pub/Sub) vs EventBridge (Bus)
- [ ] **Monitoring:** CloudWatch (Metrics/Logs) vs CloudTrail (Auditing)
- [ ] **Governance:** AWS Config, Trusted Advisor

## Quick Reference

| Service | Category | Use Case |
|---------|----------|----------|
| **EC2** | Compute | Virtual servers in the cloud |
| **S3** | Storage | Object storage for files, backups, static web |
| **VPC** | Networking | Virtual Private Cloud (Your private network) |
| **IAM** | Security | Identity and Access Management |
| **RDS** | Database | Managed Relational Database (MySQL, Postgres) |
| **DynamoDB** | Database | Managed NoSQL Key-Value store |
| **Lambda** | Compute | Serverless functions |
| **CloudFront**| CDN | Global Content Delivery Network |
| **Route53** | Networking | DNS and Traffic Management |
| **SNS** | Integration | Pub/Sub messaging |
| **SQS** | Integration | Message Queuing |

## Resources

- [Questions & Answers](./questions-and-answers.md)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)

## Progress Tracker

- [ ] Design a Multi-AZ, High Availability architecture
- [ ] Configure a VPC with Public/Private subnets and NAT Gateway
- [ ] Secure an S3 bucket with Bucket Policies
- [ ] Implement an Auto Scaling Group behind an ALB
- [ ] Calculate costs for a proposed architecture
