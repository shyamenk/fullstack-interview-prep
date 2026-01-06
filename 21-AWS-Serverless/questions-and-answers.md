# AWS Serverless Interview Questions & Answers

## Table of Contents

1. [What is Serverless?](#q1-what-does-serverless-mean-in-aws)
2. [Lambda Cold Starts](#q2-what-is-a-lambda-cold-start-and-how-do-you-mitigate-it)
3. [Lambda Execution Limits](#q3-what-are-the-default-execution-limits-of-lambda)
4. [API Gateway Integration Types](#q4-what-are-the-integration-types-in-api-gateway)
5. [Lambda Layers](#q5-what-are-lambda-layers)
6. [DynamoDB On-Demand vs Provisioned](#q6-what-is-the-difference-between-provisioned-and-on-demand-capacity-in-dynamodb)
7. [Step Functions vs Lambda Chaining](#q7-why-use-step-functions-instead-of-chaining-lambdas)
8. [Event Source Mappings (SQS/Kinesis)](#q8-how-does-lambda-read-from-sqs-or-kinesis)
9. [SAM (Serverless Application Model)](#q9-what-is-aws-sam)
10. [API Gateway Throttling](#q10-how-does-api-gateway-handle-throttling)
11. [Lambda Concurrency (Reserved vs Provisioned)](#q11-explain-lambda-reserved-vs-provisioned-concurrency)
12. [Dead Letter Queues (DLQ) in Serverless](#q12-how-do-you-handle-failures-in-async-lambda-invocations)
13. [Lambda Destinations](#q13-what-are-lambda-destinations)
14. [Serverless vs Containers](#q14-when-should-you-choose-lambda-over-fargate)
15. [Global Tables (DynamoDB)](#q15-what-are-dynamodb-global-tables)
16. [X-Ray Tracing](#q16-how-do-you-debug-and-trace-serverless-applications)
17. [Cognito Authorizers](#q17-how-do-you-secure-api-gateway-endpoints)
18. [Lambda Versioning & Aliases](#q18-how-do-you-manage-lambda-versions)
19. [EventBridge Scheduler](#q19-how-do-you-schedule-serverless-tasks)
20. [Serverless Cost Model](#q20-how-is-aws-lambda-pricing-calculated)

---

## Q1. What does "Serverless" mean in AWS?
Concept:
It doesn't mean "no servers"; it means **you don't manage the servers**.
- **No provisioning/patching:** AWS handles OS and infrastructure.
- **Auto-scaling:** Scales automatically from 0 to thousands.
- **Pay-for-value:** You pay only when code runs (execution duration), not for idle time.

Example Services: Lambda, DynamoDB, API Gateway, Fargate, S3, EventBridge.

⚡ One-liner to remember:
Serverless = Zero management + Auto-scaling + Pay-per-use.

## Q2. What is a Lambda "Cold Start" and how do you mitigate it?
Concept:
When a Lambda function is invoked for the first time (or after being idle), AWS must spin up a new container environment, download code, and start the runtime. This adds latency (100ms - seconds).

Mitigation:
1.  **Provisioned Concurrency:** Keeps a set number of environments "warm" (Costs extra).
2.  **Minimize Package Size:** Smaller code = faster download.
3.  **Choose Runtime:** Node.js/Python start faster than Java/C#.
4.  **Keep Warm:** Ping the function every 5 mins (Less effective now).

Analogy:
- **Cold Start:** Starting a car on a freezing morning (takes time to warm up).
- **Warm:** A taxi with the engine running, waiting for you.

⚡ One-liner to remember:
Latency during initialization of a new container; fix with Provisioned Concurrency.

## Q3. What are the default execution limits of Lambda?
Concept:
- **Timeout:** 15 minutes (900 seconds).
- **Memory:** 128 MB to 10 GB.
- **Temp Storage (/tmp):** 512 MB to 10 GB.
- **Concurrency:** 1000 concurrent executions per region (can be increased).
- **Payload:** 6 MB (Sync), 256 KB (Async).

⚡ One-liner to remember:
Max 15 minutes runtime; built for short-lived tasks.

## Q4. What are the integration types in API Gateway?
Concept:
1.  **Lambda Proxy Integration:** Passes the raw HTTP request directly to Lambda. Lambda is responsible for parsing and formatting the response. (Most common).
2.  **Lambda Non-Proxy (Custom):** API Gateway transforms the request/response using VTL mapping templates before sending to Lambda.
3.  **HTTP Proxy:** Proxies request to another HTTP endpoint.
4.  **Mock:** Returns a static response without backend.

⚡ One-liner to remember:
Use "Proxy Integration" to let Lambda handle the full Request/Response object.

## Q5. What are Lambda Layers?
Concept:
A way to pull common code (dependencies, libraries, custom runtimes) out of your function code and into a shared "layer".
- Reduces deployment package size.
- Promotes code reuse across multiple functions.

Analogy:
Libraries in a programming language. Instead of copying `node_modules` into every single project folder, you reference a global shared library.

⚡ One-liner to remember:
Shared zip files containing libraries/dependencies mounted to `/opt`.

## Q6. What is the difference between Provisioned and On-Demand Capacity in DynamoDB?
Concept:
- **Provisioned:** You specify WCU/RCU (Write/Read Capacity Units). Cheaper if predictable.
- **On-Demand:** Auto-scales instantly. Pay-per-request. Good for spiky/unknown traffic.

⚡ One-liner to remember:
Provisioned for steady; On-Demand for unpredictable.

## Q7. Why use Step Functions instead of chaining Lambdas?
Concept:
- **Chaining (Lambda A -> Lambda B):** Tight coupling. Hard to debug. If B fails, A might timeout waiting. "Spaghetti code".
- **Step Functions:** State machine orchestrator. Visual workflow. Handles retries, error handling, parallel execution, and wait states natively. Decoupled.

Analogy:
- **Chaining:** A relay race where runners have to shout instructions to each other.
- **Step Functions:** A conductor leading an orchestra. The conductor tells each musician when to play.

⚡ One-liner to remember:
Step Functions orchestrate workflows; Lambda executes tasks.

## Q8. How does Lambda read from SQS or Kinesis?
Concept:
Lambda uses an **Event Source Mapping** (Poll-based).
An internal AWS process polls the SQS queue or Kinesis stream, batches the records, and invokes your Lambda function synchronously with the batch.

- **Batch Size:** Configurable (e.g., 10 messages).
- **Batch Window:** Wait time to fill batch.

⚡ One-liner to remember:
AWS polls the queue for you and invokes Lambda with a batch of events.

## Q9. What is AWS SAM?
Concept:
Serverless Application Model (SAM) is an open-source framework (extension of CloudFormation) for building serverless apps.
- Simplified syntax for defining Lambdas, APIs, Tables (`AWS::Serverless::Function`).
- **SAM CLI:** Local testing, building, and deploying.

Analogy:
React Create App for Serverless. It sets up the boilerplate and simplifies the complex configuration.

⚡ One-liner to remember:
IaC tool specifically designed for Serverless resources.

## Q10. How does API Gateway handle throttling?
Concept:
- **Account Level:** Default 10,000 requests/sec.
- **Stage Level:** Can limit specific stages (dev/prod).
- **Method Level:** Can limit specific routes (e.g., POST /login).
- **Usage Plans:** Throttling per API Key (Client).
Returns `429 Too Many Requests`.

⚡ One-liner to remember:
Token Bucket algorithm to limit RPS (Requests Per Second) and Burst.

## Q11. Explain Lambda Reserved vs Provisioned Concurrency.
Concept:
- **Reserved Concurrency:** A **Limit**. Guarantees a function *can* scale up to X, but also *stops* it from scaling beyond X (Protecting downstream DBs).
- **Provisioned Concurrency:** A **Performance Booster**. Keeps X instances initialized and ready to respond (No cold starts).

Analogy:
- **Reserved:** "This room can hold MAX 50 people".
- **Provisioned:** "Keep 10 seats warm and reserved for VIPs".

⚡ One-liner to remember:
Reserved limits max scale; Provisioned eliminates cold starts.

## Q12. How do you handle failures in Async Lambda invocations?
Concept:
For async events (S3, SNS), Lambda retries twice (3 attempts total).
If it still fails, the event is discarded unless a **DLQ (Dead Letter Queue)** or **Lambda Destination** is configured to capture the failed event.

⚡ One-liner to remember:
Configure a DLQ (SQS/SNS) to catch events after retries are exhausted.

## Q13. What are Lambda Destinations?
Concept:
A newer feature (better than DLQ) for Async invocations.
Can route *both* **Success** and **Failure** results to other services (SQS, SNS, Lambda, EventBridge).
Provides more context (stack trace, request payload) than standard DLQ.

⚡ One-liner to remember:
Route success/failure results to other services without writing code.

## Q14. When should you choose Lambda over Fargate?
Concept:
- **Lambda:** Spiky traffic, short execution (<15 min), event-driven, zero maintenance.
- **Fargate (Serverless Containers):** Long-running processes, consistent high load, need full control over OS/networking, legacy apps that are hard to refactor.

⚡ One-liner to remember:
Lambda for functions; Fargate for long-running containers.

## Q15. What are DynamoDB Global Tables?
Concept:
Multi-Region, multi-master database.
- Replicates data automatically across chosen regions.
- Provides local read/write performance for global users.
- Handles conflict resolution (Last Writer Wins).

⚡ One-liner to remember:
Active-Active multi-region replication for DynamoDB.

## Q16. How do you debug and trace Serverless applications?
Concept:
Use **AWS X-Ray**.
- Provides a "Service Map" visualization.
- Traces requests as they travel from API Gateway -> Lambda -> DynamoDB.
- Identifies latency bottlenecks and errors.

⚡ One-liner to remember:
Enable Active Tracing in Lambda/API Gateway to visualize the request path.

## Q17. How do you secure API Gateway endpoints?
Concept:
1.  **API Keys:** For usage throttling (not secure for auth).
2.  **IAM Authorization:** Internal AWS access (e.g., frontend signed with SigV4).
3.  **Cognito User Pools:** JWT tokens for app users.
4.  **Lambda Authorizer (Custom):** Custom logic (validate 3rd party tokens).

⚡ One-liner to remember:
Cognito for users; IAM for services; Lambda Authorizer for custom logic.

## Q18. How do you manage Lambda versions?
Concept:
- **$LATEST:** The mutable working version.
- **Versions:** Immutable snapshots of code (v1, v2).
- **Aliases:** Pointers to versions (e.g., "PROD" -> v1, "DEV" -> $LATEST). allows Blue/Green deployment by shifting traffic weights between aliases.

⚡ One-liner to remember:
Publish Versions (Immutable) and point Aliases (Mutable) to them.

## Q19. How do you schedule Serverless tasks?
Concept:
Use **EventBridge Scheduler** (formerly CloudWatch Events Rules).
- Cron expressions (e.g., "rate(1 day)").
- One-time schedules.
- Triggers Lambda, SQS, Step Functions.

⚡ One-liner to remember:
Serverless Cron jobs via EventBridge.

## Q20. How is AWS Lambda pricing calculated?
Concept:
1.  **Requests:** $0.20 per 1 million requests.
2.  **Duration:** GB-seconds (Memory allocated * Execution time in ms).
3.  **Ephemeral Storage:** Charged if > 512 MB.

*Note: x86 and ARM (Graviton) have different pricing.*

⚡ One-liner to remember:
Pay for number of hits + (Memory × Time).
