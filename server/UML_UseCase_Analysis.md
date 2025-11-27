# Learning Management System - UML Use Case Analysis

## Overview
This document provides a comprehensive UML use case analysis of the Learning Management System (LMS) based on the source code analysis. The system supports multiple user roles with different access levels and functionality.

## System Architecture Summary

### User Roles
1. **Guest** - Unauthenticated users who can browse courses and contact support
2. **Student** - Registered users who can purchase subscriptions and access limited content
3. **Subscriber** - Users with active subscriptions who can access all course content
4. **Admin** - Users with administrative privileges who can manage courses, users, and system

### Core Modules
1. **User Management** - Authentication, profile management, admin request system
2. **Course Management** - CRUD operations for courses and lectures
3. **Payment System** - Subscription management via Razorpay integration
4. **Administration** - User management, analytics, and system monitoring

## Use Case Diagrams Analysis

### 1. User Management Use Cases

**Primary Actors:** User, Admin, System

**Key Use Cases:**
- **Register Account**: Creates new user with email validation and default avatar
- **Authentication Flow**: Login/logout with JWT token management
- **Profile Management**: Update personal information and avatar upload
- **Password Management**: Change password and forgot/reset password flow
- **Admin Request System**: Users can request admin access with reason and experience

**Notable Features:**
- JWT-based authentication with httpOnly cookies
- Password hashing with bcrypt
- Email-based password reset with token expiration
- Admin promotion system with approval workflow
- Avatar upload with Cloudinary integration

### 2. Course Management Use Cases

**Primary Actors:** Student, Subscriber, Admin

**Key Use Cases:**
- **Course Browsing**: View course catalog (public access)
- **Content Access**: Access course lectures (subscription required)
- **Course Administration**: Full CRUD operations for courses and lectures
- **Media Management**: Upload thumbnails and lecture videos

**Access Control:**
- Public access to course listings
- Subscription required for lecture access
- Admin bypass for all content
- File upload handling with Cloudinary

### 3. Payment & Subscription Use Cases

**Primary Actors:** User, Admin, Razorpay

**Key Use Cases:**
- **Subscription Management**: Buy, verify, and cancel subscriptions
- **Payment Processing**: Razorpay integration with signature verification
- **Refund Management**: 14-day refund policy with automatic processing
- **Analytics**: Payment statistics and monthly sales tracking

**Payment Features:**
- 12-month recurring subscription model
- Admin cannot purchase subscriptions
- Signature verification for security
- Automatic refund processing within policy

### 4. System Administration Use Cases

**Primary Actors:** Admin, User, System

**Key Use Cases:**
- **Contact Management**: Contact form with email notifications
- **User Analytics**: Registration and subscription statistics
- **System Monitoring**: Debug endpoints for authentication and payment issues

## Technical Implementation Details

### Authentication & Authorization
```javascript
// JWT Token Structure
{
  id: user.id,
  role: user.role,
  subscription: user.subscription
}
```

### Database Models
1. **User Model**: Profile, authentication, subscription status
2. **Course Model**: Course information, lectures, thumbnails
3. **Payment Model**: Razorpay payment details
4. **AdminRequest Model**: Admin access request workflow

### API Endpoints Structure
```
/api/v1/user/*     - User management
/api/v1/course/*   - Course operations
/api/v1/payment/*  - Payment processing
/api/v1/*          - Miscellaneous (contact, stats)
```

### Security Features
- Password hashing with bcrypt
- JWT token authentication
- CORS configuration for frontend domains
- File upload validation
- Payment signature verification
- Role-based access control

### File Management
- Cloudinary integration for image/video storage
- Local file cleanup after upload
- Thumbnail generation for courses
- Video upload for lectures

## Business Logic Flow

### User Registration Flow
1. User provides email, password, fullName
2. System validates email uniqueness
3. Password is hashed and stored
4. Default avatar is assigned
5. JWT token is generated and set as cookie
6. User is redirected to dashboard

### Course Access Flow
1. Guest/User browses course catalog (no auth required)
2. User clicks on course for details
3. If accessing lectures:
   - Check user authentication
   - Verify subscription status
   - Allow access if subscribed or admin
   - Deny access with appropriate message

### Payment Flow
1. User requests to buy subscription
2. System validates user eligibility
3. Razorpay subscription is created
4. User completes payment
5. System verifies payment signature
6. User subscription status is updated
7. Access to premium content is granted

### Admin Request Flow
1. User submits admin request with reason/experience
2. System validates request uniqueness
3. Admin reviews request
4. Admin approves/rejects with notes
5. If approved, user role is updated to ADMIN
6. User receives notification of status change

## Error Handling & Validation

### Input Validation
- Email format validation
- Password strength requirements
- File type and size validation
- Required field validation

### Error Responses
- Structured error responses with AppError class
- HTTP status codes for different error types
- Detailed error messages for debugging
- Graceful handling of external service failures

### Logging
- Authentication attempts logging
- Payment processing logs
- File upload logs
- System error logging

## Integration Points

### External Services
1. **Cloudinary**: Image and video storage
2. **Razorpay**: Payment processing
3. **Email Service**: Notifications and password reset
4. **MongoDB**: Primary database

### Frontend Integration
- CORS configuration for multiple domains
- Cookie-based authentication
- File upload support
- Real-time payment status updates

## Scalability Considerations

### Database Design
- Indexed fields for performance
- Referenced relationships for data integrity
- Timestamp tracking for auditing

### File Storage
- Cloud-based storage for scalability
- Automatic cleanup of temporary files
- Optimized image/video processing

### Payment Processing
- Asynchronous payment verification
- Retry mechanisms for failed transactions
- Comprehensive payment logging

## Security Measures

### Authentication Security
- JWT token expiration
- HttpOnly cookies
- Password complexity requirements
- Account lockout mechanisms

### Payment Security
- Signature verification
- Encrypted payment data
- Secure API key management
- PCI compliance considerations

### Data Protection
- Input sanitization
- SQL injection prevention
- XSS protection
- File upload restrictions

This UML use case analysis provides a comprehensive view of the Learning Management System's functionality, user interactions, and technical implementation based on the actual source code.
