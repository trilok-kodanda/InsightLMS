# UML Use Case Diagrams for Learning Management System

This directory contains individual PlantUML files for each use case diagram, optimized for easy viewing and importing into draw.io.

## Files Overview

### 1. `01_User_Management_UseCases.puml`
**Focus**: User authentication, profile management, and admin request system
- **Actors**: User, Admin, System
- **Key Use Cases**: Registration, Login/Logout, Profile Management, Password Reset, Admin Requests
- **Total Use Cases**: 14

### 2. `02_Course_Management_UseCases.puml`
**Focus**: Course browsing, content access, and course administration
- **Actors**: Student, Subscriber, Admin
- **Key Use Cases**: Course Browsing, Lecture Access, Course CRUD, Media Upload
- **Total Use Cases**: 11

### 3. `03_Payment_Subscription_UseCases.puml`
**Focus**: Subscription management and payment processing
- **Actors**: User, Admin, Razorpay
- **Key Use Cases**: Subscription Purchase, Payment Verification, Refund Processing, Analytics
- **Total Use Cases**: 10

### 4. `04_System_Administration_UseCases.puml`
**Focus**: System administration, monitoring, and support
- **Actors**: Admin, User, System
- **Key Use Cases**: Contact Management, User Analytics, System Monitoring
- **Total Use Cases**: 7

### 5. `05_Overall_System_Architecture.puml`
**Focus**: High-level system overview with all actors and major use cases
- **Actors**: Guest, Student, Subscriber, Admin, PaymentGateway, EmailService
- **Key Use Cases**: All major system functionalities grouped by modules
- **Total Use Cases**: 15

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

## Diagram Features

Each diagram includes:
- ✅ **Clear actor definitions** with role-based access
- ✅ **Use case relationships** (includes, extends, associations)
- ✅ **Package organization** by functional modules
- ✅ **Detailed notes** explaining key functionality
- ✅ **Consistent styling** with `!theme plain` for clean appearance

## System Overview

**Total Actors**: 6 (Guest, Student, Subscriber, Admin, PaymentGateway, EmailService)
**Total Use Cases**: 57 across all diagrams
**Core Modules**: 5 (User Management, Course Management, Payment System, Administration, Public Access)

## Quick Reference

| Diagram | Primary Focus | Main Actors | Use Cases |
|---------|---------------|-------------|-----------|
| 01 | User Management | User, Admin, System | 14 |
| 02 | Course Management | Student, Subscriber, Admin | 11 |
| 03 | Payment System | User, Admin, Razorpay | 10 |
| 04 | System Admin | Admin, User, System | 7 |
| 05 | Overall Architecture | All Actors | 15 |

## Tips for Draw.io Import

1. **Copy the entire content** of each `.puml` file (including `@startuml` and `@enduml`)
2. **Use the PlantUML import** feature in draw.io for best results
3. **Adjust layout** if needed after import
4. **Export as PNG/SVG** for presentations or documentation

These diagrams are based on the actual source code analysis of your Learning Management System and represent the complete functionality implemented in your application.

