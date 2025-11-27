# Design Patterns Documentation
## Learning Management System

### Table of Contents
1. [Overview](#overview)
2. [Creational Patterns](#creational-patterns)
3. [Structural Patterns](#structural-patterns)
4. [Behavioral Patterns](#behavioral-patterns)
5. [Architectural Patterns](#architectural-patterns)
6. [Express.js Specific Patterns](#expressjs-specific-patterns)
7. [Implementation Examples](#implementation-examples)
8. [Benefits and Trade-offs](#benefits-and-trade-offs)

---

## Overview

This document identifies and documents the design patterns implemented in the Learning Management System (LMS) codebase. The system demonstrates various design patterns that improve code organization, maintainability, and scalability.

### Pattern Categories Identified:
- **Creational Patterns**: Singleton, Factory, Builder
- **Structural Patterns**: Adapter, Decorator, Facade, Proxy
- **Behavioral Patterns**: Middleware, Chain of Responsibility, Observer, Strategy, Template Method
- **Architectural Patterns**: MVC, Layered Architecture, Repository Pattern
- **Express.js Patterns**: Middleware Pipeline, Route Handlers, Error Handling

---

## Creational Patterns

### 1. Singleton Pattern

**Purpose**: Ensure a class has only one instance and provide global access to it.

**Implementation**: Database connections, logger instances, and external service configurations.

#### Example: Logger Singleton
```javascript
// utils/logger.js
const logger = winston.createLogger({
    level: 'error',
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.label({ label: 'user-service' }),
        winston.format.printf(({ level, message, name, code, stack, label, timestamp }) => {
            return `${timestamp} [${label}] ${level}: ${message} : ${name} : ${code} : ${stack}`;
        })
    ),
    transports: [
        new winston.transports.File({ filename: 'error.log', level: 'error' }),
    ],
});

export default logger; // Single instance exported
```

**Usage**: Used throughout the application for consistent logging.

### 2. Factory Pattern

**Purpose**: Create objects without specifying their exact classes.

**Implementation**: Multer configuration factory, Cloudinary upload factory.

#### Example: Multer Upload Factory
```javascript
// middlewares/multer.middleware.js
const upload = multer({
    dest: "uploads/",
    limits: {
        fileSize: 1024 * 1024 * 50 // 50MB
    },
    storage: multer.diskStorage({
        destination: "uploads/",
        filename: (_req, file, cb) => {
            cb(null, `${file.fieldname}-${Date.now()}${path.extname(file.originalname)}`)
        },
    }),
    fileFilter: (_req, file, cb) => {
        let ext = path.extname(file.originalname);
        if (ext !== '.png' && ext !== '.jpg' && ext !== '.jpeg' && ext !== '.webp' && ext !== '.mp4') {
            cb(new Error('File type is not supported'), false)
            return;
        }
        cb(null, true);
    },
});

export default upload; // Factory instance
```

**Usage**: Creates configured multer instances for different file upload scenarios.

### 3. Builder Pattern

**Purpose**: Construct complex objects step by step.

**Implementation**: JWT token building, email message building, database query building.

#### Example: Cookie Options Builder
```javascript
// controllers/user.controller.js
const cookieOptions = {
    expires: new Date(Date.now() + process.env.JWT_COOKIE_EXPIRY * 24 * 60 * 60 * 1000),
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: process.env.NODE_ENV === 'production' ? 'none' : 'lax',
    path: '/'
}
```

**Usage**: Builds cookie configuration objects with different properties based on environment.

---

## Structural Patterns

### 1. Adapter Pattern

**Purpose**: Allow incompatible interfaces to work together.

**Implementation**: External service adapters (Razorpay, Cloudinary, Email).

#### Example: Razorpay Adapter
```javascript
// server.js
export var razorpay = new Razorpay({
    key_id: process.env.RAZORPAY_KEY_ID,
    key_secret: process.env.RAZORPAY_SECRET,
});

// controllers/payment.controller.js
const subscription = await razorpay.subscriptions.create({
    plan_id: process.env.RAZORPAY_PLAN_ID,
    customer_notify: 1,
    total_count: 12
});
```

**Usage**: Adapts Razorpay API to internal payment service interface.

### 2. Decorator Pattern

**Purpose**: Add behavior to objects dynamically.

**Implementation**: Middleware decorators, validation decorators.

#### Example: Async Handler Decorator
```javascript
// middlewares/asyncHandler.middleware.js
const asyncHandler = (fn) => {
    return (req, res, next) => {
        fn(req, res, next).catch((err) => next(err));
    };
};

export default asyncHandler;
```

**Usage**: Decorates async route handlers to automatically catch and forward errors.

### 3. Facade Pattern

**Purpose**: Provide a simplified interface to a complex subsystem.

**Implementation**: Email service facade, file upload facade.

#### Example: Email Service Facade
```javascript
// utils/sendEmail.js
const sendEmail = async function(email, subject, message) {
    let transporter = nodemailer.createTransporter({
        host: process.env.SMTP_HOST,
        port: process.env.SMTP_PORT,
        secure: false,
        auth: {
            user: process.env.SMTP_USERNAME,
            pass: process.env.SMTP_PASSWORD
        },
    });

    await transporter.sendMail({
        from: process.env.SMTP_FROM_EMAIL,
        to: email,
        subject: subject,
        html: message
    });
};

export default sendEmail;
```

**Usage**: Simplifies complex nodemailer configuration and usage.

### 4. Proxy Pattern

**Purpose**: Provide a placeholder or surrogate for another object.

**Implementation**: Database connection proxy, file upload proxy.

#### Example: Database Connection Proxy
```javascript
// config/dbConnection.js
const connectToDB = async () => {
    try {
        const { connection } = await mongoose.connect(process.env.MONGO_URI);
        console.log(`MongoDB connected to ${connection.host}`);
    } catch (error) {
        console.log(`Error: ${error.message}`);
        process.exit(1);
    }
}
```

**Usage**: Proxies MongoDB connections with error handling and logging.

---

## Behavioral Patterns

### 1. Middleware Pattern (Chain of Responsibility)

**Purpose**: Pass requests along a chain of handlers.

**Implementation**: Express.js middleware stack, authentication chain.

#### Example: Authentication Middleware Chain
```javascript
// middlewares/auth.middleware.js
export const isLoggedIn = (req, res, next) => {
    const { token } = req.cookies;
    
    if (!token) {
        return next(new AppError('Please login to continue. No authentication token found.', 401));
    }
    
    try {
        const decode = jwt.verify(token, process.env.JWT_SECRET);
        req.user = decode;
        next();
    } catch (error) {
        return next(new AppError('Authentication failed', 401));
    }
}

export const authorizedRoles = (...roles) => (req, res, next) => {
    const currentRole = req.user;
    if (!roles.includes(currentRole.role)) {
        return next(new AppError('You are not authorized to access this route', 403));
    }
    next();
};
```

**Usage**: Chains authentication and authorization middleware for protected routes.

### 2. Observer Pattern

**Purpose**: Define a one-to-many dependency between objects.

**Implementation**: Event logging, error tracking.

#### Example: Error Observer
```javascript
// middlewares/error.middleware.js
const errorMiddleware = (err, req, res, next) => {
    req.statusCode = req.statusCode || 500;
    req.statusMessage = req.statusMessage || 'Internal Server Error caught by errorMiddleware';

    logger.error({
        message: req.statusMessage,
        name: err.name,
        code: err.code,
        stack: err.stack
    });

    return res.status(req.statusCode).json({
        success: false,
        message: req.statusMessage,
        name: err.name,
        code: err.code,
        stack: err.stack
    });
}
```

**Usage**: Observes errors and notifies logging system and error handlers.

### 3. Strategy Pattern

**Purpose**: Define a family of algorithms and make them interchangeable.

**Implementation**: Authentication strategies, validation strategies.

#### Example: Password Validation Strategy
```javascript
// models/user.model.js
userSchema.methods = {
    comparePassword: function(plainTextPassword) {
        return bcrypt.compare(plainTextPassword, this.password);
    },
    generateJWTToken: async function() {
        return await jwt.sign({
            id: this.id,
            role: this.role,
            subscription: this.subscription
        }, process.env.JWT_SECRET, {
            expiresIn: process.env.JWT_EXPIRY || '7d'
        });
    }
};
```

**Usage**: Different strategies for password comparison and token generation.

### 4. Template Method Pattern

**Purpose**: Define the skeleton of an algorithm in a method.

**Implementation**: Controller method templates, service method templates.

#### Example: Controller Template Method
```javascript
// controllers/course.controller.js
export const getAllCourses = async (req, res, next) => {
    try {
        const courses = await Course.find({}).select("-lectures");
        res.status(200).json({
            status: "success",
            message: "All courses",
            data: courses
        })
    } catch (error) {
        return next(new AppError(error.message || "unable to fetch the course", 500))
    }
};
```

**Usage**: Template for standard controller response format with error handling.

---

## Architectural Patterns

### 1. Model-View-Controller (MVC) Pattern

**Purpose**: Separate application logic into three interconnected components.

**Implementation**: 
- **Models**: User, Course, Payment, AdminRequest
- **Views**: JSON responses
- **Controllers**: UserController, CourseController, PaymentController

#### Example: MVC Structure
```
models/user.model.js     -> Data and business logic
controllers/user.controller.js -> Request handling
routes/user.routes.js    -> URL mapping
```

**Usage**: Organizes code into logical components with clear separation of concerns.

### 2. Layered Architecture Pattern

**Purpose**: Organize code into horizontal layers with specific responsibilities.

**Implementation**: 
- **Presentation Layer**: Routes, Controllers
- **Business Logic Layer**: Services, Models
- **Data Access Layer**: Mongoose models
- **Infrastructure Layer**: External services

#### Example: Layered Structure
```
routes/           -> Presentation Layer
controllers/      -> Presentation Layer
models/           -> Data Access Layer
middlewares/      -> Infrastructure Layer
utils/            -> Infrastructure Layer
```

**Usage**: Provides clear separation of concerns and maintainable code structure.

### 3. Repository Pattern

**Purpose**: Encapsulate data access logic and provide a uniform interface.

**Implementation**: Mongoose models as repositories.

#### Example: User Repository
```javascript
// models/user.model.js
const User = model('User', userSchema);

// Usage in controllers
const user = await User.findById(id);
const users = await User.find({});
await User.create(userData);
```

**Usage**: Abstracts database operations and provides consistent data access interface.

---

## Express.js Specific Patterns

### 1. Middleware Pipeline Pattern

**Purpose**: Process requests through a series of middleware functions.

**Implementation**: Express.js middleware stack.

#### Example: Middleware Pipeline
```javascript
// app.js
app.use(express.json());                    // Body parsing
app.use(express.urlencoded({ extended: true })); // URL encoding
app.use(cookieParser());                    // Cookie parsing
app.use(morgan('dev'));                     // Logging
app.use(cors({...}));                       // CORS handling
app.use('/api/v1/user', userRoutes);        // Route mounting
app.use(errorMiddleware);                   // Error handling
```

**Usage**: Processes requests through a defined pipeline of middleware functions.

### 2. Route Handler Pattern

**Purpose**: Handle HTTP requests for specific routes.

**Implementation**: Express.js route handlers.

#### Example: Route Handler
```javascript
// routes/user.routes.js
router.post('/register', upload.single('avatar'), register);
router.post('/login', login);
router.get('/profile', isLoggedIn, getProfile);
```

**Usage**: Maps HTTP methods and URLs to controller functions with middleware.

### 3. Error Handling Pattern

**Purpose**: Centralized error handling and response formatting.

**Implementation**: Error middleware and custom error classes.

#### Example: Error Handling
```javascript
// utils/appError.js
class AppError extends Error {
    constructor(message, statusCode) {
        super(message);
        this.statusCode = statusCode;
        Error.captureStackTrace(this, this.constructor);
    }
}

// middlewares/error.middleware.js
const errorMiddleware = (err, req, res, next) => {
    // Error processing logic
};
```

**Usage**: Provides consistent error handling and response formatting across the application.

---

## Implementation Examples

### 1. Authentication Flow Pattern
```javascript
// Route with middleware chain
router.get('/profile', isLoggedIn, getProfile);

// Middleware chain execution
isLoggedIn -> getProfile -> response
```

### 2. File Upload Pattern
```javascript
// Multer middleware configuration
const upload = multer({
    dest: "uploads/",
    limits: { fileSize: 1024 * 1024 * 50 },
    fileFilter: (req, file, cb) => {
        // File validation logic
    }
});

// Usage in routes
router.post('/register', upload.single('avatar'), register);
```

### 3. Database Operation Pattern
```javascript
// Repository pattern with error handling
export const getAllCourses = async (req, res, next) => {
    try {
        const courses = await Course.find({}).select("-lectures");
        res.status(200).json({
            status: "success",
            message: "All courses",
            data: courses
        });
    } catch (error) {
        return next(new AppError(error.message || "unable to fetch the course", 500));
    }
};
```

---

## Benefits and Trade-offs

### Benefits

1. **Maintainability**: Clear separation of concerns makes code easier to maintain
2. **Scalability**: Modular architecture supports system growth
3. **Reusability**: Patterns enable code reuse across different parts of the system
4. **Testability**: Separated components are easier to unit test
5. **Consistency**: Standardized patterns provide consistent code structure
6. **Error Handling**: Centralized error handling improves reliability

### Trade-offs

1. **Complexity**: Additional abstraction layers can increase complexity
2. **Performance**: Some patterns may introduce slight performance overhead
3. **Learning Curve**: Team members need to understand the implemented patterns
4. **Over-engineering**: Risk of implementing patterns where simpler solutions would suffice

### Recommendations

1. **Use patterns consistently** across the codebase
2. **Document pattern usage** for team understanding
3. **Balance abstraction** with simplicity
4. **Regular code reviews** to ensure pattern adherence
5. **Refactor when patterns** become outdated or inappropriate

---

## Conclusion

The Learning Management System implements a comprehensive set of design patterns that improve code organization, maintainability, and scalability. The combination of creational, structural, behavioral, and architectural patterns creates a robust and well-structured application that follows industry best practices.

These patterns work together to provide:
- Clear separation of concerns
- Consistent error handling
- Modular architecture
- Reusable components
- Scalable design

The implementation demonstrates how design patterns can be effectively applied in a Node.js/Express.js application to create maintainable and professional-grade software.
