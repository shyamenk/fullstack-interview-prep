# Quick References & Cheat Sheets

## 🚀 Frontend Quick Reference

### React Hooks Cheat Sheet
```javascript
// useState - State management
const [state, setState] = useState(initialValue);

// useEffect - Side effects
useEffect(() => {
  // Effect logic
  return () => {
    // Cleanup
  };
}, [dependencies]);

// useContext - Context consumption
const value = useContext(MyContext);

// useReducer - Complex state management
const [state, dispatch] = useReducer(reducer, initialState);

// useMemo - Expensive calculations
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);

// useCallback - Function memoization
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);

// useRef - DOM access and mutable values
const ref = useRef(initialValue);

// Custom Hook Pattern
function useCustomHook(initialValue) {
  const [value, setValue] = useState(initialValue);

  const updateValue = useCallback((newValue) => {
    setValue(newValue);
  }, []);

  return [value, updateValue];
}
```

### Next.js Key Functions
```javascript
// getServerSideProps - SSR
export async function getServerSideProps(context) {
  const data = await fetchData();
  return { props: { data } };
}

// getStaticProps - SSG
export async function getStaticProps() {
  const data = await fetchData();
  return {
    props: { data },
    revalidate: 60 // ISR
  };
}

// getStaticPaths - Dynamic SSG
export async function getStaticPaths() {
  const paths = await getPaths();
  return {
    paths,
    fallback: 'blocking'
  };
}

// API Route
export default function handler(req, res) {
  if (req.method === 'POST') {
    // Handle POST
    res.status(200).json({ success: true });
  }
}

// Middleware (Next.js 12+)
export function middleware(request) {
  // Logic
  return NextResponse.redirect(new URL('/login', request.url));
}
```

## 🔧 Backend Quick Reference

### Express.js Essentials
```javascript
// Basic Express setup
const express = require('express');
const app = express();

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
app.get('/api/users', (req, res) => {
  res.json({ users: [] });
});

app.post('/api/users', (req, res) => {
  const user = req.body;
  // Process user
  res.status(201).json({ user });
});

// Error handling middleware
app.use((err, req, res, next) => {
  res.status(err.status || 500).json({
    message: err.message,
    error: process.env.NODE_ENV === 'development' ? err : {}
  });
});

// Route parameters
app.get('/api/users/:id', (req, res) => {
  const { id } = req.params;
  // Find user by id
});

// Query parameters
app.get('/api/posts', (req, res) => {
  const { page = 1, limit = 10 } = req.query;
  // Paginated results
});
```

### Node.js Core Patterns
```javascript
// Async/Await with Error Handling
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

// EventEmitter Pattern
const EventEmitter = require('events');
class MyEmitter extends EventEmitter {}
const myEmitter = new MyEmitter();

// Stream Processing
const { Transform } = require('stream');
class MyTransform extends Transform {
  _transform(chunk, encoding, callback) {
    const data = chunk.toString().toUpperCase();
    callback(null, data);
  }
}

// File Operations
const fs = require('fs').promises;
await fs.readFile('file.txt', 'utf8');
await fs.writeFile('output.txt', data);
```

### FastAPI Essentials
```python
from fastapi import FastAPI, HTTPException, Depends
from pydantic import BaseModel

app = FastAPI()

# Pydantic models
class User(BaseModel):
    name: str
    email: str
    age: int

# Basic routes
@app.get("/")
async def root():
    return {"message": "Hello World"}

@app.post("/users/")
async def create_user(user: User):
    return {"user": user}

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    if user_id not in users_db:
        raise HTTPException(status_code=404, detail="User not found")
    return users_db[user_id]

# Dependency injection
async def get_db():
    db = DatabaseSession()
    try:
        yield db
    finally:
        db.close()

@app.get("/items/")
async def read_items(db: Session = Depends(get_db)):
    return db.query(Item).all()
```

## 🗄️ Database Quick Reference

### PostgreSQL Common Queries
```sql
-- Basic CRUD operations
SELECT * FROM users WHERE age > 18;
INSERT INTO users (name, email) VALUES ('John', 'john@example.com');
UPDATE users SET age = 25 WHERE id = 1;
DELETE FROM users WHERE id = 1;

-- Joins
SELECT u.name, p.title
FROM users u
JOIN posts p ON u.id = p.user_id;

-- Aggregations
SELECT COUNT(*), AVG(age), MAX(created_at) FROM users;

-- Window functions
SELECT name, age,
       RANK() OVER (ORDER BY age DESC) as age_rank
FROM users;

-- Indexing
CREATE INDEX idx_user_email ON users(email);
CREATE INDEX idx_post_user_id ON posts(user_id);

-- Performance optimization
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'john@example.com';
```

### MongoDB Operations
```javascript
// Basic operations
db.users.find({ age: { $gt: 18 } });
db.users.insertOne({ name: "John", email: "john@example.com" });
db.users.updateOne({ _id: ObjectId("...") }, { $set: { age: 25 } });
db.users.deleteOne({ _id: ObjectId("...") });

// Aggregation pipeline
db.users.aggregate([
  { $match: { age: { $gte: 18 } } },
  { $group: { _id: "$department", avgAge: { $avg: "$age" } } },
  { $sort: { avgAge: -1 } }
]);

// Indexing
db.users.createIndex({ email: 1 });
db.users.createIndex({ name: "text" });

// Mongoose schemas
const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, unique: true },
  age: { type: Number, min: 0 }
});
```

### DynamoDB Operations
```javascript
// AWS SDK v3
import { DynamoDBClient, PutItemCommand, GetItemCommand } from "@aws-sdk/client-dynamodb";

const client = new DynamoDBClient({ region: "us-east-1" });

// Put item
const putCommand = new PutItemCommand({
  TableName: "Users",
  Item: {
    userId: { S: "123" },
    name: { S: "John Doe" },
    email: { S: "john@example.com" }
  }
});

// Get item
const getCommand = new GetItemCommand({
  TableName: "Users",
  Key: {
    userId: { S: "123" }
  }
});

// Query with GSI
const queryCommand = new QueryCommand({
  TableName: "Users",
  IndexName: "EmailIndex",
  KeyConditionExpression: "email = :email",
  ExpressionAttributeValues: {
    ":email": { S: "john@example.com" }
  }
});
```

## ☁️ AWS Quick Reference

### Essential AWS CLI Commands
```bash
# S3 operations
aws s3 ls s3://bucket-name/
aws s3 cp file.txt s3://bucket-name/
aws s3 sync ./dist s3://bucket-name/ --delete

# Lambda operations
aws lambda create-function --function-name myFunction
aws lambda invoke --function-name myFunction response.json
aws lambda update-function-code --function-name myFunction --zip-file fileb://function.zip

# EC2 operations
aws ec2 describe-instances
aws ec2 start-instances --instance-ids i-1234567890abcdef0
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# RDS operations
aws rds describe-db-instances
aws rds create-db-snapshot --db-instance-identifier mydb --db-snapshot-identifier mydb-snapshot
```

### AWS SDK (Node.js) Examples
```javascript
// S3 operations
const { S3Client, PutObjectCommand } = require("@aws-sdk/client-s3");
const s3 = new S3Client({ region: "us-east-1" });

const uploadParams = {
  Bucket: "my-bucket",
  Key: "file.txt",
  Body: "Hello World"
};

await s3.send(new PutObjectCommand(uploadParams));

// Lambda operations
const { LambdaClient, InvokeCommand } = require("@aws-sdk/client-lambda");
const lambda = new LambdaClient({ region: "us-east-1" });

const invokeParams = {
  FunctionName: "myFunction",
  Payload: JSON.stringify({ key: "value" })
};

await lambda.send(new InvokeCommand(invokeParams));
```

## 🧪 Testing Quick Reference

### Jest Testing Patterns
```javascript
// Basic test structure
describe('Calculator', () => {
  test('should add two numbers', () => {
    expect(add(2, 3)).toBe(5);
  });

  test('should handle edge cases', () => {
    expect(add(0, 0)).toBe(0);
    expect(add(-1, 1)).toBe(0);
  });
});

// Async testing
test('async data fetch', async () => {
  const data = await fetchData();
  expect(data).toHaveProperty('id');
});

// Mocking
jest.mock('./api');
const mockApi = require('./api');
mockApi.fetchUser.mockResolvedValue({ id: 1, name: 'John' });

// React Testing Library
import { render, screen, fireEvent } from '@testing-library/react';

test('button click increments counter', () => {
  render(<Counter />);
  const button = screen.getByRole('button');
  fireEvent.click(button);
  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});
```

### API Testing with Supertest
```javascript
const request = require('supertest');
const app = require('../app');

describe('GET /api/users', () => {
  test('should return all users', async () => {
    const response = await request(app)
      .get('/api/users')
      .expect(200)
      .expect('Content-Type', /json/);

    expect(response.body).toHaveProperty('users');
    expect(Array.isArray(response.body.users)).toBe(true);
  });
});

describe('POST /api/users', () => {
  test('should create a new user', async () => {
    const userData = {
      name: 'John Doe',
      email: 'john@example.com'
    };

    const response = await request(app)
      .post('/api/users')
      .send(userData)
      .expect(201);

    expect(response.body.user).toMatchObject(userData);
  });
});
```

## 🔒 Security Cheat Sheet

### Input Validation & Sanitization
```javascript
// Express-validator
const { body, validationResult } = require('express-validator');

const validateUser = [
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 8 }),
  (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    next();
  }
];

// Helmet.js security headers
const helmet = require('helmet');
app.use(helmet());

// Rate limiting
const rateLimit = require('express-rate-limit');
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});
app.use(limiter);
```

### JWT Authentication
```javascript
const jwt = require('jsonwebtoken');

// Generate token
const token = jwt.sign(
  { userId: user.id, email: user.email },
  process.env.JWT_SECRET,
  { expiresIn: '24h' }
);

// Verify token middleware
const authenticate = (req, res, next) => {
  const token = req.header('Authorization')?.replace('Bearer ', '');

  if (!token) {
    return res.status(401).json({ error: 'Access denied' });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
};
```

## 📊 Performance Optimization

### Frontend Performance Tips
```javascript
// Code splitting with React.lazy
const LazyComponent = React.lazy(() => import('./LazyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </Suspense>
  );
}

// Memoization
const MemoizedComponent = React.memo(({ data }) => {
  return <div>{data.title}</div>;
});

const memoizedValue = useMemo(() => {
  return expensiveCalculation(data);
}, [data]);

// Virtual scrolling pattern
const VirtualList = ({ items, itemHeight, containerHeight }) => {
  const [scrollTop, setScrollTop] = useState(0);

  const startIndex = Math.floor(scrollTop / itemHeight);
  const endIndex = Math.min(
    startIndex + Math.ceil(containerHeight / itemHeight),
    items.length - 1
  );

  const visibleItems = items.slice(startIndex, endIndex + 1);

  return (
    <div
      style={{ height: containerHeight, overflowY: 'auto' }}
      onScroll={(e) => setScrollTop(e.target.scrollTop)}
    >
      <div style={{ height: items.length * itemHeight }}>
        <div style={{ transform: `translateY(${startIndex * itemHeight}px)` }}>
          {visibleItems.map((item, index) => (
            <div key={startIndex + index} style={{ height: itemHeight }}>
              {item.content}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
};
```

### Backend Performance Tips
```javascript
// Database connection pooling
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// Redis caching
const redis = require('redis');
const client = redis.createClient();

const cacheMiddleware = (duration = 300) => {
  return async (req, res, next) => {
    const key = req.originalUrl;
    const cached = await client.get(key);

    if (cached) {
      return res.json(JSON.parse(cached));
    }

    res.sendResponse = res.json;
    res.json = (body) => {
      client.setex(key, duration, JSON.stringify(body));
      res.sendResponse(body);
    };

    next();
  };
};

// Compression
const compression = require('compression');
app.use(compression());

// Clustering
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
} else {
  app.listen(3000);
}
```

## 📱 Git Commands Quick Reference

```bash
# Basic commands
git clone <repository>
git add .
git commit -m "message"
git push origin main
git pull origin main

# Branching
git branch feature-branch
git checkout feature-branch
git checkout -b feature-branch  # create and switch
git merge feature-branch
git branch -d feature-branch

# Advanced
git rebase main
git cherry-pick <commit-hash>
git stash
git stash pop
git reset --hard HEAD~1
git revert <commit-hash>

# Useful aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.cm commit
```

## 🚀 Deployment Cheat Sheet

### Docker Commands
```bash
# Build and run
docker build -t app-name .
docker run -p 3000:3000 app-name
docker run -d --name my-app -p 3000:3000 app-name

# Management
docker ps
docker stop container-id
docker rm container-id
docker images
docker rmi image-id

# Docker Compose
docker-compose up
docker-compose up -d
docker-compose down
docker-compose logs
```

### Environment Variables
```bash
# Node.js
NODE_ENV=production
PORT=3000
DATABASE_URL=postgresql://user:pass@localhost:5432/db
JWT_SECRET=your-secret-key
REDIS_URL=redis://localhost:6379

# AWS
AWS_ACCESS_KEY_ID=your-key
AWS_SECRET_ACCESS_KEY=your-secret
AWS_REGION=us-east-1

# Database
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=myapp
POSTGRES_USER=user
POSTGRES_PASSWORD=password
```

This quick reference serves as a handy lookup guide during development and interviews. Keep it accessible for rapid consultation of syntax, patterns, and best practices.