# Express.js Interview Questions & Answers

## Table of Contents

1. [Middleware Types & Execution](#q1-what-are-the-types-of-middleware-in-express)
2. [app.use() vs app.get()](#q2-what-is-the-difference-between-appuse-and-appget)
3. [The `next` Function](#q3-what-is-the-role-of-the-next-function)
4. [Error Handling (Sync vs Async)](#q4-how-does-express-handle-errors)
5. [req.params vs req.query vs req.body](#q5-what-is-the-difference-between-reqparams-reqquery-and-reqbody)
6. [res.send() vs res.json() vs res.end()](#q6-what-is-the-difference-between-ressend-resjson-and-resend)
7. [Express Router](#q7-what-is-the-express-router-and-why-use-it)
8. [Body Parsing](#q8-how-do-you-handle-request-bodies-in-express)
9. [Serving Static Files](#q9-how-do-you-serve-static-files-in-express)
10. [CORS (Cross-Origin Resource Sharing)](#q10-how-do-you-handle-cors-in-express)
11. [Security Headers (Helmet)](#q11-how-do-you-secure-http-headers-in-express)
12. [Rate Limiting](#q12-how-do-you-implement-rate-limiting-in-express)
13. [Route Chaining](#q13-what-is-approute-used-for)
14. [Cookie Management](#q14-how-do-you-handle-cookies-in-express)
15. [File Uploads](#q15-how-do-you-handle-file-uploads-in-express)
16. [Validation](#q16-how-do-you-validate-input-data-in-express)
17. [Template Engines](#q17-what-are-template-engines-in-express)
18. [App Settings (app.set vs app.use)](#q18-what-is-the-difference-between-appset-and-appuse)
19. [Trust Proxy](#q19-what-is-trust-proxy-and-when-do-you-need-it)
20. [Graceful Shutdown](#q20-how-do-you-handle-graceful-shutdowns-in-express)

---

## Q1. What are the types of middleware in Express?
Concept:
Middleware functions have access to the request object (`req`), the response object (`res`), and the next function (`next`).
Express has 5 types:
1.  **Application-level:** Bound to `app` (e.g., `app.use()`).
2.  **Router-level:** Bound to `express.Router()`.
3.  **Error-handling:** Takes 4 arguments `(err, req, res, next)`.
4.  **Built-in:** `express.static`, `express.json`.
5.  **Third-party:** `morgan`, `cors`, `helmet`.

Example:
```javascript
// Application-level
app.use((req, res, next) => {
  console.log('Time:', Date.now());
  next();
});

// Error-handling
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Broken!');
});
```

Analogy:
An assembly line in a factory. The raw material (Request) passes through various stations (Middleware). One station paints it, another checks for defects (Error handling), another packages it, until it's finished (Response).

⚡ One-liner to remember:
Middleware are functions that run in the middle of the request-response cycle.

## Q2. What is the difference between `app.use()` and `app.get()`?
Concept:
- **`app.use()`:** Mounts middleware for **all** HTTP methods (GET, POST, PUT, etc.) on the specified path (or all paths if none specified).
- **`app.get()`:** Mounts middleware/handlers **only** for HTTP GET requests on the specified path.

Example:
```javascript
// Matches GET, POST, PUT, DELETE... on /api
app.use('/api', (req, res, next) => {
  // Check auth for ALL API calls
  next();
});

// Matches ONLY GET on /api/users
app.get('/api/users', (req, res) => {
  res.json({ users: [] });
});
```

Analogy:
- `app.use()`: A security guard at the building entrance. Everyone (all methods) must pass him.
- `app.get()`: A receptionist at the specific "Deliveries" desk. Only delivery drivers (GET requests) go there.

⚡ One-liner to remember:
`use()` matches any method; `get()` matches only GET.

## Q3. What is the role of the `next` function?
Concept:
`next` is a callback function passed to middleware.
- Calling `next()` passes control to the **next middleware** function.
- Calling `next(error)` skips all remaining non-error middleware and goes to **error handlers**.
- Calling `next('route')` skips the rest of the middleware in the current router stack.

Example:
```javascript
app.get('/user/:id', (req, res, next) => {
  if (req.params.id === '0') next('route'); // Skip to next route
  else next(); // Go to next middleware in this stack
}, (req, res, next) => {
  res.send('Regular');
});

app.get('/user/:id', (req, res, next) => {
  res.send('Special'); // '0' lands here
});
```

Analogy:
A relay race baton. You run your leg (middleware logic), then pass the baton (`next()`) to the next runner. If you drop the baton and yell "Help!" (`next(err)`), the medics (Error Handler) come instead of the next runner.

⚡ One-liner to remember:
`next()` says "I'm done, your turn"; `next(err)` says "Something broke, help!".

## Q4. How does Express handle errors?
Concept:
- **Synchronous Errors:** Express catches them automatically.
- **Asynchronous Errors (Pre-Express 5):** You MUST catch them and pass them to `next(err)`. If you throw inside an async function without try/catch, the app crashes.
- **Express 5:** Handles rejected promises automatically.

Example:
```javascript
// Async Error Handling (Express 4)
app.get('/', async (req, res, next) => {
  try {
    const user = await getUser();
    throw new Error("Oops");
  } catch (err) {
    next(err); // Must pass to next explicitly
  }
});

// Error Handler (Must have 4 args)
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message });
});
```

Analogy:
- Sync: You trip on the sidewalk, a safety net catches you immediately.
- Async: You trip while jumping out of a plane. You must manually pull the chute (`next(err)`). If you don't, you crash.

⚡ One-liner to remember:
Catch async errors and pass them to `next(err)`; define error handlers with 4 arguments.

## Q5. What is the difference between `req.params`, `req.query`, and `req.body`?
Concept:
- **`req.params`:** Route parameters (part of the URL path). e.g., `/users/:id`.
- **`req.query`:** URL query string parameters. e.g., `/users?sort=desc`.
- **`req.body`:** Data sent in the request body (JSON, form data). Requires parsing middleware.

Example:
```javascript
// URL: POST /users/123?active=true
// Body: { "name": "Alice" }

app.post('/users/:id', (req, res) => {
  console.log(req.params.id); // "123"
  console.log(req.query.active); // "true"
  console.log(req.body.name); // "Alice"
});
```

Analogy:
- `params`: The address on the envelope (House Number).
- `query`: Notes written on the outside of the envelope ("Urgent", "Confidential").
- `body`: The letter inside the envelope.

⚡ One-liner to remember:
`params` = URL path; `query` = ?search; `body` = JSON payload.

## Q6. What is the difference between `res.send()`, `res.json()`, and `res.end()`?
Concept:
- **`res.send(body)`:** Sends the HTTP response. Automatically sets `Content-Type` based on body type (HTML for strings, JSON for objects).
- **`res.json(body)`:** Sends a JSON response. Forces `Content-Type: application/json` and formats the output.
- **`res.end()`:** Ends the response process quickly without data. Used if you just want to send a status (e.g., 404) or stream data manually.

Example:
```javascript
res.send('<h1>Hi</h1>'); // Content-Type: text/html
res.send({ a: 1 });      // Content-Type: application/json
res.json({ a: 1 });      // Content-Type: application/json (Explicit)
res.status(404).end();   // Empty response
```

Analogy:
- `res.send()`: Sending a package (detects if it's a letter or a box).
- `res.json()`: Sending a package specifically labeled "Fragile/JSON".
- `res.end()`: Hanging up the phone.

⚡ One-liner to remember:
Use `res.json()` for APIs, `res.send()` for general data, `res.end()` to just stop.

## Q7. What is the Express Router and why use it?
Concept:
A `Router` instance is a complete middleware and routing system; essentially a "mini-app". It allows you to modularize your routes into separate files.

Example:
```javascript
// users.js
const router = express.Router();
router.get('/', (req, res) => { ... });
router.get('/:id', (req, res) => { ... });
module.exports = router;

// app.js
const userRoutes = require('./users');
app.use('/users', userRoutes); // Mounts at /users
```

Analogy:
Instead of one giant map for the whole world, you have an atlas with separate pages for each country (Router). You combine them to make the full map (App).

⚡ One-liner to remember:
Router = Modular mini-applications to keep your code organized.

## Q8. How do you handle request bodies in Express?
Concept:
Express (since v4.16.0) has built-in middleware for body parsing. You must mount them to access `req.body`.
- `express.json()`: Parses JSON.
- `express.urlencoded()`: Parses URL-encoded form data.

Example:
```javascript
const express = require('express');
const app = express();

app.use(express.json()); // Parses application/json
app.use(express.urlencoded({ extended: true })); // Parses form data

app.post('/', (req, res) => {
  console.log(req.body); // Now defined
});
```

Analogy:
A translator. The raw request speaks a binary language. The middleware translates it into a JavaScript Object (`req.body`) so you can understand it.

⚡ One-liner to remember:
Use `app.use(express.json())` to unlock the data inside `req.body`.

## Q9. How do you serve static files in Express?
Concept:
Use the built-in `express.static` middleware. You can serve images, CSS, JS files directly from a folder.

Example:
```javascript
// Serve files from 'public' directory
app.use(express.static('public'));

// Access: http://localhost:3000/images/logo.png
// (assuming public/images/logo.png exists)
```

Analogy:
A "Take One" pamphlet stand in a lobby. You don't need a receptionist (route handler) to hand them out; people just take them directly from the stand (static folder).

⚡ One-liner to remember:
`express.static('folder')` exposes a directory to the world.

## Q10. How do you handle CORS in Express?
Concept:
Cross-Origin Resource Sharing (CORS) restricts browser requests from other domains. Use the `cors` middleware package to allow/configure access.

Example:
```javascript
const cors = require('cors');

// Allow everyone
app.use(cors());

// Or restrict
app.use(cors({
  origin: 'https://example.com',
  methods: ['GET', 'POST']
}));
```

Analogy:
A bouncer at a club. By default, only locals (same domain) enter. CORS tells the bouncer to let in people from "Specific Town" (origin) or "Everyone" (`*`).

⚡ One-liner to remember:
Use the `cors` package to whitelist who can call your API.

## Q11. How do you secure HTTP headers in Express?
Concept:
Use the `helmet` middleware. It automatically sets various HTTP headers (like `X-Frame-Options`, `Strict-Transport-Security`, `X-Content-Type-Options`) to protect against common attacks (XSS, Clickjacking).

Example:
```javascript
const helmet = require('helmet');
app.use(helmet());
```

Analogy:
Putting on a seatbelt, helmet, and kneepads before riding a bike. It doesn't guarantee you won't crash, but it protects you from standard injuries.

⚡ One-liner to remember:
`app.use(helmet())` adds a layer of security headers instantly.

## Q12. How do you implement rate limiting in Express?
Concept:
Use `express-rate-limit` middleware to limit repeated requests to public APIs and/or endpoints such as password reset. Prevents DoS/Brute Force attacks.

Example:
```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: "Too many requests, please try again later."
});

app.use('/api', limiter);
```

Analogy:
A "One Sample Per Person" sign at Costco. Even if you want more, the attendant stops you after your limit.

⚡ One-liner to remember:
Rate limiting stops one user from flooding your server.

## Q13. What is `app.route()` used for?
Concept:
It creates a chainable route handler for a route path. Useful to reduce redundancy and typos when defining multiple methods (GET, POST, PUT) for the *same path*.

Example:
```javascript
// Without app.route (Repetitive)
app.get('/book', (req, res) => { ... });
app.post('/book', (req, res) => { ... });

// With app.route (Clean)
app.route('/book')
  .get((req, res) => { ... })
  .post((req, res) => { ... })
  .put((req, res) => { ... });
```

Analogy:
Instead of writing the full address on three separate envelopes to the same person, you put all three letters in one big envelope addressed once.

⚡ One-liner to remember:
Chain multiple method handlers to a single path URL.

## Q14. How do you handle cookies in Express?
Concept:
Use `cookie-parser` middleware to read cookies (`req.cookies`). Use `res.cookie()` to set them.

Example:
```javascript
const cookieParser = require('cookie-parser');
app.use(cookieParser());

app.get('/', (req, res) => {
  // Set cookie
  res.cookie('name', 'express', { httpOnly: true, secure: true });
  // Read cookie
  console.log(req.cookies.name);
  res.send('Cookie set');
});
```

Analogy:
A wristband at a concert. The venue gives it to you (`res.cookie`). Every time you walk past security, they check your wrist (`req.cookies`) to see where you can go.

⚡ One-liner to remember:
`cookie-parser` reads them; `res.cookie` sets them.

## Q15. How do you handle file uploads in Express?
Concept:
Express handles body data but not multi-part form data (files) by default. Use `multer` middleware.

Example:
```javascript
const multer  = require('multer');
const upload = multer({ dest: 'uploads/' });

app.post('/profile', upload.single('avatar'), (req, res) => {
  // req.file is the `avatar` file
  // req.body will hold the text fields
  console.log(req.file);
  res.send('Uploaded!');
});
```

Analogy:
The mailroom. Regular letters go to the front desk. Heavy packages (files) go to the loading dock (`multer`) to be processed and stored differently.

⚡ One-liner to remember:
Use `multer` for `multipart/form-data` file uploads.

## Q16. How do you validate input data in Express?
Concept:
Use `express-validator`. It is a set of express.js middlewares that wraps validator.js validator and sanitizer functions.

Example:
```javascript
const { body, validationResult } = require('express-validator');

app.post('/user', [
  body('email').isEmail(),
  body('password').isLength({ min: 5 })
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  // Proceed...
});
```

Analogy:
A nightclub ID check. They check if the ID is valid (date, photo) before letting you in. If invalid, you get bounced immediately.

⚡ One-liner to remember:
Validate inputs before processing logic to save resources and ensure safety.

## Q17. What are Template Engines in Express?
Concept:
They allow you to use static template files (like Pug, EJS, Handlebars) and replace variables with actual data at runtime to generate HTML.

Example:
```javascript
app.set('view engine', 'pug');

app.get('/', (req, res) => {
  // Renders 'index.pug' with title and message
  res.render('index', { title: 'Hey', message: 'Hello there!' });
});
```

Analogy:
Mad Libs. You have a pre-written story (Template) with blanks. You give it words (Data), and it fills them in to create the final story (HTML).

⚡ One-liner to remember:
Server-side rendering: Data + Template = HTML.

## Q18. What is the difference between `app.set()` and `app.use()`?
Concept:
- **`app.set(key, value)`:** Sets application settings/variables (e.g., 'view engine', 'port').
- **`app.use(path, callback)`:** Mounts middleware functions.

Example:
```javascript
app.set('title', 'My App'); // Setting a variable
app.use(express.json());    // Mounting logic
```

Analogy:
- `app.set`: Adjusting the thermostat setting (Configuration).
- `app.use`: Turning on the fan that actually blows air (Action/Middleware).

⚡ One-liner to remember:
`set` configures values; `use` applies behavior.

## Q19. What is "Trust Proxy" and when do you need it?
Concept:
When running Express behind a reverse proxy (like Nginx, AWS ELB, Heroku), `req.ip` and `req.protocol` might be incorrect (they show the proxy's IP).
Setting `app.enable('trust proxy')` tells Express to respect the `X-Forwarded-*` headers set by the proxy.

Example:
```javascript
// Required if behind Nginx/Heroku for correct IP rate limiting or secure cookies
app.enable('trust proxy');
```

Analogy:
If you receive a letter forwarded by a friend, the return address says "Friend's House". "Trust Proxy" means you look inside for the "Original Sender" note to know who actually wrote it.

⚡ One-liner to remember:
Enable this if you are behind a load balancer (Nginx/AWS) to get the real client IP.

## Q20. How do you handle graceful shutdowns in Express?
Concept:
Listen for process signals (`SIGTERM`, `SIGINT`). Close the server to stop accepting new connections, finish existing requests, and close DB connections before exiting.

Example:
```javascript
const server = app.listen(3000);

process.on('SIGTERM', () => {
  console.log('SIGTERM signal received: closing HTTP server');
  server.close(() => {
    console.log('HTTP server closed');
    // Close DB connections here
    process.exit(0);
  });
});
```

Analogy:
Closing a shop. You flip the sign to "Closed" (stop accepting new customers), but you let the current customers finish their shopping and pay before you lock the door.

⚡ One-liner to remember:
Don't just kill the process; finish pending requests and close connections cleanly.
