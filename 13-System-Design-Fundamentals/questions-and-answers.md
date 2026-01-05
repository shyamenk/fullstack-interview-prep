# System Design Fundamentals Interview Questions & Answers

## Table of Contents

1. [How do you design a REST API and what are the best practices?](#q1-how-do-you-design-a-rest-api-and-what-are-the-best-practices)
2. [How do you secure REST APIs?](#q2-how-do-you-secure-rest-apis)
3. [How do you implement pagination in APIs?](#q3-how-do-you-implement-pagination-in-apis)
4. [What is the difference between Offset-based and Cursor-based pagination?](#q4-what-is-the-difference-between-offset-based-and-cursor-based-pagination)
5. [How do you implement rate limiting and what algorithms are used?](#q5-how-do-you-implement-rate-limiting-and-what-algorithms-are-used)
6. [How do you implement caching in a Next.js PostgreSQL app?](#q6-how-do-you-implement-caching-in-a-nextjs-postgresql-app)
7. [How would you design a scalable notification system?](#q7-how-would-you-design-a-scalable-notification-system)
8. [REST vs GraphQL - when to use each?](#q8-rest-vs-graphql---when-to-use-each)
9. [Why is API versioning needed and what are its advantages?](#q9-why-is-api-versioning-needed-and-what-are-its-advantages)
10. [What is idempotency and why is it critical for APIs?](#q10-what-is-idempotency-and-why-is-it-critical-for-apis)
11. [How do you design an idempotent payment API?](#q11-how-do-you-design-an-idempotent-payment-api)

---

## Q1. How do you design a REST API and what are the best practices?

### Concept:

REST (Representational State Transfer) is an architectural style for designing APIs using HTTP methods to interact with resources.

### Core Principles:

1. **Resources** - Everything is a resource (users, orders, products)
2. **URIs** - Unique identifiers for resources
3. **HTTP Methods** - Standard actions (GET, POST, PUT, DELETE)
4. **Stateless** - Each request contains all needed information
5. **Client-Server** - Separation of concerns
6. **Representation** - Resources returned as JSON/XML

### HTTP Methods:

| Method | Purpose | Example | Idempotent |
|--------|---------|---------|------------|
| GET | Retrieve resource(s) | `GET /users` | ✅ Yes |
| POST | Create new resource | `POST /users` | ❌ No |
| PUT | Replace entire resource | `PUT /users/123` | ✅ Yes |
| PATCH | Partial update | `PATCH /users/123` | ❌ No |
| DELETE | Remove resource | `DELETE /users/123` | ✅ Yes |

### Best Practices:

#### 1. **Use Nouns, Not Verbs**

**Bad:**
```
GET  /getUsers
POST /createUser
GET  /getUserById/123
```

**Good:**
```
GET    /users          # Get all users
POST   /users          # Create a user
GET    /users/123      # Get user by ID
PUT    /users/123      # Update user
DELETE /users/123      # Delete user
```

#### 2. **Use Plural Nouns**

**Bad:**
```
GET /user/123
GET /order/456
```

**Good:**
```
GET /users/123
GET /orders/456
```

#### 3. **Use Proper HTTP Status Codes**

| Code | Meaning | Use Case |
|------|---------|----------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Missing/invalid auth |
| 403 | Forbidden | Authenticated but no permission |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate resource |
| 500 | Server Error | Internal error |

**Example:**
```typescript
// Create user
app.post("/users", async (req, res) => {
  const user = await createUser(req.body);
  res.status(201).json(user);  // ✅ 201 Created
});

// User not found
app.get("/users/:id", async (req, res) => {
  const user = await findUser(req.params.id);
  if (!user) {
    return res.status(404).json({ error: "User not found" });  // ✅ 404
  }
  res.status(200).json(user);  // ✅ 200 OK
});
```

#### 4. **Nested Resources**

**Good structure:**
```
GET    /users/123/orders           # Get user's orders
POST   /users/123/orders           # Create order for user
GET    /users/123/orders/456       # Get specific order
DELETE /users/123/orders/456       # Delete order
```

**Limit nesting to 2 levels:**
```
# ❌ Too deep
GET /users/123/orders/456/items/789/reviews

# ✅ Better
GET /order-items/789/reviews
```

#### 5. **Filtering, Sorting, Pagination**

**Filtering:**
```
GET /users?role=admin
GET /users?status=active&country=US
GET /products?price[gte]=100&price[lte]=500
```

**Sorting:**
```
GET /users?sort=createdAt          # Ascending
GET /users?sort=-createdAt         # Descending (- prefix)
GET /users?sort=name,-createdAt    # Multiple fields
```

**Pagination:**
```
GET /users?page=2&limit=20
GET /users?offset=40&limit=20
```

**Implementation:**
```typescript
app.get("/users", async (req, res) => {
  const {
    role,
    status,
    sort = "-createdAt",
    page = 1,
    limit = 20
  } = req.query;

  const filters = {};
  if (role) filters.role = role;
  if (status) filters.status = status;

  const users = await db.query(`
    SELECT * FROM users
    WHERE role = ? AND status = ?
    ORDER BY ${sort}
    LIMIT ? OFFSET ?
  `, [role, status, limit, (page - 1) * limit]);

  res.json({
    data: users,
    pagination: {
      page: parseInt(page),
      limit: parseInt(limit),
      total: await db.count("users", filters)
    }
  });
});
```

#### 6. **Versioning**

**Option 1: URI versioning (most common):**
```
GET /v1/users
GET /v2/users
```

**Option 2: Header versioning:**
```
GET /users
Accept: application/vnd.api.v1+json
```

**Implementation:**
```typescript
// URI versioning
app.use("/v1", v1Routes);
app.use("/v2", v2Routes);

// v1 routes
const v1Router = express.Router();
v1Router.get("/users", getUsersV1);

// v2 routes
const v2Router = express.Router();
v2Router.get("/users", getUsersV2);
```

#### 7. **Consistent Response Format**

**Success response:**
```json
{
  "data": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com"
  },
  "meta": {
    "timestamp": "2024-01-01T10:00:00Z"
  }
}
```

**Error response:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email"
      }
    ]
  }
}
```

**Implementation:**
```typescript
// Success wrapper
function successResponse(data, meta = {}) {
  return {
    data,
    meta: {
      timestamp: new Date().toISOString(),
      ...meta
    }
  };
}

// Error wrapper
function errorResponse(code, message, details = []) {
  return {
    error: {
      code,
      message,
      details
    }
  };
}

// Usage
app.get("/users/:id", async (req, res) => {
  const user = await findUser(req.params.id);
  if (!user) {
    return res.status(404).json(
      errorResponse("NOT_FOUND", "User not found")
    );
  }
  res.json(successResponse(user));
});
```

#### 8. **HATEOAS (Hypermedia)**

Include links to related resources:

```json
{
  "data": {
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com"
  },
  "links": {
    "self": "/users/123",
    "orders": "/users/123/orders",
    "profile": "/users/123/profile"
  }
}
```

#### 9. **Idempotency**

**Idempotent methods** produce the same result when called multiple times:

```typescript
// GET - Always returns same data (idempotent)
GET /users/123

// PUT - Replacing entire resource (idempotent)
PUT /users/123
{ "name": "John", "email": "john@example.com" }

// DELETE - Deleting same resource (idempotent)
DELETE /users/123  // First call: deletes user
DELETE /users/123  // Second call: 404, but no side effects

// POST - Creates new resource each time (NOT idempotent)
POST /users
{ "name": "John" }  // Creates user with ID 1
POST /users
{ "name": "John" }  // Creates user with ID 2
```

**Idempotency keys for POST:**
```typescript
app.post("/orders", async (req, res) => {
  const idempotencyKey = req.headers["idempotency-key"];

  // Check if request with this key was already processed
  const existing = await redis.get(`idempotency:${idempotencyKey}`);
  if (existing) {
    return res.status(200).json(JSON.parse(existing));
  }

  // Process new request
  const order = await createOrder(req.body);

  // Cache result with TTL
  await redis.setex(
    `idempotency:${idempotencyKey}`,
    86400,  // 24 hours
    JSON.stringify(order)
  );

  res.status(201).json(order);
});
```

#### 10. **Rate Limiting**

```typescript
import rateLimit from "express-rate-limit";

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,  // Limit each IP to 100 requests per window
  message: "Too many requests, please try again later"
});

app.use("/api/", limiter);

// Response headers
res.headers["X-RateLimit-Limit"] = "100";
res.headers["X-RateLimit-Remaining"] = "87";
res.headers["X-RateLimit-Reset"] = "1640000000";
```

#### 11. **Documentation**

Use OpenAPI/Swagger:

```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0

paths:
  /users:
    get:
      summary: Get all users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
        - name: limit
          in: query
          schema:
            type: integer
      responses:
        200:
          description: Successful response
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
```

#### 12. **Security Best Practices**

```typescript
// HTTPS only
if (process.env.NODE_ENV === "production" && req.protocol !== "https") {
  return res.redirect(`https://${req.hostname}${req.url}`);
}

// Authentication
app.use("/api", authenticate);

// CORS
app.use(cors({
  origin: ["https://example.com"],
  credentials: true
}));

// Input validation
app.post("/users", validate(userSchema), createUser);

// Helmet for security headers
app.use(helmet());
```

### Complete Example:

```typescript
import express from "express";
import { z } from "zod";

const app = express();
app.use(express.json());

// Validation schema
const userSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  role: z.enum(["user", "admin"])
});

// GET all users with filtering, sorting, pagination
app.get("/api/v1/users", async (req, res) => {
  const {
    role,
    status,
    sort = "-createdAt",
    page = 1,
    limit = 20
  } = req.query;

  const users = await db.users.findMany({
    where: { role, status },
    orderBy: parseSort(sort),
    skip: (page - 1) * limit,
    take: limit
  });

  const total = await db.users.count({ where: { role, status } });

  res.json({
    data: users,
    pagination: {
      page: parseInt(page),
      limit: parseInt(limit),
      total,
      pages: Math.ceil(total / limit)
    },
    links: {
      self: `/api/v1/users?page=${page}`,
      next: page < Math.ceil(total / limit) ? `/api/v1/users?page=${parseInt(page) + 1}` : null,
      prev: page > 1 ? `/api/v1/users?page=${parseInt(page) - 1}` : null
    }
  });
});

// GET single user
app.get("/api/v1/users/:id", async (req, res) => {
  const user = await db.users.findUnique({
    where: { id: req.params.id }
  });

  if (!user) {
    return res.status(404).json({
      error: {
        code: "NOT_FOUND",
        message: "User not found"
      }
    });
  }

  res.json({
    data: user,
    links: {
      self: `/api/v1/users/${user.id}`,
      orders: `/api/v1/users/${user.id}/orders`
    }
  });
});

// POST create user
app.post("/api/v1/users", async (req, res) => {
  const validation = userSchema.safeParse(req.body);

  if (!validation.success) {
    return res.status(400).json({
      error: {
        code: "VALIDATION_ERROR",
        message: "Invalid input",
        details: validation.error.errors
      }
    });
  }

  const user = await db.users.create({
    data: validation.data
  });

  res.status(201).json({
    data: user,
    links: {
      self: `/api/v1/users/${user.id}`
    }
  });
});

// PUT update user
app.put("/api/v1/users/:id", async (req, res) => {
  const validation = userSchema.safeParse(req.body);

  if (!validation.success) {
    return res.status(400).json({
      error: {
        code: "VALIDATION_ERROR",
        message: "Invalid input",
        details: validation.error.errors
      }
    });
  }

  const user = await db.users.update({
    where: { id: req.params.id },
    data: validation.data
  });

  res.json({ data: user });
});

// DELETE user
app.delete("/api/v1/users/:id", async (req, res) => {
  await db.users.delete({
    where: { id: req.params.id }
  });

  res.status(204).send();  // No content
});

// Nested resource
app.get("/api/v1/users/:userId/orders", async (req, res) => {
  const orders = await db.orders.findMany({
    where: { userId: req.params.userId }
  });

  res.json({ data: orders });
});
```

### REST API Checklist:

- [ ] Use nouns for resources, not verbs
- [ ] Use plural nouns (`/users`, not `/user`)
- [ ] Use proper HTTP methods (GET, POST, PUT, DELETE)
- [ ] Return correct HTTP status codes
- [ ] Implement filtering, sorting, pagination
- [ ] Version your API (`/v1/users`)
- [ ] Use consistent response format
- [ ] Validate input data
- [ ] Implement rate limiting
- [ ] Add authentication and authorization
- [ ] Use HTTPS in production
- [ ] Document API (OpenAPI/Swagger)
- [ ] Handle errors consistently
- [ ] Support CORS for browser clients
- [ ] Implement idempotency for critical operations

### Common Mistakes to Avoid:

| Mistake | Correct Approach |
|---------|-----------------|
| `GET /getUser/123` | `GET /users/123` |
| Return 200 for errors | Return appropriate status (400, 404, 500) |
| No input validation | Validate all inputs |
| Expose internal errors | Return generic errors to client |
| No pagination | Always paginate large lists |
| Inconsistent naming | Use consistent casing (camelCase or snake_case) |
| No versioning | Version from day 1 |

### Analogy:

Think of a REST API like a **restaurant menu**:
- **Resources** = Dishes (burgers, pizzas, drinks)
- **HTTP Methods** = Actions (view menu, order, modify order, cancel)
- **URIs** = Menu sections (`/appetizers`, `/main-courses`)
- **Status Codes** = Waiter responses (order confirmed, dish unavailable, kitchen error)
- **Pagination** = Menu pages (page 1: appetizers, page 2: mains)

### ⚡ One-liner to remember:
> REST API = Nouns for resources, HTTP methods for actions, proper status codes, and consistent structure.

---

## Q2. How do you secure REST APIs?

### Concept:

Securing REST APIs involves multiple layers of protection: authentication, authorization, encryption, input validation, and rate limiting.

### Security Layers:

```
┌─────────────────────────────────────┐
│ 1. HTTPS/TLS (Encryption)           │
├─────────────────────────────────────┤
│ 2. Authentication (Who are you?)    │
├─────────────────────────────────────┤
│ 3. Authorization (What can you do?) │
├─────────────────────────────────────┤
│ 4. Input Validation                 │
├─────────────────────────────────────┤
│ 5. Rate Limiting                    │
├─────────────────────────────────────┤
│ 6. Security Headers                 │
└─────────────────────────────────────┘
```

### 1. **HTTPS/TLS (Encryption in Transit)**

**Always use HTTPS in production:**

```typescript
import https from "https";
import fs from "fs";

const options = {
  key: fs.readFileSync("private-key.pem"),
  cert: fs.readFileSync("certificate.pem")
};

https.createServer(options, app).listen(443);

// Redirect HTTP to HTTPS
app.use((req, res, next) => {
  if (req.protocol !== "https" && process.env.NODE_ENV === "production") {
    return res.redirect(`https://${req.hostname}${req.url}`);
  }
  next();
});
```

### 2. **Authentication**

#### Option A: JWT (Stateless)

```typescript
import jwt from "jsonwebtoken";

// Login endpoint
app.post("/api/login", async (req, res) => {
  const { email, password } = req.body;

  const user = await db.users.findOne({ email });
  if (!user || !(await bcrypt.compare(password, user.passwordHash))) {
    return res.status(401).json({ error: "Invalid credentials" });
  }

  const token = jwt.sign(
    { userId: user.id, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: "15m" }
  );

  res.json({ token });
});

// Authentication middleware
function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(" ")[1];

  if (!token) {
    return res.status(401).json({ error: "No token provided" });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET, {
      algorithms: ["HS256"]
    });
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: "Invalid token" });
  }
}

// Protected route
app.get("/api/profile", authenticate, (req, res) => {
  res.json({ user: req.user });
});
```

#### Option B: API Keys

```typescript
// Store API keys hashed in database
app.use("/api", async (req, res, next) => {
  const apiKey = req.headers["x-api-key"];

  if (!apiKey) {
    return res.status(401).json({ error: "API key required" });
  }

  const hashedKey = crypto.createHash("sha256").update(apiKey).digest("hex");
  const client = await db.apiKeys.findOne({ key: hashedKey, active: true });

  if (!client) {
    return res.status(401).json({ error: "Invalid API key" });
  }

  req.client = client;
  next();
});
```

#### Option C: OAuth 2.0

```typescript
import passport from "passport";
import { Strategy as GoogleStrategy } from "passport-google-oauth20";

passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  callbackURL: "/auth/google/callback"
}, async (accessToken, refreshToken, profile, done) => {
  const user = await db.users.findOrCreate({ googleId: profile.id });
  done(null, user);
}));

app.get("/auth/google", passport.authenticate("google", {
  scope: ["profile", "email"]
}));

app.get("/auth/google/callback",
  passport.authenticate("google", { session: false }),
  (req, res) => {
    const token = jwt.sign({ userId: req.user.id }, process.env.JWT_SECRET);
    res.redirect(`/dashboard?token=${token}`);
  }
);
```

### 3. **Authorization (RBAC)**

```typescript
// Role-based access control
const permissions = {
  admin: ["read", "write", "delete"],
  user: ["read", "write"],
  guest: ["read"]
};

function authorize(requiredPermission: string) {
  return (req, res, next) => {
    const userPermissions = permissions[req.user.role] || [];

    if (!userPermissions.includes(requiredPermission)) {
      return res.status(403).json({ error: "Forbidden" });
    }

    next();
  };
}

// Usage
app.delete("/api/users/:id",
  authenticate,
  authorize("delete"),
  deleteUser
);

// Resource ownership check
app.get("/api/posts/:id", authenticate, async (req, res) => {
  const post = await db.posts.findOne({ id: req.params.id });

  if (!post) {
    return res.status(404).json({ error: "Not found" });
  }

  // Check if user owns the resource or is admin
  if (post.authorId !== req.user.id && req.user.role !== "admin") {
    return res.status(403).json({ error: "Forbidden" });
  }

  res.json({ data: post });
});
```

### 4. **Input Validation & Sanitization**

```typescript
import { z } from "zod";
import DOMPurify from "dompurify";

// Schema validation
const createUserSchema = z.object({
  name: z.string().min(2).max(50),
  email: z.string().email(),
  age: z.number().min(18).max(120),
  role: z.enum(["user", "admin"])
});

// Validation middleware
function validate(schema: z.ZodSchema) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);

    if (!result.success) {
      return res.status(400).json({
        error: "Validation failed",
        details: result.error.errors
      });
    }

    req.body = result.data;  // Use validated data
    next();
  };
}

// Sanitization
app.post("/api/posts", authenticate, validate(postSchema), async (req, res) => {
  const sanitized = {
    title: DOMPurify.sanitize(req.body.title),
    content: DOMPurify.sanitize(req.body.content)
  };

  const post = await db.posts.create(sanitized);
  res.status(201).json({ data: post });
});

// SQL Injection prevention - use parameterized queries
// ❌ Vulnerable
const user = await db.query(`SELECT * FROM users WHERE email = '${email}'`);

// ✅ Safe
const user = await db.query("SELECT * FROM users WHERE email = ?", [email]);
```

### 5. **Rate Limiting**

```typescript
import rateLimit from "express-rate-limit";
import RedisStore from "rate-limit-redis";
import Redis from "ioredis";

const redis = new Redis();

// General rate limit
const generalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,  // 100 requests per window
  message: "Too many requests from this IP",
  standardHeaders: true,
  legacyHeaders: false,
  store: new RedisStore({
    client: redis,
    prefix: "rl:"
  })
});

// Stricter limit for auth endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,  // 5 login attempts per 15 minutes
  skipSuccessfulRequests: true,
  message: "Too many login attempts"
});

// Apply rate limiting
app.use("/api/", generalLimiter);
app.use("/api/login", authLimiter);

// Per-user rate limiting
async function userRateLimit(req, res, next) {
  const key = `rate:${req.user.id}`;
  const requests = await redis.incr(key);

  if (requests === 1) {
    await redis.expire(key, 60);  // 60 seconds window
  }

  if (requests > 100) {
    return res.status(429).json({ error: "Rate limit exceeded" });
  }

  next();
}
```

### 6. **CORS (Cross-Origin Resource Sharing)**

```typescript
import cors from "cors";

// Development - allow all origins
app.use(cors());

// Production - whitelist specific origins
const corsOptions = {
  origin: (origin, callback) => {
    const allowedOrigins = [
      "https://example.com",
      "https://app.example.com"
    ];

    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error("Not allowed by CORS"));
    }
  },
  credentials: true,  // Allow cookies
  optionsSuccessStatus: 200
};

app.use(cors(corsOptions));

// Handle preflight requests
app.options("*", cors(corsOptions));
```

### 7. **Security Headers**

```typescript
import helmet from "helmet";

// Use helmet for security headers
app.use(helmet());

// Custom security headers
app.use((req, res, next) => {
  // Prevent clickjacking
  res.setHeader("X-Frame-Options", "DENY");

  // Prevent MIME sniffing
  res.setHeader("X-Content-Type-Options", "nosniff");

  // XSS protection
  res.setHeader("X-XSS-Protection", "1; mode=block");

  // Content Security Policy
  res.setHeader(
    "Content-Security-Policy",
    "default-src 'self'; script-src 'self' 'unsafe-inline'"
  );

  // HSTS - force HTTPS
  res.setHeader(
    "Strict-Transport-Security",
    "max-age=31536000; includeSubDomains"
  );

  next();
});
```

### 8. **Prevent Common Attacks**

#### XSS (Cross-Site Scripting)

```typescript
// Sanitize user input
import DOMPurify from "dompurify";

app.post("/api/comments", async (req, res) => {
  const clean = DOMPurify.sanitize(req.body.comment);
  await db.comments.create({ content: clean });
  res.status(201).send();
});

// Escape output
res.json({
  message: escapeHtml(userInput)
});
```

#### CSRF (Cross-Site Request Forgery)

```typescript
import csurf from "csurf";

const csrfProtection = csurf({
  cookie: {
    httpOnly: true,
    secure: true,
    sameSite: "strict"
  }
});

app.use(csrfProtection);

// Send CSRF token to client
app.get("/api/csrf-token", (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

// Verify CSRF token on state-changing requests
app.post("/api/transfer", csrfProtection, (req, res) => {
  // Token automatically verified by middleware
  // Process transfer...
});
```

#### SQL Injection

```typescript
// ❌ Vulnerable
const query = `SELECT * FROM users WHERE id = ${userId}`;

// ✅ Safe - parameterized queries
const query = "SELECT * FROM users WHERE id = ?";
const user = await db.query(query, [userId]);

// ✅ Safe - ORM
const user = await db.users.findOne({ where: { id: userId } });
```

#### NoSQL Injection

```typescript
// ❌ Vulnerable
const user = await db.users.findOne({ email: req.body.email });

// Attacker sends: { "email": { "$gt": "" } } - returns all users

// ✅ Safe - validate input type
if (typeof req.body.email !== "string") {
  return res.status(400).json({ error: "Invalid email" });
}

const user = await db.users.findOne({ email: req.body.email });
```

### 9. **Logging & Monitoring**

```typescript
import winston from "winston";

const logger = winston.createLogger({
  level: "info",
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: "error.log", level: "error" }),
    new winston.transports.File({ filename: "combined.log" })
  ]
});

// Log authentication attempts
app.post("/api/login", async (req, res) => {
  const { email, password } = req.body;

  logger.info("Login attempt", {
    email,
    ip: req.ip,
    userAgent: req.headers["user-agent"]
  });

  const user = await authenticateUser(email, password);

  if (!user) {
    logger.warn("Failed login", { email, ip: req.ip });
    return res.status(401).json({ error: "Invalid credentials" });
  }

  logger.info("Successful login", { userId: user.id, ip: req.ip });
  res.json({ token: generateToken(user) });
});

// Log suspicious activity
app.use((req, res, next) => {
  if (req.path.includes("../") || req.path.includes("..\\")) {
    logger.warn("Path traversal attempt", {
      path: req.path,
      ip: req.ip
    });
    return res.status(400).json({ error: "Invalid request" });
  }
  next();
});
```

### 10. **Secrets Management**

```typescript
// ❌ Bad - hardcoded secrets
const secret = "my-secret-key";

// ✅ Good - environment variables
const secret = process.env.JWT_SECRET;

// ✅ Better - secrets manager (AWS Secrets Manager)
import { SecretsManagerClient, GetSecretValueCommand } from "@aws-sdk/client-secrets-manager";

const client = new SecretsManagerClient({ region: "us-east-1" });

async function getSecret(secretName: string) {
  const command = new GetSecretValueCommand({ SecretId: secretName });
  const response = await client.send(command);
  return JSON.parse(response.SecretString);
}

const dbCredentials = await getSecret("prod/db/credentials");
```

### Complete Secure API Example:

```typescript
import express from "express";
import helmet from "helmet";
import cors from "cors";
import rateLimit from "express-rate-limit";
import { z } from "zod";
import jwt from "jsonwebtoken";

const app = express();

// 1. Security headers
app.use(helmet());

// 2. CORS
app.use(cors({
  origin: ["https://example.com"],
  credentials: true
}));

// 3. Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100
});
app.use("/api/", limiter);

// 4. Body parsing with size limit
app.use(express.json({ limit: "10kb" }));

// 5. HTTPS redirect
app.use((req, res, next) => {
  if (req.protocol !== "https" && process.env.NODE_ENV === "production") {
    return res.redirect(`https://${req.hostname}${req.url}`);
  }
  next();
});

// 6. Authentication middleware
function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(" ")[1];

  if (!token) {
    return res.status(401).json({ error: "No token" });
  }

  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET, {
      algorithms: ["HS256"]
    });
    next();
  } catch {
    return res.status(401).json({ error: "Invalid token" });
  }
}

// 7. Authorization middleware
function authorize(role: string) {
  return (req, res, next) => {
    if (req.user.role !== role) {
      return res.status(403).json({ error: "Forbidden" });
    }
    next();
  };
}

// 8. Validation
const userSchema = z.object({
  name: z.string().min(2),
  email: z.string().email()
});

function validate(schema: z.ZodSchema) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(400).json({ error: result.error.errors });
    }
    req.body = result.data;
    next();
  };
}

// 9. Protected endpoint with all security layers
app.post("/api/users",
  authenticate,
  authorize("admin"),
  validate(userSchema),
  async (req, res) => {
    const user = await db.users.create(req.body);
    res.status(201).json({ data: user });
  }
);

// 10. Error handling
app.use((err, req, res, next) => {
  logger.error(err);

  // Don't expose internal errors
  res.status(500).json({
    error: "Internal server error"
  });
});

// Start server
https.createServer(sslOptions, app).listen(443);
```

### Security Checklist:

- [ ] Use HTTPS/TLS in production
- [ ] Implement authentication (JWT, OAuth, API keys)
- [ ] Add authorization checks (RBAC)
- [ ] Validate all input with schemas
- [ ] Sanitize user input to prevent XSS
- [ ] Use parameterized queries to prevent SQL injection
- [ ] Implement rate limiting
- [ ] Set security headers (helmet)
- [ ] Configure CORS properly
- [ ] Add CSRF protection for state-changing operations
- [ ] Store secrets in environment variables or secrets manager
- [ ] Log security events
- [ ] Limit request body size
- [ ] Keep dependencies updated
- [ ] Use security scanners (npm audit, Snyk)

### Security Layers Comparison:

| Layer | Purpose | Tools |
|-------|---------|-------|
| Transport | Encrypt data in transit | HTTPS/TLS |
| Authentication | Verify identity | JWT, OAuth, API keys |
| Authorization | Control access | RBAC, permissions |
| Input | Validate & sanitize | Zod, Joi, DOMPurify |
| Rate Limiting | Prevent abuse | express-rate-limit |
| Headers | Browser protections | helmet |
| CORS | Control origins | cors middleware |
| Logging | Detect attacks | Winston, Datadog |

### Analogy:

Think of a **secure building**:
- **HTTPS** = Encrypted communication channels
- **Authentication** = ID badge to enter
- **Authorization** = Badge determines which floors you can access
- **Input Validation** = Security screening at entrance
- **Rate Limiting** = Prevents someone from entering 100 times/minute
- **CORS** = Only allow visitors from approved addresses
- **Security Headers** = Building security protocols
- **Logging** = Security cameras recording everything

### ⚡ One-liner to remember:
> Secure REST APIs = HTTPS + Auth + Authorization + Input Validation + Rate Limiting + Security Headers.

---

## Q3. How do you implement pagination in APIs?

### Concept:

Pagination breaks large datasets into smaller chunks (pages) to improve performance, reduce memory usage, and enhance user experience.

### Why Pagination is Important:

| Without Pagination | With Pagination |
|-------------------|-----------------|
| Returns 10,000 records at once | Returns 20 records per page |
| Slow response time | Fast response time |
| High memory usage | Low memory usage |
| Poor user experience | Smooth scrolling/navigation |
| Database overload | Efficient queries |

### Basic Implementation (Offset-based):

```typescript
// GET /api/users?page=2&limit=20

app.get("/api/users", async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 20;
  const offset = (page - 1) * limit;

  // Fetch paginated data
  const users = await db.query(
    "SELECT * FROM users ORDER BY id LIMIT ? OFFSET ?",
    [limit, offset]
  );

  // Get total count for metadata
  const totalCount = await db.query("SELECT COUNT(*) FROM users");
  const totalPages = Math.ceil(totalCount / limit);

  res.json({
    data: users,
    pagination: {
      page,
      limit,
      totalPages,
      totalCount,
      hasNextPage: page < totalPages,
      hasPrevPage: page > 1
    }
  });
});
```

### Response Format:

```json
{
  "data": [
    { "id": 21, "name": "User 21" },
    { "id": 22, "name": "User 22" }
  ],
  "pagination": {
    "page": 2,
    "limit": 20,
    "totalPages": 50,
    "totalCount": 1000,
    "hasNextPage": true,
    "hasPrevPage": true
  },
  "links": {
    "self": "/api/users?page=2&limit=20",
    "first": "/api/users?page=1&limit=20",
    "prev": "/api/users?page=1&limit=20",
    "next": "/api/users?page=3&limit=20",
    "last": "/api/users?page=50&limit=20"
  }
}
```

### Implementation with Prisma (ORM):

```typescript
app.get("/api/users", async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 20;

  const [users, totalCount] = await Promise.all([
    prisma.user.findMany({
      skip: (page - 1) * limit,
      take: limit,
      orderBy: { createdAt: "desc" }
    }),
    prisma.user.count()
  ]);

  res.json({
    data: users,
    pagination: {
      page,
      limit,
      totalCount,
      totalPages: Math.ceil(totalCount / limit)
    }
  });
});
```

### Implementation with MongoDB:

```typescript
app.get("/api/products", async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 20;

  const products = await db.collection("products")
    .find({})
    .sort({ createdAt: -1 })
    .skip((page - 1) * limit)
    .limit(limit)
    .toArray();

  const totalCount = await db.collection("products").countDocuments();

  res.json({
    data: products,
    pagination: {
      page,
      limit,
      totalCount,
      totalPages: Math.ceil(totalCount / limit)
    }
  });
});
```

### Advanced: Filtering + Sorting + Pagination

```typescript
app.get("/api/products", async (req, res) => {
  const {
    page = 1,
    limit = 20,
    category,
    minPrice,
    maxPrice,
    sort = "-createdAt"
  } = req.query;

  // Build filter
  const filter: any = {};
  if (category) filter.category = category;
  if (minPrice || maxPrice) {
    filter.price = {};
    if (minPrice) filter.price.$gte = parseFloat(minPrice);
    if (maxPrice) filter.price.$lte = parseFloat(maxPrice);
  }

  // Parse sort (e.g., "-price" = descending, "price" = ascending)
  const sortField = sort.startsWith("-") ? sort.slice(1) : sort;
  const sortOrder = sort.startsWith("-") ? -1 : 1;

  const [products, totalCount] = await Promise.all([
    db.collection("products")
      .find(filter)
      .sort({ [sortField]: sortOrder })
      .skip((page - 1) * limit)
      .limit(limit)
      .toArray(),
    db.collection("products").countDocuments(filter)
  ]);

  res.json({
    data: products,
    pagination: {
      page: parseInt(page),
      limit: parseInt(limit),
      totalCount,
      totalPages: Math.ceil(totalCount / limit)
    },
    filters: { category, minPrice, maxPrice, sort }
  });
});
```

### Client-Side Usage:

**React Example:**
```tsx
function UserList() {
  const [users, setUsers] = useState([]);
  const [pagination, setPagination] = useState({});
  const [page, setPage] = useState(1);

  useEffect(() => {
    fetch(`/api/users?page=${page}&limit=20`)
      .then(res => res.json())
      .then(data => {
        setUsers(data.data);
        setPagination(data.pagination);
      });
  }, [page]);

  return (
    <div>
      {users.map(user => (
        <div key={user.id}>{user.name}</div>
      ))}

      <div>
        <button
          disabled={!pagination.hasPrevPage}
          onClick={() => setPage(p => p - 1)}
        >
          Previous
        </button>

        <span>Page {pagination.page} of {pagination.totalPages}</span>

        <button
          disabled={!pagination.hasNextPage}
          onClick={() => setPage(p => p + 1)}
        >
          Next
        </button>
      </div>
    </div>
  );
}
```

### Best Practices:

- [ ] Set default and maximum limits (prevent abuse)
- [ ] Always include total count in response
- [ ] Provide navigation links (first, prev, next, last)
- [ ] Use consistent query parameter names
- [ ] Return empty array (not error) when no results
- [ ] Include current page info in response
- [ ] Cache total count for performance
- [ ] Validate page and limit parameters

### Performance Optimization:

```typescript
// Cache total count (expensive operation)
const CACHE_TTL = 60; // 60 seconds

app.get("/api/users", async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 20;

  // Get count from cache or database
  let totalCount = await redis.get("users:count");

  if (!totalCount) {
    totalCount = await db.query("SELECT COUNT(*) FROM users");
    await redis.setex("users:count", CACHE_TTL, totalCount);
  }

  const users = await db.query(
    "SELECT * FROM users LIMIT ? OFFSET ?",
    [limit, (page - 1) * limit]
  );

  res.json({
    data: users,
    pagination: {
      page,
      limit,
      totalCount: parseInt(totalCount),
      totalPages: Math.ceil(totalCount / limit)
    }
  });
});
```

### Analogy:

Think of a **phone book**:
- Without pagination = Reading the entire phone book at once
- With pagination = Turning one page at a time
- Page number = Which page you're on
- Limit = How many entries per page
- Total pages = How thick the book is

### ⚡ One-liner to remember:
> Pagination = Break large datasets into pages using page number + limit, return data + metadata.

---

## Q4. What is the difference between Offset-based and Cursor-based pagination?

### Concept:

**Offset-based** uses page number and limit (skip N records).
**Cursor-based** uses a unique identifier to fetch records after a specific point.

### Comparison:

| Aspect | Offset-based | Cursor-based |
|--------|--------------|--------------|
| Query params | `?page=2&limit=20` | `?cursor=abc123&limit=20` |
| Implementation | Simple | More complex |
| Performance | Degrades with large offsets | Consistent performance |
| Duplicate results | Possible with live data | No duplicates |
| Missing results | Possible with deletions | No missing results |
| Use case | Small datasets, admin panels | Infinite scroll, real-time feeds |
| Jump to page | Easy (page=50) | Not possible |

### 1. Offset-based Pagination:

**How it works:**
```
Page 1: LIMIT 20 OFFSET 0   → Records 1-20
Page 2: LIMIT 20 OFFSET 20  → Records 21-40
Page 3: LIMIT 20 OFFSET 40  → Records 41-60
```

**Implementation:**
```typescript
// GET /api/posts?page=2&limit=20

app.get("/api/posts", async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 20;
  const offset = (page - 1) * limit;

  const posts = await db.query(
    "SELECT * FROM posts ORDER BY created_at DESC LIMIT ? OFFSET ?",
    [limit, offset]
  );

  const totalCount = await db.query("SELECT COUNT(*) FROM posts");

  res.json({
    data: posts,
    pagination: {
      page,
      limit,
      totalCount,
      totalPages: Math.ceil(totalCount / limit)
    }
  });
});
```

**Problems:**

1. **Performance degrades with large offsets:**
```sql
-- Fast: OFFSET 0 (first page)
SELECT * FROM posts ORDER BY id LIMIT 20 OFFSET 0;

-- Slow: OFFSET 10000 (page 500)
SELECT * FROM posts ORDER BY id LIMIT 20 OFFSET 10000;
-- Database scans 10,020 rows to skip first 10,000
```

2. **Inconsistent results with live data:**
```
User on page 2 (records 21-40)
→ New record inserted at position 1
→ User clicks next → sees record 40 again (shifted to position 41)
```

### 2. Cursor-based Pagination:

**How it works:**
```
Cursor = Last record's unique identifier (ID, timestamp)
Next page: Fetch records AFTER cursor
```

**Implementation:**
```typescript
// GET /api/posts?cursor=2024-01-01T10:00:00Z&limit=20

app.get("/api/posts", async (req, res) => {
  const cursor = req.query.cursor; // ISO timestamp
  const limit = parseInt(req.query.limit) || 20;

  let query = "SELECT * FROM posts";
  let params = [];

  if (cursor) {
    query += " WHERE created_at < ?";
    params.push(cursor);
  }

  query += " ORDER BY created_at DESC LIMIT ?";
  params.push(limit + 1); // Fetch one extra to check if more exist

  const posts = await db.query(query, params);

  // Check if there are more results
  const hasMore = posts.length > limit;
  if (hasMore) {
    posts.pop(); // Remove extra record
  }

  // Next cursor is the last record's timestamp
  const nextCursor = hasMore ? posts[posts.length - 1].created_at : null;

  res.json({
    data: posts,
    pagination: {
      nextCursor,
      hasMore
    }
  });
});
```

**Using ID as cursor:**
```typescript
app.get("/api/posts", async (req, res) => {
  const cursor = req.query.cursor; // Last seen ID
  const limit = parseInt(req.query.limit) || 20;

  const posts = await db.query(
    cursor
      ? "SELECT * FROM posts WHERE id < ? ORDER BY id DESC LIMIT ?"
      : "SELECT * FROM posts ORDER BY id DESC LIMIT ?",
    cursor ? [cursor, limit + 1] : [limit + 1]
  );

  const hasMore = posts.length > limit;
  if (hasMore) posts.pop();

  res.json({
    data: posts,
    pagination: {
      nextCursor: hasMore ? posts[posts.length - 1].id : null,
      hasMore
    }
  });
});
```

**Benefits:**

1. **Consistent performance:**
```sql
-- Always fast (uses index on created_at or id)
SELECT * FROM posts
WHERE created_at < '2024-01-01T10:00:00Z'
ORDER BY created_at DESC
LIMIT 20;
```

2. **No duplicate/missing results:**
```
User fetches page with cursor=100
→ New record inserted at position 1
→ User fetches next page with cursor=80
→ No duplicates (always gets records < 80)
```

### Client-Side Usage:

**Offset-based (Numbered pages):**
```tsx
function PostList() {
  const [posts, setPosts] = useState([]);
  const [page, setPage] = useState(1);
  const [totalPages, setTotalPages] = useState(0);

  useEffect(() => {
    fetch(`/api/posts?page=${page}&limit=20`)
      .then(res => res.json())
      .then(data => {
        setPosts(data.data);
        setTotalPages(data.pagination.totalPages);
      });
  }, [page]);

  return (
    <div>
      {posts.map(post => <Post key={post.id} {...post} />)}

      <Pagination>
        {[...Array(totalPages)].map((_, i) => (
          <button key={i} onClick={() => setPage(i + 1)}>
            {i + 1}
          </button>
        ))}
      </Pagination>
    </div>
  );
}
```

**Cursor-based (Infinite scroll):**
```tsx
function PostFeed() {
  const [posts, setPosts] = useState([]);
  const [cursor, setCursor] = useState(null);
  const [hasMore, setHasMore] = useState(true);

  const loadMore = async () => {
    const url = cursor
      ? `/api/posts?cursor=${cursor}&limit=20`
      : `/api/posts?limit=20`;

    const res = await fetch(url);
    const data = await res.json();

    setPosts(prev => [...prev, ...data.data]);
    setCursor(data.pagination.nextCursor);
    setHasMore(data.pagination.hasMore);
  };

  useEffect(() => {
    loadMore();
  }, []);

  return (
    <div>
      {posts.map(post => <Post key={post.id} {...post} />)}

      {hasMore && (
        <button onClick={loadMore}>Load More</button>
      )}
    </div>
  );
}
```

### When to Use Each:

**Use Offset-based when:**
- Small to medium datasets (< 10,000 records)
- Need to jump to specific pages
- Admin panels with numbered pagination
- Data doesn't change frequently
- Simple implementation preferred

**Use Cursor-based when:**
- Large datasets (> 100,000 records)
- Real-time feeds (social media, chat)
- Infinite scroll UI
- Data changes frequently
- Performance is critical

### Hybrid Approach:

```typescript
// Support both pagination types
app.get("/api/posts", async (req, res) => {
  const { page, cursor, limit = 20 } = req.query;

  let posts;

  if (cursor) {
    // Cursor-based pagination
    posts = await db.query(
      "SELECT * FROM posts WHERE id < ? ORDER BY id DESC LIMIT ?",
      [cursor, limit + 1]
    );
  } else if (page) {
    // Offset-based pagination
    const offset = (page - 1) * limit;
    posts = await db.query(
      "SELECT * FROM posts ORDER BY id DESC LIMIT ? OFFSET ?",
      [limit, offset]
    );
  } else {
    // Default: first page
    posts = await db.query(
      "SELECT * FROM posts ORDER BY id DESC LIMIT ?",
      [limit]
    );
  }

  // Return appropriate pagination metadata
  res.json({
    data: posts,
    pagination: cursor
      ? { nextCursor: posts[posts.length - 1]?.id, hasMore: posts.length > limit }
      : { page: parseInt(page) || 1, limit: parseInt(limit) }
  });
});
```

### Performance Comparison:

| Records | Offset-based (OFFSET 10000) | Cursor-based |
|---------|----------------------------|--------------|
| 10K | ~50ms | ~10ms |
| 100K | ~500ms | ~10ms |
| 1M | ~5000ms | ~10ms |

### Analogy:

**Offset-based** = Reading a book by page number
- "Go to page 50" (need to flip through 49 pages first)
- Easy to jump to any page
- Slow if book is very thick

**Cursor-based** = Following a bookmark
- "Continue from where I left off" (instant)
- Fast no matter how thick the book
- Can't jump to random pages

### ⚡ One-liner to remember:
> Offset = Simple but slow with large data. Cursor = Fast and consistent, ideal for infinite scroll.

---

## Q5. How do you implement rate limiting and what algorithms are used?

### Concept:

Rate limiting controls how many requests a client can make in a given time period to prevent abuse, ensure fair usage, and protect server resources.

### Why Rate Limiting is Important:

| Without Rate Limiting | With Rate Limiting |
|----------------------|-------------------|
| DDoS attacks succeed | Blocks excessive requests |
| Server overload | Controlled resource usage |
| Abuse/spam | Fair usage for all users |
| High costs | Predictable costs |
| Poor experience for all | Consistent performance |

### Common Rate Limiting Algorithms:

| Algorithm | How It Works | Use Case | Pros | Cons |
|-----------|--------------|----------|------|------|
| **Fixed Window** | Count requests per time window | Simple APIs | Easy to implement | Burst at window edges |
| **Sliding Window** | Rolling time window | Better precision | Smooth distribution | More complex |
| **Token Bucket** | Tokens refill at rate | Allow bursts | Flexible | Needs token tracking |
| **Leaky Bucket** | Process at constant rate | Steady flow | Smooth output | Strict, no bursts |

---

## 1. Fixed Window Algorithm

### How It Works:

```
Window 1: 00:00 - 01:00 → 100 requests allowed
Window 2: 01:00 - 02:00 → 100 requests allowed

Problem: User can make 100 requests at 00:59 and 100 at 01:00 = 200 in 1 minute
```

### Implementation (Redis):

```typescript
import Redis from "ioredis";

const redis = new Redis();

async function fixedWindowRateLimit(userId: string) {
  const limit = 100; // requests
  const window = 60; // seconds

  const key = `rate:${userId}:${Math.floor(Date.now() / 1000 / window)}`;
  const current = await redis.incr(key);

  if (current === 1) {
    await redis.expire(key, window); // Set expiry on first request
  }

  if (current > limit) {
    return {
      allowed: false,
      remaining: 0,
      resetAt: Math.ceil(Date.now() / 1000 / window) * window
    };
  }

  return {
    allowed: true,
    remaining: limit - current,
    resetAt: Math.ceil(Date.now() / 1000 / window) * window
  };
}

// Express middleware
async function rateLimitMiddleware(req, res, next) {
  const userId = req.user.id;
  const result = await fixedWindowRateLimit(userId);

  res.setHeader("X-RateLimit-Limit", "100");
  res.setHeader("X-RateLimit-Remaining", result.remaining);
  res.setHeader("X-RateLimit-Reset", result.resetAt);

  if (!result.allowed) {
    return res.status(429).json({
      error: "Too many requests",
      retryAfter: result.resetAt - Math.floor(Date.now() / 1000)
    });
  }

  next();
}
```

**Pros:**
- Simple to implement
- Memory efficient
- Easy to understand

**Cons:**
- Allows burst at window edges
- Not perfectly accurate

---

## 2. Sliding Window Algorithm

### How It Works:

```
Uses weighted count from previous and current window

Example:
- Previous window (11:00-11:01): 80 requests
- Current window (11:01-11:02): 40 requests
- At 11:01:30 (50% into current window):
  Weight = (50% × 80) + 40 = 80 requests
```

### Implementation:

```typescript
async function slidingWindowRateLimit(userId: string) {
  const limit = 100;
  const window = 60; // seconds

  const now = Date.now() / 1000;
  const currentWindow = Math.floor(now / window);
  const previousWindow = currentWindow - 1;

  const currentKey = `rate:${userId}:${currentWindow}`;
  const previousKey = `rate:${userId}:${previousWindow}`;

  const [currentCount, previousCount] = await Promise.all([
    redis.get(currentKey).then(v => parseInt(v) || 0),
    redis.get(previousKey).then(v => parseInt(v) || 0)
  ]);

  // Calculate weighted count
  const percentageInCurrent = (now % window) / window;
  const weightedCount =
    (previousCount * (1 - percentageInCurrent)) + currentCount;

  if (weightedCount >= limit) {
    return { allowed: false, remaining: 0 };
  }

  // Increment current window
  await redis.incr(currentKey);
  await redis.expire(currentKey, window * 2);

  return {
    allowed: true,
    remaining: Math.floor(limit - weightedCount - 1)
  };
}
```

**Pros:**
- More accurate than fixed window
- Prevents burst at edges
- Smooth rate limiting

**Cons:**
- More complex
- Slightly more memory

---

## 3. Token Bucket Algorithm

### How It Works:

```
Bucket holds tokens (max capacity)
Tokens refill at constant rate
Each request consumes 1 token
If no tokens → reject request

Example:
- Capacity: 100 tokens
- Refill rate: 10 tokens/second
- Allows bursts up to capacity
```

### Implementation:

```typescript
interface TokenBucket {
  tokens: number;
  lastRefill: number;
}

async function tokenBucketRateLimit(userId: string) {
  const capacity = 100; // max tokens
  const refillRate = 10; // tokens per second

  const key = `bucket:${userId}`;
  const now = Date.now() / 1000;

  // Get current bucket state
  const data = await redis.get(key);
  let bucket: TokenBucket = data
    ? JSON.parse(data)
    : { tokens: capacity, lastRefill: now };

  // Calculate tokens to add
  const timePassed = now - bucket.lastRefill;
  const tokensToAdd = timePassed * refillRate;
  bucket.tokens = Math.min(capacity, bucket.tokens + tokensToAdd);
  bucket.lastRefill = now;

  // Check if request is allowed
  if (bucket.tokens < 1) {
    await redis.set(key, JSON.stringify(bucket), "EX", 3600);
    return { allowed: false, remaining: 0 };
  }

  // Consume token
  bucket.tokens -= 1;
  await redis.set(key, JSON.stringify(bucket), "EX", 3600);

  return {
    allowed: true,
    remaining: Math.floor(bucket.tokens)
  };
}
```

**Pros:**
- Allows controlled bursts
- Smooth over time
- Flexible

**Cons:**
- Complex implementation
- Needs state storage

---

## 4. Leaky Bucket Algorithm

### How It Works:

```
Requests enter bucket (queue)
Process at constant rate (leak)
If bucket full → reject

Example:
- Process 10 requests/second
- Queue capacity: 50
- Excess requests dropped
```

### Implementation:

```typescript
async function leakyBucketRateLimit(userId: string) {
  const processRate = 10; // requests per second
  const capacity = 50; // max queue size

  const key = `leaky:${userId}`;
  const now = Date.now() / 1000;

  const data = await redis.get(key);
  let bucket = data ? JSON.parse(data) : { queue: 0, lastLeak: now };

  // Calculate leaked requests
  const timePassed = now - bucket.lastLeak;
  const leaked = timePassed * processRate;
  bucket.queue = Math.max(0, bucket.queue - leaked);
  bucket.lastLeak = now;

  // Check if bucket is full
  if (bucket.queue >= capacity) {
    await redis.set(key, JSON.stringify(bucket), "EX", 3600);
    return { allowed: false, remaining: 0 };
  }

  // Add request to queue
  bucket.queue += 1;
  await redis.set(key, JSON.stringify(bucket), "EX", 3600);

  return {
    allowed: true,
    remaining: Math.floor(capacity - bucket.queue)
  };
}
```

**Pros:**
- Smooth, constant rate
- Prevents bursts
- Fair processing

**Cons:**
- No burst allowance
- Complex queue management

---

## Complete Implementation with Express

```typescript
import express from "express";
import Redis from "ioredis";
import rateLimit from "express-rate-limit";
import RedisStore from "rate-limit-redis";

const app = express();
const redis = new Redis();

// Option 1: Using express-rate-limit (Fixed Window)
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: "Too many requests from this IP",
  standardHeaders: true, // Return rate limit info in headers
  legacyHeaders: false,
  store: new RedisStore({
    client: redis,
    prefix: "rl:"
  })
});

app.use("/api/", limiter);

// Option 2: Custom Sliding Window
async function customRateLimiter(req, res, next) {
  const identifier = req.user?.id || req.ip;
  const result = await slidingWindowRateLimit(identifier);

  res.setHeader("X-RateLimit-Limit", "100");
  res.setHeader("X-RateLimit-Remaining", result.remaining);

  if (!result.allowed) {
    return res.status(429).json({
      error: "Too many requests",
      message: "Please wait before making more requests"
    });
  }

  next();
}

app.use("/api/v2/", customRateLimiter);

// Option 3: Different limits for different endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5, // Only 5 login attempts
  skipSuccessfulRequests: true
});

app.post("/api/login", authLimiter, loginHandler);

// Option 4: User-tier based limits
async function tierBasedLimiter(req, res, next) {
  const limits = {
    free: 100,
    pro: 1000,
    enterprise: 10000
  };

  const userTier = req.user?.tier || "free";
  const limit = limits[userTier];

  const result = await checkRateLimit(req.user.id, limit);

  res.setHeader("X-RateLimit-Limit", limit);
  res.setHeader("X-RateLimit-Remaining", result.remaining);

  if (!result.allowed) {
    return res.status(429).json({
      error: "Rate limit exceeded",
      upgrade: "Upgrade to Pro for higher limits"
    });
  }

  next();
}
```

---

## Response Headers

```http
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1640000000
Retry-After: 60

{
  "error": "Too many requests",
  "message": "Please wait 60 seconds before retrying"
}
```

---

## Best Practices

- [ ] Set appropriate limits per endpoint
- [ ] Return rate limit info in headers
- [ ] Use distributed storage (Redis) for scale
- [ ] Different limits for different user tiers
- [ ] Stricter limits for expensive operations
- [ ] Implement exponential backoff
- [ ] Log rate limit violations
- [ ] Monitor rate limit metrics
- [ ] Whitelist internal services
- [ ] Graceful degradation

---

## Distributed Rate Limiting (Microservices)

```typescript
// Using Redis for shared state across servers
class DistributedRateLimiter {
  constructor(private redis: Redis) {}

  async check(key: string, limit: number, window: number) {
    const multi = this.redis.multi();

    multi.incr(key);
    multi.expire(key, window);

    const [count] = await multi.exec();

    return {
      allowed: count <= limit,
      remaining: Math.max(0, limit - count),
      reset: Math.ceil(Date.now() / 1000) + window
    };
  }
}

// Usage across multiple servers
const limiter = new DistributedRateLimiter(redis);

app.use(async (req, res, next) => {
  const key = `rate:${req.ip}:${Math.floor(Date.now() / 60000)}`;
  const result = await limiter.check(key, 100, 60);

  if (!result.allowed) {
    return res.status(429).json({ error: "Rate limit exceeded" });
  }

  next();
});
```

---

## Algorithm Comparison Example

```typescript
// Scenario: 100 requests/minute limit

// Fixed Window (00:00 - 01:00)
// Time 00:59: 100 requests ✅
// Time 01:00: 100 requests ✅ (new window)
// Total in 1 minute: 200 requests ❌

// Sliding Window
// Time 00:59: 100 requests ✅
// Time 01:00: Counts previous window
// Total: Limited to ~100 ✅

// Token Bucket (capacity: 100, refill: 1.67/sec)
// Allows burst of 100
// Then steady 1.67/sec
// Good for APIs with occasional spikes

// Leaky Bucket (process: 1.67/sec, queue: 50)
// Strict 1.67/sec processing
// No bursts allowed
// Good for background jobs
```

---

## Monitoring and Alerts

```typescript
// Track rate limit metrics
async function trackRateLimitMetrics(userId: string, blocked: boolean) {
  await redis.incr(`metrics:ratelimit:${blocked ? "blocked" : "allowed"}`);

  if (blocked) {
    // Alert if user is consistently hitting limits
    const blockedCount = await redis.incr(`metrics:user:${userId}:blocked`);

    if (blockedCount > 10) {
      await sendAlert({
        type: "rate_limit_abuse",
        userId,
        count: blockedCount
      });
    }
  }
}
```

---

### Analogy:

**Fixed Window** = Hourly parking ticket (expires at exact hour)

**Sliding Window** = Rolling 60-minute meter (smooth transition)

**Token Bucket** = Coffee shop punch card (collect tokens, use when needed, refills over time)

**Leaky Bucket** = Water bucket with hole (constant drip rate, overflow rejected)

---

### ⚡ One-liner to remember:
> Rate limiting = Control request frequency using algorithms: Fixed/Sliding Window (simple), Token Bucket (bursts allowed), Leaky Bucket (strict flow).

---

## Q6. How do you implement caching in a Next.js PostgreSQL app?

### Concept:

Caching stores frequently accessed data in memory to reduce database queries, improve response times, and lower infrastructure costs. In a Next.js + PostgreSQL app, you have multiple caching layers.

### Caching Layers:

```
┌─────────────────────────────────────────┐
│ 1. Next.js Built-in Caching             │ ← Static pages, ISR, Route caching
├─────────────────────────────────────────┤
│ 2. Client-Side Caching                  │ ← React Query, SWR
├─────────────────────────────────────────┤
│ 3. API Response Caching                 │ ← CDN, HTTP headers
├─────────────────────────────────────────┤
│ 4. Application Cache (Redis)            │ ← Query results, sessions
├─────────────────────────────────────────┤
│ 5. Database Query Cache                 │ ← PostgreSQL cache
└─────────────────────────────────────────┘
```

---

## 1. Next.js Built-in Caching

### Static Site Generation (SSG)

**Cached at build time:**

```tsx
// app/products/page.tsx (App Router)
export default async function ProductsPage() {
  // Fetched at build time, cached forever
  const products = await db.query("SELECT * FROM products");

  return (
    <div>
      {products.map(product => (
        <ProductCard key={product.id} {...product} />
      ))}
    </div>
  );
}
```

### Incremental Static Regeneration (ISR)

**Re-generate static pages after a time interval:**

```tsx
// app/products/page.tsx
export const revalidate = 60; // Revalidate every 60 seconds

export default async function ProductsPage() {
  const products = await db.query(
    "SELECT * FROM products ORDER BY created_at DESC"
  );

  return (
    <div>
      {products.map(product => <ProductCard key={product.id} {...product} />)}
    </div>
  );
}
```

**How ISR works:**
```
User 1 visits at 0s   → Serves cached version
User 2 visits at 61s  → Serves stale version, triggers rebuild
User 3 visits at 62s  → Serves fresh version
```

### Dynamic Routes with ISR

```tsx
// app/products/[id]/page.tsx
export const revalidate = 3600; // 1 hour

export async function generateStaticParams() {
  // Pre-render popular products at build time
  const products = await db.query(
    "SELECT id FROM products WHERE popular = true LIMIT 100"
  );

  return products.map(product => ({
    id: product.id.toString()
  }));
}

export default async function ProductPage({ params }: { params: { id: string } }) {
  const product = await db.query(
    "SELECT * FROM products WHERE id = ?",
    [params.id]
  );

  return <ProductDetails product={product} />;
}
```

### App Router Cache Configuration

```tsx
// app/api/products/route.ts
import { NextResponse } from "next/server";

export async function GET(request: Request) {
  const products = await db.query("SELECT * FROM products");

  return NextResponse.json(products, {
    headers: {
      "Cache-Control": "public, s-maxage=60, stale-while-revalidate=120"
      // s-maxage: CDN caches for 60s
      // stale-while-revalidate: Serve stale for 120s while revalidating
    }
  });
}
```

### On-Demand Revalidation

```tsx
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from "next/cache";
import { NextRequest, NextResponse } from "next/server";

export async function POST(request: NextRequest) {
  const { path, tag } = await request.json();

  if (path) {
    revalidatePath(path);
  }

  if (tag) {
    revalidateTag(tag);
  }

  return NextResponse.json({ revalidated: true });
}

// Trigger revalidation after database update
async function updateProduct(id: string, data: any) {
  await db.query("UPDATE products SET ? WHERE id = ?", [data, id]);

  // Clear cache for this product
  await fetch("/api/revalidate", {
    method: "POST",
    body: JSON.stringify({ path: `/products/${id}` })
  });
}
```

### Tagged Caching

```tsx
// app/products/page.tsx
import { unstable_cache } from "next/cache";

const getCachedProducts = unstable_cache(
  async () => {
    return await db.query("SELECT * FROM products");
  },
  ["products-list"],
  {
    tags: ["products"],
    revalidate: 60
  }
);

export default async function ProductsPage() {
  const products = await getCachedProducts();
  return <ProductList products={products} />;
}

// Clear cache when products change
import { revalidateTag } from "next/cache";

async function addProduct(product: Product) {
  await db.query("INSERT INTO products SET ?", [product]);
  revalidateTag("products"); // Clear all cached queries tagged with "products"
}
```

---

## 2. Client-Side Caching with React Query

```tsx
// app/providers.tsx
"use client";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactNode, useState } from "react";

export function Providers({ children }: { children: ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60 * 1000, // Data fresh for 60s
            cacheTime: 5 * 60 * 1000, // Keep cache for 5 minutes
            refetchOnWindowFocus: false
          }
        }
      })
  );

  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}

// app/layout.tsx
import { Providers } from "./providers";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

### Using React Query

```tsx
// components/ProductList.tsx
"use client";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";

function ProductList() {
  const queryClient = useQueryClient();

  // Fetch and cache products
  const { data: products, isLoading } = useQuery({
    queryKey: ["products"],
    queryFn: async () => {
      const res = await fetch("/api/products");
      return res.json();
    },
    staleTime: 60 * 1000, // Fresh for 60 seconds
    cacheTime: 5 * 60 * 1000 // Keep in cache for 5 minutes
  });

  // Mutation with cache invalidation
  const addProduct = useMutation({
    mutationFn: async (product: Product) => {
      const res = await fetch("/api/products", {
        method: "POST",
        body: JSON.stringify(product)
      });
      return res.json();
    },
    onSuccess: () => {
      // Invalidate and refetch products
      queryClient.invalidateQueries({ queryKey: ["products"] });
    }
  });

  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      {products.map(product => (
        <ProductCard key={product.id} {...product} />
      ))}
      <button onClick={() => addProduct.mutate(newProduct)}>
        Add Product
      </button>
    </div>
  );
}
```

### Optimistic Updates

```tsx
const updateProduct = useMutation({
  mutationFn: async ({ id, data }: { id: string; data: Product }) => {
    const res = await fetch(`/api/products/${id}`, {
      method: "PUT",
      body: JSON.stringify(data)
    });
    return res.json();
  },
  onMutate: async ({ id, data }) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: ["products"] });

    // Snapshot current value
    const previous = queryClient.getQueryData(["products"]);

    // Optimistically update
    queryClient.setQueryData(["products"], (old: Product[]) =>
      old.map(p => (p.id === id ? { ...p, ...data } : p))
    );

    return { previous };
  },
  onError: (err, variables, context) => {
    // Rollback on error
    queryClient.setQueryData(["products"], context.previous);
  },
  onSettled: () => {
    // Refetch after mutation
    queryClient.invalidateQueries({ queryKey: ["products"] });
  }
});
```

### Prefetching

```tsx
// Prefetch data on hover
function ProductCard({ id }: { id: string }) {
  const queryClient = useQueryClient();

  const prefetchProduct = () => {
    queryClient.prefetchQuery({
      queryKey: ["product", id],
      queryFn: async () => {
        const res = await fetch(`/api/products/${id}`);
        return res.json();
      }
    });
  };

  return (
    <Link
      href={`/products/${id}`}
      onMouseEnter={prefetchProduct} // Prefetch on hover
    >
      View Product
    </Link>
  );
}
```

---

## 3. Redis Caching Layer

### Setup Redis

```typescript
// lib/redis.ts
import Redis from "ioredis";

const redis = new Redis({
  host: process.env.REDIS_HOST || "localhost",
  port: parseInt(process.env.REDIS_PORT || "6379"),
  password: process.env.REDIS_PASSWORD
});

export default redis;
```

### Cache Database Queries

```typescript
// lib/cache.ts
import redis from "./redis";

export async function cachedQuery<T>(
  key: string,
  queryFn: () => Promise<T>,
  ttl: number = 60 // seconds
): Promise<T> {
  // Try to get from cache
  const cached = await redis.get(key);

  if (cached) {
    return JSON.parse(cached);
  }

  // Query database
  const result = await queryFn();

  // Store in cache
  await redis.setex(key, ttl, JSON.stringify(result));

  return result;
}

// Usage
import { cachedQuery } from "@/lib/cache";

export async function getProducts() {
  return cachedQuery(
    "products:all",
    async () => {
      return await db.query("SELECT * FROM products");
    },
    60 // Cache for 60 seconds
  );
}
```

### API Route with Redis Caching

```typescript
// app/api/products/route.ts
import { NextResponse } from "next/server";
import redis from "@/lib/redis";
import db from "@/lib/db";

export async function GET() {
  const cacheKey = "products:all";

  // Try cache first
  const cached = await redis.get(cacheKey);
  if (cached) {
    return NextResponse.json(JSON.parse(cached), {
      headers: { "X-Cache": "HIT" }
    });
  }

  // Query database
  const products = await db.query(
    "SELECT * FROM products ORDER BY created_at DESC"
  );

  // Cache for 60 seconds
  await redis.setex(cacheKey, 60, JSON.stringify(products));

  return NextResponse.json(products, {
    headers: {
      "X-Cache": "MISS",
      "Cache-Control": "public, s-maxage=60"
    }
  });
}
```

### Cache Patterns

**1. Cache-Aside (Lazy Loading):**

```typescript
async function getUser(id: string) {
  const cacheKey = `user:${id}`;

  // Check cache
  let user = await redis.get(cacheKey);

  if (user) {
    return JSON.parse(user);
  }

  // Cache miss - fetch from DB
  user = await db.query("SELECT * FROM users WHERE id = ?", [id]);

  // Store in cache
  await redis.setex(cacheKey, 300, JSON.stringify(user));

  return user;
}
```

**2. Write-Through Cache:**

```typescript
async function updateUser(id: string, data: User) {
  // Update database
  await db.query("UPDATE users SET ? WHERE id = ?", [data, id]);

  // Update cache immediately
  const cacheKey = `user:${id}`;
  await redis.setex(cacheKey, 300, JSON.stringify(data));

  return data;
}
```

**3. Write-Behind (Write-Back) Cache:**

```typescript
async function logEvent(event: Event) {
  // Write to cache immediately
  await redis.lpush("events:queue", JSON.stringify(event));

  // Background job writes to DB later
  // (Processed by a separate worker)
}

// Background worker
async function processEventQueue() {
  while (true) {
    const event = await redis.rpop("events:queue");
    if (event) {
      await db.query("INSERT INTO events SET ?", [JSON.parse(event)]);
    }
    await sleep(100);
  }
}
```

---

## 4. Cache Invalidation Strategies

### 1. Time-Based Expiration (TTL)

```typescript
// Short TTL for frequently changing data
await redis.setex("products:featured", 60, JSON.stringify(products)); // 60s

// Long TTL for static data
await redis.setex("categories:all", 3600, JSON.stringify(categories)); // 1 hour
```

### 2. Event-Based Invalidation

```typescript
// When product is updated, clear related caches
async function updateProduct(id: string, data: Product) {
  await db.query("UPDATE products SET ? WHERE id = ?", [data, id]);

  // Clear specific caches
  await Promise.all([
    redis.del(`product:${id}`),
    redis.del("products:all"),
    redis.del("products:featured"),
    redis.del(`products:category:${data.categoryId}`)
  ]);
}
```

### 3. Tag-Based Invalidation

```typescript
// Tag cache keys
async function cacheWithTags(key: string, value: any, tags: string[], ttl: number) {
  await redis.setex(key, ttl, JSON.stringify(value));

  // Store key in tag sets
  for (const tag of tags) {
    await redis.sadd(`tag:${tag}`, key);
  }
}

// Invalidate by tag
async function invalidateTag(tag: string) {
  const keys = await redis.smembers(`tag:${tag}`);

  if (keys.length > 0) {
    await redis.del(...keys);
    await redis.del(`tag:${tag}`);
  }
}

// Usage
await cacheWithTags(
  "product:123",
  product,
  ["products", "category:electronics"],
  300
);

// Clear all products
await invalidateTag("products");
```

### 4. Cache Stampede Prevention

**Problem:** Many requests hit empty cache simultaneously → all query DB

**Solution: Lock-based caching**

```typescript
async function cachedQueryWithLock<T>(
  key: string,
  queryFn: () => Promise<T>,
  ttl: number = 60
): Promise<T> {
  // Try to get from cache
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);

  // Try to acquire lock
  const lockKey = `lock:${key}`;
  const locked = await redis.set(lockKey, "1", "EX", 10, "NX"); // 10s lock

  if (locked) {
    try {
      // Query database
      const result = await queryFn();

      // Cache result
      await redis.setex(key, ttl, JSON.stringify(result));

      return result;
    } finally {
      // Release lock
      await redis.del(lockKey);
    }
  } else {
    // Wait for lock holder to populate cache
    await sleep(100);
    return cachedQueryWithLock(key, queryFn, ttl); // Retry
  }
}
```

---

## 5. PostgreSQL Query Optimization

### Materialized Views (Pre-computed queries)

```sql
-- Create materialized view for expensive aggregation
CREATE MATERIALIZED VIEW product_stats AS
SELECT
  category_id,
  COUNT(*) as product_count,
  AVG(price) as avg_price,
  SUM(stock) as total_stock
FROM products
GROUP BY category_id;

-- Refresh periodically
REFRESH MATERIALIZED VIEW product_stats;
```

```typescript
// Use in API
export async function GET() {
  // Fast query (pre-computed)
  const stats = await db.query("SELECT * FROM product_stats");

  return NextResponse.json(stats);
}

// Refresh view when products change
async function updateProduct(id: string, data: Product) {
  await db.query("UPDATE products SET ? WHERE id = ?", [data, id]);

  // Refresh materialized view
  await db.query("REFRESH MATERIALIZED VIEW product_stats");
}
```

### Database Connection Pooling

```typescript
// lib/db.ts
import { Pool } from "pg";

const pool = new Pool({
  host: process.env.DB_HOST,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  max: 20, // Max connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000
});

export async function query(text: string, params?: any[]) {
  const start = Date.now();
  const res = await pool.query(text, params);
  const duration = Date.now() - start;

  console.log("Query executed", { text, duration, rows: res.rowCount });

  return res.rows;
}
```

---

## Complete Example: E-commerce Product API

```typescript
// lib/productService.ts
import redis from "@/lib/redis";
import db from "@/lib/db";

export class ProductService {
  private CACHE_TTL = {
    PRODUCT: 300, // 5 minutes
    LIST: 60, // 1 minute
    FEATURED: 120 // 2 minutes
  };

  async getProduct(id: string) {
    const cacheKey = `product:${id}`;

    // Try cache
    const cached = await redis.get(cacheKey);
    if (cached) {
      return JSON.parse(cached);
    }

    // Query database
    const product = await db.query(
      "SELECT * FROM products WHERE id = ?",
      [id]
    );

    if (product) {
      // Cache with tags
      await this.cacheWithTags(
        cacheKey,
        product,
        ["products", `category:${product.categoryId}`],
        this.CACHE_TTL.PRODUCT
      );
    }

    return product;
  }

  async getProducts(filters: { category?: string; page?: number; limit?: number }) {
    const { category, page = 1, limit = 20 } = filters;
    const cacheKey = `products:${category || "all"}:${page}:${limit}`;

    // Try cache
    const cached = await redis.get(cacheKey);
    if (cached) {
      return JSON.parse(cached);
    }

    // Query database
    const offset = (page - 1) * limit;
    let query = "SELECT * FROM products";
    const params: any[] = [];

    if (category) {
      query += " WHERE category_id = ?";
      params.push(category);
    }

    query += " ORDER BY created_at DESC LIMIT ? OFFSET ?";
    params.push(limit, offset);

    const products = await db.query(query, params);

    // Cache
    await redis.setex(cacheKey, this.CACHE_TTL.LIST, JSON.stringify(products));

    return products;
  }

  async updateProduct(id: string, data: Product) {
    // Update database
    await db.query("UPDATE products SET ? WHERE id = ?", [data, id]);

    // Invalidate caches
    await this.invalidateProductCaches(id, data.categoryId);

    // Revalidate Next.js pages
    await fetch("/api/revalidate", {
      method: "POST",
      body: JSON.stringify({ path: `/products/${id}` })
    });

    return data;
  }

  private async cacheWithTags(
    key: string,
    value: any,
    tags: string[],
    ttl: number
  ) {
    await redis.setex(key, ttl, JSON.stringify(value));

    for (const tag of tags) {
      await redis.sadd(`tag:${tag}`, key);
    }
  }

  private async invalidateProductCaches(productId: string, categoryId: string) {
    // Clear specific product
    await redis.del(`product:${productId}`);

    // Clear lists
    const listKeys = await redis.keys("products:*");
    if (listKeys.length > 0) {
      await redis.del(...listKeys);
    }

    // Clear category cache
    await redis.del(`products:category:${categoryId}`);
  }
}

// API Route
// app/api/products/[id]/route.ts
import { NextRequest, NextResponse } from "next/server";
import { ProductService } from "@/lib/productService";

const productService = new ProductService();

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const product = await productService.getProduct(params.id);

  if (!product) {
    return NextResponse.json({ error: "Not found" }, { status: 404 });
  }

  return NextResponse.json(product, {
    headers: {
      "Cache-Control": "public, s-maxage=60, stale-while-revalidate=120"
    }
  });
}

export async function PUT(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const data = await request.json();
  const product = await productService.updateProduct(params.id, data);

  return NextResponse.json(product);
}
```

---

## Monitoring Cache Performance

```typescript
// lib/cacheMonitor.ts
import redis from "./redis";

export class CacheMonitor {
  async getStats() {
    const info = await redis.info("stats");

    return {
      hits: this.parseInfo(info, "keyspace_hits"),
      misses: this.parseInfo(info, "keyspace_misses"),
      hitRate: this.calculateHitRate(info)
    };
  }

  private parseInfo(info: string, key: string): number {
    const match = info.match(new RegExp(`${key}:(\\d+)`));
    return match ? parseInt(match[1]) : 0;
  }

  private calculateHitRate(info: string): number {
    const hits = this.parseInfo(info, "keyspace_hits");
    const misses = this.parseInfo(info, "keyspace_misses");
    const total = hits + misses;

    return total > 0 ? (hits / total) * 100 : 0;
  }
}

// Usage in API
export async function GET() {
  const monitor = new CacheMonitor();
  const stats = await monitor.getStats();

  return NextResponse.json(stats);
}
```

---

## Best Practices

- [ ] Use Next.js ISR for pages that change infrequently
- [ ] Use React Query for client-side data caching
- [ ] Add Redis for API response caching
- [ ] Set appropriate TTL based on data freshness requirements
- [ ] Implement cache invalidation on data updates
- [ ] Use cache tags for grouped invalidation
- [ ] Prevent cache stampede with locks
- [ ] Monitor cache hit rates
- [ ] Use CDN for static assets
- [ ] Cache at multiple layers (CDN → Redis → DB)
- [ ] Use materialized views for complex queries
- [ ] Implement stale-while-revalidate pattern
- [ ] Add cache headers for browser/CDN caching

---

## Caching Strategy by Data Type

| Data Type | Caching Strategy | TTL | Invalidation |
|-----------|------------------|-----|--------------|
| User profile | Redis + React Query | 5 min | On update |
| Product list | Next.js ISR + Redis | 1 min | On product change |
| Product detail | Next.js ISR | 5 min | On update |
| Shopping cart | Client-side only | Session | On change |
| Order history | React Query | 10 min | On new order |
| Static content | Next.js SSG + CDN | Forever | On deployment |
| Real-time data | No cache / Short TTL | 5s | Polling/WebSocket |

---

## Performance Comparison

| Scenario | Without Cache | With Redis | With Next.js ISR |
|----------|---------------|------------|------------------|
| Database query | 50-100ms | 5-10ms | 0ms (static) |
| API response time | 200ms | 20ms | 10ms |
| Database load | 1000 req/s | 10 req/s | 0 req/s |
| Cost (hypothetical) | $500/month | $100/month | $50/month |

---

## Analogy:

Think of caching like a **library system**:
- **Next.js ISR** = Photocopying popular books (static copies)
- **Redis** = Reserving books at the front desk (quick access)
- **React Query** = Your personal bookshelf (client-side cache)
- **Database** = Main archive (slow but complete)
- **Cache invalidation** = Updating outdated editions

---

### ⚡ One-liner to remember:
> Next.js caching = ISR for pages, Redis for API data, React Query for client-side, invalidate on updates.

---

## Q7. How would you design a scalable notification system?

### Concept:

A scalable notification system handles multi-channel delivery (email, SMS, push) with different priority levels, automatic retries, and failure handling.

### Requirements (Clarify First):

| Requirement | Details |
|-------------|---------|
| **Channels** | Email, SMS, Push notifications |
| **Volume** | 200+ daily (can scale to millions) |
| **Priority** | Critical alerts = real-time, Reports = batched |
| **Reliability** | Automatic retries, failure tracking |
| **Delivery Status** | Track sent/delivered/failed |

---

## Architecture

```
┌──────────┐     ┌─────────┐     ┌──────────┐     ┌───────────┐
│   App    │ ──▶ │   SQS   │ ──▶ │  Lambda  │ ──▶ │ Channels  │
│(Producer)│     │ (Queue) │     │ (Worker) │     │ SES/SNS   │
└──────────┘     └─────────┘     └──────────┘     └───────────┘
                      │                                 │
                      ▼                                 ▼
                 ┌─────────┐                     ┌───────────┐
                 │   DLQ   │                     │ DynamoDB  │
                 │(Failures)│                     │ (Status)  │
                 └─────────┘                     └───────────┘
```

**Components:**
1. **Producer (App)** - Sends notification requests
2. **SQS Queue** - Decouples producer/consumer, handles spikes
3. **Lambda Worker** - Processes notifications, sends to channels
4. **Channels** - SES (email), SNS (SMS), Firebase (push)
5. **DLQ** - Captures failed messages for retry
6. **DynamoDB** - Tracks delivery status

---

## Implementation

### 1. Producer (App sends to queue)

```typescript
// src/services/notificationService.ts
import { SQSClient, SendMessageCommand } from "@aws-sdk/client-sqs";

const sqs = new SQSClient({ region: "us-east-1" });

interface Notification {
  userId: string;
  type: "email" | "sms" | "push";
  priority: "high" | "low";
  template: string;
  data: Record<string, any>;
  recipient: {
    email?: string;
    phone?: string;
    deviceToken?: string;
  };
}

export async function sendNotification(notification: Notification) {
  const queueUrl = notification.priority === "high"
    ? process.env.HIGH_PRIORITY_QUEUE
    : process.env.LOW_PRIORITY_QUEUE;

  await sqs.send(
    new SendMessageCommand({
      QueueUrl: queueUrl,
      MessageBody: JSON.stringify({
        id: crypto.randomUUID(),
        userId: notification.userId,
        type: notification.type,
        priority: notification.priority,
        template: notification.template,
        data: notification.data,
        recipient: notification.recipient,
        timestamp: Date.now()
      }),
      MessageGroupId: notification.userId, // FIFO ordering per user
      MessageDeduplicationId: `${notification.userId}-${Date.now()}` // Prevent duplicates
    })
  );

  console.log(`Notification queued: ${notification.type} to ${notification.userId}`);
}

// Usage in application
await sendNotification({
  userId: "user123",
  type: "email",
  priority: "high",
  template: "appointment_reminder",
  data: {
    appointmentDate: "2026-01-10",
    doctorName: "Dr. Smith",
    appointmentTime: "10:00 AM"
  },
  recipient: {
    email: "patient@example.com"
  }
});
```

### 2. Consumer (Lambda processes queue)

```typescript
// lambda/notificationWorker.ts
import { SQSEvent, SQSHandler } from "aws-lambda";
import { SESClient, SendEmailCommand } from "@aws-sdk/client-ses";
import { SNSClient, PublishCommand } from "@aws-sdk/client-sns";
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, PutCommand } from "@aws-sdk/lib-dynamodb";

const ses = new SESClient({ region: "us-east-1" });
const sns = new SNSClient({ region: "us-east-1" });
const dynamodb = DynamoDBDocumentClient.from(new DynamoDBClient({}));

export const handler: SQSHandler = async (event: SQSEvent) => {
  for (const record of event.Records) {
    const notification = JSON.parse(record.body);

    try {
      let deliveryResult;

      switch (notification.type) {
        case "email":
          deliveryResult = await sendEmail(notification);
          break;
        case "sms":
          deliveryResult = await sendSMS(notification);
          break;
        case "push":
          deliveryResult = await sendPush(notification);
          break;
        default:
          throw new Error(`Unknown notification type: ${notification.type}`);
      }

      // Track delivery status in DynamoDB
      await dynamodb.send(
        new PutCommand({
          TableName: process.env.NOTIFICATIONS_TABLE,
          Item: {
            id: notification.id,
            userId: notification.userId,
            type: notification.type,
            status: "delivered",
            deliveryId: deliveryResult.messageId,
            timestamp: Date.now(),
            ttl: Math.floor(Date.now() / 1000) + 30 * 24 * 60 * 60 // 30 days
          }
        })
      );

      console.log(`✓ Notification delivered: ${notification.id}`);
    } catch (error) {
      console.error(`✗ Notification failed: ${notification.id}`, error);

      // Track failure
      await dynamodb.send(
        new PutCommand({
          TableName: process.env.NOTIFICATIONS_TABLE,
          Item: {
            id: notification.id,
            userId: notification.userId,
            type: notification.type,
            status: "failed",
            error: error.message,
            timestamp: Date.now()
          }
        })
      );

      // Throw error to move message to DLQ
      throw error;
    }
  }
};

async function sendEmail(notification: any) {
  const template = await loadTemplate(notification.template);
  const html = renderTemplate(template, notification.data);

  const result = await ses.send(
    new SendEmailCommand({
      Source: process.env.FROM_EMAIL,
      Destination: {
        ToAddresses: [notification.recipient.email]
      },
      Message: {
        Subject: { Data: notification.data.subject || "Notification" },
        Body: {
          Html: { Data: html }
        }
      }
    })
  );

  return { messageId: result.MessageId };
}

async function sendSMS(notification: any) {
  const result = await sns.send(
    new PublishCommand({
      PhoneNumber: notification.recipient.phone,
      Message: renderTemplate(notification.template, notification.data)
    })
  );

  return { messageId: result.MessageId };
}

async function sendPush(notification: any) {
  // Using Firebase Cloud Messaging (FCM)
  const admin = require("firebase-admin");

  const result = await admin.messaging().send({
    token: notification.recipient.deviceToken,
    notification: {
      title: notification.data.title,
      body: notification.data.body
    },
    data: notification.data
  });

  return { messageId: result };
}

function loadTemplate(templateName: string): string {
  // Load from S3 or database
  const templates = {
    appointment_reminder: `
      <h1>Appointment Reminder</h1>
      <p>You have an appointment with {{doctorName}} on {{appointmentDate}} at {{appointmentTime}}.</p>
    `
  };
  return templates[templateName] || "";
}

function renderTemplate(template: string, data: Record<string, any>): string {
  return template.replace(/\{\{(\w+)\}\}/g, (_, key) => data[key] || "");
}
```

### 3. Infrastructure Setup (AWS CDK)

```typescript
// lib/notification-stack.ts
import * as cdk from "aws-cdk-lib";
import * as sqs from "aws-cdk-lib/aws-sqs";
import * as lambda from "aws-cdk-lib/aws-lambda";
import * as dynamodb from "aws-cdk-lib/aws-dynamodb";
import { SqsEventSource } from "aws-cdk-lib/aws-lambda-event-sources";

export class NotificationStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string) {
    super(scope, id);

    // DynamoDB table for tracking notifications
    const notificationsTable = new dynamodb.Table(this, "NotificationsTable", {
      partitionKey: { name: "id", type: dynamodb.AttributeType.STRING },
      sortKey: { name: "timestamp", type: dynamodb.AttributeType.NUMBER },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
      timeToLiveAttribute: "ttl"
    });

    // Dead Letter Queue
    const dlq = new sqs.Queue(this, "NotificationDLQ", {
      retentionPeriod: cdk.Duration.days(14)
    });

    // High priority queue (real-time)
    const highPriorityQueue = new sqs.Queue(this, "HighPriorityQueue", {
      fifo: true,
      contentBasedDeduplication: true,
      visibilityTimeout: cdk.Duration.seconds(30),
      deadLetterQueue: {
        queue: dlq,
        maxReceiveCount: 3 // Retry 3 times before moving to DLQ
      }
    });

    // Low priority queue (batched)
    const lowPriorityQueue = new sqs.Queue(this, "LowPriorityQueue", {
      fifo: true,
      contentBasedDeduplication: true,
      visibilityTimeout: cdk.Duration.minutes(5),
      deadLetterQueue: {
        queue: dlq,
        maxReceiveCount: 3
      }
    });

    // Lambda worker
    const worker = new lambda.Function(this, "NotificationWorker", {
      runtime: lambda.Runtime.NODEJS_18_X,
      handler: "index.handler",
      code: lambda.Code.fromAsset("lambda"),
      timeout: cdk.Duration.seconds(30),
      environment: {
        NOTIFICATIONS_TABLE: notificationsTable.tableName,
        FROM_EMAIL: "noreply@example.com"
      }
    });

    // Grant permissions
    notificationsTable.grantWriteData(worker);

    // Connect queues to Lambda
    worker.addEventSource(
      new SqsEventSource(highPriorityQueue, {
        batchSize: 1 // Process one at a time for real-time
      })
    );

    worker.addEventSource(
      new SqsEventSource(lowPriorityQueue, {
        batchSize: 10 // Batch for efficiency
      })
    );
  }
}
```

---

## Key Design Decisions

| Decision | Why | Alternative |
|----------|-----|-------------|
| **SQS Queue** | Decouples producer/consumer, handles traffic spikes | Direct Lambda invocation (no buffering) |
| **Lambda** | Auto-scales, pay per use, serverless | EC2/ECS (more operational overhead) |
| **DLQ (Dead Letter Queue)** | Captures failures for debugging and manual retry | Lose failed messages |
| **DynamoDB** | Fast lookups, scales automatically | RDS (slower, needs provisioning) |
| **Priority Queues** | Separate SLA for critical vs non-critical | Single queue (everything same priority) |
| **FIFO Queues** | Ensures ordering per user | Standard queue (duplicates possible) |

---

## Scaling Strategies

| Volume | Architecture | Notes |
|--------|--------------|-------|
| **Low (200/day)** | Single SQS + Lambda | Simple, cost-effective |
| **Medium (10K/day)** | Add batching, increase Lambda concurrency | Optimize batch sizes |
| **High (100K+/day)** | Multiple queues by channel, increase Lambda reserved concurrency | Separate email/SMS/push queues |
| **Very High (1M+/day)** | Kinesis Data Streams for real-time, maintain SQS for batched | Use Kinesis for high-throughput streaming |

**Optimizations:**
```typescript
// Batching for low-priority notifications
const batchNotifications = async (notifications: Notification[]) => {
  // Group by type
  const grouped = notifications.reduce((acc, n) => {
    acc[n.type] = acc[n.type] || [];
    acc[n.type].push(n);
    return acc;
  }, {});

  // Send in batches
  for (const [type, batch] of Object.entries(grouped)) {
    if (type === "email") {
      await ses.sendBulkTemplatedEmail({
        Source: process.env.FROM_EMAIL,
        Destinations: batch.map(n => ({
          Destination: { ToAddresses: [n.recipient.email] },
          ReplacementTemplateData: JSON.stringify(n.data)
        })),
        Template: "notification-template"
      });
    }
  }
};
```

---

## Monitoring and Observability

```typescript
// CloudWatch metrics
import { CloudWatch } from "@aws-sdk/client-cloudwatch";

const cloudwatch = new CloudWatch({});

async function trackMetrics(notification: any, status: string) {
  await cloudwatch.putMetricData({
    Namespace: "Notifications",
    MetricData: [
      {
        MetricName: "NotificationsSent",
        Value: 1,
        Unit: "Count",
        Dimensions: [
          { Name: "Type", Value: notification.type },
          { Name: "Status", Value: status },
          { Name: "Priority", Value: notification.priority }
        ]
      }
    ]
  });
}

// CloudWatch alarm for DLQ
// Alert when messages pile up in DLQ
const dlqAlarm = new cloudwatch.Alarm(this, "DLQAlarm", {
  metric: dlq.metricApproximateNumberOfMessagesVisible(),
  threshold: 10,
  evaluationPeriods: 1,
  alarmDescription: "Alert when DLQ has messages"
});
```

---

## Real-World Experience

**From DrChrono EHR Integration:**

> "I built this notification system for DrChrono webhook integration. When patient data syncs from the EHR, it triggers appointment reminders via AWS SES. Using SQS decoupling, we handle 200+ daily notifications with automatic retries for failed deliveries. Failed messages go to DLQ where we can debug and manually retry. This reduced manual reminder processes by 75% and improved patient show-up rates by 20%."

**Key Learnings:**
- FIFO queues prevent duplicate notifications (important for patient communications)
- DLQ saved us during SES rate limit issues - messages weren't lost
- Separate high/low priority queues improved critical alert delivery time
- DynamoDB TTL automatically cleans up old notification records

---

## Testing Strategy

```typescript
// tests/notification.test.ts
import { sendNotification } from "../src/services/notificationService";
import { handler } from "../lambda/notificationWorker";

describe("Notification System", () => {
  it("should queue notification successfully", async () => {
    const result = await sendNotification({
      userId: "test123",
      type: "email",
      priority: "high",
      template: "test",
      data: { subject: "Test" },
      recipient: { email: "test@example.com" }
    });

    expect(result).toBeDefined();
  });

  it("should process email notification", async () => {
    const event = {
      Records: [
        {
          body: JSON.stringify({
            id: "test-id",
            type: "email",
            recipient: { email: "test@example.com" },
            data: { subject: "Test" }
          })
        }
      ]
    };

    await expect(handler(event as any)).resolves.not.toThrow();
  });

  it("should move to DLQ after max retries", async () => {
    // Simulate failure 3 times
    const failingEvent = {
      Records: [
        {
          body: JSON.stringify({
            type: "email",
            recipient: { email: "invalid@" } // Invalid email
          }),
          attributes: {
            ApproximateReceiveCount: "4" // Exceeds max retries
          }
        }
      ]
    };

    // Should throw and be moved to DLQ
    await expect(handler(failingEvent as any)).rejects.toThrow();
  });
});
```

---

## Best Practices

- [ ] Use separate queues for different priorities
- [ ] Implement idempotency (prevent duplicate sends)
- [ ] Set appropriate visibility timeouts
- [ ] Configure DLQ for failure handling
- [ ] Track delivery status in database
- [ ] Monitor queue depth and processing time
- [ ] Use batching for non-critical notifications
- [ ] Implement rate limiting for external APIs
- [ ] Add retry logic with exponential backoff
- [ ] Use templates for consistent messaging
- [ ] Test with production-like volumes
- [ ] Set up alerts for DLQ messages

---

## Cost Estimation

| Component | Cost (200 notifications/day) | Cost (10K/day) |
|-----------|------------------------------|----------------|
| SQS | ~$0.001 | ~$0.05 |
| Lambda | ~$0.01 | ~$0.50 |
| SES | ~$0.02 | ~$1.00 |
| DynamoDB | ~$0.01 | ~$0.25 |
| **Total** | **~$0.04/day** | **~$1.80/day** |

**Monthly cost:** $1.20 (low volume) to $54 (medium volume)

---

## Analogy:

Think of a notification system like a **postal service**:
- **SQS Queue** = Mailbox (collects letters)
- **Lambda Worker** = Mail carrier (delivers to destinations)
- **Priority Queues** = Express vs regular mail
- **DLQ** = Return to sender (undeliverable items)
- **DynamoDB** = Tracking system (delivery confirmation)
- **Channels** = Delivery methods (truck, plane, drone)

---

### ⚡ One-liner to remember:
> Producer → Queue (SQS) → Worker (Lambda) → Channels (SES/SNS) → Track (DynamoDB) + DLQ for failures.

---

## Q8. REST vs GraphQL - when to use each?

### Bottom Line (TL;DR):

**Use REST** when you want simplicity, caching, and clear resource boundaries.

**Use GraphQL** when you need flexible data fetching and fewer network calls.

---

## REST — Use When

### Why:
- Simple mental model (resources + HTTP verbs)
- Works great with HTTP caching, CDNs
- Easy to debug, monitor, and scale

### Best For:
- CRUD-heavy APIs
- Public APIs
- Microservices with clear boundaries

### Example:

```typescript
// REST - Multiple requests needed
GET /users/1
GET /users/1/orders

// Response 1
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com"
}

// Response 2
{
  "orders": [
    { "id": 101, "total": 99.99 },
    { "id": 102, "total": 149.99 }
  ]
}
```

**Trade-off:** Multiple calls, but responses are predictable and cacheable.

---

## GraphQL — Use When

### Why:
- Client asks exactly what it needs (no over/under-fetching)
- Fewer round trips
- Strongly typed schema

### Best For:
- Mobile apps (slow networks, battery concerns)
- Complex UIs (dashboards, feeds)
- Rapid frontend iteration

### Example:

```graphql
# GraphQL - One request, tailored response
query {
  user(id: 1) {
    name
    email
    orders {
      id
      total
    }
  }
}

# Response - Single call
{
  "data": {
    "user": {
      "name": "John Doe",
      "email": "john@example.com",
      "orders": [
        { "id": 101, "total": 99.99 },
        { "id": 102, "total": 149.99 }
      ]
    }
  }
}
```

**Trade-off:** One request, but requires GraphQL server setup and caching strategy.

---

## Key Differences (Quick Scan)

| Aspect | REST | GraphQL |
|--------|------|---------|
| **Data fetching** | Fixed responses | Client-defined |
| **Network calls** | Often many | Usually one |
| **Caching** | Native HTTP (CDN-friendly) | Manual (requires library) |
| **Learning curve** | Low | Medium |
| **Tooling** | Mature (Postman, curl) | Growing fast (Apollo, Relay) |
| **Versioning** | `/v1/`, `/v2/` | Schema evolution |
| **Over-fetching** | Common | Eliminated |
| **Under-fetching** | Common (N+1) | Eliminated |
| **File uploads** | Native | Requires extensions |
| **Real-time** | WebSockets/SSE | Subscriptions built-in |

---

## Practical Rule of Thumb

```
Backend-driven systems     → REST
Frontend-driven products   → GraphQL

Public API                 → REST
Internal product UI        → GraphQL

Simple CRUD                → REST
Complex data requirements  → GraphQL

Need HTTP caching          → REST
Need flexible queries      → GraphQL
```

---

## Code Comparison

### REST Implementation:

```typescript
// Express REST API
app.get("/api/users/:id", async (req, res) => {
  const user = await db.users.findOne({ id: req.params.id });
  res.json(user);
});

app.get("/api/users/:id/orders", async (req, res) => {
  const orders = await db.orders.findMany({ userId: req.params.id });
  res.json(orders);
});

// Client - Multiple requests
const user = await fetch("/api/users/1").then(r => r.json());
const orders = await fetch("/api/users/1/orders").then(r => r.json());
```

### GraphQL Implementation:

```typescript
// GraphQL Schema
const typeDefs = `
  type User {
    id: ID!
    name: String!
    email: String!
    orders: [Order!]!
  }

  type Order {
    id: ID!
    total: Float!
  }

  type Query {
    user(id: ID!): User
  }
`;

// Resolvers
const resolvers = {
  Query: {
    user: (_, { id }) => db.users.findOne({ id })
  },
  User: {
    orders: (user) => db.orders.findMany({ userId: user.id })
  }
};

// Client - Single request
const { data } = await client.query({
  query: gql`
    query GetUser($id: ID!) {
      user(id: $id) {
        name
        email
        orders {
          id
          total
        }
      }
    }
  `,
  variables: { id: "1" }
});
```

---

## When to Use Both (Hybrid)

Some companies use both:

```
REST       → Public API, webhooks, simple CRUD
GraphQL    → Internal dashboard, mobile app

Example: GitHub
- REST API v3 (public)
- GraphQL API v4 (flexible queries)
```

---

## Migration Path

**REST → GraphQL:**

```typescript
// Step 1: Wrap existing REST endpoints
const typeDefs = `
  type User {
    id: ID!
    name: String!
  }

  type Query {
    user(id: ID!): User
  }
`;

const resolvers = {
  Query: {
    user: async (_, { id }) => {
      // Call existing REST endpoint
      const response = await fetch(`/api/users/${id}`);
      return response.json();
    }
  }
};

// Step 2: Gradually replace with direct DB calls
// Step 3: Deprecate REST endpoints
```

---

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Using GraphQL for simple CRUD | Stick with REST |
| Not implementing caching in GraphQL | Use DataLoader, Apollo Cache |
| Over-nesting GraphQL queries | Limit query depth |
| Exposing REST endpoints without pagination | Always paginate |
| No rate limiting on GraphQL | Implement query cost analysis |

---

## Real-World Examples

**REST Wins:**
- Stripe API (payments)
- Twilio API (SMS)
- AWS S3 API (file storage)

**GraphQL Wins:**
- GitHub v4 API (complex queries)
- Shopify Storefront API (flexible product data)
- Airbnb (mobile app performance)

**Hybrid:**
- Netflix (REST for CDN, GraphQL for UI)
- Facebook (REST for legacy, GraphQL for new features)

---

## Performance Comparison

| Scenario | REST | GraphQL |
|----------|------|---------|
| **Fetch user + orders** | 2 requests (100ms each) = 200ms | 1 request (120ms) = 120ms |
| **Mobile on 3G** | Multiple round trips = slow | Single request = faster |
| **CDN caching** | Native support | Requires custom setup |
| **Over-fetching** | Returns all fields | Returns only requested |

---

## Analogy:

**REST** = **Restaurant menu**
- Fixed dishes (endpoints)
- Order item by item (separate calls)
- Clear prices (predictable responses)
- Easy for everyone to understand

**GraphQL** = **Build-your-own-bowl**
- Pick exactly what you want (custom queries)
- One order, custom ingredients (single request)
- Requires understanding the menu (learning curve)
- Perfect for picky eaters (flexible clients)

---

### ⚡ One-liner to remember:
> REST = Simple, cacheable, predictable. GraphQL = Flexible, efficient, tailored.

---

## Q9. Why is API versioning needed and what are its advantages?

### Bottom Line:

**API versioning lets you change an API without breaking existing clients.**

---

## Why API Versioning is Needed

As APIs evolve, you will:
- Add new fields
- Change response structure
- Deprecate or remove features
- Fix design mistakes

**Without versioning → client apps break unexpectedly.**

---

## Common Versioning Approaches

### 1. URL Versioning (Most Common)

```
GET /api/v1/users
GET /api/v2/users
```

**Use when:** Public or long-lived APIs
**Pros:** Clear, explicit, easy to manage
**Cons:** URL duplication

**Example:**
```typescript
// v1: Returns basic user data
app.get("/api/v1/users/:id", (req, res) => {
  const user = db.users.findOne({ id: req.params.id });
  res.json({ id: user.id, name: user.name });
});

// v2: Returns additional data
app.get("/api/v2/users/:id", (req, res) => {
  const user = db.users.findOne({ id: req.params.id });
  res.json({
    id: user.id,
    name: user.name,
    email: user.email,
    profile: user.profile
  });
});
```

### 2. Header Versioning

```
Accept: application/vnd.myapi.v2+json
API-Version: 2
```

**Use when:** You want clean URLs
**Pros:** REST-pure, flexible
**Cons:** Harder to debug, less visible

**Example:**
```typescript
app.get("/api/users/:id", (req, res) => {
  const version = req.headers["api-version"] || "1";
  const user = db.users.findOne({ id: req.params.id });

  if (version === "1") {
    res.json({ id: user.id, name: user.name });
  } else if (version === "2") {
    res.json({
      id: user.id,
      name: user.name,
      email: user.email,
      profile: user.profile
    });
  }
});
```

### 3. Query Parameter Versioning

```
GET /api/users?version=2
```

**Use when:** Internal or temporary APIs
**Pros:** Simple, no route changes
**Cons:** Easy to misuse, less strict

---

## Advantages of API Versioning

| Advantage | Description |
|-----------|-------------|
| **Backward compatibility** | Old clients keep working while new clients upgrade |
| **Safe evolution** | Refactor or redesign without fear |
| **Clear deprecation** | Gradually sunset old versions instead of sudden breaks |
| **Parallel development** | Multiple teams can move at different speeds |
| **Consumer trust** | Clients know changes won't surprise them |

---

## Versioning Strategy

### Breaking Changes (Version Required):

```typescript
// v1: Returns array
GET /api/v1/users
{ "users": [...] }

// v2: Returns object with metadata (BREAKING)
GET /api/v2/users
{
  "data": [...],
  "pagination": { "page": 1, "total": 100 }
}
```

### Non-Breaking Changes (No Version Needed):

```typescript
// v1: Original
GET /api/v1/users
{ "id": 1, "name": "John" }

// v1: Add optional field (NOT BREAKING)
GET /api/v1/users
{ "id": 1, "name": "John", "avatar": "url" }  // New field, backward compatible
```

---

## Best Practices

- [ ] Version only when breaking changes occur
- [ ] Keep versions few (max 2-3 active)
- [ ] Document each version thoroughly
- [ ] Announce deprecations early (6-12 months notice)
- [ ] Avoid versioning for small, backward-compatible changes
- [ ] Set sunset dates for old versions
- [ ] Monitor usage of deprecated versions
- [ ] Provide migration guides

---

## Deprecation Strategy

```typescript
// Step 1: Announce deprecation
app.get("/api/v1/users", (req, res) => {
  res.setHeader("X-API-Deprecated", "true");
  res.setHeader("X-API-Sunset-Date", "2026-12-31");
  res.setHeader("X-API-Migration-Guide", "https://docs.example.com/migrate-v2");

  // Still return v1 response
  const users = db.users.findAll();
  res.json(users);
});

// Step 2: After sunset date, return error
app.get("/api/v1/users", (req, res) => {
  res.status(410).json({
    error: "API version v1 has been sunset",
    message: "Please upgrade to v2",
    migrationGuide: "https://docs.example.com/migrate-v2"
  });
});
```

---

## Real-World Example

**Twitter API Evolution:**

```
v1.1 (2012-2018)
GET /1.1/statuses/user_timeline.json

v2 (2020+)
GET /2/users/:id/tweets
```

**Changes:**
- New data model
- More granular permissions
- Better pagination
- Standardized error responses

**Strategy:**
- Ran both versions simultaneously for 2 years
- Provided migration tools
- Deprecated v1.1 gradually

---

## Comparison Table

| Approach | URL Clarity | Debugging | Caching | Adoption |
|----------|-------------|-----------|---------|----------|
| URL versioning | ✅ Excellent | ✅ Easy | ✅ Simple | 🟢 High |
| Header versioning | ⚠️ Hidden | ❌ Hard | ⚠️ Complex | 🟡 Medium |
| Query parameter | ⚠️ Unclear | ⚠️ Medium | ❌ Tricky | 🔴 Low |

---

## Rule of Thumb

```
If a change can break a client     → Version it
If it only adds optional data      → Don't version

Examples:
- Rename field          → Version (breaking)
- Remove field          → Version (breaking)
- Change data type      → Version (breaking)
- Add optional field    → No version (compatible)
- Add new endpoint      → No version (compatible)
```

---

## Analogy:

Think of API versioning like **software releases**:
- **v1** = First stable version (Windows 10)
- **v2** = Major update with new features (Windows 11)
- **Both run simultaneously** = Users upgrade when ready
- **Deprecation** = End of support for old version
- **Migration guide** = Upgrade documentation

---

### ⚡ One-liner to remember:
> Version APIs to evolve safely — breaking changes get new versions, additions don't.

---

## Q10. What is idempotency and why is it critical for APIs?

### Definition:

**An operation is idempotent if repeating it multiple times gives the same result as doing it once.**

---

## Simple Analogy

**Light switch:**
- Switch ON once → light is ON
- Switch ON again → light is still ON
- **No extra effect after the first action → idempotent**

---

## HTTP Method Examples

### Idempotent Methods:

```typescript
// GET - Always returns same data
GET /users/1
GET /users/1  // Same result

// PUT - Replaces resource (same result every time)
PUT /users/1
{
  "name": "Shyam"
}
// Send 1 time or 10 times → user name remains "Shyam"

// DELETE - Deletes resource
DELETE /users/1
DELETE /users/1  // Already deleted, same state

// PATCH - Can be designed to be idempotent
PATCH /users/1
{
  "name": "Shyam"  // Setting absolute value (idempotent)
}
```

### Non-Idempotent Method:

```typescript
// POST - Creates new resource each time
POST /payments
{
  "amount": 100
}
// Send it twice → ₹200 charged ❌ DANGEROUS
```

---

## Why Idempotency Matters

| Scenario | Without Idempotency | With Idempotency |
|----------|---------------------|------------------|
| **Network retry** | Duplicate payment | Safe retry |
| **Mobile app resend** | Double order | Same order |
| **Gateway timeout** | Uncertain state | Predictable result |
| **User double-click** | Multiple charges | Single charge |

**Critical for:**
- Payment APIs
- Order creation
- Inventory updates
- Financial transactions

---

## Real-World Problem

```typescript
// User submits payment
POST /payments { amount: 1000 }

// Network timeout (but server processed it)
// User clicks "Retry"
POST /payments { amount: 1000 }

// Result: ₹2000 charged 😱
```

---

## HTTP Methods Summary

| Method | Idempotent? | Why |
|--------|-------------|-----|
| **GET** | ✅ Yes | Only reads, no side effects |
| **PUT** | ✅ Yes | Replaces entire resource with same data |
| **DELETE** | ✅ Yes | Deleting twice has same effect |
| **PATCH** | ⚠️ Maybe | Depends on implementation |
| **POST** | ❌ No | Creates new resource each time |

---

## PATCH: Idempotent vs Non-Idempotent

**Non-Idempotent PATCH:**
```typescript
// Increments balance
PATCH /accounts/1
{
  "operation": "increment",
  "balance": 100
}
// Called twice → balance += 200 ❌
```

**Idempotent PATCH:**
```typescript
// Sets absolute value
PATCH /accounts/1
{
  "balance": 500
}
// Called twice → balance = 500 ✅
```

---

## Best Practices

- [ ] Use GET for safe, read-only operations
- [ ] Use PUT for full resource replacement
- [ ] Use DELETE for removals
- [ ] **Avoid POST for critical operations** (use idempotency keys instead)
- [ ] Design PATCH to be idempotent when possible
- [ ] Always implement idempotency for financial operations

---

## Analogy:

**Idempotent** = **Turning off a light that's already off**
- First time: Light goes OFF
- Second time: Light stays OFF (no change)
- Safe to repeat

**Non-Idempotent** = **Putting money in a piggy bank**
- First time: Add $10
- Second time: Add another $10
- Total: $20 (different result)

---

### ⚡ One-liner to remember:
> Idempotent = Safe to retry. Critical for payments, retries, and reliability.

---

## Q11. How do you design an idempotent payment API?

### Bottom Line:

**Use an Idempotency Key to ensure retries don't duplicate payments.**

---

## The Problem

```typescript
// Payment request times out
POST /payments { amount: 100 }
// Timeout... but payment processed?

// User retries
POST /payments { amount: 100 }
// Result: ₹200 charged 😱
```

---

## Solution: Idempotency Key

### Step 1: Client sends unique key

```typescript
POST /payments
Idempotency-Key: abc-123-uuid
Content-Type: application/json

{
  "amount": 100,
  "currency": "INR",
  "userId": "user123"
}
```

### Step 2: Server checks the key

```typescript
app.post("/payments", async (req, res) => {
  const idempotencyKey = req.headers["idempotency-key"];

  // Check if we've seen this key before
  const existing = await db.idempotencyKeys.findOne({
    key: idempotencyKey,
    userId: req.body.userId
  });

  if (existing) {
    // Already processed → return stored response
    return res.status(existing.statusCode).json(existing.response);
  }

  // New request → process payment
  // ...
});
```

### Step 3: Process & store result

```typescript
app.post("/payments", async (req, res) => {
  const idempotencyKey = req.headers["idempotency-key"];

  // Check existing (as above)
  const existing = await checkExisting(idempotencyKey, req.body.userId);
  if (existing) return res.status(existing.statusCode).json(existing.response);

  // Lock to prevent race conditions
  const lock = await acquireLock(idempotencyKey);

  try {
    // Process payment
    const payment = await processPayment(req.body);

    // Store result
    await db.idempotencyKeys.create({
      key: idempotencyKey,
      userId: req.body.userId,
      requestHash: hashRequest(req.body),
      response: {
        paymentId: payment.id,
        status: "SUCCESS",
        amount: payment.amount
      },
      statusCode: 201,
      createdAt: new Date(),
      expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000) // 24 hours
    });

    return res.status(201).json({
      paymentId: payment.id,
      status: "SUCCESS",
      amount: payment.amount
    });
  } finally {
    await releaseLock(lock);
  }
});
```

### Step 4: Return same response on retry

```typescript
// First request
POST /payments
Idempotency-Key: abc-123

Response: 201 Created
{
  "paymentId": "pay_001",
  "status": "SUCCESS",
  "amount": 100
}

// Retry (same key)
POST /payments
Idempotency-Key: abc-123

Response: 201 Created (from cache)
{
  "paymentId": "pay_001",
  "status": "SUCCESS",
  "amount": 100
}
// ✅ No duplicate charge!
```

---

## Complete Implementation

```typescript
import { v4 as uuidv4 } from "uuid";
import crypto from "crypto";

// Database schema
interface IdempotencyRecord {
  key: string;
  userId: string;
  requestHash: string;
  response: any;
  statusCode: number;
  createdAt: Date;
  expiresAt: Date;
}

// Middleware to enforce idempotency key
function requireIdempotencyKey(req, res, next) {
  if (!req.headers["idempotency-key"]) {
    return res.status(400).json({
      error: "Idempotency-Key header is required"
    });
  }
  next();
}

// Hash request body for validation
function hashRequest(body: any): string {
  return crypto.createHash("sha256")
    .update(JSON.stringify(body))
    .digest("hex");
}

// Payment endpoint
app.post(
  "/payments",
  requireIdempotencyKey,
  async (req, res) => {
    const idempotencyKey = req.headers["idempotency-key"];
    const userId = req.body.userId;

    // Check if already processed
    const existing = await db.idempotencyKeys.findOne({
      key: idempotencyKey,
      userId: userId
    });

    if (existing) {
      // Validate request hasn't changed
      const currentHash = hashRequest(req.body);
      if (currentHash !== existing.requestHash) {
        return res.status(422).json({
          error: "Request body has changed for this idempotency key"
        });
      }

      // Return cached response
      console.log("Returning cached response for:", idempotencyKey);
      return res.status(existing.statusCode).json(existing.response);
    }

    // Acquire distributed lock (Redis)
    const lockKey = `payment-lock:${idempotencyKey}`;
    const locked = await redis.set(lockKey, "1", "EX", 30, "NX");

    if (!locked) {
      return res.status(409).json({
        error: "Payment is being processed, please retry"
      });
    }

    try {
      // Process payment
      const payment = await stripe.charges.create({
        amount: req.body.amount,
        currency: req.body.currency,
        source: req.body.source,
        metadata: { userId: req.body.userId }
      });

      const response = {
        paymentId: payment.id,
        status: payment.status,
        amount: payment.amount,
        currency: payment.currency
      };

      // Store result with TTL
      await db.idempotencyKeys.create({
        key: idempotencyKey,
        userId: userId,
        requestHash: hashRequest(req.body),
        response: response,
        statusCode: 201,
        createdAt: new Date(),
        expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000)
      });

      return res.status(201).json(response);
    } catch (error) {
      // Store failure too
      await db.idempotencyKeys.create({
        key: idempotencyKey,
        userId: userId,
        requestHash: hashRequest(req.body),
        response: { error: error.message },
        statusCode: 500,
        createdAt: new Date(),
        expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000)
      });

      return res.status(500).json({ error: error.message });
    } finally {
      // Release lock
      await redis.del(lockKey);
    }
  }
);

// Client usage
const idempotencyKey = uuidv4(); // Generate once per payment attempt

async function makePayment() {
  try {
    const response = await fetch("/payments", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Idempotency-Key": idempotencyKey // Same key for retries
      },
      body: JSON.stringify({
        amount: 10000,
        currency: "INR",
        userId: "user123",
        source: "tok_visa"
      })
    });

    return await response.json();
  } catch (error) {
    // Safe to retry with same key
    console.log("Retrying with same idempotency key...");
    return makePayment(); // Uses same idempotencyKey
  }
}
```

---

## Flow Diagram

```
Client                    Server                     Database
  │                         │                            │
  ├─── POST + Key ─────────▶│                            │
  │                         ├─── Check key ────────────▶│
  │                         │◀─── Not found ─────────────┤
  │                         │                            │
  │                         ├─── Acquire lock ─────────▶│
  │                         │◀─── Lock acquired ─────────┤
  │                         │                            │
  │                         ├─── Process payment         │
  │                         │                            │
  │                         ├─── Store result ─────────▶│
  │                         │◀─── Stored ────────────────┤
  │                         │                            │
  │◀─── Response ───────────┤                            │
  │                         │                            │
  ├─── RETRY (same key) ───▶│                            │
  │                         ├─── Check key ────────────▶│
  │                         │◀─── Found ─────────────────┤
  │◀─── Cached response ────┤                            │
```

---

## Best Practices

- [ ] **Key should be UUID** - Unique per payment attempt
- [ ] **Key scoped per user** - Different users can use same key
- [ ] **Expire keys after 24 hours** - Clean up old records
- [ ] **Lock during processing** - Prevent race conditions
- [ ] **Store request hash** - Validate request hasn't changed
- [ ] **Store both success and failure** - Cache all outcomes
- [ ] **Client generates key** - Server shouldn't create it
- [ ] **Use distributed lock** - For multi-server setups

---

## Real-World Examples

**Stripe:**
```bash
curl https://api.stripe.com/v1/charges \
  -H "Idempotency-Key: {YOUR_KEY}" \
  -u sk_test_xxx: \
  -d amount=1000
```

**PayPal:**
```bash
curl -X POST https://api.paypal.com/v2/checkout/orders \
  -H "PayPal-Request-Id: {YOUR_KEY}"
```

**Industry standard** for payment APIs.

---

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Server generates key | Client must generate |
| No expiration | Set 24-48 hour TTL |
| No locking | Use Redis locks for race conditions |
| Key not validated | Check request hash matches |
| Only storing success | Store failures too |

---

## Analogy:

**Idempotency Key** = **Ticket stub at a concert**
- First entry: Stub checked, you enter
- Try to enter again: Stub already used, denied
- Same stub = same result (no duplicate entry)

---

### ⚡ One-liner to remember:
> Idempotency keys make retries safe — non-negotiable for payments.

---
