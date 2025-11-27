# Class Diagrams for Learning Management System

This directory contains comprehensive class diagrams for the Learning Management System, showing the object-oriented design, class relationships, and system architecture.

## Class Diagrams Overview

### 1. `01_Model_Classes.puml`
**Focus**: Data model classes and their relationships
- **Classes**: User, Course, Lecture, Payment, AdminRequest
- **Relationships**: One-to-many, many-to-one relationships
- **Methods**: Model methods for business logic
- **Use Case**: Understanding data models and their behavior

### 2. `02_Controller_Classes.puml`
**Focus**: Controller classes and their methods
- **Classes**: UserController, CourseController, PaymentController, MiscellaneousController
- **Methods**: HTTP request handling methods
- **Dependencies**: External services and models
- **Use Case**: API endpoint implementation understanding

### 3. `03_Middleware_Classes.puml`
**Focus**: Middleware classes and their functionality
- **Classes**: AuthMiddleware, ErrorMiddleware, AsyncHandler, MulterMiddleware, Logger, AppError, SendEmail
- **Functionality**: Authentication, error handling, file uploads, logging
- **Use Case**: Understanding middleware stack and cross-cutting concerns

### 4. `04_Service_Classes.puml`
**Focus**: Service layer classes and business logic
- **Classes**: UserService, CourseService, PaymentService, EmailService, FileService, ValidationService
- **Business Logic**: Core application functionality
- **External Integrations**: Third-party service interactions
- **Use Case**: Business logic implementation and service design

### 5. `05_Route_Classes.puml`
**Focus**: Route configuration and URL mapping
- **Classes**: UserRoutes, CourseRoutes, PaymentRoutes, MiscellaneousRoutes, App, Server, DatabaseConnection
- **Route Configuration**: URL patterns and middleware application
- **Use Case**: API endpoint mapping and route organization

### 6. `06_Overall_Class_Diagram.puml`
**Focus**: Complete system class diagram with all layers
- **Layers**: Presentation, Controller, Service, Data Access, Middleware, Utility, External Services
- **Architecture**: Layered architecture pattern
- **Dependencies**: Inter-layer and inter-class dependencies
- **Use Case**: System architecture overview and design understanding

## Class Structure Summary

### Model Classes (Data Layer)

#### User Class
- **Attributes**: _id, fullName, email, password, role, avatar, subscription, timestamps
- **Methods**: comparePassword(), generateJWTToken(), generatePasswordResetToken()
- **Validation**: Email format, password strength, field constraints
- **Security**: Password hashing, JWT token generation

#### Course Class
- **Attributes**: _id, title, description, category, lectures, thumbnail, numberOfLectures, createdBy
- **Methods**: addLecture(), removeLecture(), updateLecture(), updateNumberOfLectures()
- **Embedded**: Lecture documents for efficient querying
- **Media**: Thumbnail and video file management

#### Lecture Class
- **Attributes**: _id, title, description, lecture (video info), timestamps
- **Methods**: updateTitle(), updateDescription(), updateVideo(), getVideoUrl()
- **Embedded**: Within Course documents
- **Media**: Video file references and metadata

#### Payment Class
- **Attributes**: _id, razorpay_payment_id, razorpay_subscription_id, razorpay_signature
- **Methods**: validateSignature(), getPaymentId(), getSubscriptionId()
- **Integration**: Razorpay payment gateway integration
- **Security**: Signature verification for payment validation

#### AdminRequest Class
- **Attributes**: _id, user, reason, experience, status, reviewedBy, reviewedAt, reviewNotes
- **Methods**: approve(), reject(), isPending(), isApproved(), isRejected()
- **Workflow**: Admin approval process with status tracking
- **Constraints**: Unique pending requests per user

### Controller Classes (Presentation Layer)

#### UserController
- **Methods**: register, login, logout, getProfile, forgetPassword, resetPassword, changePassword, updateUser, makeUserAdmin, requestAdminAccess, getAdminRequests, reviewAdminRequest, getUserAdminRequest
- **Dependencies**: User model, AdminRequest model, AppError, Cloudinary, EmailService
- **Functionality**: User authentication, profile management, admin request workflow

#### CourseController
- **Methods**: getAllCourses, getLecturesByCourseId, createCourse, updateCourse, deleteCourse, addLectureToCourseById, deleteLectureById, updateLecture
- **Dependencies**: Course model, AppError, Cloudinary, FileSystem
- **Functionality**: Course CRUD operations, lecture management, file uploads

#### PaymentController
- **Methods**: getRazorpayApiKey, buySubscription, verifySubscription, cancelSubscription, getAllPayments
- **Dependencies**: User model, Payment model, Razorpay, AppError, Crypto
- **Functionality**: Payment processing, subscription management, refund handling

#### MiscellaneousController
- **Methods**: contactUs, userStats
- **Dependencies**: EmailService, User model, AppError
- **Functionality**: Contact form handling, user statistics

### Service Classes (Business Logic Layer)

#### UserService
- **Methods**: registerUser, authenticateUser, updateUserProfile, changeUserPassword, requestPasswordReset, resetPassword, makeUserAdmin, requestAdminAccess, getAdminRequests, reviewAdminRequest, getUserAdminRequest
- **Dependencies**: User model, AdminRequest model, bcrypt, jwt, crypto, cloudinary, email service
- **Functionality**: User management, authentication, admin workflow

#### CourseService
- **Methods**: getAllCourses, getCourseById, getCourseLectures, createCourse, updateCourse, deleteCourse, addLectureToCourse, updateLecture, deleteLecture
- **Dependencies**: Course model, cloudinary, file system
- **Functionality**: Course management, lecture handling, file operations

#### PaymentService
- **Methods**: getRazorpayApiKey, createSubscription, verifyPayment, cancelSubscription, processRefund, getAllPayments, getPaymentStatistics
- **Dependencies**: User model, Payment model, razorpay, crypto
- **Functionality**: Payment processing, subscription management, analytics

#### EmailService
- **Methods**: sendPasswordResetEmail, sendContactFormEmail, sendAdminNotificationEmail, sendWelcomeEmail
- **Dependencies**: nodemailer
- **Functionality**: Email notifications, templating, SMTP configuration

#### FileService
- **Methods**: uploadImage, uploadVideo, deleteFile, cleanupLocalFile, validateFileType, validateFileSize
- **Dependencies**: cloudinary, file system, path
- **Functionality**: File upload, validation, cleanup operations

#### ValidationService
- **Methods**: validateEmail, validatePassword, validateUserData, validateCourseData, validateLectureData, validatePaymentData, validateAdminRequestData, validateFileUpload
- **Dependencies**: validator
- **Functionality**: Input validation, data sanitization, business rule validation

### Middleware Classes (Cross-cutting Concerns)

#### AuthMiddleware
- **Methods**: isLoggedIn, authorizedRoles, authorizedSubscriber
- **Dependencies**: AppError, jwt, logger, asyncHandler
- **Functionality**: Authentication, authorization, role-based access control

#### ErrorMiddleware
- **Methods**: errorMiddleware
- **Dependencies**: logger
- **Functionality**: Centralized error handling, logging, response formatting

#### AsyncHandler
- **Methods**: asyncHandler
- **Functionality**: Async function wrapper, error catching

#### MulterMiddleware
- **Methods**: upload (multer instance)
- **Dependencies**: multer, path
- **Functionality**: File upload handling, validation, storage configuration

#### Logger
- **Methods**: info, warn, error, debug
- **Dependencies**: winston
- **Functionality**: Application logging, file transport, formatted output

#### AppError
- **Methods**: constructor
- **Inheritance**: Extends Error
- **Functionality**: Custom error class, status code management

#### SendEmail
- **Methods**: sendEmail
- **Dependencies**: nodemailer
- **Functionality**: Email sending, SMTP configuration

### Route Classes (URL Mapping)

#### UserRoutes
- **Routes**: /register, /login, /logout, /profile, /reset, /changePassword, /update, /request-admin, /admin-requests, /my-admin-request
- **Middleware**: Authentication, file upload, role authorization
- **Functionality**: User-related API endpoints

#### CourseRoutes
- **Routes**: / (GET, POST), /:courseId (GET, PUT, DELETE, POST), /:courseId/lectures/:lectureId (DELETE, PUT)
- **Middleware**: Authentication, file upload, subscription authorization
- **Functionality**: Course-related API endpoints

#### PaymentRoutes
- **Routes**: /razorpay-key, /subscribe, /verify, /unsubscribe, / (GET)
- **Middleware**: Authentication, role authorization
- **Functionality**: Payment-related API endpoints

#### MiscellaneousRoutes
- **Routes**: /contact (POST), /admin/stats/users (GET)
- **Middleware**: Authentication, role authorization
- **Functionality**: Miscellaneous API endpoints

## Architecture Patterns

### 1. **Layered Architecture**
- **Presentation Layer**: Controllers and Routes
- **Business Logic Layer**: Services
- **Data Access Layer**: Models
- **Cross-cutting Concerns**: Middleware and Utilities

### 2. **MVC Pattern**
- **Models**: Data models with business logic methods
- **Views**: JSON responses (API-based)
- **Controllers**: HTTP request handling

### 3. **Service Layer Pattern**
- **Business Logic**: Encapsulated in service classes
- **Separation of Concerns**: Controllers handle HTTP, services handle business logic
- **Reusability**: Services can be used by multiple controllers

### 4. **Middleware Pattern**
- **Cross-cutting Concerns**: Authentication, logging, error handling
- **Request Pipeline**: Sequential processing of requests
- **Reusability**: Middleware can be applied to multiple routes

### 5. **Repository Pattern**
- **Data Access**: Models handle database operations
- **Abstraction**: Business logic separated from data access
- **Consistency**: Standardized data access methods

## Class Relationships

### Inheritance
- **AppError** extends **Error**

### Composition
- **Course** contains **Lecture** (embedded documents)
- **User** makes **Payment** (one-to-many)
- **User** creates **Course** (one-to-many)
- **User** requests **AdminRequest** (one-to-many)
- **AdminRequest** reviewed by **User** (many-to-one)

### Dependencies
- **Controllers** depend on **Services**
- **Services** depend on **Models**
- **Controllers** use **Middleware**
- **Services** use **External Services**

## How to Use These Diagrams

### In VS Code with PlantUML Extension:
1. Open any `.puml` file
2. Press `Ctrl+Shift+P` → "PlantUML: Preview Current Diagram"
3. Or right-click → "Preview Current Diagram"

### In draw.io:
1. Go to [app.diagrams.net](https://app.diagrams.net)
2. Create new diagram
3. Go to "Arrange" → "Insert" → "Advanced" → "PlantUML"
4. Copy and paste the content of any `.puml` file
5. Click "Insert"

### Export Options:
- **PNG**: Good for presentations and documents
- **SVG**: Vector format, scalable, good for web
- **PDF**: Good for documentation and printing

## Quick Reference

| Diagram | Primary Focus | Target Audience | Use Case |
|---------|---------------|-----------------|----------|
| 01 | Data Models | Database Designers, Developers | Understanding data structure |
| 02 | Controllers | API Developers, Frontend Developers | API implementation |
| 03 | Middleware | System Architects, Developers | Cross-cutting concerns |
| 04 | Services | Business Analysts, Developers | Business logic understanding |
| 05 | Routes | API Designers, Developers | URL mapping and organization |
| 06 | Overall Architecture | Architects, Stakeholders | System design overview |

These class diagrams provide a comprehensive view of the Learning Management System's object-oriented design, showing all classes, their relationships, methods, and dependencies across the entire system architecture.

