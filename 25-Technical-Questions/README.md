# Technical Interview Questions

## 🎯 Question Categories

### Frontend Technical Questions
Deep dives into React, JavaScript, CSS, and web technologies

### Backend Technical Questions
Node.js, Express, API design, and server-side concepts

### Database Technical Questions
SQL, NoSQL, optimization, and design patterns

### System Design Technical Questions
Architecture decisions, scalability, and trade-offs

## ⚛️ React & Frontend Questions

### React Core Concepts
**Q: Explain the Virtual DOM and how React's diffing algorithm works**
**A:** The Virtual DOM is a JavaScript representation of the actual DOM. React maintains a virtual copy in memory and uses a diffing algorithm to:
1. Compare the new virtual DOM tree with the previous one
2. Identify the minimum set of changes needed
3. Update only the changed elements in the real DOM
4. Use techniques like component keys to optimize list updates

**Follow-up:** What are the performance benefits and limitations?

**Q: What's the difference between useState and useReducer? When would you use each?**
**A:**
- **useState**: Best for simple state (strings, numbers, booleans)
- **useReducer**: Better for complex state objects, multiple related state variables, or when next state depends on previous state

```javascript
// useState - simple state
const [count, setCount] = useState(0);

// useReducer - complex state
const [state, dispatch] = useReducer(reducer, {
  count: 0,
  loading: false,
  error: null
});
```

**Q: How do you optimize React performance? Explain React.memo, useMemo, and useCallback**
**A:**
- **React.memo**: Prevents re-renders if props haven't changed
- **useMemo**: Memoizes expensive calculations
- **useCallback**: Memoizes functions to prevent child re-renders

```javascript
// React.memo for component optimization
const ExpensiveComponent = React.memo(({ data, onUpdate }) => {
  // Only re-renders if data or onUpdate changes
  return <div>{data.title}</div>;
});

// useMemo for expensive calculations
const expensiveValue = useMemo(() => {
  return data.reduce((sum, item) => sum + item.value, 0);
}, [data]);

// useCallback for function memoization
const handleClick = useCallback((id) => {
  onUpdate(id);
}, [onUpdate]);
```

### JavaScript Deep Dive
**Q: Explain event loops, call stack, and task queue in JavaScript**
**A:**
1. **Call Stack**: LIFO structure for function execution
2. **Web APIs**: Handle async operations (setTimeout, HTTP requests)
3. **Task Queue**: Holds callbacks ready for execution
4. **Event Loop**: Moves tasks from queue to stack when stack is empty

```javascript
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve().then(() => console.log('3'));

console.log('4');

// Output: 1, 4, 3, 2
// Microtasks (Promises) have higher priority than macrotasks (setTimeout)
```

**Q: What are closures and provide a practical example**
**A:** Closures give access to outer function's scope from inner function, even after outer function returns.

```javascript
function createCounter(initialValue = 0) {
  let count = initialValue;

  return {
    increment: () => ++count,
    decrement: () => --count,
    getValue: () => count,
    reset: () => { count = initialValue; }
  };
}

const counter = createCounter(10);
console.log(counter.increment()); // 11
console.log(counter.getValue()); // 11
```

## 🔧 Node.js & Backend Questions

### Node.js Fundamentals
**Q: How does Node.js handle concurrency without threads?**
**A:** Node.js uses:
1. **Single-threaded event loop**: Handles I/O operations
2. **Non-blocking I/O**: Operations don't block the main thread
3. **Thread pool**: For file system operations and CPU-intensive tasks
4. **Event-driven architecture**: Callbacks, promises, async/await

```javascript
// Non-blocking I/O example
const fs = require('fs').promises;

async function readFiles() {
  try {
    // These operations run concurrently
    const [file1, file2, file3] = await Promise.all([
      fs.readFile('file1.txt', 'utf8'),
      fs.readFile('file2.txt', 'utf8'),
      fs.readFile('file3.txt', 'utf8')
    ]);

    console.log('All files read successfully');
  } catch (error) {
    console.error('Error reading files:', error);
  }
}
```

**Q: Explain the difference between process.nextTick() and setImmediate()**
**A:**
- **process.nextTick()**: Executes before any other I/O events in the same phase
- **setImmediate()**: Executes in the next iteration of the event loop

```javascript
console.log('Start');

setImmediate(() => console.log('setImmediate'));
process.nextTick(() => console.log('nextTick'));

setTimeout(() => console.log('setTimeout'), 0);

console.log('End');

// Output: Start, End, nextTick, setTimeout, setImmediate
```

### Express.js and API Design
**Q: How would you implement rate limiting in Express.js?**
**A:** Multiple approaches depending on requirements:

```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const redis = require('redis');

// Basic in-memory rate limiting
const basicLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP'
});

// Redis-based rate limiting for distributed systems
const redisClient = redis.createClient();
const distributedLimiter = rateLimit({
  store: new RedisStore({
    client: redisClient,
    prefix: 'rl:'
  }),
  windowMs: 15 * 60 * 1000,
  max: 100
});

// Custom rate limiting with different limits per endpoint
const createCustomLimiter = (windowMs, max) => rateLimit({
  windowMs,
  max,
  keyGenerator: (req) => `${req.ip}:${req.route.path}`,
  skip: (req) => req.user?.role === 'admin' // Skip for admins
});

app.use('/api/auth', createCustomLimiter(60 * 1000, 5)); // 5 per minute
app.use('/api', distributedLimiter);
```

**Q: How do you handle errors in Express.js applications?**
**A:** Comprehensive error handling strategy:

```javascript
// Custom error class
class AppError extends Error {
  constructor(message, statusCode, isOperational = true) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = isOperational;
    Error.captureStackTrace(this, this.constructor);
  }
}

// Async error wrapper
const catchAsync = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

// Route with error handling
app.get('/api/users/:id', catchAsync(async (req, res, next) => {
  const user = await User.findById(req.params.id);

  if (!user) {
    return next(new AppError('User not found', 404));
  }

  res.json({ success: true, data: user });
}));

// Global error handler
app.use((err, req, res, next) => {
  err.statusCode = err.statusCode || 500;

  if (process.env.NODE_ENV === 'development') {
    sendErrorDev(err, res);
  } else {
    sendErrorProd(err, res);
  }
});

function sendErrorProd(err, res) {
  if (err.isOperational) {
    res.status(err.statusCode).json({
      status: 'error',
      message: err.message
    });
  } else {
    console.error('ERROR:', err);
    res.status(500).json({
      status: 'error',
      message: 'Something went wrong!'
    });
  }
}
```

## 🗄️ Database Technical Questions

### SQL and PostgreSQL
**Q: Explain the difference between different types of JOINs with examples**
**A:**

```sql
-- Sample tables
CREATE TABLE users (id, name, department_id);
CREATE TABLE departments (id, name);

-- INNER JOIN: Only matching records
SELECT u.name, d.name as department
FROM users u
INNER JOIN departments d ON u.department_id = d.id;

-- LEFT JOIN: All users, matching departments
SELECT u.name, d.name as department
FROM users u
LEFT JOIN departments d ON u.department_id = d.id;

-- RIGHT JOIN: All departments, matching users
SELECT u.name, d.name as department
FROM users u
RIGHT JOIN departments d ON u.department_id = d.id;

-- FULL OUTER JOIN: All records from both tables
SELECT u.name, d.name as department
FROM users u
FULL OUTER JOIN departments d ON u.department_id = d.id;
```

**Q: How would you optimize a slow-performing query?**
**A:** Systematic optimization approach:

```sql
-- 1. Analyze the query execution plan
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE o.created_at >= '2023-01-01'
  AND c.status = 'active'
ORDER BY o.created_at DESC
LIMIT 100;

-- 2. Add appropriate indexes
CREATE INDEX idx_orders_created_at ON orders(created_at DESC);
CREATE INDEX idx_customers_status ON customers(status);
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

-- 3. Consider composite indexes
CREATE INDEX idx_orders_customer_created ON orders(customer_id, created_at DESC);

-- 4. Rewrite query to use indexes effectively
SELECT o.*, c.name, c.email
FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE c.status = 'active'
  AND o.created_at >= '2023-01-01'
ORDER BY o.created_at DESC
LIMIT 100;

-- 5. Use covering indexes when possible
CREATE INDEX idx_orders_covering ON orders(created_at DESC, customer_id)
INCLUDE (total_amount, status);
```

### NoSQL and MongoDB
**Q: When would you use MongoDB over PostgreSQL?**
**A:** Consider MongoDB when:

1. **Flexible Schema**: Frequent schema changes
2. **Horizontal Scaling**: Need to distribute across servers
3. **Document Structure**: Data naturally fits document model
4. **Rapid Development**: Prototype phase with changing requirements
5. **JSON-heavy**: Working primarily with JSON data

```javascript
// MongoDB excels at nested, varying structures
const userProfile = {
  _id: ObjectId("..."),
  email: "user@example.com",
  profile: {
    personal: {
      firstName: "John",
      lastName: "Doe",
      addresses: [
        { type: "home", street: "123 Main St", city: "NYC" },
        { type: "work", street: "456 Office Blvd", city: "SF" }
      ]
    },
    professional: {
      skills: ["JavaScript", "Python", "MongoDB"],
      experience: [
        {
          company: "TechCorp",
          role: "Developer",
          duration: { start: ISODate("2020-01-01"), end: ISODate("2022-12-31") }
        }
      ]
    },
    preferences: {
      notifications: { email: true, sms: false },
      privacy: { profileVisible: true }
    }
  }
};
```

**Q: Explain MongoDB's aggregation pipeline with a complex example**
**A:**

```javascript
// Complex analytics query: User engagement metrics
db.posts.aggregate([
  // Stage 1: Match recent posts
  {
    $match: {
      createdAt: { $gte: ISODate("2023-01-01") },
      status: "published"
    }
  },

  // Stage 2: Lookup author information
  {
    $lookup: {
      from: "users",
      localField: "authorId",
      foreignField: "_id",
      as: "author",
      pipeline: [
        { $project: { name: 1, email: 1, department: 1 } }
      ]
    }
  },

  // Stage 3: Unwind authors array
  { $unwind: "$author" },

  // Stage 4: Group by department and calculate metrics
  {
    $group: {
      _id: "$author.department",
      totalPosts: { $sum: 1 },
      totalViews: { $sum: "$views" },
      totalLikes: { $sum: "$likes" },
      avgViews: { $avg: "$views" },
      topPost: { $max: { views: "$views", title: "$title" } },
      authors: { $addToSet: "$author.name" }
    }
  },

  // Stage 5: Calculate engagement rate
  {
    $addFields: {
      engagementRate: {
        $multiply: [
          { $divide: ["$totalLikes", "$totalViews"] },
          100
        ]
      },
      authorCount: { $size: "$authors" }
    }
  },

  // Stage 6: Sort by engagement rate
  { $sort: { engagementRate: -1 } },

  // Stage 7: Format output
  {
    $project: {
      department: "$_id",
      metrics: {
        posts: "$totalPosts",
        views: "$totalViews",
        likes: "$totalLikes",
        avgViews: { $round: ["$avgViews", 2] },
        engagementRate: { $round: ["$engagementRate", 2] }
      },
      topPost: 1,
      authorCount: 1,
      _id: 0
    }
  }
]);
```

## 🏗️ System Design Technical Questions

### Scalability and Performance
**Q: How would you design a system to handle 1 million concurrent users?**
**A:** Multi-layered approach:

```
Load Balancer (HAProxy/NGINX)
    ↓
API Gateway (Rate limiting, authentication)
    ↓
Application Servers (Multiple instances)
    ↓
Microservices Architecture
    ↓
Database Layer (Read replicas, sharding)
    ↓
Caching Layer (Redis/Memcached)
    ↓
CDN for static assets
```

**Key strategies:**
1. **Horizontal scaling**: Multiple server instances
2. **Database optimization**: Read replicas, connection pooling
3. **Caching**: Multiple levels (CDN, application, database)
4. **Async processing**: Message queues for heavy operations
5. **Monitoring**: Real-time metrics and alerting

**Q: Explain different caching strategies and when to use each**
**A:**

```javascript
// 1. Cache-Aside (Lazy Loading)
async function getCacheAside(key) {
  let data = await cache.get(key);
  if (!data) {
    data = await database.get(key);
    await cache.set(key, data, TTL);
  }
  return data;
}

// 2. Write-Through
async function writeThrough(key, data) {
  await cache.set(key, data);
  await database.set(key, data);
}

// 3. Write-Behind (Write-Back)
async function writeBehind(key, data) {
  await cache.set(key, data);
  // Queue for background database write
  writeQueue.add({ key, data });
}

// 4. Refresh-Ahead
async function refreshAhead(key) {
  const data = await cache.get(key);
  if (data && data.ttl < REFRESH_THRESHOLD) {
    // Refresh in background
    backgroundRefresh(key);
  }
  return data;
}
```

### Security and Authentication
**Q: How would you implement JWT authentication securely?**
**A:** Comprehensive JWT implementation:

```javascript
const jwt = require('jsonwebtoken');
const crypto = require('crypto');

class AuthService {
  static generateTokens(user) {
    const payload = {
      userId: user.id,
      email: user.email,
      role: user.role
    };

    // Short-lived access token
    const accessToken = jwt.sign(payload, process.env.JWT_SECRET, {
      expiresIn: '15m',
      issuer: 'myapp',
      audience: 'myapp-users'
    });

    // Long-lived refresh token
    const refreshToken = jwt.sign(
      { userId: user.id, tokenId: crypto.randomUUID() },
      process.env.REFRESH_SECRET,
      { expiresIn: '7d' }
    );

    return { accessToken, refreshToken };
  }

  static async verifyAccessToken(token) {
    try {
      const decoded = jwt.verify(token, process.env.JWT_SECRET, {
        issuer: 'myapp',
        audience: 'myapp-users'
      });

      // Check if user still exists and is active
      const user = await User.findById(decoded.userId);
      if (!user || !user.isActive) {
        throw new Error('User not found or inactive');
      }

      return decoded;
    } catch (error) {
      throw new Error('Invalid or expired token');
    }
  }

  static async refreshTokens(refreshToken) {
    const decoded = jwt.verify(refreshToken, process.env.REFRESH_SECRET);

    // Verify refresh token exists in database
    const storedToken = await RefreshToken.findOne({
      userId: decoded.userId,
      tokenId: decoded.tokenId
    });

    if (!storedToken) {
      throw new Error('Invalid refresh token');
    }

    const user = await User.findById(decoded.userId);
    if (!user) {
      throw new Error('User not found');
    }

    // Generate new tokens
    const tokens = this.generateTokens(user);

    // Rotate refresh token
    await RefreshToken.deleteOne({ tokenId: decoded.tokenId });
    await RefreshToken.create({
      userId: user.id,
      tokenId: tokens.refreshToken.tokenId
    });

    return tokens;
  }
}

// Middleware implementation
const authenticate = async (req, res, next) => {
  try {
    const token = req.headers.authorization?.split(' ')[1];

    if (!token) {
      return res.status(401).json({ error: 'No token provided' });
    }

    const decoded = await AuthService.verifyAccessToken(token);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ error: error.message });
  }
};
```

## 🧪 Testing Technical Questions

**Q: What's your approach to testing a full-stack application?**
**A:** Comprehensive testing strategy:

```javascript
// 1. Unit Tests - Individual functions/components
describe('UserService', () => {
  test('should create user with valid data', async () => {
    const userData = {
      email: 'test@example.com',
      password: 'SecurePass123!'
    };

    const user = await UserService.createUser(userData);

    expect(user).toHaveProperty('id');
    expect(user.email).toBe(userData.email);
    expect(user.password).not.toBe(userData.password); // Should be hashed
  });
});

// 2. Integration Tests - API endpoints
describe('POST /api/auth/login', () => {
  test('should login with valid credentials', async () => {
    const response = await request(app)
      .post('/api/auth/login')
      .send({
        email: 'test@example.com',
        password: 'password123'
      });

    expect(response.status).toBe(200);
    expect(response.body).toHaveProperty('accessToken');
    expect(response.body).toHaveProperty('refreshToken');
  });
});

// 3. Component Tests - React components
test('LoginForm submits correct data', () => {
  const mockSubmit = jest.fn();
  render(<LoginForm onSubmit={mockSubmit} />);

  fireEvent.change(screen.getByLabelText(/email/i), {
    target: { value: 'test@example.com' }
  });
  fireEvent.change(screen.getByLabelText(/password/i), {
    target: { value: 'password123' }
  });
  fireEvent.click(screen.getByRole('button', { name: /login/i }));

  expect(mockSubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    password: 'password123'
  });
});

// 4. E2E Tests - Full user workflows
describe('User Authentication Flow', () => {
  test('user can register, login, and access protected content', async () => {
    await page.goto('/register');
    await page.fill('[data-testid="email"]', 'test@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    await page.click('[data-testid="register-button"]');

    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('[data-testid="welcome-message"]')).toBeVisible();
  });
});
```

## 📊 Performance Optimization Questions

**Q: How would you optimize the performance of a React application?**
**A:** Multi-level optimization approach:

```javascript
// 1. Component Level Optimizations
const OptimizedUserList = React.memo(({ users, onUserClick }) => {
  return (
    <VirtualizedList
      items={users}
      renderItem={({ item, index }) => (
        <UserItem
          key={item.id}
          user={item}
          onClick={onUserClick}
        />
      )}
      itemHeight={60}
      windowHeight={400}
    />
  );
});

// 2. State Management Optimization
const useOptimizedUserData = () => {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);

  const debouncedSearch = useMemo(
    () => debounce((query) => {
      setLoading(true);
      searchUsers(query).then(setUsers).finally(() => setLoading(false));
    }, 300),
    []
  );

  const sortedUsers = useMemo(() => {
    return users.sort((a, b) => a.name.localeCompare(b.name));
  }, [users]);

  return { users: sortedUsers, loading, search: debouncedSearch };
};

// 3. Code Splitting
const LazyDashboard = lazy(() =>
  import('./Dashboard').then(module => ({
    default: module.Dashboard
  }))
);

// 4. Bundle Optimization with webpack
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
        common: {
          name: 'common',
          minChunks: 2,
          chunks: 'all',
          enforce: true,
        },
      },
    },
  },
};
```

## ✅ Preparation Strategy

### Technical Question Categories
- [ ] **JavaScript Fundamentals**: Closures, prototypes, async/await
- [ ] **React Concepts**: Hooks, lifecycle, performance
- [ ] **Node.js Core**: Event loop, streams, clustering
- [ ] **Database Design**: Normalization, indexing, queries
- [ ] **System Architecture**: Scaling, caching, security
- [ ] **Performance**: Optimization techniques, monitoring
- [ ] **Testing**: Unit, integration, E2E strategies
- [ ] **Security**: Authentication, authorization, best practices

### Practice Schedule
- **Daily (30 minutes)**: Review 3-5 technical questions
- **Weekly (2 hours)**: Deep dive into one complex topic
- **Bi-weekly**: Mock technical interview session

### Answer Structure
1. **Direct Answer**: Address the question immediately
2. **Explanation**: Provide technical details
3. **Code Example**: Show practical implementation
4. **Trade-offs**: Discuss alternatives and considerations
5. **Follow-up**: Ask clarifying questions or suggest improvements

**Target**: Master technical concepts and articulate solutions clearly for confident technical discussions