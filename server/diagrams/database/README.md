# Database Diagrams for Learning Management System

This directory contains comprehensive database diagrams for the Learning Management System, including Entity Relationship (ER) diagrams, data flow diagrams, and database schema documentation.

## Database Diagrams Overview

### 1. `01_ER_Diagram.puml`
**Focus**: Entity Relationship Diagram showing all entities and their relationships
- **Entities**: User, Course, Lecture, Payment, AdminRequest
- **Relationships**: One-to-many and many-to-one relationships
- **Attributes**: Complete field definitions with constraints
- **Use Case**: Database design documentation and relationship understanding

### 2. `02_Data_Flow_Diagram.puml`
**Focus**: Data flow through the system for all major operations
- **Flows**: User registration, login, course management, payment processing, admin requests
- **Processes**: Complete step-by-step data flow for each operation
- **External Systems**: Integration with Cloudinary, Razorpay, Email Service
- **Use Case**: Understanding data movement and system interactions

### 3. `03_Database_Schema.puml`
**Focus**: Detailed database schema with field types, constraints, and indexes
- **Collections**: Complete MongoDB collection schemas
- **Field Definitions**: Data types, validation rules, required fields
- **Indexes**: Database indexes for performance optimization
- **Methods**: Schema methods for business logic
- **Use Case**: Database implementation and optimization

### 4. `04_Data_Relationships.puml`
**Focus**: Detailed relationship mapping between entities
- **Cardinality**: One-to-many, many-to-one relationships
- **Relationship Types**: Ownership, reference, and dependency relationships
- **Constraints**: Unique constraints and referential integrity
- **Use Case**: Database design validation and relationship understanding

### 5. `05_Data_Operations_Flow.puml`
**Focus**: CRUD operations flow for all database entities
- **Operations**: Create, Read, Update, Delete operations
- **External Integrations**: File uploads, payment processing, email services
- **Process Flows**: Complete operation workflows
- **Use Case**: Implementation planning and operation understanding

## Database Structure Summary

### Collections Overview

#### User Collection
- **Primary Key**: `_id` (ObjectId)
- **Unique Fields**: `email`
- **Required Fields**: `fullName`, `email`, `password`
- **Embedded Objects**: `avatar`, `subscription`
- **Indexes**: `email` (unique), `createdAt`
- **Methods**: `comparePassword()`, `generateJWTToken()`, `generatePasswordResetToken()`

#### Course Collection
- **Primary Key**: `_id` (ObjectId)
- **Required Fields**: `title`, `description`, `category`, `createdBy`
- **Embedded Arrays**: `lectures[]`
- **Embedded Objects**: `thumbnail`
- **Indexes**: `title` (text), `category`, `createdBy`, `createdAt`
- **Auto-calculated**: `numberOfLectures`

#### Lecture Collection (Embedded)
- **Primary Key**: `_id` (ObjectId, auto-generated)
- **Required Fields**: `lecture.public_id`, `lecture.secure_url`
- **Fields**: `title`, `description`
- **Embedded Objects**: `lecture` (video file info)

#### Payment Collection
- **Primary Key**: `_id` (ObjectId)
- **Required Fields**: `razorpay_payment_id`, `razorpay_subscription_id`, `razorpay_signature`
- **Indexes**: `razorpay_payment_id` (unique), `razorpay_subscription_id`, `createdAt`

#### AdminRequest Collection
- **Primary Key**: `_id` (ObjectId)
- **Required Fields**: `user`, `reason`, `experience`
- **Foreign Keys**: `user` (ref: User), `reviewedBy` (ref: User)
- **Indexes**: `user` (unique, partial: status=pending), `status`, `reviewedBy`, `createdAt`

### Relationship Summary

| Entity | Relationship | Target | Type | Description |
|--------|-------------|--------|------|-------------|
| User | 1:N | Payment | One-to-Many | User makes multiple payments |
| User | 1:N | AdminRequest | One-to-Many | User can make multiple admin requests |
| User | 1:N | Course | One-to-Many | User creates multiple courses |
| Course | 1:N | Lecture | One-to-Many | Course contains multiple lectures |
| AdminRequest | N:1 | User | Many-to-One | Admin reviews admin requests |

### Key Features

#### Data Validation
- **Email Validation**: Unique, lowercase, format validation
- **Password Security**: Minimum 6 characters, bcrypt hashing
- **Field Constraints**: Length limits, required fields, enum values
- **Input Sanitization**: Trim whitespace, lowercase conversion

#### Security Features
- **Password Hashing**: bcrypt with salt rounds
- **JWT Tokens**: Secure authentication tokens
- **Password Reset**: Time-limited reset tokens
- **Role-based Access**: USER and ADMIN roles

#### External Integrations
- **Cloudinary**: File storage for avatars, thumbnails, and videos
- **Razorpay**: Payment processing and subscription management
- **Email Service**: Password reset and notification emails

#### Performance Optimizations
- **Database Indexes**: Optimized queries for common operations
- **Embedded Documents**: Lectures embedded in courses for efficiency
- **Connection Pooling**: MongoDB connection management
- **Query Optimization**: Selective field queries and pagination

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

## Database Design Patterns

### 1. **Document-based Design**
- MongoDB document structure
- Embedded documents for related data
- Flexible schema for evolving requirements

### 2. **Reference vs Embedding**
- **Embedded**: Lectures in courses (one-to-many, small documents)
- **Referenced**: User references in admin requests (many-to-one, large documents)

### 3. **Index Strategy**
- **Unique Indexes**: Email, payment IDs
- **Compound Indexes**: User + status for admin requests
- **Text Indexes**: Course titles for search
- **Partial Indexes**: Pending admin requests only

### 4. **Data Integrity**
- **Required Fields**: Critical data validation
- **Enum Constraints**: Status fields with predefined values
- **Reference Integrity**: ObjectId references between collections
- **Unique Constraints**: Prevent duplicate data

## Quick Reference

| Diagram | Primary Focus | Target Audience | Use Case |
|---------|---------------|-----------------|----------|
| 01 | Entity Relationships | Database Designers, Developers | Database design validation |
| 02 | Data Flow | System Architects, Developers | Understanding data movement |
| 03 | Database Schema | Database Admins, Developers | Implementation reference |
| 04 | Data Relationships | Database Designers, Analysts | Relationship mapping |
| 05 | Operations Flow | Developers, QA Engineers | Implementation planning |

These database diagrams provide a comprehensive view of the Learning Management System's data structure, relationships, and operations, covering all aspects from high-level entity relationships to detailed implementation patterns.

