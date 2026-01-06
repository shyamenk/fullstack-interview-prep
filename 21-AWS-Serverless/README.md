# AWS Serverless Mastery

## Overview

Mastering the Serverless paradigm on AWS. Focuses on building event-driven, scalable, and cost-effective applications using Lambda, API Gateway, and managed services without provisioning servers.

## Study Checklist

### Compute & API
- [ ] **Lambda:** Lifecycle, Cold Starts, Concurrency, Layers, Versions/Aliases
- [ ] **API Gateway:** REST vs HTTP APIs, Integrations (Proxy), Throttling, Auth (Cognito/Lambda Authorizers)
- [ ] **Step Functions:** State Machines, Orchestration patterns, Error handling

### Database & Storage
- [ ] **DynamoDB:** Single Table Design, Global Tables, Streams, Capacity Modes
- [ ] **S3:** Event Notifications (triggering Lambda)

### Event-Driven Architecture
- [ ] **EventBridge:** Rules, Buses, Schema Registry, Scheduler
- [ ] **SQS/SNS:** Fan-out pattern, DLQs, Decoupling
- [ ] **Kinesis:** Data Streams vs Firehose (Real-time processing)

### Developer Tools
- [ ] **SAM (Serverless Application Model):** Template.yaml, CLI commands
- [ ] **Monitoring:** CloudWatch Logs/Metrics, X-Ray Tracing
- [ ] **CI/CD:** CodePipeline for Serverless

## Quick Reference

| Service | Role | Key Concept |
|---------|------|-------------|
| **Lambda** | Compute | Run code without servers (15 min limit) |
| **API Gateway** | Front Door | Expose Lambda as HTTP endpoints |
| **DynamoDB** | Database | Serverless NoSQL (fast, scalable) |
| **Step Functions** | Orchestration | Coordinate multi-step workflows |
| **EventBridge** | Bus | Route events between services (Content-based filtering) |
| **SQS** | Queue | Buffer requests for async processing |
| **SAM** | IaC | Framework to define serverless apps |

## Resources

- [Questions & Answers](./questions-and-answers.md)
- [Serverless Land (AWS)](https://serverlessland.com/)

## Progress Tracker

- [ ] Build a REST API with API Gateway + Lambda + DynamoDB
- [ ] Implement an Async processing flow (S3 -> Lambda -> SQS)
- [ ] Orchestrate a workflow with Step Functions
- [ ] Deploy an app using SAM CLI
- [ ] Optimize Lambda cold starts and costs
