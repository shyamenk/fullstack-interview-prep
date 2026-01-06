# AWS DevOps & CI/CD Mastery

## Overview

Focuses on automating software delivery, infrastructure management, and monitoring using AWS Developer Tools and practices. Essential for "You Build It, You Run It" culture.

## Study Checklist

### CI/CD Pipelines
- [ ] **CodeCommit:** Git repositories
- [ ] **CodeBuild:** Build specifications (buildspec.yml)
- [ ] **CodeDeploy:** Deployment strategies (In-place vs Blue/Green)
- [ ] **CodePipeline:** Orchestrating the full release process

### Infrastructure as Code (IaC)
- [ ] **CloudFormation:** Templates, Stacks, Drift Detection
- [ ] **AWS CDK:** Defining infra in TypeScript/Python
- [ ] **Systems Manager (SSM):** Parameter Store, Patch Manager

### Monitoring & Logging
- [ ] **CloudWatch:** Alarms, Logs Insights, Dashboards
- [ ] **EventBridge:** Event-driven automation
- [ ] **X-Ray:** Distributed tracing

### Deployment Strategies
- [ ] **Blue/Green:** Zero downtime, instant rollback
- [ ] **Canary:** Risk mitigation, gradual traffic shift
- [ ] **Rolling:** Update batch by batch

## Quick Reference

| Service | Category | Function |
|---------|----------|----------|
| **CodePipeline** | CICD | Orchestrates build, test, deploy workflow |
| **CodeBuild** | Build | Compiles code, runs tests, produces artifacts |
| **CodeDeploy** | Deploy | Automates deployment to EC2/Lambda/ECS |
| **CloudFormation** | IaC | Declarative infrastructure management (JSON/YAML) |
| **CDK** | IaC | Imperative infrastructure management (Code) |
| **SSM** | Mgmt | Parameter Store, Patching, Remote Execution |
| **CloudWatch** | Monitor | Metrics, Logs, Alarms |

## Resources

- [Questions & Answers](./questions-and-answers.md)
- [AWS DevOps Blog](https://aws.amazon.com/blogs/devops/)

## Progress Tracker

- [ ] Create a 3-stage Pipeline (Source -> Build -> Deploy)
- [ ] Write a CloudFormation template to launch an EC2 instance
- [ ] Implement a Blue/Green deployment for Lambda
- [ ] Configure a CloudWatch Alarm to trigger an SNS email
- [ ] Store configuration in SSM Parameter Store
