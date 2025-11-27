# Project Modules Documentation
## Learning Management System

## Overview

This document provides a comprehensive list of all modules involved in the Learning Management System project, categorized by type and functionality.

---

## 1. Node.js Dependencies (NPM Packages)

### Core Framework Modules
| Module | Version | Purpose |
|--------|---------|---------|
| **express** | ^4.18.2 | Web application framework for Node.js |
| **mongoose** | ^8.0.0 | MongoDB object modeling (ODM) |
| **dotenv** | ^16.3.1 | Environment variable management |

### Authentication & Security Modules
| Module | Version | Purpose |
|--------|---------|---------|
| **jsonwebtoken** | ^9.0.2 | JWT token generation and verification |
| **bcryptjs** | ^2.4.3 | Password hashing and comparison |
| **cookie-parser** | ^1.4.6 | Cookie parsing middleware |
| **validator** | ^13.11.0 | Input validation and sanitization |

### File Upload & Media Modules
| Module | Version | Purpose |
|--------|---------|---------|
| **multer** | ^1.4.5-lts.1 | File upload handling middleware |
| **cloudinary** | ^1.41.0 | Cloud-based image and video storage |
| **path** | ^0.12.7 | File path utilities |

### Payment Processing Modules
| Module | Version | Purpose |
|--------|---------|---------|
| **razorpay** | ^2.9.2 | Payment gateway integration |

### Communication Modules
| Module | Version | Purpose |
|--------|---------|---------|
| **nodemailer** | ^6.9.7 | Email sending functionality |

### Utility & Logging Modules
| Module | Version | Purpose |
|--------|---------|---------|
| **winston** | ^3.11.0 | Logging library |
| **morgan** | ^1.10.0 | HTTP request logger middleware |
| **cors** | ^2.8.5 | Cross-Origin Resource Sharing middleware |

### Development Modules
| Module | Version | Purpose |
|--------|---------|---------|
| **nodemon** | ^3.0.1 | Development server with auto-restart |

---

## 2. Application Modules (Internal Structure)

### 2.1 Core Application Modules

#### Entry Point Module
- **server.js** - Application entry point, server initialization
- **app.js** - Express application configuration and middleware setup

#### Configuration Modules
- **config/dbConnection.js** - MongoDB database connection configuration

### 2.2 Controller Modules (Business Logic)

| Module | File | Purpose |
|--------|------|---------|
| **User Controller** | `controllers/user.controller.js` | User authentication, profile management, admin requests |
| **Course Controller** | `controllers/course.controller.js` | Course CRUD operations, lecture management |
| **Payment Controller** | `controllers/payment.controller.js` | Subscription management, payment processing |
| **Miscellaneous Controller** | `controllers/miscellaneous.controller.js` | Contact form, user statistics |

### 2.3 Model Modules (Data Layer)

| Module | File | Purpose |
|--------|------|---------|
| **User Model** | `models/user.model.js` | User schema, authentication methods |
| **Course Model** | `models/courser.model.js` | Course schema with embedded lectures |
| **Payment Model** | `models/payment.model.js` | Payment transaction schema |
| **AdminRequest Model** | `models/adminRequest.model.js` | Admin access request workflow schema |

### 2.4 Route Modules (API Endpoints)

| Module | File | Purpose |
|--------|------|---------|
| **User Routes** | `routes/user.routes.js` | User-related API endpoints |
| **Course Routes** | `routes/course.routes.js` | Course-related API endpoints |
| **Payment Routes** | `routes/payment.routes.js` | Payment-related API endpoints |
| **Miscellaneous Routes** | `routes/miscellaneous.routes.js` | Contact and statistics endpoints |

### 2.5 Middleware Modules (Cross-cutting Concerns)

| Module | File | Purpose |
|--------|------|---------|
| **Authentication Middleware** | `middlewares/auth.middleware.js` | JWT authentication, role-based authorization |
| **Error Middleware** | `middlewares/error.middleware.js` | Centralized error handling |
| **Async Handler** | `middlewares/asyncHandler.middleware.js` | Async function error wrapper |
| **Multer Middleware** | `middlewares/multer.middleware.js` | File upload configuration |

### 2.6 Utility Modules

| Module | File | Purpose |
|--------|------|---------|
| **AppError** | `utils/appError.js` | Custom error class |
| **Logger** | `utils/logger.js` | Winston-based logging utility |
| **SendEmail** | `utils/sendEmail.js` | Email sending utility |

---

## 3. Functional Modules (By Feature)

### 3.1 User Management Module
**Description:**
Handles all user-related operations including authentication, profile management, and administrative access requests. This module provides secure user registration, login/logout functionality, password reset mechanisms, and manages the admin request workflow. It integrates JWT tokens for session management and bcrypt for secure password storage.

**Components:**
- User Controller
- User Model
- User Routes
- Authentication Middleware
- Password hashing (bcryptjs)
- JWT token management (jsonwebtoken)

**Features:**
- User registration
- User login/logout
- Profile management
- Password reset
- Admin request system

### 3.2 Course Management Module
**Description:**
Manages all course-related operations including creation, updates, deletion, and lecture management. This module handles course content organization, thumbnail uploads for course previews, and video uploads for lecture materials. It implements subscription-based access control to ensure only authorized users can access course content.

**Components:**
- Course Controller
- Course Model
- Course Routes
- File upload (multer)
- Cloudinary integration

**Features:**
- Course CRUD operations
- Lecture management
- Thumbnail upload
- Video upload
- Content access control

### 3.3 Payment & Subscription Module
**Description:**
Manages all payment and subscription operations through Razorpay integration. This module handles subscription purchases, payment verification with signature validation, subscription cancellation, and refund processing within the 14-day refund policy. It also provides payment analytics and monthly sales tracking for administrative purposes.

**Components:**
- Payment Controller
- Payment Model
- Payment Routes
- Razorpay integration

**Features:**
- Subscription purchase
- Payment verification
- Subscription cancellation
- Refund processing
- Payment analytics

### 3.4 File Management Module
**Description:**
Handles all file upload and storage operations using Multer for local file processing and Cloudinary for cloud-based storage. This module manages image uploads for user avatars and course thumbnails, video uploads for lecture content, and implements file validation, size limits, and automatic cleanup of temporary files. All files are stored in Cloudinary's CDN for fast global delivery.

**Components:**
- Multer middleware
- Cloudinary SDK
- File system utilities

**Features:**
- Image upload (avatars, thumbnails)
- Video upload (lectures)
- File validation
- Cloud storage
- File deletion

### 3.5 Communication Module
**Description:**
Manages all email communication using Nodemailer for SMTP email delivery. This module sends password reset emails with secure tokens, processes contact form submissions, and handles various email notifications. It supports HTML email templates and provides reliable email delivery through configured SMTP servers.

**Components:**
- SendEmail utility
- Nodemailer integration

**Features:**
- Password reset emails
- Contact form emails
- Email notifications

### 3.6 Security Module
**Description:**
Provides comprehensive security features including JWT-based authentication, role-based authorization (USER/ADMIN), and subscription-based access control. This module implements secure password hashing with bcrypt, manages authentication tokens, and configures CORS policies for secure cross-origin requests. It ensures that only authenticated and authorized users can access protected resources.

**Components:**
- Authentication middleware
- Authorization middleware
- JWT tokens
- Password hashing
- CORS configuration

**Features:**
- User authentication
- Role-based access control
- Subscription-based access
- Secure password storage

### 3.7 Logging & Monitoring Module
**Description:**
Provides comprehensive logging and monitoring capabilities using Winston for application logging and Morgan for HTTP request logging. This module logs all application events, errors with stack traces, authentication attempts, and system performance metrics. It helps in debugging, monitoring system health, and tracking user activities for security and analytics purposes.

**Components:**
- Logger utility (Winston)
- Morgan middleware
- Error middleware

**Features:**
- Application logging
- Error logging
- Request logging
- Performance monitoring

---

## 4. External Service Modules

### 4.1 Database Service
- **MongoDB** - Primary database for storing all application data including users, courses, payments, and admin requests. It provides document-based storage with flexible schema design and supports complex queries and aggregations. The database handles data persistence, relationships between collections, and ensures data integrity through validation and indexing.

- **Mongoose ODM** - Database abstraction layer that provides schema modeling, validation, and query building for MongoDB. It simplifies database operations, manages connections, and provides middleware hooks for data transformation and business logic execution.

### 4.2 File Storage Service
- **Cloudinary** - Cloud-based media storage and CDN service that handles all image and video uploads for the application. It provides automatic image optimization, resizing, and format conversion, as well as video processing and streaming capabilities. The service delivers content through a global CDN for fast access worldwide, reducing server load and improving user experience.

### 4.3 Payment Service
- **Razorpay** - Payment gateway that processes all subscription purchases, one-time payments, and refund transactions. It manages recurring subscription billing, payment verification through signature validation, and handles refund processing within the defined policy period. The service provides secure payment processing with PCI compliance and supports multiple payment methods.

### 4.4 Email Service
- **Nodemailer** - Email delivery service that sends transactional emails including password reset links, contact form notifications, and system alerts. It configures SMTP connections, supports HTML email templates, and provides email delivery tracking. The service ensures reliable email delivery through various email providers and SMTP servers.

---

## 5. Configuration Modules

### 5.1 Environment Configuration
- **.env file** - Environment variables
  - `MONGO_URI` - Database connection
  - `JWT_SECRET` - JWT token secret
  - `CLOUDINARY_CLOUD_NAME` - Cloudinary account
  - `CLOUDINARY_API_KEY` - Cloudinary API key
  - `CLOUDINARY_API_SECRET` - Cloudinary API secret
  - `RAZORPAY_KEY_ID` - Razorpay key
  - `RAZORPAY_SECRET` - Razorpay secret
  - `SMTP_HOST` - Email server
  - `SMTP_PORT` - Email port
  - `SMTP_USERNAME` - Email username
  - `SMTP_PASSWORD` - Email password

### 5.2 Application Configuration
- **app.js** - Express app configuration
  - CORS settings
  - Middleware stack
  - Route mounting
  - Error handling

---

## 6. Module Dependencies Diagram

```
┌─────────────────────────────────────────────────┐
│  Entry Point                                    │
│  server.js → app.js                             │
└──────────────────┬──────────────────────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
        ▼                     ▼
┌──────────────┐    ┌─────────────────┐
│  Database     │    │  External       │
│  Connection   │    │  Services       │
│  (Mongoose)   │    │  (Cloudinary,   │
└──────────────┘    │   Razorpay)     │
        │           └─────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────┐
│  Application Layers                             │
│  ┌─────────────┐  ┌─────────────┐              │
│  │  Routes     │→ │ Controllers │              │
│  └─────────────┘  └──────┬──────┘              │
│                          │                      │
│  ┌─────────────┐  ┌──────▼──────┐              │
│  │ Middleware  │→ │   Models     │              │
│  └─────────────┘  └──────┬──────┘              │
│                          │                      │
│  ┌─────────────┐  ┌──────▼──────┐              │
│  │   Utils     │→ │  Database    │              │
│  └─────────────┘  └─────────────┘              │
└─────────────────────────────────────────────────┘
```

---

## 7. Module Summary Statistics

### Total Modules Count

| Category | Count | Details |
|----------|-------|---------|
| **NPM Dependencies** | 15 | External packages |
| **Controllers** | 4 | Business logic handlers |
| **Models** | 4 | Database schemas |
| **Routes** | 4 | API endpoint definitions |
| **Middlewares** | 4 | Request processing |
| **Utils** | 3 | Utility functions |
| **Config** | 1 | Configuration files |
| **Core Files** | 2 | Entry points |
| **Total Application Modules** | 22 | Internal modules |
| **Total External Modules** | 15 | NPM packages |
| **Grand Total** | 37 | All modules |

---

## 8. Module Responsibilities

### Core Modules
- **server.js**: Application startup, external service initialization
- **app.js**: Express configuration, middleware setup, route mounting

### Data Layer Modules
- **Models**: Data structure definition, validation, business logic methods
- **Database Connection**: MongoDB connection management

### Business Logic Modules
- **Controllers**: Request handling, business logic, response formatting

### API Layer Modules
- **Routes**: URL mapping, HTTP method handling, middleware application

### Security Modules
- **Auth Middleware**: Authentication and authorization
- **Error Middleware**: Error handling and logging

### Utility Modules
- **Logger**: Application logging
- **AppError**: Custom error handling
- **SendEmail**: Email communication

---

## 9. Module Integration Points

### Authentication Flow
```
User Request → Routes → Auth Middleware → Controller → Model → Database
```

### File Upload Flow
```
User Upload → Routes → Multer Middleware → Controller → Cloudinary → Database
```

### Payment Flow
```
User Payment → Routes → Auth Middleware → Controller → Razorpay → Payment Model → Database
```

### Error Handling Flow
```
Any Error → Error Middleware → Logger → Response
```

---

## 10. Module Dependencies

### Critical Dependencies
- **express** → All route and controller modules
- **mongoose** → All model modules
- **jsonwebtoken** → Authentication middleware
- **bcryptjs** → User model (password hashing)
- **cloudinary** → Course and User controllers (file uploads)
- **razorpay** → Payment controller
- **nodemailer** → SendEmail utility

### Optional Dependencies
- **winston** → Logger utility
- **morgan** → Request logging
- **validator** → Input validation
- **cors** → Cross-origin requests

---

## Conclusion

The Learning Management System consists of **37 total modules**:
- **15 NPM packages** (external dependencies)
- **22 application modules** (internal code)

These modules work together to provide:
- User management and authentication
- Course and content management
- Payment and subscription processing
- File upload and storage
- Email communication
- Security and error handling
- Logging and monitoring

All modules follow a modular architecture pattern, ensuring maintainability, scalability, and testability of the application.
