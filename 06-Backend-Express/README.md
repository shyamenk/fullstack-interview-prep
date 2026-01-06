# Express.js Framework Mastery

## 🎯 Express.js Fundamentals

### Framework Basics
- [ ] **Application Setup**
  - Express app initialization
  - Basic routing setup
  - Server configuration
  - Environment-based configuration
- [ ] **Middleware Concept**
  - Middleware execution flow
  - Built-in vs custom middleware
  - Error handling middleware
  - Third-party middleware integration

### Routing System
- [ ] **Route Methods**
  - GET, POST, PUT, DELETE, PATCH
  - Route parameters and wildcards
  - Query strings and body parsing
  - Route handlers and multiple callbacks
- [ ] **Router Module**
  - Modular routing with express.Router()
  - Route organization and separation
  - Nested routers and mounting
  - Route-specific middleware

## 🚀 Advanced Express Features

### Middleware Deep Dive
- [ ] **Custom Middleware**
  - Request/response modification
  - Authentication middleware
  - Logging and monitoring middleware
  - Rate limiting and security
- [ ] **Error Handling**
  - Global error handling middleware
  - Custom error classes
  - Async error handling
  - Error response formatting

### Request/Response Handling
- [ ] **Request Processing**
  - Body parsing (JSON, URL-encoded, multipart)
  - File upload handling
  - Cookie and session management
  - Header manipulation
- [ ] **Response Methods**
  - JSON responses and status codes
  - File serving and downloads
  - Redirects and rendering
  - Stream responses

## 💻 Common Interview Questions

> **[View Detailed Questions & Answers](./questions-and-answers.md)**

### Conceptual Questions
1. **Explain the middleware stack in Express.js**
2. **What's the difference between app.use() and app.get()?**
3. **How does Express handle errors in async routes?**
4. **What are the benefits of using express.Router()?**
5. **Explain the req, res, and next parameters**

### Implementation Challenges
1. **Build a RESTful API with CRUD operations**
2. **Implement JWT authentication middleware**
3. **Create a file upload endpoint with validation**
4. **Build rate limiting middleware**
5. **Implement API versioning**

### Architecture Questions
1. **How would you structure a large Express application?**
2. **What's your approach to error handling in Express?**
3. **How do you implement logging and monitoring?**
4. **Explain your strategy for API documentation**

## 🛠️ Practical Implementation Examples

### Complete Express Server Setup
```javascript
const express = require('express');
const cors = require('cors');
const morgan = require('morgan');
const helmet = require('helmet');
const compression = require('compression');
const rateLimit = require('express-rate-limit');

const app = express();

// Security middleware
app.use(helmet());

// CORS configuration
app.use(cors({
  origin: process.env.NODE_ENV === 'production'
    ? ['https://yourdomain.com']
    : ['http://localhost:3000'],
  credentials: true
}));

// Logging
app.use(morgan('combined'));

// Compression
app.use(compression());

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});
app.use('/api/', limiter);

// Body parsing
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// Static files
app.use('/uploads', express.static('uploads'));

// Routes
app.use('/api/v1/auth', require('./routes/auth'));
app.use('/api/v1/users', require('./routes/users'));
app.use('/api/v1/posts', require('./routes/posts'));

// Health check
app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'OK',
    timestamp: new Date().toISOString(),
    uptime: process.uptime()
  });
});

// 404 handler
app.use('*', (req, res) => {
  res.status(404).json({
    error: 'Route not found',
    method: req.method,
    url: req.originalUrl
  });
});

// Global error handler
app.use((err, req, res, next) => {
  console.error(err.stack);

  if (err.name === 'ValidationError') {
    return res.status(400).json({
      error: 'Validation Error',
      details: err.message
    });
  }

  if (err.name === 'CastError') {
    return res.status(400).json({
      error: 'Invalid ID format'
    });
  }

  res.status(err.statusCode || 500).json({
    error: process.env.NODE_ENV === 'production'
      ? 'Something went wrong!'
      : err.message
  });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

module.exports = app;
```

### RESTful API Route Structure
```javascript
// routes/posts.js
const express = require('express');
const router = express.Router();
const { authenticate, authorize } = require('../middleware/auth');
const { validatePost } = require('../middleware/validation');
const upload = require('../middleware/upload');
const PostController = require('../controllers/PostController');

// GET /api/v1/posts - Get all posts with pagination
router.get('/', PostController.getAllPosts);

// GET /api/v1/posts/:id - Get single post
router.get('/:id', PostController.getPostById);

// POST /api/v1/posts - Create new post (authenticated users only)
router.post('/',
  authenticate,
  upload.array('images', 5),
  validatePost,
  PostController.createPost
);

// PUT /api/v1/posts/:id - Update post (owner or admin only)
router.put('/:id',
  authenticate,
  authorize(['owner', 'admin']),
  validatePost,
  PostController.updatePost
);

// DELETE /api/v1/posts/:id - Delete post (owner or admin only)
router.delete('/:id',
  authenticate,
  authorize(['owner', 'admin']),
  PostController.deletePost
);

// POST /api/v1/posts/:id/like - Like/unlike post
router.post('/:id/like',
  authenticate,
  PostController.toggleLike
);

// GET /api/v1/posts/:id/comments - Get post comments
router.get('/:id/comments', PostController.getComments);

// POST /api/v1/posts/:id/comments - Add comment
router.post('/:id/comments',
  authenticate,
  PostController.addComment
);

module.exports = router;
```

### Custom Middleware Examples
```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');
const User = require('../models/User');

const authenticate = async (req, res, next) => {
  try {
    const token = req.header('Authorization')?.replace('Bearer ', '');

    if (!token) {
      return res.status(401).json({ error: 'Access denied. No token provided.' });
    }

    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    const user = await User.findById(decoded.id).select('-password');

    if (!user) {
      return res.status(401).json({ error: 'Token is not valid.' });
    }

    req.user = user;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Token is not valid.' });
  }
};

const authorize = (roles = []) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Access denied.' });
    }

    if (roles.length && !roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Insufficient permissions.' });
    }

    next();
  };
};

// Rate limiting middleware
const createRateLimiter = (windowMs, max, message) => {
  return rateLimit({
    windowMs,
    max,
    message: { error: message },
    standardHeaders: true,
    legacyHeaders: false,
  });
};

module.exports = {
  authenticate,
  authorize,
  createRateLimiter
};
```

### Validation Middleware
```javascript
// middleware/validation.js
const { body, validationResult } = require('express-validator');

const handleValidationErrors = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({
      error: 'Validation failed',
      details: errors.array()
    });
  }
  next();
};

const validatePost = [
  body('title')
    .trim()
    .isLength({ min: 1, max: 200 })
    .withMessage('Title must be between 1 and 200 characters'),

  body('content')
    .trim()
    .isLength({ min: 1, max: 5000 })
    .withMessage('Content must be between 1 and 5000 characters'),

  body('tags')
    .optional()
    .isArray({ max: 10 })
    .withMessage('Tags must be an array with maximum 10 items'),

  body('published')
    .optional()
    .isBoolean()
    .withMessage('Published must be a boolean'),

  handleValidationErrors
];

const validateUser = [
  body('email')
    .isEmail()
    .normalizeEmail()
    .withMessage('Please provide a valid email'),

  body('password')
    .isLength({ min: 8 })
    .withMessage('Password must be at least 8 characters long')
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/)
    .withMessage('Password must contain at least one uppercase letter, one lowercase letter, one number, and one special character'),

  body('name')
    .trim()
    .isLength({ min: 2, max: 50 })
    .withMessage('Name must be between 2 and 50 characters'),

  handleValidationErrors
];

module.exports = {
  validatePost,
  validateUser,
  handleValidationErrors
};
```

## 🔧 Advanced Patterns

### Controller Pattern
```javascript
// controllers/PostController.js
const Post = require('../models/Post');
const { asyncHandler } = require('../utils/asyncHandler');
const { AppError } = require('../utils/AppError');
const { APIFeatures } = require('../utils/APIFeatures');

class PostController {
  // Get all posts with filtering, sorting, and pagination
  static getAllPosts = asyncHandler(async (req, res) => {
    const features = new APIFeatures(Post.find(), req.query)
      .filter()
      .sort()
      .limitFields()
      .paginate();

    const posts = await features.query.populate('author', 'name email');

    res.status(200).json({
      status: 'success',
      results: posts.length,
      data: { posts }
    });
  });

  static getPostById = asyncHandler(async (req, res, next) => {
    const post = await Post.findById(req.params.id)
      .populate('author', 'name email')
      .populate('comments.author', 'name');

    if (!post) {
      return next(new AppError('Post not found', 404));
    }

    res.status(200).json({
      status: 'success',
      data: { post }
    });
  });

  static createPost = asyncHandler(async (req, res) => {
    const postData = {
      ...req.body,
      author: req.user._id,
      images: req.files?.map(file => file.path) || []
    };

    const post = await Post.create(postData);

    res.status(201).json({
      status: 'success',
      data: { post }
    });
  });

  static updatePost = asyncHandler(async (req, res, next) => {
    const post = await Post.findById(req.params.id);

    if (!post) {
      return next(new AppError('Post not found', 404));
    }

    // Check ownership
    if (post.author.toString() !== req.user._id.toString() && req.user.role !== 'admin') {
      return next(new AppError('Not authorized to update this post', 403));
    }

    const updatedPost = await Post.findByIdAndUpdate(
      req.params.id,
      req.body,
      { new: true, runValidators: true }
    );

    res.status(200).json({
      status: 'success',
      data: { post: updatedPost }
    });
  });

  static deletePost = asyncHandler(async (req, res, next) => {
    const post = await Post.findById(req.params.id);

    if (!post) {
      return next(new AppError('Post not found', 404));
    }

    // Check ownership
    if (post.author.toString() !== req.user._id.toString() && req.user.role !== 'admin') {
      return next(new AppError('Not authorized to delete this post', 403));
    }

    await Post.findByIdAndDelete(req.params.id);

    res.status(204).json({
      status: 'success',
      data: null
    });
  });
}

module.exports = PostController;
```

### Service Layer Pattern
```javascript
// services/PostService.js
class PostService {
  static async getAllPosts(query) {
    const features = new APIFeatures(Post.find(), query)
      .filter()
      .sort()
      .limitFields()
      .paginate();

    return await features.query.populate('author', 'name email');
  }

  static async getPostById(id) {
    const post = await Post.findById(id)
      .populate('author', 'name email')
      .populate('comments.author', 'name');

    if (!post) {
      throw new AppError('Post not found', 404);
    }

    return post;
  }

  static async createPost(postData, userId, files) {
    const post = await Post.create({
      ...postData,
      author: userId,
      images: files?.map(file => file.path) || []
    });

    return post;
  }

  static async updatePost(id, updateData, userId, userRole) {
    const post = await Post.findById(id);

    if (!post) {
      throw new AppError('Post not found', 404);
    }

    if (post.author.toString() !== userId && userRole !== 'admin') {
      throw new AppError('Not authorized to update this post', 403);
    }

    return await Post.findByIdAndUpdate(id, updateData, {
      new: true,
      runValidators: true
    });
  }

  static async deletePost(id, userId, userRole) {
    const post = await Post.findById(id);

    if (!post) {
      throw new AppError('Post not found', 404);
    }

    if (post.author.toString() !== userId && userRole !== 'admin') {
      throw new AppError('Not authorized to delete this post', 403);
    }

    await Post.findByIdAndDelete(id);
  }
}

module.exports = PostService;
```

## 🔒 Security Implementation

### Input Sanitization
```javascript
const mongoSanitize = require('express-mongo-sanitize');
const xss = require('xss-clean');
const hpp = require('hpp');

// Data sanitization against NoSQL query injection
app.use(mongoSanitize());

// Data sanitization against XSS
app.use(xss());

// Prevent parameter pollution
app.use(hpp({
  whitelist: ['sort', 'fields', 'page', 'limit']
}));
```

### File Upload Security
```javascript
// middleware/upload.js
const multer = require('multer');
const path = require('path');
const { AppError } = require('../utils/AppError');

const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/');
  },
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
  }
});

const fileFilter = (req, file, cb) => {
  const allowedTypes = /jpeg|jpg|png|gif|pdf|doc|docx/;
  const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
  const mimetype = allowedTypes.test(file.mimetype);

  if (mimetype && extname) {
    return cb(null, true);
  } else {
    cb(new AppError('Invalid file type', 400));
  }
};

const upload = multer({
  storage,
  fileFilter,
  limits: {
    fileSize: 5 * 1024 * 1024 // 5MB limit
  }
});

module.exports = upload;
```

## 📊 Testing Express Applications

### Integration Testing
```javascript
// tests/integration/posts.test.js
const request = require('supertest');
const app = require('../../app');
const User = require('../../models/User');
const Post = require('../../models/Post');

describe('Posts API', () => {
  let authToken;
  let userId;

  beforeEach(async () => {
    // Create test user and get auth token
    const user = await User.create({
      name: 'Test User',
      email: 'test@example.com',
      password: 'Password123!'
    });
    userId = user._id;
    authToken = user.generateAuthToken();
  });

  afterEach(async () => {
    await User.deleteMany({});
    await Post.deleteMany({});
  });

  describe('GET /api/v1/posts', () => {
    it('should return all posts', async () => {
      const posts = [
        { title: 'Post 1', content: 'Content 1', author: userId },
        { title: 'Post 2', content: 'Content 2', author: userId }
      ];
      await Post.insertMany(posts);

      const res = await request(app)
        .get('/api/v1/posts')
        .expect(200);

      expect(res.body.status).toBe('success');
      expect(res.body.results).toBe(2);
      expect(res.body.data.posts).toHaveLength(2);
    });

    it('should support pagination', async () => {
      const posts = Array.from({ length: 15 }, (_, i) => ({
        title: `Post ${i + 1}`,
        content: `Content ${i + 1}`,
        author: userId
      }));
      await Post.insertMany(posts);

      const res = await request(app)
        .get('/api/v1/posts?page=2&limit=5')
        .expect(200);

      expect(res.body.data.posts).toHaveLength(5);
    });
  });

  describe('POST /api/v1/posts', () => {
    it('should create a new post when authenticated', async () => {
      const postData = {
        title: 'New Post',
        content: 'This is a new post content'
      };

      const res = await request(app)
        .post('/api/v1/posts')
        .set('Authorization', `Bearer ${authToken}`)
        .send(postData)
        .expect(201);

      expect(res.body.status).toBe('success');
      expect(res.body.data.post.title).toBe(postData.title);
      expect(res.body.data.post.author).toBe(userId.toString());
    });

    it('should return 401 when not authenticated', async () => {
      const postData = {
        title: 'New Post',
        content: 'This is a new post content'
      };

      await request(app)
        .post('/api/v1/posts')
        .send(postData)
        .expect(401);
    });

    it('should validate required fields', async () => {
      const res = await request(app)
        .post('/api/v1/posts')
        .set('Authorization', `Bearer ${authToken}`)
        .send({})
        .expect(400);

      expect(res.body.error).toBe('Validation failed');
    });
  });
});
```

## 📚 Learning Resources

### Official Documentation
- [Express.js Documentation](https://expressjs.com/)
- [Express.js API Reference](https://expressjs.com/en/4x/api.html)
- [Express.js Guide](https://expressjs.com/en/guide/)

### Recommended Books
- "Express in Action" by Evan Hahn
- "Web Development with Node and Express" by Ethan Brown
- "Express.js Deep API Reference" by Azat Mardan

## ✅ Progress Tracker

### Core Concepts
- [ ] Express app setup and configuration mastered
- [ ] Middleware concepts and implementation understood
- [ ] Routing system and modular organization practiced
- [ ] Request/response handling patterns learned

### Advanced Features
- [ ] Custom middleware development completed
- [ ] Error handling patterns implemented
- [ ] Authentication and authorization set up
- [ ] File upload and processing configured

### Best Practices
- [ ] Security measures implemented
- [ ] Input validation and sanitization applied
- [ ] Testing strategies practiced
- [ ] Performance optimization techniques learned

### Practical Experience
- [ ] 3+ Express.js APIs built
- [ ] RESTful API design principles applied
- [ ] Database integration completed
- [ ] Production deployment achieved

**Target**: Master Express.js for building scalable, secure APIs and web applications