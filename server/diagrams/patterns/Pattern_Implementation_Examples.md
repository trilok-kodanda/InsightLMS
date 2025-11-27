# Design Pattern Implementation Examples
## Learning Management System

## Code Examples and Implementation Details

### Table of Contents
1. [Creational Patterns](#creational-patterns)
2. [Structural Patterns](#structural-patterns)
3. [Behavioral Patterns](#behavioral-patterns)
4. [Architectural Patterns](#architectural-patterns)
5. [Express.js Patterns](#expressjs-patterns)
6. [Pattern Testing Examples](#pattern-testing-examples)

---

## Creational Patterns

### 1. Singleton Pattern Implementation

#### Logger Singleton
```javascript
// utils/logger.js
import winston from 'winston';

class Logger {
    constructor() {
        if (Logger.instance) {
            return Logger.instance;
        }
        
        this.logger = winston.createLogger({
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
        
        Logger.instance = this;
    }
    
    info(message, meta) {
        this.logger.info(message, meta);
    }
    
    error(message, meta) {
        this.logger.error(message, meta);
    }
    
    warn(message, meta) {
        this.logger.warn(message, meta);
    }
}

export default new Logger(); // Single instance
```

#### Usage Example:
```javascript
// controllers/user.controller.js
import logger from '../utils/logger.js';

export const register = async (req, res, next) => {
    try {
        // Registration logic
        logger.info('User registration successful', { userId: user._id });
    } catch (error) {
        logger.error('Registration failed', { error: error.message });
    }
};
```

### 2. Factory Pattern Implementation

#### File Upload Factory
```javascript
// middlewares/multer.middleware.js
import multer from 'multer';
import path from 'path';

class FileUploadFactory {
    static createUploader(options = {}) {
        const defaultOptions = {
            dest: "uploads/",
            limits: {
                fileSize: 1024 * 1024 * 50 // 50MB
            },
            storage: multer.diskStorage({
                destination: "uploads/",
                filename: (_req, file, cb) => {
                    cb(null, `${file.fieldname}-${Date.now()}${path.extname(file.originalname)}`);
                },
            }),
            fileFilter: (_req, file, cb) => {
                let ext = path.extname(file.originalname);
                const allowedTypes = ['.png', '.jpg', '.jpeg', '.webp', '.mp4'];
                
                if (!allowedTypes.includes(ext)) {
                    cb(new Error('File type is not supported'), false);
                    return;
                }
                cb(null, true);
            },
        };
        
        return multer({ ...defaultOptions, ...options });
    }
    
    static createImageUploader() {
        return this.createUploader({
            fileFilter: (_req, file, cb) => {
                const allowedTypes = ['.png', '.jpg', '.jpeg', '.webp'];
                const ext = path.extname(file.originalname);
                
                if (!allowedTypes.includes(ext)) {
                    cb(new Error('Only image files are allowed'), false);
                    return;
                }
                cb(null, true);
            }
        });
    }
    
    static createVideoUploader() {
        return this.createUploader({
            fileFilter: (_req, file, cb) => {
                const allowedTypes = ['.mp4', '.avi', '.mov'];
                const ext = path.extname(file.originalname);
                
                if (!allowedTypes.includes(ext)) {
                    cb(new Error('Only video files are allowed'), false);
                    return;
                }
                cb(null, true);
            }
        });
    }
}

export default FileUploadFactory.createUploader();
```

#### Usage Example:
```javascript
// routes/user.routes.js
import FileUploadFactory from '../middlewares/multer.middleware.js';

const imageUpload = FileUploadFactory.createImageUploader();
const videoUpload = FileUploadFactory.createVideoUploader();

router.post('/register', imageUpload.single('avatar'), register);
router.post('/upload-video', videoUpload.single('video'), uploadVideo);
```

### 3. Builder Pattern Implementation

#### Email Builder
```javascript
// utils/emailBuilder.js
class EmailBuilder {
    constructor() {
        this.email = {
            to: '',
            subject: '',
            html: '',
            from: process.env.SMTP_FROM_EMAIL
        };
    }
    
    setTo(email) {
        this.email.to = email;
        return this;
    }
    
    setSubject(subject) {
        this.email.subject = subject;
        return this;
    }
    
    setHtml(html) {
        this.email.html = html;
        return this;
    }
    
    buildPasswordResetEmail(resetUrl) {
        this.email.subject = 'Password Reset Request';
        this.email.html = `
            <div style="font-family: Arial, sans-serif; padding: 20px;">
                <h2>Password Reset Request</h2>
                <p>Click the link below to reset your password:</p>
                <a href="${resetUrl}" style="background-color: #4CAF50; color: white; padding: 10px 20px; text-decoration: none; border-radius: 4px;">Reset Password</a>
                <p>If you didn't request this, please ignore this email.</p>
            </div>
        `;
        return this;
    }
    
    buildWelcomeEmail(userName) {
        this.email.subject = 'Welcome to Learning Management System';
        this.email.html = `
            <div style="font-family: Arial, sans-serif; padding: 20px;">
                <h2>Welcome ${userName}!</h2>
                <p>Thank you for joining our Learning Management System.</p>
                <p>You can now access all our courses and start learning.</p>
            </div>
        `;
        return this;
    }
    
    build() {
        return this.email;
    }
}

export default EmailBuilder;
```

#### Usage Example:
```javascript
// controllers/user.controller.js
import EmailBuilder from '../utils/emailBuilder.js';
import sendEmail from '../utils/sendEmail.js';

export const forgetPassword = async (req, res, next) => {
    const { email } = req.body;
    
    const resetUrl = `${process.env.FRONTEND_URL}/reset-password/${resetToken}`;
    
    const emailData = new EmailBuilder()
        .setTo(email)
        .buildPasswordResetEmail(resetUrl)
        .build();
    
    await sendEmail(emailData.to, emailData.subject, emailData.html);
};
```

---

## Structural Patterns

### 1. Adapter Pattern Implementation

#### Razorpay Adapter
```javascript
// services/razorpayAdapter.js
import Razorpay from 'razorpay';

class RazorpayAdapter {
    constructor() {
        this.razorpay = new Razorpay({
            key_id: process.env.RAZORPAY_KEY_ID,
            key_secret: process.env.RAZORPAY_SECRET,
        });
    }
    
    async createSubscription(planId, customerNotify = 1, totalCount = 12) {
        try {
            const subscription = await this.razorpay.subscriptions.create({
                plan_id: planId,
                customer_notify: customerNotify,
                total_count: totalCount
            });
            return {
                success: true,
                data: subscription,
                message: 'Subscription created successfully'
            };
        } catch (error) {
            return {
                success: false,
                error: error.message,
                message: 'Failed to create subscription'
            };
        }
    }
    
    async cancelSubscription(subscriptionId) {
        try {
            const subscription = await this.razorpay.subscriptions.cancel(subscriptionId);
            return {
                success: true,
                data: subscription,
                message: 'Subscription cancelled successfully'
            };
        } catch (error) {
            return {
                success: false,
                error: error.message,
                message: 'Failed to cancel subscription'
            };
        }
    }
    
    async processRefund(paymentId, amount = null) {
        try {
            const refundOptions = { speed: "optimum" };
            if (amount) refundOptions.amount = amount;
            
            const refund = await this.razorpay.payments.refund(paymentId, refundOptions);
            return {
                success: true,
                data: refund,
                message: 'Refund processed successfully'
            };
        } catch (error) {
            return {
                success: false,
                error: error.message,
                message: 'Failed to process refund'
            };
        }
    }
}

export default new RazorpayAdapter();
```

#### Usage Example:
```javascript
// controllers/payment.controller.js
import razorpayAdapter from '../services/razorpayAdapter.js';

export const buySubscription = async (req, res, next) => {
    try {
        const { id } = req.user;
        const user = await User.findById(id);
        
        const result = await razorpayAdapter.createSubscription(
            process.env.RAZORPAY_PLAN_ID
        );
        
        if (result.success) {
            user.subscription.id = result.data.id;
            user.subscription.status = result.data.status;
            await user.save();
            
            res.status(200).json({
                success: true,
                message: "Subscription created successfully",
                subscription_id: result.data.id
            });
        } else {
            return next(new AppError(result.message, 400));
        }
    } catch (error) {
        return next(new AppError(error.message, 500));
    }
};
```

### 2. Decorator Pattern Implementation

#### Async Handler Decorator
```javascript
// middlewares/asyncHandler.middleware.js
const asyncHandler = (fn) => {
    return (req, res, next) => {
        Promise.resolve(fn(req, res, next)).catch(next);
    };
};

// Enhanced version with logging
const asyncHandlerWithLogging = (fn, operation = 'Unknown') => {
    return (req, res, next) => {
        const startTime = Date.now();
        
        Promise.resolve(fn(req, res, next))
            .then(() => {
                const duration = Date.now() - startTime;
                console.log(`${operation} completed in ${duration}ms`);
            })
            .catch((error) => {
                const duration = Date.now() - startTime;
                console.error(`${operation} failed after ${duration}ms:`, error.message);
                next(error);
            });
    };
};

// Validation decorator
const withValidation = (schema) => {
    return (fn) => {
        return async (req, res, next) => {
            try {
                await schema.validateAsync(req.body);
                return fn(req, res, next);
            } catch (error) {
                return next(new AppError(error.details[0].message, 400));
            }
        };
    };
};

export { asyncHandler, asyncHandlerWithLogging, withValidation };
```

#### Usage Example:
```javascript
// controllers/user.controller.js
import { asyncHandler, withValidation } from '../middlewares/asyncHandler.middleware.js';
import Joi from 'joi';

const userRegistrationSchema = Joi.object({
    fullName: Joi.string().min(3).max(30).required(),
    email: Joi.string().email().required(),
    password: Joi.string().min(6).required()
});

export const register = asyncHandler(
    withValidation(userRegistrationSchema)(
        async (req, res, next) => {
            // Registration logic
            const { fullName, email, password } = req.body;
            // ... implementation
        }
    )
);
```

### 3. Facade Pattern Implementation

#### File Upload Facade
```javascript
// services/fileUploadFacade.js
import cloudinary from 'cloudinary';
import fs from 'fs/promises';
import path from 'path';

class FileUploadFacade {
    constructor() {
        this.cloudinary = cloudinary;
    }
    
    async uploadImage(file, folder = 'lms') {
        try {
            const result = await this.cloudinary.v2.uploader.upload(file.path, {
                folder: folder,
                width: 250,
                height: 250,
                gravity: 'faces',
                crop: 'fill'
            });
            
            // Clean up local file
            await this.cleanupLocalFile(file.path);
            
            return {
                success: true,
                public_id: result.public_id,
                secure_url: result.secure_url
            };
        } catch (error) {
            return {
                success: false,
                error: error.message
            };
        }
    }
    
    async uploadVideo(file, folder = 'lms') {
        try {
            const result = await this.cloudinary.v2.uploader.upload(file.path, {
                resource_type: 'video',
                folder: folder,
                chunk_size: 5000000 // 50MB
            });
            
            // Clean up local file
            await this.cleanupLocalFile(file.path);
            
            return {
                success: true,
                public_id: result.public_id,
                secure_url: result.secure_url
            };
        } catch (error) {
            return {
                success: false,
                error: error.message
            };
        }
    }
    
    async deleteFile(publicId, resourceType = 'image') {
        try {
            await this.cloudinary.v2.uploader.destroy(publicId, {
                resource_type: resourceType
            });
            return { success: true };
        } catch (error) {
            return {
                success: false,
                error: error.message
            };
        }
    }
    
    async cleanupLocalFile(filePath) {
        try {
            await fs.rm(filePath);
        } catch (error) {
            console.error('Failed to cleanup local file:', error.message);
        }
    }
}

export default new FileUploadFacade();
```

#### Usage Example:
```javascript
// controllers/course.controller.js
import fileUploadFacade from '../services/fileUploadFacade.js';

export const createCourse = async (req, res, next) => {
    try {
        const { title, description, category, createdBy } = req.body;
        
        const course = await Course.create({
            title,
            description,
            category,
            createdBy,
            thumbnail: {
                public_id: 'dummy',
                secure_url: 'dummy'
            },
        });
        
        if (req.file) {
            const uploadResult = await fileUploadFacade.uploadImage(req.file, 'course-thumbnails');
            
            if (uploadResult.success) {
                course.thumbnail.public_id = uploadResult.public_id;
                course.thumbnail.secure_url = uploadResult.secure_url;
                await course.save();
            }
        }
        
        res.status(200).json({
            success: true,
            message: "Course created successfully",
            data: course
        });
    } catch (error) {
        return next(new AppError(error.message || "Unable to create your course", 400));
    }
};
```

---

## Behavioral Patterns

### 1. Middleware Pattern Implementation

#### Authentication Middleware Chain
```javascript
// middlewares/auth.middleware.js
import AppError from "../utils/appError.js";
import jwt from 'jsonwebtoken';
import logger from '../utils/logger.js';

class AuthMiddleware {
    static isLoggedIn(req, res, next) {
        const { token } = req.cookies;
        
        logger.info("Authentication Debug:", {
            route: req.originalUrl,
            method: req.method,
            hasToken: !!token,
            timestamp: new Date().toISOString()
        });
        
        if (!token) {
            logger.warn("Authentication failed: No token found");
            return next(new AppError('Please login to continue. No authentication token found.', 401));
        }
        
        try {
            const decode = jwt.verify(token, process.env.JWT_SECRET);
            req.user = decode;
            logger.info("Authentication successful", {
                userId: decode.id,
                userEmail: decode.email,
                route: req.originalUrl
            });
            next();
        } catch (error) {
            logger.error("Error in isLoggedIn middleware: ", {
                error: error.message,
                route: req.originalUrl
            });
            
            if (error.name === 'TokenExpiredError') {
                return next(new AppError('Token expired, please login again', 401));
            } else if (error.name === 'JsonWebTokenError') {
                return next(new AppError('Invalid token, please login again', 401));
            } else {
                return next(new AppError('Authentication failed', 401));
            }
        }
    }
    
    static authorizedRoles(...roles) {
        return (req, res, next) => {
            const currentRole = req.user.role;
            
            if (!roles.includes(currentRole)) {
                return next(new AppError('You are not authorized to access this route', 403));
            }
            
            next();
        };
    }
    
    static authorizedSubscriber(req, res, next) {
        const subscriptionStatus = req.user.subscription.status;
        const currentRole = req.user.role;
        
        if (currentRole !== 'ADMIN' && subscriptionStatus !== 'Active') {
            return next(new AppError('You are not authorized to access this route', 403));
        }
        
        next();
    }
}

export const { isLoggedIn, authorizedRoles, authorizedSubscriber } = AuthMiddleware;
```

#### Usage Example:
```javascript
// routes/course.routes.js
import { isLoggedIn, authorizedRoles, authorizedSubscriber } from '../middlewares/auth.middleware.js';

router.get('/:courseId', 
    isLoggedIn, 
    authorizedSubscriber, 
    getLecturesByCourseId
);

router.post('/', 
    isLoggedIn, 
    authorizedRoles('ADMIN'), 
    upload.single('thumbnail'), 
    createCourse
);
```

### 2. Observer Pattern Implementation

#### Error Observer System
```javascript
// services/errorObserver.js
class ErrorObserver {
    constructor() {
        this.observers = [];
    }
    
    subscribe(observer) {
        this.observers.push(observer);
    }
    
    unsubscribe(observer) {
        this.observers = this.observers.filter(obs => obs !== observer);
    }
    
    notify(error, context) {
        this.observers.forEach(observer => {
            observer.update(error, context);
        });
    }
}

// Concrete observers
class LoggingObserver {
    update(error, context) {
        console.error('Error logged:', {
            message: error.message,
            stack: error.stack,
            context: context,
            timestamp: new Date().toISOString()
        });
    }
}

class EmailNotificationObserver {
    async update(error, context) {
        if (error.statusCode >= 500) {
            // Send email to admin for critical errors
            await this.sendAdminNotification(error, context);
        }
    }
    
    async sendAdminNotification(error, context) {
        // Implementation for sending admin notification
        console.log('Admin notification sent for critical error');
    }
}

class MetricsObserver {
    update(error, context) {
        // Update error metrics
        console.log('Error metrics updated:', {
            errorType: error.name,
            statusCode: error.statusCode,
            endpoint: context.route
        });
    }
}

// Singleton instance
const errorObserver = new ErrorObserver();
errorObserver.subscribe(new LoggingObserver());
errorObserver.subscribe(new EmailNotificationObserver());
errorObserver.subscribe(new MetricsObserver());

export default errorObserver;
```

#### Usage Example:
```javascript
// middlewares/error.middleware.js
import errorObserver from '../services/errorObserver.js';

const errorMiddleware = (err, req, res, next) => {
    req.statusCode = req.statusCode || 500;
    req.statusMessage = req.statusMessage || 'Internal Server Error caught by errorMiddleware';
    
    // Notify all observers
    errorObserver.notify(err, {
        route: req.originalUrl,
        method: req.method,
        timestamp: new Date().toISOString()
    });
    
    return res.status(req.statusCode).json({
        success: false,
        message: req.statusMessage,
        name: err.name,
        code: err.code,
        stack: err.stack
    });
};

export default errorMiddleware;
```

### 3. Strategy Pattern Implementation

#### Authentication Strategy
```javascript
// strategies/authStrategy.js
class AuthStrategy {
    authenticate(credentials) {
        throw new Error('Authentication method must be implemented');
    }
}

class JWTAuthStrategy extends AuthStrategy {
    async authenticate(credentials) {
        const { token } = credentials;
        
        try {
            const decode = jwt.verify(token, process.env.JWT_SECRET);
            return {
                success: true,
                user: decode
            };
        } catch (error) {
            return {
                success: false,
                error: error.message
            };
        }
    }
}

class PasswordAuthStrategy extends AuthStrategy {
    async authenticate(credentials) {
        const { email, password } = credentials;
        
        try {
            const user = await User.findOne({ email }).select('+password');
            
            if (!user || !(await user.comparePassword(password))) {
                return {
                    success: false,
                    error: 'Invalid credentials'
                };
            }
            
            return {
                success: true,
                user: user
            };
        } catch (error) {
            return {
                success: false,
                error: error.message
            };
        }
    }
}

class AuthContext {
    constructor(strategy) {
        this.strategy = strategy;
    }
    
    setStrategy(strategy) {
        this.strategy = strategy;
    }
    
    async authenticate(credentials) {
        return await this.strategy.authenticate(credentials);
    }
}

export { AuthContext, JWTAuthStrategy, PasswordAuthStrategy };
```

#### Usage Example:
```javascript
// controllers/user.controller.js
import { AuthContext, JWTAuthStrategy, PasswordAuthStrategy } from '../strategies/authStrategy.js';

export const login = async (req, res, next) => {
    try {
        const { email, password } = req.body;
        
        const authContext = new AuthContext(new PasswordAuthStrategy());
        const result = await authContext.authenticate({ email, password });
        
        if (result.success) {
            const token = await result.user.generateJWTToken();
            res.cookie('token', token, cookieOptions);
            
            res.status(201).json({
                success: true,
                message: 'User logged in successfully',
                data: result.user
            });
        } else {
            return next(new AppError(result.error, 401));
        }
    } catch (error) {
        return next(new AppError(error.message, 500));
    }
};
```

---

## Architectural Patterns

### 1. Repository Pattern Implementation

#### Base Repository
```javascript
// repositories/baseRepository.js
class BaseRepository {
    constructor(model) {
        this.model = model;
    }
    
    async findById(id) {
        return await this.model.findById(id);
    }
    
    async findOne(conditions) {
        return await this.model.findOne(conditions);
    }
    
    async find(conditions = {}) {
        return await this.model.find(conditions);
    }
    
    async create(data) {
        return await this.model.create(data);
    }
    
    async update(id, data) {
        return await this.model.findByIdAndUpdate(id, data, { new: true });
    }
    
    async delete(id) {
        return await this.model.findByIdAndDelete(id);
    }
    
    async count(conditions = {}) {
        return await this.model.countDocuments(conditions);
    }
    
    async paginate(conditions = {}, options = {}) {
        const page = options.page || 1;
        const limit = options.limit || 10;
        const skip = (page - 1) * limit;
        
        const [data, total] = await Promise.all([
            this.model.find(conditions).skip(skip).limit(limit),
            this.model.countDocuments(conditions)
        ]);
        
        return {
            data,
            total,
            page,
            limit,
            pages: Math.ceil(total / limit)
        };
    }
}

export default BaseRepository;
```

#### User Repository
```javascript
// repositories/userRepository.js
import BaseRepository from './baseRepository.js';
import User from '../models/user.model.js';

class UserRepository extends BaseRepository {
    constructor() {
        super(User);
    }
    
    async findByEmail(email) {
        return await this.model.findOne({ email: email.toLowerCase() });
    }
    
    async findByEmailWithPassword(email) {
        return await this.model.findOne({ email: email.toLowerCase() }).select('+password');
    }
    
    async updateSubscription(userId, subscriptionData) {
        return await this.model.findByIdAndUpdate(
            userId,
            { subscription: subscriptionData },
            { new: true }
        );
    }
    
    async updateRole(userId, role) {
        return await this.model.findByIdAndUpdate(
            userId,
            { role },
            { new: true }
        );
    }
    
    async getSubscribedUsers() {
        return await this.model.find({
            'subscription.status': 'Active'
        });
    }
    
    async getUsersByRole(role) {
        return await this.model.find({ role });
    }
}

export default new UserRepository();
```

#### Usage Example:
```javascript
// controllers/user.controller.js
import userRepository from '../repositories/userRepository.js';

export const register = async (req, res, next) => {
    try {
        const { fullName, email, password } = req.body;
        
        // Check if user already exists
        const existingUser = await userRepository.findByEmail(email);
        if (existingUser) {
            return next(new AppError('User already exists', 400));
        }
        
        // Create new user
        const user = await userRepository.create({
            fullName,
            email,
            password,
            avatar: {
                public_id: email,
                secure_url: 'https://res.cloudinary.com/du9jzqlpt/image/upload/v1674647316/avatar_drzgxv.jpg'
            }
        });
        
        // Handle avatar upload if provided
        if (req.file) {
            // Upload logic
        }
        
        const token = await user.generateJWTToken();
        res.cookie('token', token, cookieOptions);
        
        res.status(200).json({
            success: true,
            message: 'User created successfully',
            data: user
        });
    } catch (error) {
        return next(new AppError(error.message, 500));
    }
};
```

---

## Express.js Patterns

### 1. Middleware Pipeline Implementation

#### Custom Middleware Pipeline
```javascript
// middlewares/pipeline.js
class MiddlewarePipeline {
    constructor() {
        this.middlewares = [];
    }
    
    use(middleware) {
        this.middlewares.push(middleware);
        return this;
    }
    
    async execute(req, res, next) {
        let index = 0;
        
        const runNext = async () => {
            if (index < this.middlewares.length) {
                const middleware = this.middlewares[index++];
                await middleware(req, res, runNext);
            } else {
                next();
            }
        };
        
        try {
            await runNext();
        } catch (error) {
            next(error);
        }
    }
}

// Usage
const pipeline = new MiddlewarePipeline()
    .use(loggingMiddleware)
    .use(authMiddleware)
    .use(validationMiddleware);

export default pipeline;
```

### 2. Route Handler Pattern Implementation

#### Route Handler Factory
```javascript
// handlers/routeHandlerFactory.js
class RouteHandlerFactory {
    static createHandler(controller, method, middlewares = []) {
        return async (req, res, next) => {
            try {
                // Apply middlewares
                for (const middleware of middlewares) {
                    await middleware(req, res, next);
                }
                
                // Execute controller method
                await controller[method](req, res, next);
            } catch (error) {
                next(error);
            }
        };
    }
    
    static createCRUDHandlers(controller) {
        return {
            create: this.createHandler(controller, 'create', [isLoggedIn, authorizedRoles('ADMIN')]),
            read: this.createHandler(controller, 'read'),
            update: this.createHandler(controller, 'update', [isLoggedIn, authorizedRoles('ADMIN')]),
            delete: this.createHandler(controller, 'delete', [isLoggedIn, authorizedRoles('ADMIN')])
        };
    }
}

export default RouteHandlerFactory;
```

---

## Pattern Testing Examples

### 1. Testing Singleton Pattern
```javascript
// tests/logger.test.js
import logger from '../utils/logger.js';

describe('Logger Singleton', () => {
    test('should return the same instance', () => {
        const logger1 = require('../utils/logger.js').default;
        const logger2 = require('../utils/logger.js').default;
        
        expect(logger1).toBe(logger2);
    });
    
    test('should log messages correctly', () => {
        const consoleSpy = jest.spyOn(console, 'log').mockImplementation();
        
        logger.info('Test message');
        
        expect(consoleSpy).toHaveBeenCalled();
        consoleSpy.mockRestore();
    });
});
```

### 2. Testing Factory Pattern
```javascript
// tests/fileUploadFactory.test.js
import FileUploadFactory from '../middlewares/multer.middleware.js';

describe('File Upload Factory', () => {
    test('should create image uploader', () => {
        const imageUploader = FileUploadFactory.createImageUploader();
        expect(imageUploader).toBeDefined();
        expect(typeof imageUploader.single).toBe('function');
    });
    
    test('should create video uploader', () => {
        const videoUploader = FileUploadFactory.createVideoUploader();
        expect(videoUploader).toBeDefined();
        expect(typeof videoUploader.single).toBe('function');
    });
});
```

### 3. Testing Strategy Pattern
```javascript
// tests/authStrategy.test.js
import { AuthContext, JWTAuthStrategy, PasswordAuthStrategy } from '../strategies/authStrategy.js';

describe('Authentication Strategy', () => {
    test('should authenticate with JWT strategy', async () => {
        const authContext = new AuthContext(new JWTAuthStrategy());
        const result = await authContext.authenticate({ token: 'valid-jwt-token' });
        
        expect(result.success).toBe(true);
    });
    
    test('should authenticate with password strategy', async () => {
        const authContext = new AuthContext(new PasswordAuthStrategy());
        const result = await authContext.authenticate({ 
            email: 'test@example.com', 
            password: 'password123' 
        });
        
        expect(result.success).toBe(true);
    });
});
```

---

## Conclusion

These implementation examples demonstrate how design patterns are applied in the Learning Management System to create maintainable, scalable, and testable code. Each pattern serves a specific purpose and contributes to the overall architecture quality of the application.

The examples show:
- **Real-world implementation** of each pattern
- **Practical usage scenarios** in the context of the LMS
- **Testing strategies** for pattern validation
- **Code organization** and maintainability benefits

This comprehensive approach to design pattern implementation ensures the codebase follows industry best practices and provides a solid foundation for future development and maintenance.
