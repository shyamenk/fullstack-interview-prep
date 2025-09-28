# AWS Services for Fullstack Developers

## 🎯 Core AWS Services Overview

### Compute Services
- [ ] **Amazon EC2** - Virtual servers in the cloud
- [ ] **AWS Lambda** - Serverless compute functions
- [ ] **Amazon ECS** - Container orchestration service
- [ ] **AWS Fargate** - Serverless containers
- [ ] **Amazon EKS** - Managed Kubernetes service

### Storage Services
- [ ] **Amazon S3** - Object storage service
- [ ] **Amazon EBS** - Block storage for EC2
- [ ] **Amazon EFS** - Managed file system
- [ ] **AWS Storage Gateway** - Hybrid cloud storage

### Database Services
- [ ] **Amazon RDS** - Managed relational databases
- [ ] **Amazon DynamoDB** - NoSQL database
- [ ] **Amazon ElastiCache** - In-memory caching
- [ ] **Amazon Aurora** - Cloud-native relational database

### Networking & Content Delivery
- [ ] **Amazon VPC** - Virtual private cloud
- [ ] **Amazon CloudFront** - Content delivery network
- [ ] **Amazon Route 53** - DNS web service
- [ ] **Application Load Balancer** - Load balancing

### Security & Identity
- [ ] **AWS IAM** - Identity and access management
- [ ] **AWS KMS** - Key management service
- [ ] **AWS Certificate Manager** - SSL/TLS certificates
- [ ] **AWS Secrets Manager** - Secrets management

### Developer Tools
- [ ] **AWS CodeCommit** - Source code repository
- [ ] **AWS CodeBuild** - Build service
- [ ] **AWS CodeDeploy** - Deployment automation
- [ ] **AWS CodePipeline** - CI/CD pipeline

### Monitoring & Management
- [ ] **Amazon CloudWatch** - Monitoring and observability
- [ ] **AWS CloudTrail** - API logging and auditing
- [ ] **AWS X-Ray** - Distributed tracing
- [ ] **AWS Systems Manager** - Operational insights

### Application Integration
- [ ] **Amazon SQS** - Message queuing service
- [ ] **Amazon SNS** - Pub/sub messaging
- [ ] **Amazon EventBridge** - Event bus service
- [ ] **AWS Step Functions** - Workflow orchestration

## 💻 Common Fullstack Use Cases

### Web Application Hosting
```
Frontend: S3 + CloudFront
Backend: EC2/Fargate + ALB
Database: RDS/DynamoDB
Cache: ElastiCache
CDN: CloudFront
```

### Serverless Architecture
```
Frontend: S3 + CloudFront
API: API Gateway + Lambda
Database: DynamoDB
Authentication: Cognito
File Storage: S3
```

### Microservices Architecture
```
Container Orchestration: ECS/EKS
Service Discovery: AWS Cloud Map
Load Balancing: ALB/NLB
Messaging: SQS/SNS
Database: RDS + DynamoDB
```

### Real-time Applications
```
WebSocket API: API Gateway WebSocket
Real-time Processing: Lambda + Kinesis
Database: DynamoDB
Cache: ElastiCache
Notifications: SNS
```

## 🚀 Hands-on Practice Projects

### Beginner Level
- [ ] **Static Website Hosting**
  - S3 bucket configuration
  - CloudFront distribution
  - Route 53 domain setup
  - Certificate Manager SSL

- [ ] **Simple Lambda Function**
  - Basic Lambda function
  - API Gateway integration
  - CloudWatch monitoring
  - Environment variables

### Intermediate Level
- [ ] **Full-Stack Web App**
  - React frontend on S3/CloudFront
  - Node.js backend on EC2/Fargate
  - RDS PostgreSQL database
  - ElastiCache for caching

- [ ] **Serverless API**
  - Lambda functions with Node.js
  - DynamoDB for data storage
  - API Gateway for REST API
  - Cognito for authentication

### Advanced Level
- [ ] **Microservices Platform**
  - ECS/EKS container deployment
  - Service mesh implementation
  - Auto-scaling configuration
  - Multi-environment setup

- [ ] **Event-Driven Architecture**
  - SQS/SNS messaging
  - Lambda event processing
  - Step Functions workflows
  - EventBridge integration

## 📋 Service-Specific Deep Dives

### Amazon EC2
- [ ] **Instance Types** - Compute, memory, storage optimized
- [ ] **Auto Scaling** - Horizontal scaling configuration
- [ ] **Load Balancers** - ALB, NLB, CLB comparison
- [ ] **Security Groups** - Network-level security
- [ ] **Key Pairs** - SSH access management

### AWS Lambda
- [ ] **Function Configuration** - Runtime, memory, timeout
- [ ] **Event Sources** - Triggers and integrations
- [ ] **Environment Variables** - Configuration management
- [ ] **Layers** - Code sharing and dependencies
- [ ] **Cold Starts** - Performance optimization

### Amazon S3
- [ ] **Bucket Policies** - Access control
- [ ] **Storage Classes** - Cost optimization
- [ ] **Versioning** - Object version management
- [ ] **Cross-Region Replication** - Data durability
- [ ] **Static Website Hosting** - Frontend deployment

### Amazon DynamoDB
- [ ] **Partition Keys** - Data distribution
- [ ] **Global Secondary Indexes** - Query flexibility
- [ ] **Auto Scaling** - Capacity management
- [ ] **Streams** - Change data capture
- [ ] **Global Tables** - Multi-region replication

### Amazon RDS
- [ ] **Multi-AZ Deployment** - High availability
- [ ] **Read Replicas** - Read scaling
- [ ] **Automated Backups** - Data protection
- [ ] **Parameter Groups** - Database configuration
- [ ] **Performance Insights** - Query optimization

## 🔧 AWS CLI and SDK Usage

### AWS CLI Commands
```bash
# S3 operations
aws s3 ls
aws s3 cp file.txt s3://bucket-name/
aws s3 sync ./dist s3://bucket-name/

# Lambda operations
aws lambda create-function --function-name myFunction
aws lambda invoke --function-name myFunction response.json
aws lambda update-function-code --function-name myFunction

# DynamoDB operations
aws dynamodb create-table --table-name MyTable
aws dynamodb put-item --table-name MyTable --item '{}'
aws dynamodb scan --table-name MyTable
```

### Node.js SDK Examples
```javascript
// S3 file upload
const AWS = require('aws-sdk');
const s3 = new AWS.S3();

const uploadParams = {
  Bucket: 'my-bucket',
  Key: 'file.txt',
  Body: 'Hello World'
};

s3.upload(uploadParams).promise()
  .then(data => console.log('Upload successful'))
  .catch(err => console.error(err));

// DynamoDB operations
const dynamodb = new AWS.DynamoDB.DocumentClient();

const putParams = {
  TableName: 'Users',
  Item: {
    id: '123',
    name: 'John Doe',
    email: 'john@example.com'
  }
};

dynamodb.put(putParams).promise()
  .then(() => console.log('Item added'))
  .catch(err => console.error(err));
```

## 💰 Cost Optimization Strategies

### Compute Costs
- [ ] **Right-sizing** - Choose appropriate instance types
- [ ] **Reserved Instances** - Long-term capacity planning
- [ ] **Spot Instances** - Cost-effective for fault-tolerant workloads
- [ ] **Lambda optimization** - Memory and execution time tuning

### Storage Costs
- [ ] **S3 storage classes** - Lifecycle policies
- [ ] **EBS volume types** - Performance vs cost balance
- [ ] **Data transfer** - CloudFront and regional optimization

### Database Costs
- [ ] **RDS instance sizing** - CPU and memory optimization
- [ ] **DynamoDB capacity** - On-demand vs provisioned
- [ ] **Read replicas** - Geographic distribution strategy

## 🔒 Security Best Practices

### IAM (Identity and Access Management)
- [ ] **Principle of least privilege** - Minimal permissions
- [ ] **Role-based access** - Service roles vs user roles
- [ ] **MFA enforcement** - Multi-factor authentication
- [ ] **Access key rotation** - Regular credential updates

### Network Security
- [ ] **VPC configuration** - Private and public subnets
- [ ] **Security groups** - Application-level firewalls
- [ ] **NACLs** - Network-level access control
- [ ] **VPC endpoints** - Private service access

### Data Protection
- [ ] **Encryption at rest** - S3, RDS, DynamoDB encryption
- [ ] **Encryption in transit** - SSL/TLS certificates
- [ ] **Key management** - KMS key policies and rotation
- [ ] **Secrets management** - Secrets Manager integration

## 📊 Monitoring and Troubleshooting

### CloudWatch Monitoring
- [ ] **Metrics collection** - Default and custom metrics
- [ ] **Alarms configuration** - Threshold-based alerting
- [ ] **Dashboards creation** - Visual monitoring
- [ ] **Log aggregation** - Centralized logging

### Performance Optimization
- [ ] **Application performance** - X-Ray tracing
- [ ] **Database performance** - RDS Performance Insights
- [ ] **Lambda performance** - Duration and memory metrics
- [ ] **Network performance** - VPC flow logs

## 🎓 Certification Preparation

### AWS Certified Solutions Architect Associate
- [ ] Design resilient architectures
- [ ] Design high-performing architectures
- [ ] Design secure applications and architectures
- [ ] Design cost-optimized architectures

### AWS Certified Developer Associate
- [ ] Development with AWS services
- [ ] Security implementation
- [ ] Deployment strategies
- [ ] Refactoring and optimization

## 📚 Learning Resources

### Official AWS Resources
- [AWS Documentation](https://docs.aws.amazon.com/)
- [AWS Architecture Center](https://aws.amazon.com/architecture/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Free Tier](https://aws.amazon.com/free/)

### Hands-on Learning
- [AWS Hands-on Tutorials](https://aws.amazon.com/getting-started/hands-on/)
- [AWS Workshops](https://workshops.aws/)
- [AWS Samples on GitHub](https://github.com/aws-samples)

## ✅ Progress Tracker

### Core Services Mastery
- [ ] EC2 instance management and scaling
- [ ] Lambda function development and optimization
- [ ] S3 bucket configuration and security
- [ ] DynamoDB table design and operations
- [ ] RDS database setup and maintenance

### Architecture Patterns
- [ ] Serverless architecture implementation
- [ ] Microservices deployment
- [ ] Event-driven architecture design
- [ ] Static website hosting setup

### Security Implementation
- [ ] IAM roles and policies configuration
- [ ] VPC and network security setup
- [ ] Data encryption implementation
- [ ] Secrets management integration

### DevOps Integration
- [ ] CI/CD pipeline setup
- [ ] Infrastructure as Code (CloudFormation)
- [ ] Monitoring and alerting configuration
- [ ] Cost optimization strategies

### Practical Experience
- [ ] 3+ AWS projects deployed
- [ ] Multi-service integrations implemented
- [ ] Production-ready configurations used
- [ ] Cost and performance optimized

**Target**: Achieve proficiency in AWS services for fullstack development and cloud architecture discussions in interviews