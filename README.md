# Week 2: Node.js & Backend Development Fundamentals

## Table of Contents

- [Week 2: Node.js \& Backend Development Fundamentals](#week-2-nodejs--backend-development-fundamentals)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Learning Objectives](#learning-objectives)
  - [Part I: Node.js Runtime Architecture](#part-i-nodejs-runtime-architecture)
    - [1.1 Event Loop Deep Dive](#11-event-loop-deep-dive)
      - [Node.js Architecture Diagram](#nodejs-architecture-diagram)
      - [Event Loop Phases](#event-loop-phases)
      - [Practical Example: Event Loop Execution Order](#practical-example-event-loop-execution-order)
      - [Key Takeaways](#key-takeaways)
    - [1.2 Non-blocking I/O Model](#12-non-blocking-io-model)
      - [Blocking vs Non-blocking Comparison](#blocking-vs-non-blocking-comparison)
      - [Real-World Performance Example](#real-world-performance-example)
    - [1.3 libuv \& Thread Pool](#13-libuv--thread-pool)
      - [libuv Architecture](#libuv-architecture)
      - [Thread Pool Example](#thread-pool-example)
      - [Understanding Thread Pool](#understanding-thread-pool)
  - [Part II: Express.js Framework](#part-ii-expressjs-framework)
    - [2.1 Middleware Architecture](#21-middleware-architecture)
      - [Middleware Flow](#middleware-flow)
      - [Complete Middleware Implementation](#complete-middleware-implementation)
      - [Middleware Best Practices](#middleware-best-practices)
    - [2.2 Routing Patterns](#22-routing-patterns)
      - [Project Structure](#project-structure)
      - [users.js - Route Definition](#usersjs---route-definition)
      - [userController.js - Controller Logic](#usercontrollerjs---controller-logic)
      - [app.js - Main Application](#appjs---main-application)
    - [2.3 Error Handling](#23-error-handling)
      - [Centralized Error Handler](#centralized-error-handler)
  - [Part III: RESTful API Design](#part-iii-restful-api-design)
    - [3.1 REST Principles](#31-rest-principles)
      - [Core Principles](#core-principles)
      - [Resource Naming Conventions](#resource-naming-conventions)
    - [3.2 HTTP Methods \& Status Codes](#32-http-methods--status-codes)
      - [HTTP Methods](#http-methods)
      - [HTTP Status Codes](#http-status-codes)
      - [Status Code Examples](#status-code-examples)
    - [3.3 API Versioning](#33-api-versioning)
      - [URL Versioning (Most Common)](#url-versioning-most-common)
      - [Header Versioning](#header-versioning)
  - [Part IV: Database Integration](#part-iv-database-integration)
    - [4.1 MongoDB \& Mongoose](#41-mongodb--mongoose)
      - [Installation \& Setup](#installation--setup)
    - [4.2 Schema Design](#42-schema-design)
      - [Basic Schema](#basic-schema)
      - [Advanced Schema with Relationships](#advanced-schema-with-relationships)
    - [4.3 Query Optimization](#43-query-optimization)
      - [Basic CRUD Operations](#basic-crud-operations)
      - [Advanced Queries](#advanced-queries)
      - [Performance Optimization](#performance-optimization)
  - [Part V: Authentication \& Authorization](#part-v-authentication--authorization)
    - [5.1 JWT Implementation](#51-jwt-implementation)
      - [Installation](#installation)
      - [JWT Token Structure](#jwt-token-structure)
      - [Complete Authentication System](#complete-authentication-system)
    - [5.2 Password Security](#52-password-security)
      - [Bcrypt Hashing](#bcrypt-hashing)
      - [Password Reset Flow](#password-reset-flow)
    - [5.3 Role-Based Access Control](#53-role-based-access-control)
  - [Part VI: Testing \& Debugging](#part-vi-testing--debugging)
    - [6.1 Unit Testing with Jest](#61-unit-testing-with-jest)
      - [Installation](#installation-1)
      - [package.json](#packagejson)
      - [Test Structure](#test-structure)
    - [6.2 Integration Testing](#62-integration-testing)
    - [6.3 Debugging Techniques](#63-debugging-techniques)
      - [Console Logging](#console-logging)
      - [Debug Module](#debug-module)
      - [VS Code Debugger](#vs-code-debugger)
      - [Request Logging Middleware](#request-logging-middleware)
  - [Practical Assignments](#practical-assignments)
    - [Assignment 1: Build a Blog API](#assignment-1-build-a-blog-api)
    - [Assignment 2: E-commerce Product API](#assignment-2-e-commerce-product-api)
    - [Assignment 3: Task Management API](#assignment-3-task-management-api)
  - [Assessment Criteria](#assessment-criteria)
    - [Week 2 Evaluation (100 points)](#week-2-evaluation-100-points)
  - [Reference Materials](#reference-materials)
    - [Official Documentation](#official-documentation)
    - [Recommended Reading](#recommended-reading)
    - [Video Resources](#video-resources)
    - [Tools \& Libraries](#tools--libraries)

---

## Overview

Week 2 focuses on mastering Node.js and backend development fundamentals. You'll learn how Node.js works internally, build RESTful APIs with Express.js, integrate databases, and implement secure authentication systems. These skills are essential for building production-ready server-side applications.

**Difficulty Level:** Intermediate to Advanced  
**Prerequisites:** JavaScript fundamentals, Basic understanding of HTTP

---

## Learning Objectives

By the end of Week 2, you will be able to:

- Understand Node.js event loop, non-blocking I/O, and libuv architecture
- Build scalable REST APIs using Express.js middleware patterns
- Design and implement database schemas with MongoDB and Mongoose
- Implement secure authentication using JWT and bcrypt
- Write comprehensive tests for backend applications
- Debug and optimize Node.js applications for production

---

## Part I: Node.js Runtime Architecture

### 1.1 Event Loop Deep Dive

Node.js uses a **single-threaded event loop** to handle concurrent operations efficiently without creating multiple threads.

#### Node.js Architecture Diagram

```
┌───────────────────────────────────────────────────────────────┐
│                        Node.js Process                        │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────────────┐         ┌──────────────────────┐     │
│  │   V8 JavaScript     │         │      libuv           │     │
│  │   Engine            │───────▶ │   (Event Loop)       │     │
│  │   - Execution       │         │   - Async I/O        │     │
│  │   - Memory Mgmt     │         │   - Thread Pool      │     │
│  └─────────────────────┘         └──────────────────────┘     │
│                                                               │
├───────────────────────────────────────────────────────────────┤
│                     Node.js Core Modules                      │
│     fs, http, crypto, path, os, stream, buffer, etc.          │
└───────────────────────────────────────────────────────────────┘
```

#### Event Loop Phases

```
   ┌───────────────────────────┐
┌─>│           timers          │  Execute setTimeout & setInterval
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │  Execute I/O callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │  Internal use
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐      ┌───────────────┐
│  │           poll            │ ◀────┤ Incoming I/O  │
│  │  - Execute I/O callbacks  │      │ connections,  │
│  │  - Wait for new events    │      │ data, etc.    │
│  └─────────────┬─────────────┘      └───────────────┘
│  ┌─────────────┴─────────────┐
│  │           check           │  Execute setImmediate()
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │      close callbacks      │  socket.on('close', ...)
│  └─────────────┬─────────────┘
└──────────────────────────────┘
```

#### Practical Example: Event Loop Execution Order

```javascript
const fs = require('fs');

console.log('1. Synchronous Start');

// Microtask - Highest Priority
process.nextTick(() => {
  console.log('2. process.nextTick');
});

// Microtask
Promise.resolve().then(() => {
  console.log('3. Promise.then');
});

// Timers Phase
setTimeout(() => {
  console.log('5. setTimeout 0ms');
}, 0);

// Check Phase
setImmediate(() => {
  console.log('6. setImmediate');
});

// Poll Phase - I/O
fs.readFile(__filename, () => {
  console.log('7. File Read Complete');
  
  setTimeout(() => {
    console.log('9. setTimeout inside I/O');
  }, 0);
  
  setImmediate(() => {
    console.log('8. setImmediate inside I/O');
  });
});

console.log('4. Synchronous End');

/* OUTPUT:
1. Synchronous Start
4. Synchronous End
2. process.nextTick
3. Promise.then
5. setTimeout 0ms
6. setImmediate
7. File Read Complete
8. setImmediate inside I/O
9. setTimeout inside I/O
*/
```

#### Key Takeaways

- **Microtasks** (process.nextTick, Promises) execute before each event loop phase
- **setImmediate** vs **setTimeout(0)**: setImmediate runs after poll phase, setTimeout in timers phase
- Inside I/O callbacks, **setImmediate** always executes before **setTimeout**

---

### 1.2 Non-blocking I/O Model

**Blocking I/O** pauses execution until operation completes. **Non-blocking I/O** delegates operations to the system kernel, freeing the thread for other work.

#### Blocking vs Non-blocking Comparison

```javascript
// ❌ BLOCKING - Freezes the entire server
const fs = require('fs');

console.log('Before reading file');
const data = fs.readFileSync('./large-file.txt', 'utf8'); // BLOCKS HERE
console.log('File content:', data);
console.log('After reading file');

// Server cannot handle ANY other requests during file read
```

```javascript
// ✅ NON-BLOCKING - Server stays responsive
const fs = require('fs');

console.log('Before reading file');

fs.readFile('./large-file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log('File content:', data);
});

console.log('After initiating read'); // Executes immediately

// Server can handle thousands of other requests while file loads
```

#### Real-World Performance Example

```javascript
const http = require('http');
const fs = require('fs').promises;

// Server that reads user profiles from disk
const server = http.createServer(async (req, res) => {
  if (req.url === '/profile') {
    try {
      // Non-blocking file read
      const profile = await fs.readFile('./user-profile.json', 'utf8');
      res.writeHead(200, { 'Content-Type': 'application/json' });
      res.end(profile);
    } catch (error) {
      res.writeHead(500);
      res.end('Server Error');
    }
  }
});

server.listen(3000);

// This server can handle 10,000+ concurrent requests
// because I/O operations don't block the event loop
```

---

### 1.3 libuv & Thread Pool

**libuv** is the C library that powers Node.js asynchronous I/O operations.

#### libuv Architecture

```
┌─────────────────────────────────────────────────────────┐
│                       libuv                             │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────────────┐    ┌────────────────────┐     │
│  │   Event Loop         │    │   Thread Pool      │     │
│  │   (Single Thread)    │    │   (4 threads)      │     │
│  │                      │    │                    │     │
│  │  • Timers            │    │  • fs operations   │     │
│  │  • I/O polling       │    │  • crypto          │     │
│  │  • setImmediate      │    │  • compression     │     │
│  │  • Callbacks         │    │  • DNS lookup      │     │
│  └──────────────────────┘    └────────────────────┘     │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │           Async I/O (Kernel-level)               │   │
│  │   Network I/O, TCP, UDP, HTTP (epoll/kqueue)     │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

#### Thread Pool Example

```javascript
const crypto = require('crypto');

console.log('Starting CPU-intensive tasks...');

// These operations run in the thread pool (default: 4 threads)
for (let i = 0; i < 8; i++) {
  crypto.pbkdf2('password', 'salt', 100000, 64, 'sha512', (err, key) => {
    console.log(`Hash ${i + 1} completed`);
  });
}

// Increase thread pool size for CPU-intensive apps
// process.env.UV_THREADPOOL_SIZE = 8; // Max: 1024
```

#### Understanding Thread Pool

```javascript
// Operations that use Thread Pool:
// 1. fs.* operations (file system)
// 2. crypto operations
// 3. zlib (compression)
// 4. DNS lookup (dns.lookup)

// Operations that DON'T use Thread Pool (kernel async):
// 1. Network I/O (http, tcp, udp)
// 2. Timers
// 3. setImmediate
```

---

## Part II: Express.js Framework

### 2.1 Middleware Architecture

Middleware functions have access to `request`, `response`, and `next()` in the request-response cycle.

#### Middleware Flow

```
Request
   ↓
Middleware 1 (Logger)
   ↓
Middleware 2 (Auth)
   ↓
Middleware 3 (CORS)
   ↓
Route Handler
   ↓
Response
```

#### Complete Middleware Implementation

```javascript
const express = require('express');
const app = express();

// 1. Built-in Middleware
app.use(express.json()); // Parse JSON bodies
app.use(express.urlencoded({ extended: true })); // Parse URL-encoded bodies
app.use(express.static('public')); // Serve static files

// 2. Custom Logger Middleware
app.use((req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next(); // MUST call next() to continue
});

// 3. Request Timing Middleware
app.use((req, res, next) => {
  req.startTime = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - req.startTime;
    console.log(`Request took ${duration}ms`);
  });
  
  next();
});

// 4. Authentication Middleware
const authenticateToken = (req, res, next) => {
  const token = req.headers['authorization'];
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  // Verify token (simplified)
  if (token === 'valid-token') {
    req.user = { id: 1, username: 'john' };
    next();
  } else {
    res.status(403).json({ error: 'Invalid token' });
  }
};

// 5. Role-based Authorization Middleware
const requireAdmin = (req, res, next) => {
  if (req.user && req.user.role === 'admin') {
    next();
  } else {
    res.status(403).json({ error: 'Admin access required' });
  }
};

// 6. Apply middleware to specific routes
app.get('/public', (req, res) => {
  res.json({ message: 'This is public' });
});

app.get('/protected', authenticateToken, (req, res) => {
  res.json({ message: 'This is protected', user: req.user });
});

app.delete('/users/:id', authenticateToken, requireAdmin, (req, res) => {
  res.json({ message: 'User deleted', id: req.params.id });
});

// 7. Error Handling Middleware (MUST be last)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    error: 'Something went wrong!',
    message: err.message
  });
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

#### Middleware Best Practices

```javascript
// ✅ DO: Always call next() or send response
app.use((req, res, next) => {
  if (condition) {
    return res.status(400).json({ error: 'Bad request' });
  }
  next();
});

// ❌ DON'T: Forget to call next()
app.use((req, res, next) => {
  console.log('Logging...');
  // FORGOT next() - Request will hang!
});

// ✅ DO: Pass errors to error handler
app.use((req, res, next) => {
  try {
    riskyOperation();
    next();
  } catch (error) {
    next(error); // Pass to error handler
  }
});

// ✅ DO: Use async/await with error handling
app.use(async (req, res, next) => {
  try {
    await asyncOperation();
    next();
  } catch (error) {
    next(error);
  }
});
```

---

### 2.2 Routing Patterns

Organize routes using **Express Router** for modular, maintainable code.

#### Project Structure

```
src/
├── routes/
│   ├── users.js
│   ├── products.js
│   └── orders.js
├── controllers/
│   ├── userController.js
│   ├── productController.js
│   └── orderController.js
├── middleware/
│   ├── auth.js
│   └── validation.js
└── app.js
```

#### users.js - Route Definition

```javascript
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');
const { authenticateToken } = require('../middleware/auth');
const { validateUser } = require('../middleware/validation');

// GET /api/users
router.get('/', userController.getAllUsers);

// GET /api/users/:id
router.get('/:id', userController.getUserById);

// POST /api/users
router.post('/', validateUser, userController.createUser);

// PUT /api/users/:id
router.put('/:id', authenticateToken, validateUser, userController.updateUser);

// DELETE /api/users/:id
router.delete('/:id', authenticateToken, userController.deleteUser);

// Nested routes
router.get('/:id/posts', userController.getUserPosts);
router.get('/:id/followers', userController.getUserFollowers);

module.exports = router;
```

#### userController.js - Controller Logic

```javascript
const User = require('../models/User');

// Get all users with pagination
exports.getAllUsers = async (req, res, next) => {
  try {
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 10;
    const skip = (page - 1) * limit;

    const users = await User.find()
      .skip(skip)
      .limit(limit)
      .select('-password'); // Exclude password

    const total = await User.countDocuments();

    res.json({
      data: users,
      pagination: {
        page,
        limit,
        total,
        pages: Math.ceil(total / limit)
      }
    });
  } catch (error) {
    next(error);
  }
};

// Get user by ID
exports.getUserById = async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id).select('-password');
    
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    res.json({ data: user });
  } catch (error) {
    next(error);
  }
};

// Create new user
exports.createUser = async (req, res, next) => {
  try {
    const { username, email, password } = req.body;

    // Check if user exists
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(409).json({ error: 'Email already exists' });
    }

    const user = await User.create({ username, email, password });
    
    res.status(201).json({
      message: 'User created successfully',
      data: {
        id: user._id,
        username: user.username,
        email: user.email
      }
    });
  } catch (error) {
    next(error);
  }
};

// Update user
exports.updateUser = async (req, res, next) => {
  try {
    const { username, email } = req.body;
    
    const user = await User.findByIdAndUpdate(
      req.params.id,
      { username, email },
      { new: true, runValidators: true }
    ).select('-password');

    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }

    res.json({
      message: 'User updated successfully',
      data: user
    });
  } catch (error) {
    next(error);
  }
};

// Delete user
exports.deleteUser = async (req, res, next) => {
  try {
    const user = await User.findByIdAndDelete(req.params.id);

    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }

    res.json({ message: 'User deleted successfully' });
  } catch (error) {
    next(error);
  }
};
```

#### app.js - Main Application

```javascript
const express = require('express');
const mongoose = require('mongoose');
const userRoutes = require('./routes/users');
const productRoutes = require('./routes/products');

const app = express();

// Middleware
app.use(express.json());

// Routes
app.use('/api/users', userRoutes);
app.use('/api/products', productRoutes);

// 404 Handler
app.use((req, res) => {
  res.status(404).json({ error: 'Route not found' });
});

// Error Handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(err.status || 500).json({
    error: err.message || 'Internal Server Error'
  });
});

// Database connection
mongoose.connect('mongodb://localhost:27017/myapp')
  .then(() => {
    console.log('Connected to MongoDB');
    app.listen(3000, () => console.log('Server running on port 3000'));
  })
  .catch(err => console.error('MongoDB connection error:', err));
```
---

### 2.3 Error Handling

Proper error handling is critical for production applications.

#### Centralized Error Handler

```javascript
// Custom Error Class
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith('4') ? 'fail' : 'error';
    this.isOperational = true;

    Error.captureStackTrace(this, this.constructor);
  }
}

// Error Handler Middleware
const errorHandler = (err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || 'error';

  if (process.env.NODE_ENV === 'development') {
    res.status(err.statusCode).json({
      status: err.status,
      error: err,
      message: err.message,
      stack: err.stack
    });
  } else {
    // Production - Don't leak error details
    if (err.isOperational) {
      res.status(err.statusCode).json({
        status: err.status,
        message: err.message
      });
    } else {
      console.error('ERROR 💥', err);
      res.status(500).json({
        status: 'error',
        message: 'Something went wrong'
      });
    }
  }
};

// Async Error Wrapper
const catchAsync = (fn) => {
  return (req, res, next) => {
    fn(req, res, next).catch(next);
  };
};

// Usage Example
const getUser = catchAsync(async (req, res, next) => {
  const user = await User.findById(req.params.id);
  
  if (!user) {
    return next(new AppError('User not found', 404));
  }
  
  res.json({ data: user });
});

// Handle unhandled routes
app.all('*', (req, res, next) => {
  next(new AppError(`Can't find ${req.originalUrl}`, 404));
});

app.use(errorHandler);

// Handle unhandled promise rejections
process.on('unhandledRejection', (err) => {
  console.log('UNHANDLED REJECTION! 💥 Shutting down...');
  console.error(err.name, err.message);
  server.close(() => {
    process.exit(1);
  });
});
```
---

## Part III: RESTful API Design

### 3.1 REST Principles

**REST (Representational State Transfer)** is an architectural style for designing networked applications.

#### Core Principles

1. **Client-Server** - Separation of concerns
2. **Stateless** - Each request contains all needed information
3. **Cacheable** - Responses define cacheability
4. **Uniform Interface** - Standardized communication
5. **Layered System** - Client can't tell if connected to end server
6. **Code on Demand** (optional) - Server can send executable code

#### Resource Naming Conventions

```javascript
// ✅ GOOD - Use nouns, plural form
GET    /api/users
GET    /api/users/123
POST   /api/users
PUT    /api/users/123
DELETE /api/users/123

GET    /api/users/123/posts
GET    /api/products/456/reviews

// ❌ BAD - Avoid verbs
GET    /api/getUsers
POST   /api/createUser
GET    /api/getUserById/123

// ✅ GOOD - Use query params for filtering
GET /api/products?category=electronics&sort=price
GET /api/users?page=2&limit=20&role=admin

// ❌ BAD - Endpoint proliferation
GET /api/electronics-products
GET /api/users-page-2
```

---

### 3.2 HTTP Methods & Status Codes

#### HTTP Methods

```javascript
// GET - Retrieve resource(s)
app.get('/api/users', async (req, res) => {
  const users = await User.find();
  res.json({ data: users });
});

// POST - Create new resource
app.post('/api/users', async (req, res) => {
  const user = await User.create(req.body);
  res.status(201).json({ data: user });
});

// PUT - Replace entire resource
app.put('/api/users/:id', async (req, res) => {
  const user = await User.findByIdAndUpdate(req.params.id, req.body, {
    new: true,
    overwrite: true
  });
  res.json({ data: user });
});

// PATCH - Partial update
app.patch('/api/users/:id', async (req, res) => {
  const user = await User.findByIdAndUpdate(req.params.id, req.body, {
    new: true
  });
  res.json({ data: user });
});

// DELETE - Remove resource
app.delete('/api/users/:id', async (req, res) => {
  await User.findByIdAndDelete(req.params.id);
  res.status(204).send();
});
```

#### HTTP Status Codes

```javascript
// 2xx - Success
200 OK                 // Standard success
201 Created            // Resource created
204 No Content         // Success with no response body

// 4xx - Client Errors
400 Bad Request        // Invalid request
401 Unauthorized       // Authentication required
403 Forbidden          // Authenticated but not authorized
404 Not Found          // Resource doesn't exist
409 Conflict           // Resource conflict (duplicate email)
422 Unprocessable      // Validation error

// 5xx - Server Errors
500 Internal Error     // Generic server error
502 Bad Gateway        // Upstream server error
503 Service Unavailable // Server down/maintenance
```

#### Status Code Examples

```javascript
// 200 OK
app.get('/api/users', async (req, res) => {
  const users = await User.find();
  res.status(200).json({ data: users });
});

// 201 Created
app.post('/api/users', async (req, res) => {
  const user = await User.create(req.body);
  res.status(201).json({ data: user });
});

// 204 No Content
app.delete('/api/users/:id', async (req, res) => {
  await User.findByIdAndDelete(req.params.id);
  res.status(204).send();
});

// 400 Bad Request
if (!req.body.email) {
  return res.status(400).json({ error: 'Email is required' });
}

// 401 Unauthorized
if (!token) {
  return res.status(401).json({ error: 'Authentication required' });
}

// 403 Forbidden
if (req.user.role !== 'admin') {
  return res.status(403).json({ error: 'Admin access required' });
}

// 404 Not Found
const user = await User.findById(req.params.id);
if (!user) {
  return res.status(404).json({ error: 'User not found' });
}

// 409 Conflict
const existing = await User.findOne({ email: req.body.email });
if (existing) {
  return res.status(409).json({ error: 'Email already exists' });
}

// 422 Unprocessable Entity
if (errors.length > 0) {
  return res.status(422).json({ errors });
}

// 500 Internal Server Error
try {
  // ... operation
} catch (error) {
  res.status(500).json({ error: 'Internal server error' });
}
```

---

### 3.3 API Versioning

#### URL Versioning (Most Common)

```javascript
// v1 routes
const v1Router = express.Router();
v1Router.get('/users', getUsers_v1);
app.use('/api/v1', v1Router);

// v2 routes with breaking changes
const v2Router = express.Router();
v2Router.get('/users', getUsers_v2);
app.use('/api/v2', v2Router);

// Usage:
// GET /api/v1/users
// GET /api/v2/users
```

#### Header Versioning

```javascript
app.use((req, res, next) => {
  const version = req.headers['api-version'] || '1';
  req.apiVersion = version;
  next();
});

app.get('/api/users', (req, res) => {
  if (req.apiVersion === '1') {
    return getUsers_v1(req, res);
  }
  if (req.apiVersion === '2') {
    return getUsers_v2(req, res);
  }
});

// Usage:
// GET /api/users
// Headers: { "API-Version": "2" }
```

---

## Part IV: Database Integration

### 4.1 MongoDB & Mongoose

MongoDB is a NoSQL database. Mongoose provides schema validation and query building.

#### Installation & Setup

```bash
npm install mongoose
```

```javascript
const mongoose = require('mongoose');

// Connection
mongoose.connect('mongodb://localhost:27017/myapp', {
  useNewUrlParser: true,
  useUnifiedTopology: true
})
.then(() => console.log('MongoDB connected'))
.catch(err => console.error('MongoDB connection error:', err));

// Connection events
mongoose.connection.on('connected', () => {
  console.log('Mongoose connected to MongoDB');
});

mongoose.connection.on('error', (err) => {
  console.error('Mongoose connection error:', err);
});

mongoose.connection.on('disconnected', () => {
  console.log('Mongoose disconnected');
});

// Graceful shutdown
process.on('SIGINT', async () => {
  await mongoose.connection.close();
  console.log('MongoDB connection closed');
  process.exit(0);
});
```

---

### 4.2 Schema Design

#### Basic Schema

```javascript
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: [true, 'Username is required'],
    unique: true,
    trim: true,
    minlength: [3, 'Username must be at least 3 characters'],
    maxlength: [30, 'Username cannot exceed 30 characters']
  },
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    trim: true,
    match: [/^\S+@\S+\.\S+$/, 'Please enter a valid email']
  },
  password: {
    type: String,
    required: [true, 'Password is required'],
    minlength: [8, 'Password must be at least 8 characters'],
    select: false // Don't include in queries by default
  },
  role: {
    type: String,
    enum: ['user', 'admin', 'moderator'],
    default: 'user'
  },
  avatar: {
    type: String,
    default: 'default-avatar.png'
  },
  isActive: {
    type: Boolean,
    default: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  },
  updatedAt: {
    type: Date,
    default: Date.now
  }
}, {
  timestamps: true // Automatically manage createdAt & updatedAt
});

// Indexes for performance
userSchema.index({ email: 1 });
userSchema.index({ username: 1 });
userSchema.index({ createdAt: -1 });

const User = mongoose.model('User', userSchema);
module.exports = User;
```

#### Advanced Schema with Relationships

```javascript
// Product Schema
const productSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true
  },
  description: String,
  price: {
    type: Number,
    required: true,
    min: 0
  },
  category: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Category', // Reference to Category model
    required: true
  },
  seller: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  images: [String],
  stock: {
    type: Number,
    default: 0,
    min: 0
  },
  ratings: [{
    user: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User'
    },
    rating: {
      type: Number,
      min: 1,
      max: 5
    },
    review: String,
    createdAt: {
      type: Date,
      default: Date.now
    }
  }],
  averageRating: {
    type: Number,
    default: 0
  }
}, { timestamps: true });

// Virtual field - not stored in DB
productSchema.virtual('isInStock').get(function() {
  return this.stock > 0;
});

// Pre-save middleware
productSchema.pre('save', function(next) {
  // Calculate average rating
  if (this.ratings.length > 0) {
    const sum = this.ratings.reduce((acc, r) => acc + r.rating, 0);
    this.averageRating = sum / this.ratings.length;
  }
  next();
});

const Product = mongoose.model('Product', productSchema);
```

---

### 4.3 Query Optimization

#### Basic CRUD Operations

```javascript
// CREATE
const user = await User.create({
  username: 'john',
  email: 'john@example.com',
  password: 'password123'
});

// READ - Find all
const users = await User.find();

// READ - Find one
const user = await User.findById(userId);
const user = await User.findOne({ email: 'john@example.com' });

// READ - Find with conditions
const users = await User.find({
  role: 'admin',
  isActive: true
})
.select('username email') // Only include these fields
.sort({ createdAt: -1 }) // Sort descending
.limit(10); // Limit results

// UPDATE
const user = await User.findByIdAndUpdate(
  userId,
  { username: 'newname' },
  { new: true, runValidators: true }
);

// DELETE
await User.findByIdAndDelete(userId);
```

#### Advanced Queries

```javascript
// Pagination
const page = 2;
const limit = 10;
const skip = (page - 1) * limit;

const users = await User.find()
  .skip(skip)
  .limit(limit);

const total = await User.countDocuments();

// Filtering
const products = await Product.find({
  price: { $gte: 100, $lte: 500 }, // Between 100 and 500
  stock: { $gt: 0 }, // Greater than 0
  category: { $in: ['electronics', 'gadgets'] } // In array
})
.sort({ price: 1 }); // Sort ascending

// Text Search
productSchema.index({ name: 'text', description: 'text' });

const results = await Product.find({
  $text: { $search: 'laptop gaming' }
});

// Population (JOIN)
const product = await Product.findById(productId)
  .populate('category', 'name') // Populate category, only include name
  .populate({
    path: 'seller',
    select: 'username email',
    match: { isActive: true }
  })
  .populate('ratings.user', 'username avatar');

// Aggregation
const stats = await Product.aggregate([
  { $match: { category: categoryId } },
  { $group: {
      _id: '$seller',
      totalProducts: { $sum: 1 },
      avgPrice: { $avg: '$price' },
      totalRevenue: { $sum: { $multiply: ['$price', '$stock'] } }
    }
  },
  { $sort: { totalRevenue: -1 } },
  { $limit: 10 }
]);
```

#### Performance Optimization

```javascript
// 1. Use indexes
userSchema.index({ email: 1 });
productSchema.index({ category: 1, price: 1 });

// 2. Use lean() for read-only operations (faster)
const users = await User.find().lean(); // Returns plain JS objects

// 3. Select only needed fields
const users = await User.find().select('username email');

// 4. Use limit() to reduce data transfer
const recent = await Post.find().sort({ createdAt: -1 }).limit(20);

// 5. Use populate() wisely - avoid over-population
const product = await Product.findById(id)
  .populate('category', 'name') // Only populate what you need
  .lean();
```

---

## Part V: Authentication & Authorization

### 5.1 JWT Implementation

**JWT (JSON Web Token)** is a secure way to transmit information between parties.

#### Installation

```bash
npm install jsonwebtoken bcryptjs
```

#### JWT Token Structure

```
Header.Payload.Signature

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJ1c2VySWQiOiIxMjM0NTY3ODkwIiwicm9sZSI6ImFkbWluIn0.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

#### Complete Authentication System

```javascript
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');
const User = require('../models/User');

// Generate JWT Token
const generateToken = (userId) => {
  return jwt.sign(
    { userId },
    process.env.JWT_SECRET,
    { expiresIn: '7d' }
  );
};

// REGISTER
exports.register = async (req, res, next) => {
  try {
    const { username, email, password } = req.body;

    // Check if user exists
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(409).json({ error: 'Email already registered' });
    }

    // Hash password
    const hashedPassword = await bcrypt.hash(password, 12);

    // Create user
    const user = await User.create({
      username,
      email,
      password: hashedPassword
    });

    // Generate token
    const token = generateToken(user._id);

    res.status(201).json({
      message: 'User registered successfully',
      token,
      user: {
        id: user._id,
        username: user.username,
        email: user.email
      }
    });
  } catch (error) {
    next(error);
  }
};

// LOGIN
exports.login = async (req, res, next) => {
  try {
    const { email, password } = req.body;

    // Find user (include password for verification)
    const user = await User.findOne({ email }).select('+password');
    
    if (!user) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    // Check password
    const isPasswordValid = await bcrypt.compare(password, user.password);
    
    if (!isPasswordValid) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    // Generate token
    const token = generateToken(user._id);

    res.json({
      message: 'Login successful',
      token,
      user: {
        id: user._id,
        username: user.username,
        email: user.email,
        role: user.role
      }
    });
  } catch (error) {
    next(error);
  }
};

// AUTHENTICATION MIDDLEWARE
exports.authenticate = async (req, res, next) => {
  try {
    // Get token from header
    const authHeader = req.headers.authorization;
    
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return res.status(401).json({ error: 'No token provided' });
    }

    const token = authHeader.split(' ')[1];

    // Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);

    // Find user
    const user = await User.findById(decoded.userId);
    
    if (!user) {
      return res.status(401).json({ error: 'User not found' });
    }

    // Attach user to request
    req.user = user;
    next();
  } catch (error) {
    if (error.name === 'JsonWebTokenError') {
      return res.status(401).json({ error: 'Invalid token' });
    }
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'Token expired' });
    }
    next(error);
  }
};

// Usage in routes
router.post('/register', authController.register);
router.post('/login', authController.login);
router.get('/profile', authController.authenticate, userController.getProfile);
```

---

### 5.2 Password Security

#### Bcrypt Hashing

```javascript
const bcrypt = require('bcryptjs');

// Hash password (on registration)
const hashPassword = async (password) => {
  const salt = await bcrypt.genSalt(12); // Cost factor: 12
  const hashedPassword = await bcrypt.hash(password, salt);
  return hashedPassword;
};

// Verify password (on login)
const verifyPassword = async (plainPassword, hashedPassword) => {
  const isValid = await bcrypt.compare(plainPassword, hashedPassword);
  return isValid;
};

// In User Model - Auto-hash before saving
userSchema.pre('save', async function(next) {
  // Only hash if password is modified
  if (!this.isModified('password')) return next();
  
  this.password = await bcrypt.hash(this.password, 12);
  next();
});

// In User Model - Method to verify password
userSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};
```

#### Password Reset Flow

```javascript
const crypto = require('crypto');

// 1. Request Password Reset
exports.forgotPassword = async (req, res, next) => {
  try {
    const user = await User.findOne({ email: req.body.email });
    
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }

    // Generate reset token
    const resetToken = crypto.randomBytes(32).toString('hex');
    
    // Hash token and save to database
    user.passwordResetToken = crypto
      .createHash('sha256')
      .update(resetToken)
      .digest('hex');
    
    user.passwordResetExpires = Date.now() + 10 * 60 * 1000; // 10 minutes
    await user.save();

    // Send email with reset link (simplified)
    const resetUrl = `${req.protocol}://${req.get('host')}/reset-password/${resetToken}`;
    // await sendEmail(user.email, 'Password Reset', resetUrl);

    res.json({ message: 'Reset token sent to email' });
  } catch (error) {
    next(error);
  }
};

// 2. Reset Password
exports.resetPassword = async (req, res, next) => {
  try {
    // Hash the token from URL
    const hashedToken = crypto
      .createHash('sha256')
      .update(req.params.token)
      .digest('hex');

    // Find user with valid token
    const user = await User.findOne({
      passwordResetToken: hashedToken,
      passwordResetExpires: { $gt: Date.now() }
    });

    if (!user) {
      return res.status(400).json({ error: 'Invalid or expired token' });
    }

    // Set new password
    user.password = req.body.password;
    user.passwordResetToken = undefined;
    user.passwordResetExpires = undefined;
    await user.save();

    res.json({ message: 'Password reset successful' });
  } catch (error) {
    next(error);
  }
};
```

---

### 5.3 Role-Based Access Control

```javascript
// Authorization Middleware
exports.authorize = (...roles) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        error: 'You do not have permission to perform this action'
      });
    }

    next();
  };
};

// Usage in routes
router.delete('/users/:id',
  authenticate,
  authorize('admin'),
  userController.deleteUser
);

router.post('/products',
  authenticate,
  authorize('admin', 'seller'),
  productController.createProduct
);

// Resource-based Authorization
exports.authorizeProductOwner = async (req, res, next) => {
  try {
    const product = await Product.findById(req.params.id);
    
    if (!product) {
      return res.status(404).json({ error: 'Product not found' });
    }

    // Check if user is owner or admin
    if (product.seller.toString() !== req.user.id && req.user.role !== 'admin') {
      return res.status(403).json({
        error: 'You can only modify your own products'
      });
    }

    req.product = product;
    next();
  } catch (error) {
    next(error);
  }
};

// Usage
router.put('/products/:id',
  authenticate,
  authorizeProductOwner,
  productController.updateProduct
);
```

---

## Part VI: Testing & Debugging

### 6.1 Unit Testing with Jest

#### Installation

```bash
npm install --save-dev jest supertest
```

#### package.json

```json
{
  "scripts": {
    "test": "jest --watchAll --verbose"
  },
  "jest": {
    "testEnvironment": "node"
  }
}
```

#### Test Structure

```javascript
// userController.test.js
const request = require('supertest');
const mongoose = require('mongoose');
const app = require('../app');
const User = require('../models/User');

// Setup & Teardown
beforeAll(async () => {
  await mongoose.connect('mongodb://localhost:27017/test-db');
});

afterAll(async () => {
  await mongoose.connection.close();
});

beforeEach(async () => {
  await User.deleteMany({});
});

// Test Suite
describe('User API', () => {
  
  describe('POST /api/users/register', () => {
    it('should register a new user', async () => {
      const userData = {
        username: 'testuser',
        email: 'test@example.com',
        password: 'password123'
      };

      const response = await request(app)
        .post('/api/users/register')
        .send(userData)
        .expect(201);

      expect(response.body).toHaveProperty('token');
      expect(response.body.user.username).toBe('testuser');
      expect(response.body.user.email).toBe('test@example.com');
    });

    it('should not register user with existing email', async () => {
      // Create existing user
      await User.create({
        username: 'existing',
        email: 'test@example.com',
        password: 'password123'
      });

      const response = await request(app)
        .post('/api/users/register')
        .send({
          username: 'newuser',
          email: 'test@example.com',
          password: 'password123'
        })
        .expect(409);

      expect(response.body.error).toBe('Email already registered');
    });

    it('should validate required fields', async () => {
      const response = await request(app)
        .post('/api/users/register')
        .send({
          username: 'testuser'
          // Missing email and password
        })
        .expect(400);

      expect(response.body).toHaveProperty('error');
    });
  });

  describe('POST /api/users/login', () => {
    beforeEach(async () => {
      // Create test user
      await User.create({
        username: 'testuser',
        email: 'test@example.com',
        password: await bcrypt.hash('password123', 12)
      });
    });

    it('should login with correct credentials', async () => {
      const response = await request(app)
        .post('/api/users/login')
        .send({
          email: 'test@example.com',
          password: 'password123'
        })
        .expect(200);

      expect(response.body).toHaveProperty('token');
      expect(response.body.user.email).toBe('test@example.com');
    });

    it('should not login with incorrect password', async () => {
      const response = await request(app)
        .post('/api/users/login')
        .send({
          email: 'test@example.com',
          password: 'wrongpassword'
        })
        .expect(401);

      expect(response.body.error).toBe('Invalid credentials');
    });
  });

  describe('GET /api/users/:id', () => {
    it('should get user by ID', async () => {
      const user = await User.create({
        username: 'testuser',
        email: 'test@example.com',
        password: 'password123'
      });

      const response = await request(app)
        .get(`/api/users/${user._id}`)
        .expect(200);

      expect(response.body.data.username).toBe('testuser');
    });

    it('should return 404 for non-existent user', async () => {
      const fakeId = new mongoose.Types.ObjectId();
      
      const response = await request(app)
        .get(`/api/users/${fakeId}`)
        .expect(404);

      expect(response.body.error).toBe('User not found');
    });
  });
});
```

---

### 6.2 Integration Testing

```javascript
// Integration test for complete user workflow
describe('User Workflow Integration', () => {
  let userToken;
  let userId;

  it('should complete full user lifecycle', async () => {
    // 1. Register
    const registerRes = await request(app)
      .post('/api/users/register')
      .send({
        username: 'testuser',
        email: 'test@example.com',
        password: 'password123'
      })
      .expect(201);

    userToken = registerRes.body.token;
    userId = registerRes.body.user.id;

    // 2. Login
    const loginRes = await request(app)
      .post('/api/users/login')
      .send({
        email: 'test@example.com',
        password: 'password123'
      })
      .expect(200);

    expect(loginRes.body.token).toBeDefined();

    // 3. Get Profile (Authenticated)
    const profileRes = await request(app)
      .get('/api/users/profile')
      .set('Authorization', `Bearer ${userToken}`)
      .expect(200);

    expect(profileRes.body.data.username).toBe('testuser');

    // 4. Update Profile
    const updateRes = await request(app)
      .put(`/api/users/${userId}`)
      .set('Authorization', `Bearer ${userToken}`)
      .send({ username: 'updateduser' })
      .expect(200);

    expect(updateRes.body.data.username).toBe('updateduser');

    // 5. Delete User (Admin only)
    await request(app)
      .delete(`/api/users/${userId}`)
      .set('Authorization', `Bearer ${userToken}`)
      .expect(403); // Regular user can't delete
  });
});
```

---

### 6.3 Debugging Techniques

#### Console Logging

```javascript
// Basic logging
console.log('User:', user);
console.log('Request body:', req.body);

// Structured logging
console.log({
  timestamp: new Date().toISOString(),
  method: req.method,
  url: req.url,
  userId: req.user?.id
});

// Error logging
console.error('Database error:', error);
console.error('Stack trace:', error.stack);
```

#### Debug Module

```bash
npm install debug
```

```javascript
const debug = require('debug')('app:database');

debug('Connecting to MongoDB...');
debug('User query: %O', { email: 'test@example.com' });
debug('Query took %dms', duration);

// Run with: DEBUG=app:* node server.js
```

#### VS Code Debugger

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Server",
      "program": "${workspaceFolder}/server.js",
      "restart": true,
      "console": "integratedTerminal",
      "env": {
        "NODE_ENV": "development"
      }
    }
  ]
}
```

#### Request Logging Middleware

```javascript
const morgan = require('morgan');

// Development logging
app.use(morgan('dev'));

// Production logging
app.use(morgan('combined', {
  stream: fs.createWriteStream('./access.log', { flags: 'a' })
}));

// Custom format
app.use(morgan(':method :url :status :response-time ms - :res[content-length]'));
```

---

## Practical Assignments

### Assignment 1: Build a Blog API

**Requirements:**
- User authentication (register, login)
- CRUD operations for blog posts
- Comments on posts
- Like/unlike posts
- Search and filter posts
- Pagination

**Endpoints:**
```
POST   /api/auth/register
POST   /api/auth/login
GET    /api/posts
POST   /api/posts
GET    /api/posts/:id
PUT    /api/posts/:id
DELETE /api/posts/:id
POST   /api/posts/:id/comments
POST   /api/posts/:id/like
DELETE /api/posts/:id/like
```

---

### Assignment 2: E-commerce Product API

**Requirements:**
- Product CRUD with categories
- Product search and filters
- User cart management
- Order creation and tracking
- Admin-only product management

**Database Models:**
- User (with cart)
- Product
- Category
- Order

---

### Assignment 3: Task Management API

**Requirements:**
- User authentication
- Create, read, update, delete tasks
- Task status (todo, in-progress, done)
- Assign tasks to users
- Task priorities and due dates
- Filter tasks by status, priority

---

## Assessment Criteria

### Week 2 Evaluation (100 points)

1. **Node.js Fundamentals (20 points)**
   - Event loop understanding
   - Async/await patterns
   - Non-blocking I/O

2. **Express.js Implementation (25 points)**
   - Middleware usage
   - Route organization
   - Error handling

3. **API Design (20 points)**
   - RESTful principles
   - Status codes
   - Versioning

4. **Database Integration (20 points)**
   - Schema design
   - Query optimization
   - Relationships

5. **Security (15 points)**
   - JWT implementation
   - Password hashing
   - Input validation

---

## Reference Materials

### Official Documentation
- [Node.js Documentation](https://nodejs.org/docs)
- [Express.js Guide](https://expressjs.com)
- [MongoDB Manual](https://docs.mongodb.com)
- [Mongoose Documentation](https://mongoosejs.com/docs)

### Recommended Reading
- "Node.js Design Patterns" by Mario Casciaro
- "Express in Action" by Evan Hahn
- REST API Design Rulebook by Mark Masse

### Video Resources
- Node.js Event Loop Explained
- Building REST APIs with Express
- MongoDB University Courses

### Tools & Libraries
- Postman - API testing
- MongoDB Compass - Database GUI
- Jest - Testing framework
- Morgan - HTTP logging
- Helmet - Security headers
---
**Remember: Backend development is about building reliable, scalable, and secure systems. Master the fundamentals before moving to advanced topics!**
