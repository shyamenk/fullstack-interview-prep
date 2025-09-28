# System Design Fundamentals

## 🎯 Core Concepts

### Scalability Principles
- [ ] **Horizontal vs Vertical Scaling**
  - Scale out vs scale up
  - Load distribution
  - Cost implications
- [ ] **Load Balancing**
  - Round robin, least connections, weighted
  - Layer 4 vs Layer 7 load balancing
  - Health checks and failover
- [ ] **Auto Scaling**
  - Metrics-based scaling
  - Predictive scaling
  - Scaling policies

### Reliability & Availability
- [ ] **High Availability Patterns**
  - Redundancy and failover
  - Active-active vs active-passive
  - Circuit breaker pattern
- [ ] **Fault Tolerance**
  - Graceful degradation
  - Bulkhead pattern
  - Timeout and retry mechanisms
- [ ] **Disaster Recovery**
  - RTO (Recovery Time Objective)
  - RPO (Recovery Point Objective)
  - Backup strategies

### Performance Optimization
- [ ] **Caching Strategies**
  - Client-side caching
  - CDN caching
  - Application-level caching
  - Database caching
- [ ] **Database Optimization**
  - Indexing strategies
  - Query optimization
  - Connection pooling
  - Read replicas

### Consistency & CAP Theorem
- [ ] **CAP Theorem**
  - Consistency, Availability, Partition tolerance
  - Trade-offs in distributed systems
  - Eventually consistent systems
- [ ] **ACID vs BASE**
  - ACID properties in RDBMS
  - BASE properties in NoSQL
  - Eventual consistency patterns

## 🏗️ Architecture Patterns

### Monolithic Architecture
```
┌─────────────────────────────────┐
│         Monolithic App          │
│  ┌─────┐ ┌─────┐ ┌─────┐       │
│  │ UI  │ │API  │ │ DB  │       │
│  └─────┘ └─────┘ └─────┘       │
└─────────────────────────────────┘
```
**Pros**: Simple deployment, easy debugging
**Cons**: Technology lock-in, scaling challenges

### Microservices Architecture
```
┌─────────┐    ┌─────────┐    ┌─────────┐
│Service A│    │Service B│    │Service C│
│   DB A  │    │   DB B  │    │   DB C  │
└─────────┘    └─────────┘    └─────────┘
     │              │              │
     └──────────────┼──────────────┘
                    │
            ┌───────────────┐
            │ API Gateway   │
            └───────────────┘
```
**Pros**: Independent scaling, technology diversity
**Cons**: Complexity, network latency, data consistency

### Serverless Architecture
```
┌─────────┐    ┌─────────┐    ┌─────────┐
│Function │    │Function │    │Function │
│    A    │    │    B    │    │    C    │
└─────────┘    └─────────┘    └─────────┘
     │              │              │
     └──────────────┼──────────────┘
                    │
            ┌───────────────┐
            │ Event Source  │
            └───────────────┘
```
**Pros**: No server management, automatic scaling
**Cons**: Cold starts, vendor lock-in, debugging complexity

## 🗄️ Database Design Patterns

### SQL vs NoSQL Decision Matrix
| Factor | SQL | NoSQL |
|--------|-----|-------|
| Schema | Fixed | Flexible |
| Scaling | Vertical | Horizontal |
| ACID | Strong | Eventual |
| Query | SQL | Various |
| Use Case | Complex queries | Simple lookups |

### Database Scaling Strategies
- [ ] **Read Replicas**
  - Master-slave replication
  - Read/write splitting
  - Lag considerations
- [ ] **Sharding**
  - Horizontal partitioning
  - Shard key selection
  - Cross-shard queries
- [ ] **Federation**
  - Split databases by function
  - Reduce read/write traffic
  - More complex queries

### Caching Layers
```
Client → CDN → Load Balancer → App Server → Cache → Database
```

#### Cache Patterns
- [ ] **Cache-Aside (Lazy Loading)**
  - Application manages cache
  - Cache miss handled by app
  - Good for read-heavy workloads
- [ ] **Write-Through**
  - Write to cache and database
  - Ensures data consistency
  - Higher write latency
- [ ] **Write-Behind (Write-Back)**
  - Write to cache first
  - Async write to database
  - Risk of data loss

## 🌐 Networking & Communication

### API Design Patterns
- [ ] **REST APIs**
  - HTTP methods and status codes
  - Resource-based URLs
  - Stateless communication
- [ ] **GraphQL**
  - Single endpoint
  - Client-specified queries
  - Real-time subscriptions
- [ ] **gRPC**
  - Protocol buffer serialization
  - HTTP/2 transport
  - Code generation

### Message Queues & Event Streaming
- [ ] **Message Queues (SQS, RabbitMQ)**
  - Point-to-point communication
  - Decoupling services
  - Reliability and durability
- [ ] **Pub/Sub (SNS, Kafka)**
  - One-to-many communication
  - Event-driven architecture
  - Real-time processing

### Load Balancing Strategies
- [ ] **Layer 4 (Transport)**
  - IP and port-based routing
  - Lower latency
  - Protocol agnostic
- [ ] **Layer 7 (Application)**
  - Content-based routing
  - SSL termination
  - Advanced health checks

## 🔒 Security Considerations

### Authentication & Authorization
- [ ] **Authentication Methods**
  - Session-based authentication
  - Token-based (JWT)
  - OAuth 2.0 / OpenID Connect
- [ ] **Authorization Patterns**
  - Role-based access control (RBAC)
  - Attribute-based access control (ABAC)
  - Permission-based systems

### Data Security
- [ ] **Encryption**
  - At rest encryption
  - In transit encryption (TLS)
  - Key management
- [ ] **Data Privacy**
  - PII handling
  - GDPR compliance
  - Data anonymization

## 📊 Monitoring & Observability

### The Three Pillars
- [ ] **Metrics**
  - System metrics (CPU, memory, disk)
  - Application metrics (response time, error rate)
  - Business metrics (user engagement)
- [ ] **Logs**
  - Structured logging
  - Centralized log aggregation
  - Log levels and filtering
- [ ] **Traces**
  - Distributed tracing
  - Request flow visualization
  - Performance bottleneck identification

### Key Performance Indicators (KPIs)
- [ ] **Availability** - Uptime percentage (99.9%)
- [ ] **Latency** - Response time (p50, p95, p99)
- [ ] **Throughput** - Requests per second
- [ ] **Error Rate** - Percentage of failed requests

## 🚀 Practical System Design Examples

### URL Shortener (like bit.ly)
```
Requirements:
- Shorten long URLs
- Redirect to original URL
- Custom aliases
- Analytics

Components:
- Web servers (load balanced)
- Application servers
- URL encoding service
- Database (URL mappings)
- Cache (popular URLs)
- Analytics service
```

### Chat Application (like WhatsApp)
```
Requirements:
- Send/receive messages
- Online presence
- Group chats
- Message history

Components:
- WebSocket servers
- Message queue
- User service
- Message storage
- Push notification service
- Media storage
```

### Social Media Feed (like Twitter)
```
Requirements:
- Post tweets
- Follow users
- Timeline generation
- Real-time updates

Components:
- User service
- Tweet service
- Timeline service
- Media service
- Notification service
- Search service
```

## 💡 Design Process & Best Practices

### System Design Interview Process
1. **Clarify Requirements** (5-10 minutes)
   - Functional requirements
   - Non-functional requirements
   - Scale estimates
2. **High-Level Design** (10-15 minutes)
   - Major components
   - Data flow
   - API design
3. **Detailed Design** (15-20 minutes)
   - Database schema
   - Algorithm specifics
   - Deep dive into components
4. **Scale the Design** (10-15 minutes)
   - Identify bottlenecks
   - Scaling strategies
   - Trade-off discussions

### Estimation Guidelines
- [ ] **Users**: Daily active users, peak concurrent users
- [ ] **Storage**: Data size per user, growth rate
- [ ] **Bandwidth**: Read/write requests per second
- [ ] **Memory**: Cache requirements, working set size

### Common Bottlenecks & Solutions
| Bottleneck | Solution |
|------------|----------|
| CPU | Load balancing, horizontal scaling |
| Memory | Caching, vertical scaling |
| Disk I/O | SSDs, database optimization |
| Network | CDN, data compression |
| Database | Indexing, sharding, replication |

## 📚 Study Resources

### Books
- "Designing Data-Intensive Applications" by Martin Kleppmann
- "System Design Interview" by Alex Xu
- "Building Microservices" by Sam Newman
- "High Performance MySQL" by Baron Schwartz

### Online Resources
- [High Scalability](http://highscalability.com/) - Real system architectures
- [AWS Architecture Center](https://aws.amazon.com/architecture/) - Cloud patterns
- [Google SRE Book](https://sre.google/sre-book/table-of-contents/) - Reliability engineering

### YouTube Channels
- Gaurav Sen - System design concepts
- Tech Dummies - Interview-focused content
- InfoQ - Conference talks on architecture

## ✅ Progress Tracker

### Fundamental Concepts
- [ ] Scalability principles understood
- [ ] CAP theorem mastered
- [ ] Caching strategies learned
- [ ] Load balancing concepts practiced

### Architecture Patterns
- [ ] Monolithic vs microservices compared
- [ ] Serverless patterns understood
- [ ] Event-driven architecture designed
- [ ] Database patterns practiced

### Communication Protocols
- [ ] REST API design mastered
- [ ] Message queue patterns understood
- [ ] Real-time communication implemented
- [ ] API versioning strategies learned

### System Design Practice
- [ ] 5+ system design problems solved
- [ ] Requirement gathering practiced
- [ ] Trade-off analysis performed
- [ ] Scaling strategies applied

### Real-World Application
- [ ] Architecture decisions documented
- [ ] Performance bottlenecks identified
- [ ] Monitoring strategies implemented
- [ ] Security considerations applied

**Target**: Master system design fundamentals for confident architecture discussions in senior-level interviews