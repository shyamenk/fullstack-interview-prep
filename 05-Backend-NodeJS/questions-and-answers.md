# Node.js Interview Questions & Answers

## Table of Contents

1. [How does Node.js handle many concurrent requests with a single thread?](#q1-how-does-nodejs-handle-many-concurrent-requests-with-a-single-thread)
2. [What is middleware in Express? Example use case?](#q2-what-is-middleware-in-express-example-use-case)

---

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
