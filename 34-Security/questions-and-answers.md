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
