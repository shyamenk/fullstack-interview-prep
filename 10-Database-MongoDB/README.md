# MongoDB NoSQL Database Mastery

## 🎯 MongoDB Fundamentals

### Core Concepts
- [ ] **Document-Oriented Database**
  - JSON-like BSON documents
  - Flexible schema design
  - Embedded documents vs references
  - Collections and databases
- [ ] **Data Modeling**
  - Denormalization strategies
  - Embedding vs referencing
  - One-to-one, one-to-many, many-to-many relationships
  - Schema design patterns

### MongoDB Architecture
- [ ] **Database Structure**
  - Databases contain collections
  - Collections contain documents
  - Documents contain fields
  - No enforced schema
- [ ] **Storage Engine**
  - WiredTiger storage engine
  - Memory mapping
  - Compression options
  - Journaling for durability

## 🚀 Advanced MongoDB Features

### Indexing and Performance
- [ ] **Index Types**
  - Single field indexes
  - Compound indexes
  - Multikey indexes
  - Text indexes for search
  - Geospatial indexes
  - Partial and sparse indexes
- [ ] **Query Optimization**
  - Query plans and explain()
  - Index intersection
  - Index prefixes
  - Covered queries

### Aggregation Framework
- [ ] **Pipeline Stages**
  - $match, $group, $sort, $project
  - $lookup for joins
  - $unwind for arrays
  - $facet for multiple pipelines
- [ ] **Advanced Operations**
  - $bucket and $bucketAuto
  - $graphLookup for hierarchical data
  - $merge and $out for output
  - Custom expressions and operators

## 💻 Common Interview Questions

### Conceptual Questions
1. **What's the difference between SQL and NoSQL databases?**
2. **When would you choose MongoDB over PostgreSQL?**
3. **Explain the CAP theorem in context of MongoDB**
4. **What's the difference between embedding and referencing?**
5. **How does MongoDB ensure data consistency?**

### Technical Questions
1. **Design a schema for a social media application**
2. **Write an aggregation pipeline for complex analytics**
3. **Optimize a slow-performing query**
4. **Implement real-time data synchronization**
5. **Handle large file storage with GridFS**

### Scaling Questions
1. **How would you handle 1 million documents?**
2. **Explain sharding strategies in MongoDB**
3. **Design a replication setup for high availability**
4. **Implement caching strategies with MongoDB**

## 🛠️ Practical Examples

### Database Design and Schema
```javascript
// User document with embedded profile
const userSchema = {
  _id: ObjectId("..."),
  email: "john@example.com",
  username: "johnsmith",
  profile: {
    firstName: "John",
    lastName: "Smith",
    avatar: "https://example.com/avatar.jpg",
    bio: "Software developer passionate about technology",
    location: {
      city: "New York",
      country: "USA",
      coordinates: [-74.006, 40.7128]
    },
    socialLinks: {
      twitter: "@johnsmith",
      linkedin: "linkedin.com/in/johnsmith",
      github: "github.com/johnsmith"
    }
  },
  preferences: {
    theme: "dark",
    notifications: {
      email: true,
      push: false,
      sms: true
    },
    privacy: {
      profileVisibility: "public",
      showLocation: false
    }
  },
  createdAt: ISODate("2023-01-15T10:30:00Z"),
  updatedAt: ISODate("2023-12-01T15:45:00Z"),
  isActive: true,
  lastLoginAt: ISODate("2023-12-01T15:45:00Z"),
  roles: ["user"],
  tags: ["developer", "javascript", "nodejs"]
};

// Post document with references and embedded comments
const postSchema = {
  _id: ObjectId("..."),
  authorId: ObjectId("..."), // Reference to user
  title: "Getting Started with MongoDB",
  content: "MongoDB is a powerful NoSQL database...",
  excerpt: "Learn the basics of MongoDB...",
  slug: "getting-started-with-mongodb",
  status: "published", // draft, published, archived
  categories: [
    { name: "Database", slug: "database" },
    { name: "NoSQL", slug: "nosql" }
  ],
  tags: ["mongodb", "nosql", "database", "tutorial"],
  featuredImage: {
    url: "https://example.com/image.jpg",
    alt: "MongoDB tutorial illustration",
    width: 1200,
    height: 600
  },
  metadata: {
    readTime: 5, // minutes
    difficulty: "beginner",
    language: "en"
  },
  analytics: {
    views: 1250,
    likes: 89,
    shares: 23,
    bookmarks: 45
  },
  // Embedded comments for better read performance
  comments: [
    {
      _id: ObjectId("..."),
      authorId: ObjectId("..."),
      authorName: "Jane Doe", // Denormalized for performance
      content: "Great tutorial! Very helpful.",
      createdAt: ISODate("2023-11-20T14:30:00Z"),
      likes: 5,
      replies: [
        {
          _id: ObjectId("..."),
          authorId: ObjectId("..."),
          authorName: "John Smith",
          content: "Thanks! Glad you found it useful.",
          createdAt: ISODate("2023-11-20T15:00:00Z"),
          likes: 2
        }
      ]
    }
  ],
  seo: {
    metaTitle: "Getting Started with MongoDB - Complete Guide",
    metaDescription: "Learn MongoDB fundamentals...",
    keywords: ["mongodb", "nosql", "database"]
  },
  publishedAt: ISODate("2023-11-15T09:00:00Z"),
  createdAt: ISODate("2023-11-10T16:30:00Z"),
  updatedAt: ISODate("2023-11-20T10:15:00Z")
};
```

### CRUD Operations
```javascript
// Create (Insert) operations
db.users.insertOne({
  email: "alice@example.com",
  username: "alice",
  profile: {
    firstName: "Alice",
    lastName: "Johnson"
  },
  createdAt: new Date()
});

// Insert multiple documents
db.posts.insertMany([
  {
    title: "First Post",
    authorId: ObjectId("..."),
    content: "This is my first post",
    createdAt: new Date()
  },
  {
    title: "Second Post",
    authorId: ObjectId("..."),
    content: "This is my second post",
    createdAt: new Date()
  }
]);

// Read (Find) operations
// Find all active users
db.users.find({ isActive: true });

// Find with specific fields
db.users.find(
  { "profile.location.city": "New York" },
  { email: 1, "profile.firstName": 1, "profile.lastName": 1 }
);

// Find with complex conditions
db.posts.find({
  $and: [
    { status: "published" },
    { publishedAt: { $gte: ISODate("2023-01-01") } },
    { $or: [
      { "analytics.views": { $gte: 1000 } },
      { "analytics.likes": { $gte: 50 } }
    ]}
  ]
});

// Update operations
// Update single document
db.users.updateOne(
  { _id: ObjectId("...") },
  {
    $set: {
      "profile.bio": "Updated bio",
      updatedAt: new Date()
    },
    $inc: { "analytics.profileViews": 1 }
  }
);

// Update multiple documents
db.posts.updateMany(
  { status: "draft" },
  {
    $set: { status: "archived" },
    $currentDate: { updatedAt: true }
  }
);

// Upsert operation
db.users.updateOne(
  { email: "newuser@example.com" },
  {
    $set: {
      username: "newuser",
      createdAt: new Date()
    },
    $setOnInsert: {
      isActive: true,
      roles: ["user"]
    }
  },
  { upsert: true }
);

// Delete operations
// Soft delete
db.posts.updateOne(
  { _id: ObjectId("...") },
  {
    $set: {
      status: "deleted",
      deletedAt: new Date()
    }
  }
);

// Hard delete
db.comments.deleteMany({
  createdAt: { $lt: ISODate("2022-01-01") }
});
```

### Aggregation Pipeline Examples
```javascript
// User analytics aggregation
db.posts.aggregate([
  // Match published posts from this year
  {
    $match: {
      status: "published",
      publishedAt: { $gte: ISODate("2023-01-01") }
    }
  },

  // Group by author and calculate metrics
  {
    $group: {
      _id: "$authorId",
      totalPosts: { $sum: 1 },
      totalViews: { $sum: "$analytics.views" },
      totalLikes: { $sum: "$analytics.likes" },
      avgViews: { $avg: "$analytics.views" },
      firstPost: { $min: "$publishedAt" },
      lastPost: { $max: "$publishedAt" }
    }
  },

  // Join with users collection
  {
    $lookup: {
      from: "users",
      localField: "_id",
      foreignField: "_id",
      as: "author"
    }
  },

  // Unwind author array
  { $unwind: "$author" },

  // Project final shape
  {
    $project: {
      authorName: {
        $concat: ["$author.profile.firstName", " ", "$author.profile.lastName"]
      },
      authorEmail: "$author.email",
      totalPosts: 1,
      totalViews: 1,
      totalLikes: 1,
      avgViews: { $round: ["$avgViews", 2] },
      daysSinceFirstPost: {
        $dateDiff: {
          startDate: "$firstPost",
          endDate: new Date(),
          unit: "day"
        }
      },
      engagementRate: {
        $multiply: [
          { $divide: ["$totalLikes", "$totalViews"] },
          100
        ]
      }
    }
  },

  // Sort by engagement rate
  { $sort: { engagementRate: -1 } },

  // Limit results
  { $limit: 10 }
]);

// Category analytics with facets
db.posts.aggregate([
  {
    $match: {
      status: "published",
      publishedAt: { $gte: ISODate("2023-01-01") }
    }
  },

  // Unwind categories array
  { $unwind: "$categories" },

  // Use facets for multiple analyses
  {
    $facet: {
      // Posts per category
      categoryStats: [
        {
          $group: {
            _id: "$categories.name",
            postCount: { $sum: 1 },
            totalViews: { $sum: "$analytics.views" },
            avgViews: { $avg: "$analytics.views" }
          }
        },
        { $sort: { postCount: -1 } }
      ],

      // Monthly post distribution
      monthlyDistribution: [
        {
          $group: {
            _id: {
              year: { $year: "$publishedAt" },
              month: { $month: "$publishedAt" }
            },
            postCount: { $sum: 1 },
            totalViews: { $sum: "$analytics.views" }
          }
        },
        { $sort: { "_id.year": 1, "_id.month": 1 } }
      ],

      // Overall statistics
      overallStats: [
        {
          $group: {
            _id: null,
            totalPosts: { $sum: 1 },
            totalViews: { $sum: "$analytics.views" },
            totalLikes: { $sum: "$analytics.likes" },
            avgViewsPerPost: { $avg: "$analytics.views" },
            topCategory: { $first: "$categories.name" }
          }
        }
      ]
    }
  }
]);

// Text search aggregation
db.posts.aggregate([
  // Text search
  {
    $match: {
      $text: { $search: "mongodb database tutorial" }
    }
  },

  // Add relevance score
  {
    $addFields: {
      score: { $meta: "textScore" }
    }
  },

  // Join with author data
  {
    $lookup: {
      from: "users",
      localField: "authorId",
      foreignField: "_id",
      as: "author",
      pipeline: [
        {
          $project: {
            profile: 1,
            username: 1
          }
        }
      ]
    }
  },

  // Sort by relevance
  { $sort: { score: { $meta: "textScore" } } },

  // Project final fields
  {
    $project: {
      title: 1,
      excerpt: 1,
      slug: 1,
      "author.profile.firstName": 1,
      "author.profile.lastName": 1,
      "author.username": 1,
      publishedAt: 1,
      "analytics.views": 1,
      "analytics.likes": 1,
      score: 1
    }
  }
]);
```

### Indexing Strategies
```javascript
// Single field indexes
db.users.createIndex({ email: 1 }, { unique: true });
db.posts.createIndex({ authorId: 1 });
db.posts.createIndex({ publishedAt: -1 });

// Compound indexes
db.posts.createIndex({
  status: 1,
  publishedAt: -1,
  "analytics.views": -1
});

// Text indexes for search
db.posts.createIndex({
  title: "text",
  content: "text",
  excerpt: "text"
}, {
  weights: {
    title: 10,
    excerpt: 5,
    content: 1
  }
});

// Geospatial indexes
db.users.createIndex({ "profile.location.coordinates": "2dsphere" });

// Partial indexes
db.posts.createIndex(
  { slug: 1 },
  {
    partialFilterExpression: { status: "published" },
    unique: true
  }
);

// TTL indexes for auto-deletion
db.sessions.createIndex(
  { createdAt: 1 },
  { expireAfterSeconds: 3600 } // 1 hour
);

// Multikey indexes for arrays
db.posts.createIndex({ tags: 1 });
db.posts.createIndex({ "categories.slug": 1 });

// Sparse indexes
db.users.createIndex(
  { "profile.socialLinks.twitter": 1 },
  { sparse: true }
);
```

## 🔧 MongoDB Administration

### Replica Sets and High Availability
```javascript
// Initialize replica set
rs.initiate({
  _id: "myReplicaSet",
  members: [
    { _id: 0, host: "mongo1:27017", priority: 1 },
    { _id: 1, host: "mongo2:27017", priority: 0.5 },
    { _id: 2, host: "mongo3:27017", priority: 0.5 }
  ]
});

// Check replica set status
rs.status();

// Add member to replica set
rs.add("mongo4:27017");

// Remove member from replica set
rs.remove("mongo4:27017");

// Step down primary
rs.stepDown();

// Configure read preferences
db.posts.find().readPref("secondary");
db.posts.find().readPref("secondaryPreferred");
```

### Sharding for Horizontal Scaling
```javascript
// Enable sharding on database
sh.enableSharding("myapp");

// Shard collection by user ID
sh.shardCollection("myapp.posts", { authorId: 1 });

// Shard collection with compound key
sh.shardCollection("myapp.analytics", {
  userId: 1,
  timestamp: 1
});

// Check sharding status
sh.status();

// Get shard distribution
db.posts.getShardDistribution();

// Balance shards
sh.enableBalancing("myapp.posts");
sh.disableBalancing("myapp.posts");
```

### Monitoring and Performance
```javascript
// Database statistics
db.stats();
db.posts.stats();

// Current operations
db.currentOp();

// Kill long-running operation
db.killOp(12345);

// Profiler for slow queries
db.setProfilingLevel(2, { slowms: 100 });
db.system.profile.find().sort({ ts: -1 }).limit(5);

// Index usage statistics
db.posts.aggregate([{ $indexStats: {} }]);

// Explain query execution
db.posts.find({ authorId: ObjectId("...") }).explain("executionStats");

// Validate collection
db.posts.validate();

// Compact collection
db.runCommand({ compact: "posts" });
```

## 🔒 Security Best Practices

### Authentication and Authorization
```javascript
// Create admin user
use admin
db.createUser({
  user: "admin",
  pwd: "secure_password",
  roles: ["root"]
});

// Create application user
use myapp
db.createUser({
  user: "app_user",
  pwd: "app_password",
  roles: [
    { role: "readWrite", db: "myapp" },
    { role: "read", db: "analytics" }
  ]
});

// Create read-only user
db.createUser({
  user: "readonly_user",
  pwd: "readonly_password",
  roles: [{ role: "read", db: "myapp" }]
});

// Custom role for specific operations
db.createRole({
  role: "postManager",
  privileges: [
    {
      resource: { db: "myapp", collection: "posts" },
      actions: ["find", "insert", "update", "remove"]
    },
    {
      resource: { db: "myapp", collection: "users" },
      actions: ["find"]
    }
  ],
  roles: []
});

// Grant role to user
db.grantRolesToUser("app_user", ["postManager"]);
```

### Data Encryption and Security
```javascript
// Field-level encryption setup
const clientEncryption = new ClientEncryption(keyVault, {
  keyVaultNamespace: "encryption.__keyVault",
  kmsProviders: {
    local: {
      key: localMasterKey
    }
  }
});

// Create data encryption key
const dataKeyId = await clientEncryption.createDataKey("local", {
  keyAltNames: ["user-pii-key"]
});

// Encrypt sensitive field
const encryptedSSN = await clientEncryption.encrypt(
  "123-45-6789",
  {
    algorithm: "AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic",
    keyId: dataKeyId
  }
);

// Store encrypted data
db.users.insertOne({
  name: "John Doe",
  ssn: encryptedSSN,
  email: "john@example.com"
});
```

## 🧪 Testing with MongoDB

### Unit Testing
```javascript
const { MongoMemoryServer } = require('mongodb-memory-server');
const { MongoClient } = require('mongodb');

describe('User Service', () => {
  let connection;
  let db;
  let mongod;

  beforeAll(async () => {
    mongod = await MongoMemoryServer.create();
    const uri = mongod.getUri();
    connection = await MongoClient.connect(uri);
    db = connection.db();
  });

  afterAll(async () => {
    await connection.close();
    await mongod.stop();
  });

  beforeEach(async () => {
    await db.collection('users').deleteMany({});
  });

  test('should create user successfully', async () => {
    const userData = {
      email: 'test@example.com',
      username: 'testuser',
      profile: {
        firstName: 'Test',
        lastName: 'User'
      }
    };

    const result = await db.collection('users').insertOne(userData);
    expect(result.acknowledged).toBe(true);

    const user = await db.collection('users').findOne({
      email: 'test@example.com'
    });
    expect(user).toBeTruthy();
    expect(user.username).toBe('testuser');
  });

  test('should prevent duplicate emails', async () => {
    await db.collection('users').createIndex({ email: 1 }, { unique: true });

    const userData = {
      email: 'duplicate@example.com',
      username: 'user1'
    };

    await db.collection('users').insertOne(userData);

    await expect(
      db.collection('users').insertOne({
        email: 'duplicate@example.com',
        username: 'user2'
      })
    ).rejects.toThrow();
  });
});
```

## 📚 Learning Resources

### Official Documentation
- [MongoDB Manual](https://docs.mongodb.com/manual/)
- [MongoDB University](https://university.mongodb.com/)
- [MongoDB Blog](https://www.mongodb.com/blog)

### Recommended Books
- "MongoDB: The Definitive Guide" by Shannon Bradshaw
- "MongoDB in Action" by Kyle Banker
- "MongoDB Applied Design Patterns" by Rick Copeland

### Online Courses
- "MongoDB Basics" by MongoDB University
- "MongoDB for Developers" by Pluralsight
- "Complete MongoDB Bootcamp" by Udemy

## ✅ Progress Tracker

### Core Concepts
- [ ] Document model and BSON understood
- [ ] CRUD operations mastered
- [ ] Schema design patterns learned
- [ ] Indexing strategies applied

### Advanced Features
- [ ] Aggregation pipeline mastered
- [ ] Text search implemented
- [ ] Geospatial queries practiced
- [ ] GridFS for file storage used

### Scaling and Administration
- [ ] Replica sets configured
- [ ] Sharding strategies understood
- [ ] Performance monitoring set up
- [ ] Security measures implemented

### Practical Experience
- [ ] 3+ MongoDB applications built
- [ ] Complex aggregations written
- [ ] Performance optimization performed
- [ ] Production deployment achieved

**Target**: Master MongoDB for flexible, scalable NoSQL data storage and complex document operations