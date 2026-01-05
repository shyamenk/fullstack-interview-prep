# AWS Services Interview Questions & Answers

## Table of Contents

1. [What is a Dead Letter Queue (DLQ) and why is it used?](#q1-what-is-a-dead-letter-queue-dlq-and-why-is-it-used)

---

## Q1. What is a Dead Letter Queue (DLQ) and why is it used?

### Concept:

A Dead Letter Queue (DLQ) is a **special queue that stores messages that failed to process** after multiple retry attempts.

- Prevents failed messages from blocking the main queue
- Helps debug and analyze problematic messages
- Commonly used with AWS SQS, SNS, Lambda, and other message brokers

### How It Works:

1. Message arrives in main queue
2. Consumer tries to process it
3. If processing fails → message returns to queue
4. After **max retries reached** → message moves to DLQ
5. Main queue continues processing other messages

### Example (AWS SQS):

```typescript
// Configure DLQ for main queue
const dlqArn = "arn:aws:sqs:us-east-1:123456:my-dlq";

const queueConfig = {
  QueueName: "orders-queue",
  RedrivePolicy: JSON.stringify({
    deadLetterTargetArn: dlqArn,
    maxReceiveCount: 3  // Move to DLQ after 3 failed attempts
  })
};
```

### Processing with DLQ:

```typescript
// Main queue consumer
async function processOrder(message) {
  try {
    await processPayment(message.orderId);
    // Success → delete message from queue
    await sqs.deleteMessage({ ReceiptHandle: message.ReceiptHandle });
  } catch (error) {
    // Failure → message stays in queue (will retry)
    // After 3 retries → automatically moves to DLQ
    console.error("Failed to process:", error);
  }
}

// Monitor DLQ for failed messages
async function checkDLQ() {
  const messages = await sqs.receiveMessage({ QueueUrl: dlqUrl });
  messages.forEach(msg => {
    console.log("Failed message:", msg.Body);
    // Debug, alert, or manually fix
  });
}
```

### Why Use DLQ:

| Without DLQ | With DLQ |
|-------------|----------|
| Failed messages block the queue | Failed messages isolated |
| Infinite retry loops waste resources | Limited retries, then moved |
| Hard to debug failures | Easy to inspect failed messages |
| Queue gets stuck | Queue keeps flowing |

### Common Use Cases:

1. **Payment processing failures** - invalid card, insufficient funds
2. **Third-party API errors** - timeout, service unavailable
3. **Data validation errors** - malformed JSON, missing fields
4. **Lambda function errors** - code bugs, memory issues

### AWS Services with DLQ Support:

| Service | Use Case |
|---------|----------|
| SQS | Message queue failures |
| SNS | Subscription delivery failures |
| Lambda | Async invocation failures |
| EventBridge | Event delivery failures |

### Best Practices:

- **Set appropriate retry count** (3-5 retries typical)
- **Monitor DLQ metrics** - alert when messages arrive
- **Investigate failures** - fix root cause
- **Replay messages** - reprocess after fixing issue
- **Set DLQ retention** - keep for debugging (e.g., 14 days)

### Example: Lambda with DLQ

```typescript
// Lambda function configuration
const lambdaConfig = {
  FunctionName: "processOrder",
  DeadLetterConfig: {
    TargetArn: "arn:aws:sqs:us-east-1:123456:lambda-dlq"
  }
};

// If Lambda fails after retries → event goes to DLQ
exports.handler = async (event) => {
  if (!event.orderId) {
    throw new Error("Missing orderId"); // Goes to DLQ after retries
  }
  await processOrder(event.orderId);
};
```

### Monitoring DLQ:

```typescript
// CloudWatch alarm for DLQ
const alarm = {
  AlarmName: "DLQ-Messages-Alert",
  MetricName: "ApproximateNumberOfMessagesVisible",
  Namespace: "AWS/SQS",
  Statistic: "Sum",
  Period: 300,
  EvaluationPeriods: 1,
  Threshold: 1,  // Alert if any message in DLQ
  ComparisonOperator: "GreaterThanThreshold"
};
```

### Analogy:

Think of a **hospital emergency room**:
- Main queue = Regular patients waiting for treatment
- DLQ = ICU for critical cases that need special attention
- Regular flow continues while difficult cases are isolated for deeper investigation

### ⚡ One-liner to remember:
> DLQ = Isolate failed messages after retries to prevent blocking the main queue.

---
