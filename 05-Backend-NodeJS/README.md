# Node.js Backend Development

## 🎯 Core Node.js Concepts

### Runtime Fundamentals
- [ ] **Event Loop Architecture**
  - Single-threaded, non-blocking I/O
  - Call stack, callback queue, event loop phases
  - Microtasks vs macrotasks
  - Performance implications
- [ ] **Modules System**
  - CommonJS vs ES Modules
  - require() vs import/export
  - Module caching and circular dependencies
  - Built-in modules (fs, path, http, etc.)

### Asynchronous Programming
- [ ] **Callbacks and Error Handling**
  - Error-first callback pattern
  - Callback hell and pyramid of doom
  - Error propagation strategies
- [ ] **Promises**
  - Promise states and chaining
  - Promise.all, Promise.race, Promise.allSettled
  - Error handling with .catch()
- [ ] **Async/Await**
  - Syntactic sugar over promises
  - Error handling with try/catch
  - Sequential vs parallel execution
  - Top-level await

## 🚀 Advanced Node.js Features

### Process and Performance
- [ ] **Process Management**
  - process.env and environment variables
  - process.argv command line arguments
  - process.exit() and graceful shutdown
  - Child processes and clustering
- [ ] **Memory Management**
  - Garbage collection in V8
  - Memory leaks detection
  - Heap vs stack memory
  - Buffer and stream handling

### File System & I/O
- [ ] **File Operations**
  - Synchronous vs asynchronous file operations
  - File streams for large files
  - Directory operations
  - File watching and monitoring
- [ ] **Streams**
  - Readable, writable, duplex, transform streams
  - Piping and chaining streams
  - Stream events and error handling
  - Memory efficiency with streams

## 💻 Common Interview Questions

### Conceptual Questions
1. **Explain the Event Loop in Node.js**
2. **What's the difference between process.nextTick() and setImmediate()?**
3. **How does Node.js handle concurrent requests?**
4. **What are the differences between spawn, fork, and exec?**
5. **Explain callback hell and how to avoid it**

### Coding Challenges
1. **Build a file upload server with progress tracking**
2. **Implement a rate limiter middleware**
3. **Create a URL shortener service**
4. **Build a real-time chat server**
5. **Implement JWT authentication**

### Performance Questions
1. **How would you optimize a Node.js application for high traffic?**
2. **What's the difference between clustering and worker threads?**
3. **How do you handle memory leaks in Node.js?**
4. **Explain different caching strategies**

## 🛠️ Practical Examples

### Event Loop Demonstration
```javascript
console.log('Start');

setTimeout(() => console.log('Timeout 1'), 0);
setImmediate(() => console.log('Immediate 1'));

process.nextTick(() => console.log('Next Tick 1'));

Promise.resolve().then(() => console.log('Promise 1'));

console.log('End');

// Output order: Start -> End -> Next Tick 1 -> Promise 1 -> Immediate 1 -> Timeout 1
```

### Custom Module Pattern
```javascript
// utils/logger.js
const fs = require('fs').promises;
const path = require('path');

class Logger {
  constructor(logFile = 'app.log') {
    this.logFile = path.join(process.cwd(), 'logs', logFile);
    this.initLogDirectory();
  }

  async initLogDirectory() {
    const logDir = path.dirname(this.logFile);
    try {
      await fs.mkdir(logDir, { recursive: true });
    } catch (error) {
      console.error('Failed to create log directory:', error);
    }
  }

  async log(level, message) {
    const timestamp = new Date().toISOString();
    const logEntry = `[${timestamp}] ${level.toUpperCase()}: ${message}\n`;

    try {
      await fs.appendFile(this.logFile, logEntry);
    } catch (error) {
      console.error('Failed to write to log file:', error);
    }
  }

  info(message) { return this.log('info', message); }
  error(message) { return this.log('error', message); }
  warn(message) { return this.log('warn', message); }
}

module.exports = new Logger();
```

### Stream Processing Example
```javascript
const fs = require('fs');
const { Transform } = require('stream');

// Custom transform stream to process CSV data
class CSVProcessor extends Transform {
  constructor(options = {}) {
    super({ objectMode: true });
    this.header = null;
    this.rowCount = 0;
  }

  _transform(chunk, encoding, callback) {
    const lines = chunk.toString().split('\n');

    for (const line of lines) {
      if (!line.trim()) continue;

      if (!this.header) {
        this.header = line.split(',');
        continue;
      }

      const values = line.split(',');
      const row = {};

      this.header.forEach((key, index) => {
        row[key.trim()] = values[index]?.trim();
      });

      this.rowCount++;
      this.push(row);
    }

    callback();
  }
}

// Usage
fs.createReadStream('large-file.csv')
  .pipe(new CSVProcessor())
  .on('data', (row) => {
    console.log('Processed row:', row);
  })
  .on('end', () => {
    console.log('Processing complete');
  });
```

## 🔧 Development Best Practices

### Error Handling Patterns
```javascript
// Centralized error handling
class AppError extends Error {
  constructor(message, statusCode, isOperational = true) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = isOperational;

    Error.captureStackTrace(this, this.constructor);
  }
}

// Global error handler
const globalErrorHandler = (err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || 'error';

  if (process.env.NODE_ENV === 'development') {
    sendErrorDev(err, res);
  } else {
    sendErrorProd(err, res);
  }
};

// Async error wrapper
const catchAsync = (fn) => {
  return (req, res, next) => {
    fn(req, res, next).catch(next);
  };
};
```

### Configuration Management
```javascript
// config/index.js
const config = {
  development: {
    port: process.env.PORT || 3000,
    database: {
      host: process.env.DB_HOST || 'localhost',
      port: process.env.DB_PORT || 5432,
      name: process.env.DB_NAME || 'myapp_dev'
    },
    jwt: {
      secret: process.env.JWT_SECRET || 'dev-secret',
      expiresIn: process.env.JWT_EXPIRES_IN || '24h'
    }
  },
  production: {
    port: process.env.PORT,
    database: {
      host: process.env.DB_HOST,
      port: process.env.DB_PORT,
      name: process.env.DB_NAME
    },
    jwt: {
      secret: process.env.JWT_SECRET,
      expiresIn: process.env.JWT_EXPIRES_IN || '1h'
    }
  }
};

module.exports = config[process.env.NODE_ENV || 'development'];
```

### Graceful Shutdown
```javascript
// app.js
const gracefulShutdown = (server) => {
  const shutdown = (signal) => {
    console.log(`Received ${signal}. Shutting down gracefully...`);

    server.close(() => {
      console.log('Process terminated');
      process.exit(0);
    });

    // Force shutdown after 10 seconds
    setTimeout(() => {
      console.log('Forcing shutdown...');
      process.exit(1);
    }, 10000);
  };

  process.on('SIGTERM', () => shutdown('SIGTERM'));
  process.on('SIGINT', () => shutdown('SIGINT'));
};

const server = app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});

gracefulShutdown(server);
```

## 🎨 Design Patterns

### Singleton Pattern
```javascript
class DatabaseConnection {
  constructor() {
    if (DatabaseConnection.instance) {
      return DatabaseConnection.instance;
    }

    this.connection = null;
    DatabaseConnection.instance = this;
  }

  async connect() {
    if (!this.connection) {
      this.connection = await createConnection();
    }
    return this.connection;
  }
}

module.exports = DatabaseConnection;
```

### Observer Pattern
```javascript
const EventEmitter = require('events');

class UserService extends EventEmitter {
  async createUser(userData) {
    const user = await User.create(userData);

    // Emit events for other services to listen
    this.emit('user:created', user);
    this.emit('user:welcome-email', user);

    return user;
  }
}

// Usage
const userService = new UserService();

userService.on('user:created', (user) => {
  console.log('User created:', user.email);
});

userService.on('user:welcome-email', async (user) => {
  await emailService.sendWelcomeEmail(user);
});
```

## 🔒 Security Best Practices

### Input Validation & Sanitization
```javascript
const validator = require('validator');
const rateLimit = require('express-rate-limit');
const helmet = require('helmet');

// Input validation middleware
const validateInput = (schema) => {
  return (req, res, next) => {
    const { error } = schema.validate(req.body);
    if (error) {
      return res.status(400).json({
        error: error.details[0].message
      });
    }
    next();
  };
};

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP'
});

// Security headers
app.use(helmet());
app.use(limiter);
```

### Authentication & Authorization
```javascript
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');

class AuthService {
  static async hashPassword(password) {
    const salt = await bcrypt.genSalt(12);
    return bcrypt.hash(password, salt);
  }

  static async comparePasswords(password, hashedPassword) {
    return bcrypt.compare(password, hashedPassword);
  }

  static generateToken(payload) {
    return jwt.sign(payload, process.env.JWT_SECRET, {
      expiresIn: process.env.JWT_EXPIRES_IN
    });
  }

  static verifyToken(token) {
    return jwt.verify(token, process.env.JWT_SECRET);
  }
}

// Auth middleware
const authenticate = async (req, res, next) => {
  try {
    const token = req.headers.authorization?.split(' ')[1];

    if (!token) {
      return res.status(401).json({ error: 'No token provided' });
    }

    const decoded = AuthService.verifyToken(token);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
};
```

## 📊 Performance Optimization

### Clustering for Multi-Core Usage
```javascript
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);

  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork();
  });
} else {
  // Worker process
  const app = require('./app');
  app.listen(3000, () => {
    console.log(`Worker ${process.pid} started`);
  });
}
```

### Caching Strategies
```javascript
const Redis = require('redis');
const client = Redis.createClient();

class CacheService {
  static async get(key) {
    try {
      const data = await client.get(key);
      return data ? JSON.parse(data) : null;
    } catch (error) {
      console.error('Cache get error:', error);
      return null;
    }
  }

  static async set(key, data, ttl = 3600) {
    try {
      await client.setex(key, ttl, JSON.stringify(data));
    } catch (error) {
      console.error('Cache set error:', error);
    }
  }

  static async del(key) {
    try {
      await client.del(key);
    } catch (error) {
      console.error('Cache delete error:', error);
    }
  }
}

// Cache middleware
const cacheMiddleware = (ttl = 3600) => {
  return async (req, res, next) => {
    const key = `cache:${req.originalUrl}`;
    const cached = await CacheService.get(key);

    if (cached) {
      return res.json(cached);
    }

    const originalSend = res.json;
    res.json = function(data) {
      CacheService.set(key, data, ttl);
      originalSend.call(this, data);
    };

    next();
  };
};
```

## 🧪 Testing Strategies

### Unit Testing with Jest
```javascript
// tests/utils/logger.test.js
const Logger = require('../../utils/logger');
const fs = require('fs').promises;

jest.mock('fs', () => ({
  promises: {
    mkdir: jest.fn(),
    appendFile: jest.fn()
  }
}));

describe('Logger', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  test('should log info message', async () => {
    await Logger.info('Test message');

    expect(fs.appendFile).toHaveBeenCalledWith(
      expect.stringContaining('app.log'),
      expect.stringContaining('INFO: Test message')
    );
  });

  test('should handle logging errors gracefully', async () => {
    fs.appendFile.mockRejectedValue(new Error('File write error'));
    const consoleSpy = jest.spyOn(console, 'error').mockImplementation();

    await Logger.error('Test error');

    expect(consoleSpy).toHaveBeenCalledWith(
      'Failed to write to log file:',
      expect.any(Error)
    );

    consoleSpy.mockRestore();
  });
});
```

## 📚 Learning Resources

### Official Documentation
- [Node.js Official Docs](https://nodejs.org/en/docs/)
- [NPM Documentation](https://docs.npmjs.com/)
- [Node.js API Reference](https://nodejs.org/api/)

### Recommended Books
- "Node.js Design Patterns" by Mario Casciaro
- "Learning Node.js" by Marc Wandschneider
- "Node.js in Action" by Mike Cantelon

### Online Courses
- "Node.js: The Complete Guide" by Maximilian Schwarzmüller
- "Node.js Developer Course" by Andrew Mead
- "Advanced Node.js" by Pluralsight

## ✅ Progress Tracker

### Core Concepts
- [ ] Event loop and asynchronous programming mastered
- [ ] Module system and imports/exports understood
- [ ] Error handling patterns implemented
- [ ] File system and streams practiced

### Advanced Features
- [ ] Process management and clustering explored
- [ ] Performance optimization techniques applied
- [ ] Security best practices implemented
- [ ] Design patterns utilized

### Practical Skills
- [ ] 3+ Node.js projects completed
- [ ] API development with authentication
- [ ] Database integration implemented
- [ ] Testing strategies applied
- [ ] Production deployment achieved

**Target**: Master Node.js fundamentals and advanced concepts for backend development interviews