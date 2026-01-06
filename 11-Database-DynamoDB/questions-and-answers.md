# DynamoDB Interview Questions & Answers

## Table of Contents

1. [Difference between Partition Key and Sort Key](#q1-what-is-the-difference-between-a-partition-key-and-a-sort-key)
2. [Local Secondary Index (LSI) vs Global Secondary Index (GSI)](#q2-what-is-the-difference-between-lsi-and-gsi)
3. [Scan vs Query](#q3-what-is-the-difference-between-scan-and-query)
4. [Consistency Models (Strongly vs Eventually Consistent)](#q4-what-are-the-consistency-models-in-dynamodb)
5. [Single Table Design](#q5-what-is-single-table-design-and-why-use-it)
6. [DynamoDB Streams](#q6-what-are-dynamodb-streams)
7. [Time To Live (TTL)](#q7-what-is-ttl-in-dynamodb)
8. [Provisioned vs On-Demand Capacity](#q8-what-is-the-difference-between-provisioned-and-on-demand-capacity)
9. [Conditional Writes & Optimistic Locking](#q9-how-do-you-handle-concurrent-updates-optimistic-locking)
10. [DynamoDB Accelerator (DAX)](#q10-what-is-dax-and-when-should-you-use-it)
11. [Handling Throttling](#q11-how-do-you-handle-provisionedthroughputexceededexception)
12. [Atomic Counters](#q12-how-do-you-implement-atomic-counters)
13. [Sparse Indexes](#q13-what-is-a-sparse-index-and-when-is-it-useful)
14. [Batch Operations](#q14-what-are-batch-operations-and-their-limitations)
15. [Hot Partitions](#q15-what-is-a-hot-partition-and-how-do-you-avoid-it)
16. [TransactWriteItems (ACID Transactions)](#q16-does-dynamodb-support-acid-transactions)
17. [Projection Expressions](#q17-what-is-a-projection-expression)
18. [One-to-Many Relationships](#q18-how-do-you-model-one-to-many-relationships-in-dynamodb)
19. [Many-to-Many Relationships](#q19-how-do-you-model-many-to-many-relationships-in-dynamodb)
20. [Item Size Limits](#q20-what-are-the-item-size-limits-in-dynamodb)

---

## Q1. What is the difference between a Partition Key and a Sort Key?
Concept:
DynamoDB uses primary keys to uniquely identify items.
- **Partition Key (PK):** Determines which partition (physical storage) the data is stored in. It is required.
- **Sort Key (SK):** Sorts items with the same Partition Key together. It is optional. Together they form a Composite Primary Key.

Example:
```json
// Table: Orders
// PK: UserId (Groups all orders for a user)
// SK: OrderDate (Orders user's orders by date)

{
  "UserId": "user_123",    // Partition Key
  "OrderDate": "2023-10-01", // Sort Key
  "Total": 50.00
}
```

Analogy:
Think of a library:
- **Partition Key:** The genre shelf (e.g., "Sci-Fi"). It tells you which shelf to go to.
- **Sort Key:** The author's last name. Within that shelf, books are organized by author.

⚡ One-liner to remember:
Partition Key finds the bucket; Sort Key organizes the items inside the bucket.

## Q2. What is the difference between LSI and GSI?
Concept:
Secondary Indexes allow querying by non-primary key attributes.
- **LSI (Local Secondary Index):** Uses the *same* Partition Key as the base table but a *different* Sort Key. Must be created at table creation. Strongly consistent reads supported.
- **GSI (Global Secondary Index):** Can have a *different* Partition Key and *different* Sort Key. Can be added anytime. Only supports eventually consistent reads.

Example:
```javascript
// Base Table: Music (PK: Artist, SK: SongTitle)

// LSI: PK: Artist, SK: ReleaseDate
// Query: "Get all songs by Artist 'Queen' sorted by Date"

// GSI: PK: Genre, SK: Artist
// Query: "Get all songs in 'Rock' genre by 'Queen'"
```

Analogy:
- **LSI:** An appendix at the back of a specific book (Base Table). It only helps you find things *within that book*.
- **GSI:** A master catalog for the entire library. It can look up books across all shelves (partitions) by a completely different criteria (e.g., color).

⚡ One-liner to remember:
LSI = Same partition, new sort; GSI = New partition, new sort.

## Q3. What is the difference between Scan and Query?
Concept:
- **Query:** Retrieves items based on the Primary Key (PK). It is efficient because it goes directly to the partition.
- **Scan:** Reads *every item* in the table to find matches. It is slow, expensive, and consumes a lot of throughput.

Example:
```javascript
// Efficient Query
const params = {
  TableName: "Users",
  KeyConditionExpression: "UserId = :uid",
  ExpressionAttributeValues: { ":uid": "123" }
};

// Expensive Scan
const params = {
  TableName: "Users",
  FilterExpression: "Age > :age", // Reads ALL users, then filters
  ExpressionAttributeValues: { ":age": 25 }
};
```

Analogy:
- **Query:** Looking up a word in the dictionary index to find the page.
- **Scan:** Reading every page of the dictionary one by one to find a word.

⚡ One-liner to remember:
Query is a sniper shot; Scan is a machine gun spray (expensive and wasteful).

## Q4. What are the consistency models in DynamoDB?
Concept:
DynamoDB replicates data across 3 Availability Zones (AZs).
- **Eventually Consistent (Default):** Reads might not reflect the absolute latest write (usually consistent within < 1 sec). Costs 0.5 RCU.
- **Strongly Consistent:** Returns the most up-to-date data including all successful writes prior. Costs 1.0 RCU (double the price).

Example:
```javascript
const params = {
  TableName: "BankTransactions",
  Key: { TransactionId: "tx_123" },
  ConsistentRead: true // Requesting strong consistency
};
docClient.get(params).promise();
```

Analogy:
- **Eventually Consistent:** Checking a sports score on a delayed stream (might be a few seconds behind).
- **Strongly Consistent:** Being at the stadium watching the game live.

⚡ One-liner to remember:
Eventual is fast and cheap; Strong is accurate but costs double.

## Q5. What is Single Table Design and why use it?
Concept:
Storing multiple entity types (Users, Orders, Products) in a single DynamoDB table to enable retrieving related data in a single request (joining data via PK/SK adjacency).

Reduces number of network round trips.
Optimized for read performance at scale.
Avoids expensive "joins" in application code.

Example:
```json
// One table with generic PK/SK names
[
  { "PK": "USER#123", "SK": "PROFILE", "Name": "Alice" },        // User Profile
  { "PK": "USER#123", "SK": "ORDER#999", "Total": 50 },          // User's Order
  { "PK": "ORDER#999", "SK": "ITEM#55", "Product": "Book" }      // Order Item
]
```

Analogy:
Instead of having separate file cabinets for "Invoices", "Customers", and "Products", you have one folder per Customer containing their Profile stapled to their Invoices. You pull one folder and get everything.

⚡ One-liner to remember:
Pre-join your data on write so you can fetch it all with one read.

## Q6. What are DynamoDB Streams?
Concept:
An ordered flow of information about changes (INSERT, MODIFY, REMOVE) to items in a table. Used for Event-Driven Architecture.

Triggering Lambda functions on DB changes.
Replicating data to other stores (Elasticsearch, Redshift).
Audit logging.

Example:
1. User updates profile in DynamoDB.
2. DynamoDB Stream captures the `MODIFY` event.
3. Lambda function triggers, reads the stream, and invalidates the Redis cache for that user.

Analogy:
A news ticker or RSS feed for your database. Every time something changes, a notification is broadcasted.

⚡ One-liner to remember:
Database change log that triggers events in real-time.

## Q7. What is TTL in DynamoDB?
Concept:
Time To Live (TTL) automatically deletes items after a specific timestamp (Unix epoch format). It is free of cost (doesn't consume WCU).

Great for session storage, temporary logs, or expiring coupons.

Example:
```json
{
  "SessionId": "sess_123",
  "Data": "...",
  "ExpirationTime": 1716537600 // Unix timestamp for future date
}
```
*DynamoDB background worker will delete this item after that time.*

Analogy:
Mission Impossible messages: "This message will self-destruct in 5 seconds."

⚡ One-liner to remember:
Set an expiry date on your data and let DynamoDB garbage collect it for free.

## Q8. What is the difference between Provisioned and On-Demand Capacity?
Concept:
- **Provisioned:** You specify expected reads/writes per second (RCU/WCU). Cheaper if you have predictable traffic. Supports Auto-Scaling.
- **On-Demand:** Pay-per-request. Automatically scales to zero or thousands instantly. More expensive per request but zero management.

Example:
- **Provisioned:** An e-commerce site knowing it gets 1000 orders/hr on weekdays.
- **On-Demand:** A startup launching a product with unknown traffic patterns.

Analogy:
- **Provisioned:** Buying a monthly gym membership (cheaper if you go regularly).
- **On-Demand:** Paying a drop-in fee every time you visit (flexible but pricier per visit).

⚡ One-liner to remember:
Provisioned for predictable loads; On-Demand for unpredictable or spiky workloads.

## Q9. How do you handle concurrent updates (Optimistic Locking)?
Concept:
To prevent overwriting changes made by another user, use **Conditional Writes**. Check if the item has changed since you last read it (often using a `version` number).

Example:
```javascript
const params = {
  TableName: "Products",
  Key: { Id: "prod_1" },
  UpdateExpression: "set Price = :p, Version = :v_new",
  ConditionExpression: "Version = :v_current", // Fail if version changed
  ExpressionAttributeValues: {
    ":p": 20,
    ":v_new": 2,
    ":v_current": 1
  }
};
```

Analogy:
Two people editing a Google Doc offline. When they reconnect, the system checks who has the latest version before saving to avoid overwriting each other's work.

⚡ One-liner to remember:
"Only update if the version is what I think it is."

## Q10. What is DAX and when should you use it?
Concept:
DynamoDB Accelerator (DAX) is a fully managed, in-memory cache for DynamoDB. It reduces response times from milliseconds to microseconds.

Use for read-heavy workloads (e.g., leaderboards, product catalogs).
Transparent API compatible (no code changes needed).

Example:
Instead of `const client = new DynamoDB()`, you use `const client = new AmazonDaxClient()`. The application logic remains identical.

Analogy:
DAX is the RAM of your database. Instead of reading from the hard drive (DynamoDB) every time, you keep frequently accessed data in fast memory.

⚡ One-liner to remember:
Plug-and-play cache for microseconds latency.

## Q11. How do you handle ProvisionedThroughputExceededException?
Concept:
This error occurs when your request rate exceeds your configured RCU or WCU.
Strategies:
1. **Exponential Backoff:** Retry the request with increasing delays.
2. **Jitter:** Add random delay to prevent synchronized retries (thundering herd).
3. **Increase Capacity:** Switch to On-Demand or increase Provisioned limits.

Example:
```javascript
// Pseudo-code for retry
function retry(fn, retries = 3, delay = 100) {
  try {
    return fn();
  } catch (err) {
    if (err.code === 'ProvisionedThroughputExceededException' && retries > 0) {
      sleep(delay);
      return retry(fn, retries - 1, delay * 2); // Exponential backoff
    }
    throw err;
  }
}
```

Analogy:
A traffic jam at a toll booth. If everyone tries to push through at once, it jams. If cars wait a bit (backoff) and stagger their approach (jitter), traffic flows again.

⚡ One-liner to remember:
Wait, back off exponentially, and try again.

## Q12. How do you implement atomic counters?
Concept:
Use `UpdateExpression` with `ADD` or mathematical operations to increment a value on the server side without reading it first. This is atomic and safe from concurrency issues.

Example:
```javascript
const params = {
  TableName: "Articles",
  Key: { Id: "art_1" },
  UpdateExpression: "ADD Views :inc",
  ExpressionAttributeValues: { ":inc": 1 }
};
```

Analogy:
A turnstile counter. You don't check the current number, calculate +1, and write it back. You just push the arm, and the machine mechanically clicks +1 safely.

⚡ One-liner to remember:
Server-side incrementing ensures you never lose a count.

## Q13. What is a Sparse Index and when is it useful?
Concept:
A Sparse Index is a GSI where the Partition/Sort keys are not present in every item. DynamoDB *only* writes items to the index if the index keys exist in the item.

Useful for filtering specific items (e.g., "Find all users who are admins") without scanning the whole table.

Example:
Table: Users (Millions of rows)
Attribute: `IsAdmin` (Only present on 10 users)
GSI: PK = `IsAdmin`

Since only 10 users have the `IsAdmin` attribute, the GSI only contains 10 items. Scanning the GSI is instant.

Analogy:
A "Lost and Found" box. It doesn't contain a record of every student in the school, only the few who lost something. Finding a lost item is fast because you only check the box.

⚡ One-liner to remember:
An index that ignores items missing the key, creating a tiny, efficient lookup list.

## Q14. What are batch operations and their limitations?
Concept:
`BatchGetItem` and `BatchWriteItem` allow you to read or write up to 25 items in a single network request.

Reduces latency (fewer HTTP connections).
**Limitation:** A batch request can still fail partially (some items succeed, some fail). You must handle `UnprocessedKeys` in the response.

Example:
```javascript
const params = {
  RequestItems: {
    "Table1": [ { PutRequest: { Item: { ... } } } ],
    "Table2": [ { PutRequest: { Item: { ... } } } ]
  }
};
```

Analogy:
Carrying 25 grocery bags in one trip from the car instead of 25 individual trips. Faster, but if you drop one, you have to go back for it.

⚡ One-liner to remember:
Group up to 25 actions to save network trips, but check for dropped items.

## Q15. What is a "Hot Partition" and how do you avoid it?
Concept:
A Hot Partition occurs when a disproportionate amount of traffic hits a single partition key value. Since throughput is shared, one hot key can throttle the entire table.

Solution: Use high-cardinality Partition Keys (e.g., UserID is better than Status). Add "write sharding" (suffixing random numbers 1-10 to the key) if necessary.

Example:
**Bad:** PK = "Election_2024" (Everyone votes on one key).
**Good:** PK = "Election_2024_RegionA", "Election_2024_RegionB".

Analogy:
A single checkout lane at a supermarket being flooded while others are empty. Spread the customers (traffic) to all lanes (partitions).

⚡ One-liner to remember:
Don't hammer one key; spread the load evenly across unique keys.

## Q16. Does DynamoDB support ACID transactions?
Concept:
Yes, via `TransactWriteItems` and `TransactGetItems`.
Allows you to group up to 100 actions in an all-or-nothing operation.
Costs 2x WCU/RCU compared to standard writes/reads.

Example:
Bank transfer: Deduct $50 from A AND Add $50 to B. Both must happen or neither.

```javascript
const params = {
  TransactItems: [
    { Update: { ... } }, // Deduct from A
    { Update: { ... } }  // Add to B
  ]
};
```

Analogy:
Buying a house. You sign the deed AND transfer the money. If the money transfer fails, the deed signing is cancelled. It's all or nothing.

⚡ One-liner to remember:
All-or-nothing operations for critical data integrity, at double the cost.

## Q17. What is a Projection Expression?
Concept:
A way to tell DynamoDB to return only specific attributes of an item to save bandwidth.
*Note: It does not save RCU (Read Capacity Units) because the full item is still read from disk, just filtered before sending.*

Example:
```javascript
const params = {
  TableName: "Users",
  Key: { Id: "123" },
  ProjectionExpression: "Username, Email" // Don't send the massive 'AvatarImage' blob
};
```

Analogy:
Ordering a burger but asking the chef to "hold the onions" before serving it. The chef still made the burger (cost is same), but the plate is lighter.

⚡ One-liner to remember:
Select specific columns to reduce network payload size.

## Q18. How do you model one-to-many relationships in DynamoDB?
Concept:
Two common strategies:
1.  **Denormalization (Complex Attribute):** Store children as a list inside the parent item. Good if data is small/limited.
2.  **Item Collection (Adjacency List):** Store parent and children as separate items with the same Partition Key but different Sort Keys.

Example (Strategy 2):
PK: `USER#123`
- Parent Item: SK: `PROFILE`
- Child Item 1: SK: `POST#001`
- Child Item 2: SK: `POST#002`

Query PK=`USER#123` returns the user AND all their posts.

Analogy:
A folder labeled "Project X". The folder cover is the parent info. The papers inside are the child items. You grab the folder and get both.

⚡ One-liner to remember:
Use composite keys to group parent and children under the same Partition Key.

## Q19. How do you model many-to-many relationships in DynamoDB?
Concept:
Use an Adjacency List pattern where you store the relationship twice (once for each side of the relationship) or use a GSI.

Example: Students <-> Classes
1. Main Table PK: `STUDENT#123`, SK: `CLASS#Math` (Find all classes for a student)
2. GSI PK: `CLASS#Math`, SK: `STUDENT#123` (Find all students for a class)

This allows querying efficiently from both directions.

Analogy:
A phone book (Sorted by Name) vs. A Yellow Pages (Sorted by Category). You need two lists to find "Who is John?" and "Who are all the Plumbers?".

⚡ One-liner to remember:
Duplicate the data (flip the keys) in a GSI to query from the other side.

## Q20. What are the item size limits in DynamoDB?
Concept:
- **Max Item Size:** 400 KB (includes attribute names and values).
- **Max Partition Key Length:** 2048 bytes.
- **Max Sort Key Length:** 1024 bytes.

If you need to store more than 400KB, store the data in S3 and save the S3 URL in DynamoDB.

Example:
Storing a high-res user profile image.
- DynamoDB: `{"UserId": "123", "ImageURL": "s3://bucket/img.jpg"}`
- S3: The actual 5MB image file.

Analogy:
You can't fit a sofa in your mailbox. You put a slip of paper (pointer) in the mailbox telling you the sofa is in the warehouse (S3).

⚡ One-liner to remember:
400KB limit; for anything larger, offload to S3.
