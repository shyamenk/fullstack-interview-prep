# Node.js Interview Questions & Answers

## Q1. How does Node.js handle many concurrent requests with a single thread?

### Concept:

Node.js uses a **single-threaded event loop** with **non-blocking I/O**.

- The main thread handles incoming requests
- If a request involves I/O (like reading a file or DB query), Node.js delegates it to the **worker pool** or OS-level async API
- Once the operation completes, the callback is added to the event loop to be executed
- This allows Node.js to handle thousands of concurrent requests without creating new threads for each one

### How it works:

1. **Request arrives** → Main thread receives it
2. **I/O operation needed?** → Delegate to libuv worker pool or OS async API
3. **Main thread is free** → Can handle more incoming requests
4. **I/O completes** → Callback queued in event loop
5. **Event loop** → Executes callback when call stack is empty

### Analogy:

Think of a **waiter (Node.js thread)** taking orders.

If cooking (I/O) takes time, the order goes to the **kitchen (worker pool)**, and the waiter moves on to take other orders.

When the dish is ready, the waiter delivers it.

### ⚡ One-liner to remember:
> Node.js = Single thread + non-blocking I/O + event loop = handle many concurrent requests efficiently.

---

## Q2. What is middleware in Express? Example use case?

### Concept:

Middleware is a function in Express that runs between the client request and server response.

- It can modify `req` and `res` objects
- Perform checks, logging, or handle errors
- Call `next()` to pass control to the next middleware or route handler

### Example Use Case:

```javascript
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`); // logging
  next(); // pass to next middleware or route handler
});
```

### Common middleware types:

- **Application-level:** `app.use(middleware)`
- **Router-level:** `router.use(middleware)`
- **Error-handling:** `app.use((err, req, res, next) => {...})`
- **Built-in:** `express.json()`, `express.static()`
- **Third-party:** `cors()`, `morgan()`

### Analogy:

Middleware is like a **security checkpoint at an airport**.

Every passenger (request) passes through it for checks (authentication, logging, etc.) before reaching the plane (final response).

### ⚡ One-liner to remember:
> Middleware = Functions that run between request and response, with access to req, res, and next().

---

## Q3. How do you handle authentication and authorization in a Node.js app?

### Concept:

- **Authentication:** Verifying the user's identity (who they are)
- **Authorization:** Determining what resources the user can access (permissions)

### Example Flow (Email + Password + JWT):

1. **User signs up** → Server hashes password with salt (using bcrypt) → Stores in DB
2. **User logs in** → Server hashes entered password → Compares with DB hash
3. **If valid** → Server generates JWT token → Sends to client
4. **For each protected route request** → Client sends JWT in header → Server verifies token → Grants access

### Common implementations:

- **Session-based:** Server stores session, sends session ID to client via cookie
- **Token-based (JWT):** Stateless, self-contained tokens
- **OAuth 2.0:** Third-party authentication (Google, GitHub, etc.)
- **Passport.js:** Popular authentication middleware for Node.js

### Code example:

```javascript
// Authentication middleware
const jwt = require('jsonwebtoken');

function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token provided' });

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (err) {
    res.status(401).json({ error: 'Invalid token' });
  }
}

// Authorization middleware
function authorize(roles) {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
}
```

### Analogy:

- **Authentication** → Showing your ID at the entrance (proving who you are)
- **Authorization** → Getting a keycard that opens only the rooms you're allowed to enter (what you can access)

### ⚡ One-liner to remember:
> Authentication = "Who are you?" Authorization = "What can you do?"

---

## Q4. What's the difference between JWT and sessions/cookies?

### JWT (JSON Web Token):

- A **stateless, self-contained token** used for authentication and optionally for authorization
- Contains user info (like user ID, roles) and a signature
- Sent in HTTP headers (usually `Authorization: Bearer <token>`)
- Server does **not need to store session state** — verification happens using the token signature
- Lightweight and scalable (great for APIs, microservices)

**Authentication vs Authorization:**
- JWT authenticates the user (verifies identity)
- Can also carry roles/permissions, helping with authorization

### Sessions/Cookies:

- **Session:** Server stores user info in memory or database, and assigns a session ID
- **Cookie:** Small data stored in the browser, usually contains the session ID
- Server checks the session ID to authenticate the user
- Can be **vulnerable if not secured** (HTTPOnly, Secure, SameSite flags)
- State is stored on the server (stateful), which can be a scaling bottleneck

### Comparison Table:

| Feature          | JWT                             | Session + Cookie                  |
|-----------------|---------------------------------|----------------------------------|
| State           | Stateless                        | Stateful                        |
| Storage         | Client-side (token)              | Server-side (session ID)         |
| Scalability     | Easy to scale                     | Requires shared session storage  |
| Security        | Signature prevents tampering      | Needs secure cookies (HTTPOnly) |
| Use case        | APIs, microservices, mobile apps | Traditional web apps             |

### Analogy:

- **JWT** → A sealed envelope with your ID and permissions, anyone can check it without asking the issuer again
- **Session + Cookie** → You get a numbered ticket at the counter, server keeps track of your info; you must present the ticket each time

### ⚡ One-liner to remember:
> JWT = stateless token. Session + Cookie = stateful server storage.

---