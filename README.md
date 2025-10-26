# Document Management System - Complete Implementation Guide

## Stack: Node.js + Express.js + MongoDB + React.js

---

## 📋 TABLE OF CONTENTS

1. [Project Setup & Architecture](#1-project-setup--architecture)
2. [Database Schema Design](#2-database-schema-design)
3. [Backend API Endpoints](#3-backend-api-endpoints)
4. [Backend Middleware](#4-backend-middleware-implementation)
5. [Frontend Implementation](#5-frontend-implementation)
6. [Key Features Implementation](#6-key-features-implementation)
7. [Non-Functional Requirements](#7-non-functional-requirements)
8. [Testing](#8-testing)
9. [Deployment](#9-deployment)
10. [Seed Data & Initial Setup](#10-seed-data--initial-setup)
11. [Development Best Practices](#11-development-best-practices)

---

## 1. PROJECT SETUP & ARCHITECTURE

### Backend Setup
- [ ] Initialize Node.js project with Express.js
  ```bash
  mkdir dms-backend && cd dms-backend
  npm init -y
  npm install express mongoose dotenv cors bcryptjs jsonwebtoken
  npm install multer multer-s3 @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
  npm install express-validator helmet morgan compression
  npm install express-rate-limit
  npm install --save-dev nodemon jest supertest mongodb-memory-server
  ```
- [ ] Setup MongoDB connection (local or Atlas)
- [ ] Configure environment variables (.env file)
  ```env
  PORT=5000
  MONGODB_URI=mongodb://localhost:27017/dms_nafscob
  JWT_SECRET=your-secret-key-change-in-production
  JWT_EXPIRE=7d
  AWS_ACCESS_KEY_ID=your-aws-key
  AWS_SECRET_ACCESS_KEY=your-aws-secret
  AWS_BUCKET_NAME=nafscob-dms
  AWS_REGION=ap-south-1
  NODE_ENV=development
  ```
- [ ] Setup JWT authentication middleware
- [ ] Configure CORS for frontend communication
- [ ] Setup file upload middleware (Multer + S3)
- [ ] Configure AWS S3 SDK
- [ ] Setup password hashing (bcrypt)
- [ ] Configure express-validator for input validation
- [ ] Setup error handling middleware
- [ ] Setup logging (Morgan)
- [ ] Setup helmet for security headers
- [ ] Configure compression middleware
- [ ] Setup rate limiting

### Frontend Setup
- [ ] Initialize React.js project
  ```bash
  npx create-react-app dms-frontend
  cd dms-frontend
  npm install axios react-router-dom
  npm install @mui/material @emotion/react @emotion/styled
  npm install @mui/icons-material
  npm install react-hook-form yup @hookform/resolvers
  npm install react-toastify
  npm install react-dropzone
  npm install date-fns
  npm install react-pdf
  ```
- [ ] Setup React Router for navigation
- [ ] Configure Axios for API calls (create axios instance with interceptors)
- [ ] Setup state management (Context API or Redux Toolkit)
- [ ] Install UI library (Material-UI)
- [ ] Setup protected route components
- [ ] Configure environment variables (.env)
  ```env
  REACT_APP_API_URL=http://localhost:5000/api
  ```
- [ ] Setup form validation library (React Hook Form + Yup)
- [ ] Configure toast notifications (react-toastify)

### Project Structure

**Backend:**
```
dms-backend/
├── config/
│   ├── db.js              # MongoDB connection
│   └── aws.js             # AWS S3 configuration
├── models/
│   ├── User.js
│   ├── Department.js
│   ├── Folder.js
│   ├── Document.js
│   ├── DocumentVersion.js
│   ├── AuditLog.js
│   └── Notification.js
├── middleware/
│   ├── auth.js            # JWT verification
│   ├── roleCheck.js       # Role-based access
│   ├── permissions.js     # Folder/document permissions
│   ├── upload.js          # File upload handling
│   ├── auditLog.js        # Auto-logging
│   └── errorHandler.js    # Global error handler
├── routes/
│   ├── auth.js
│   ├── users.js
│   ├── departments.js
│   ├── folders.js
│   ├── documents.js
│   ├── search.js
│   ├── auditLogs.js
│   ├── notifications.js
│   └── dashboard.js
├── controllers/
│   ├── authController.js
│   ├── userController.js
│   ├── departmentController.js
│   ├── folderController.js
│   ├── documentController.js
│   ├── searchController.js
│   ├── auditLogController.js
│   ├── notificationController.js
│   └── dashboardController.js
├── utils/
│   ├── s3Upload.js        # S3 upload helper
│   ├── generateToken.js   # JWT helper
│   └── logger.js          # Custom logger
├── seeds/
│   └── seedData.js        # Initial data seeding
├── .env
├── .gitignore
├── server.js              # Entry point
└── package.json
```

**Frontend:**
```
dms-frontend/
├── public/
├── src/
│   ├── api/
│   │   └── axios.js       # Axios instance
│   ├── components/
│   │   ├── common/
│   │   │   ├── Sidebar.js
│   │   │   ├── Navbar.js
│   │   │   ├── Loader.js
│   │   │   └── ConfirmDialog.js
│   │   ├── folders/
│   │   │   ├── FolderTree.js
│   │   │   ├── CreateFolder.js
│   │   │   └── FolderPermissions.js
│   │   ├── documents/
│   │   │   ├── DocumentList.js
│   │   │   ├── DocumentUpload.js
│   │   │   ├── DocumentViewer.js
│   │   │   └── VersionHistory.js
│   │   └── users/
│   │       ├── UserList.js
│   │       └── UserForm.js
│   ├── pages/
│   │   ├── auth/
│   │   │   ├── Login.js
│   │   │   ├── ForgotPassword.js
│   │   │   └── ResetPassword.js
│   │   ├── Dashboard.js
│   │   ├── MyDocuments.js
│   │   ├── FoldersPage.js
│   │   ├── UsersPage.js
│   │   ├── AuditLogsPage.js
│   │   └── SettingsPage.js
│   ├── context/
│   │   ├── AuthContext.js
│   │   └── NotificationContext.js
│   ├── hooks/
│   │   ├── useAuth.js
│   │   └── usePermissions.js
│   ├── utils/
│   │   ├── constants.js
│   │   └── helpers.js
│   ├── App.js
│   ├── index.js
│   └── .env
└── package.json
```

---

## 2. DATABASE SCHEMA DESIGN (MongoDB)

### Understanding the Folder Hierarchy

**IMPORTANT:** There are NO separate Category or Sub-category collections. Everything is organized using the **Folders** collection with parent-child relationships.

```
Department (folder, level 0, parentFolder: null)
  └── Category (folder, level 1, parentFolder: dept_id)
      └── Sub-category (folder, level 2, parentFolder: category_id)
          └── Sub-sub-category (folder, level 3, parentFolder: subcategory_id)
              └── Documents (stored in folders)
```

**Example Structure:**
```
HR Department (Folder)
├── Certificates (Folder = Category)
│   ├── Training Certificates (Folder = Sub-category)
│   │   ├── 2024 (Folder = Sub-sub-category)
│   │   └── 2025 (Folder = Sub-sub-category)
│   └── Audit Certificates (Folder = Sub-category)
├── Circulars (Folder = Category)
│   ├── 2024 (Folder = Sub-category)
│   └── 2025 (Folder = Sub-category)
└── Policies (Folder = Category)
```

### Collections to Create

#### 1. Departments Collection

```javascript
const departmentSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    unique: true,
    trim: true
  },
  code: {
    type: String,
    required: true,
    unique: true,
    uppercase: true,
    trim: true
  },
  description: {
    type: String,
    trim: true
  },
  head: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  },
  isActive: {
    type: Boolean,
    default: true
  },
  createdBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }
}, {
  timestamps: true
});

// Indexes
departmentSchema.index({ code: 1 });
departmentSchema.index({ isActive: 1 });
```

**Required Seed Data:**
```javascript
[
  { name: "Human Resources", code: "HR", description: "HR Department", isActive: true },
  { name: "Finance", code: "FIN", description: "Finance Department", isActive: true },
  { name: "Information Technology", code: "IT", description: "IT Department", isActive: true },
  { name: "Operations", code: "OPS", description: "Operations Department", isActive: true },
  { name: "Legal", code: "LEG", description: "Legal Department", isActive: true },
  { name: "Administration", code: "ADMIN", description: "Administration", isActive: true }
]
```

#### 2. Users Collection

```javascript
const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,
    unique: true,
    trim: true,
    lowercase: true
  },
  email: {
    type: String,
    required: true,
    unique: true,
    trim: true,
    lowercase: true
  },
  password: {
    type: String,
    required: true,
    select: false
  },
  role: {
    type: String,
    enum: ['superadmin', 'admin', 'team_member', 'member_bank'],
    default: 'team_member'
  },
  department: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Department'
  },
  isActive: {
    type: Boolean,
    default: true
  },
  mfaEnabled: {
    type: Boolean,
    default: false
  },
  mfaSecret: {
    type: String,
    select: false
  },
  lastLogin: {
    type: Date
  },
  passwordResetToken: String,
  passwordResetExpires: Date
}, {
  timestamps: true
});

// Indexes
userSchema.index({ email: 1 });
userSchema.index({ username: 1 });
userSchema.index({ department: 1 });
userSchema.index({ role: 1, isActive: 1 });

// Hash password before saving
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  this.password = await bcrypt.hash(this.password, 12);
  next();
});

// Method to compare password
userSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};
```

#### 3. Folders Collection

```javascript
const folderSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true
  },
  parentFolder: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Folder',
    default: null
  },
  department: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Department',
    required: true
  },
  path: {
    type: String,
    required: true
  },
  level: {
    type: Number,
    default: 0
  },
  createdBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  folderAccess: {
    type: String,
    enum: ['private', 'department', 'organization'],
    default: 'private'
  },
  permissions: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'FolderPermission'
  }],
  isActive: {
    type: Boolean,
    default: true
  }
}, {
  timestamps: true
});

// Indexes
folderSchema.index({ department: 1, parentFolder: 1 });
folderSchema.index({ department: 1, path: 1 }, { unique: true });
folderSchema.index({ createdBy: 1 });

```

```javascript

const folderPermissionSchema = new mongoose.Schema({
  folder: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Folder',
    required: true
  },
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  role: {
    type: String,
    enum: ['superadmin', 'admin', 'team_member', 'member_bank'],
    required: true
  },
  access: [{
    type: String,
    enum: ['view', 'upload', 'edit', 'delete', 'download']
  }]
}, { timestamps: true });

// Indexes
folderPermissionSchema.index({ folder: 1, user: 1 }, { unique: true });
folderPermissionSchema.index({ user: 1 });
folderPermissionSchema.index({ role: 1 });


```

#### 4. Documents Collection

```javascript
const documentSchema = new mongoose.Schema({
  title: {
    type: String,
    required: true,
    trim: true
  },
  originalFileName: {
    type: String,
    required: true
  },
  fileUrl: {
    type: String,
    required: true
  },
  fileKey: {
    type: String,
    required: true
  },
  fileType: {
    type: String,
    enum: ['pdf', 'docx', 'xlsx', 'jpg', 'png', 'zip'],
    required: true
  },
  fileSize: {
    type: Number,
    required: true
  },
  folder: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Folder',
    required: true
  },
  department: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Department',
    required: true
  },
  tags: [{
    type: String,
    trim: true,
    lowercase: true
  }],
  uploadedBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  version: {
    type: Number,
    default: 1
  },
  permissions: [{
    user: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User'
    },
    role: {
      type: String,
      enum: ['superadmin', 'admin', 'team_member', 'member_bank']
    },
    access: [{
      type: String,
      enum: ['view', 'edit', 'delete', 'download']
    }]
  }],
  metadata: {
    type: Map,
    of: String
  },
  isActive: {
    type: Boolean,
    default: true
  }
}, {
  timestamps: true
});

// Indexes
documentSchema.index({ folder: 1 });
documentSchema.index({ department: 1 });
documentSchema.index({ uploadedBy: 1 });
documentSchema.index({ tags: 1 });
documentSchema.index({ title: 'text', tags: 'text' });
documentSchema.index({ createdAt: -1 });
documentSchema.index({ 'permissions.user': 1 });
documentSchema.index({ 'permissions.role': 1 });
documentSchema.index({ folder: 1, isActive: 1 });
documentSchema.index({ department: 1, createdAt: -1 });
```

#### 5. DocumentVersions Collection

```javascript
const documentVersionSchema = new mongoose.Schema({
  document: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Document',
    required: true
  },
  version: {
    type: Number,
    required: true
  },
  fileUrl: {
    type: String,
    required: true
  },
  fileKey: {
    type: String,
    required: true
  },
  fileSize: {
    type: Number,
    required: true
  },
  uploadedBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  changes: {
    type: String,
    trim: true
  }
}, {
  timestamps: true
});

// Indexes
documentVersionSchema.index({ document: 1, version: -1 });
documentVersionSchema.index({ uploadedBy: 1 });
```

#### 6. AuditLogs Collection

```javascript
const auditLogSchema = new mongoose.Schema({
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  action: {
    type: String,
    enum: ['upload', 'download', 'view', 'edit', 'delete', 'rename', 'move', 'login', 'logout', 'create_folder', 'delete_folder', 'update_permissions'],
    required: true
  },
  resourceType: {
    type: String,
    enum: ['document', 'folder', 'user', 'department', 'system'],
    required: true
  },
  resourceId: {
    type: mongoose.Schema.Types.ObjectId
  },
  resourceName: {
    type: String
  },
  department: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Department'
  },
  ipAddress: {
    type: String
  },
  userAgent: {
    type: String
  },
  details: {
    type: Map,
    of: mongoose.Schema.Types.Mixed
  },
  timestamp: {
    type: Date,
    default: Date.now
  }
}, {
  timestamps: false
});

// Indexes
auditLogSchema.index({ user: 1, timestamp: -1 });
auditLogSchema.index({ action: 1, timestamp: -1 });
auditLogSchema.index({ resourceType: 1, resourceId: 1 });
auditLogSchema.index({ department: 1, timestamp: -1 });
auditLogSchema.index({ timestamp: -1 });

// TTL index - auto-delete logs older than 2 years
auditLogSchema.index({ timestamp: 1 }, { expireAfterSeconds: 63072000 });
```

#### 7. Notifications Collection

```javascript
const notificationSchema = new mongoose.Schema({
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  type: {
    type: String,
    enum: ['document_upload', 'access_granted', 'version_update', 'folder_shared'],
    required: true
  },
  title: {
    type: String,
    required: true
  },
  message: {
    type: String,
    required: true
  },
  relatedDocument: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Document'
  },
  relatedFolder: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Folder'
  },
  isRead: {
    type: Boolean,
    default: false
  },
  readAt: {
    type: Date
  }
}, {
  timestamps: true
});

// Indexes
notificationSchema.index({ user: 1, isRead: 1, createdAt: -1 });
notificationSchema.index({ user: 1, createdAt: -1 });
notificationSchema.index({ createdAt: -1 });

// TTL index - auto-delete read notifications older than 30 days
notificationSchema.index({ readAt: 1 }, { 
  expireAfterSeconds: 2592000, 
  partialFilterExpression: { isRead: true } 
});
```

---

## 3. BACKEND API ENDPOINTS

### Authentication APIs

```javascript
// routes/auth.js

POST   /api/auth/register           // Register new user (Superadmin only)
POST   /api/auth/login              // User login with JWT
POST   /api/auth/logout             // Logout
POST   /api/auth/refresh-token      // Refresh JWT token
POST   /api/auth/forgot-password    // Send password reset email
POST   /api/auth/reset-password     // Reset password with token
GET    /api/auth/me                 // Get current user info
PUT    /api/auth/update-profile     // Update own profile
PUT    /api/auth/change-password    // Change own password
```

### Department Management APIs

```javascript
// routes/departments.js

GET    /api/departments             // Get all departments
GET    /api/departments/:id         // Get department by ID
POST   /api/departments             // Create new department (Superadmin)
PUT    /api/departments/:id         // Update department (Superadmin)
DELETE /api/departments/:id         // Delete department (Superadmin)
PATCH  /api/departments/:id/toggle  // Toggle active status
GET    /api/departments/:id/stats   // Get department statistics
```

### User Management APIs

```javascript
// routes/users.js

GET    /api/users                   // Get all users (paginated)
GET    /api/users/:id               // Get user by ID
POST   /api/users                   // Create new user
PUT    /api/users/:id               // Update user
DELETE /api/users/:id               // Delete user
PATCH  /api/users/:id/toggle        // Toggle active status
GET    /api/users/department/:deptId // Get users by department
GET    /api/users/role/:role        // Get users by role
```

### Folder Management APIs

```javascript
// routes/folders.js

GET    /api/folders                 // Get all folders (tree structure)
GET    /api/folders/tree            // Get folder tree by department
GET    /api/folders/:id             // Get folder by ID with contents
POST   /api/folders                 // Create new folder
PUT    /api/folders/:id             // Update folder (rename, move)
DELETE /api/folders/:id             // Delete folder
POST   /api/folders/:id/permissions // Set folder permissions
GET    /api/folders/:id/breadcrumb  // Get folder path hierarchy
GET    /api/folders/:id/children    // Get immediate children of folder
POST   /api/folders/:id/move        // Move folder to different parent
```

### Document Management APIs

```javascript
// routes/documents.js

GET    /api/documents               // Get all documents (filtered)
GET    /api/documents/:id           // Get document by ID
POST   /api/documents/upload        // Upload new document
PUT    /api/documents/:id           // Update document metadata
DELETE /api/documents/:id           // Delete document
POST   /api/documents/:id/replace   // Replace document (new version)
GET    /api/documents/:id/versions  // Get version history
POST   /api/documents/:id/restore   // Restore old version
GET    /api/documents/:id/download  // Download document
POST   /api/documents/:id/permissions // Set document permissions
GET    /api/documents/folder/:folderId // Get documents in folder
POST   /api/documents/:id/move      // Move document to different folder
POST   /api/documents/bulk-delete   // Bulk delete documents
```

### Search & Filter APIs

```javascript
// routes/search.js

GET    /api/search/documents        // Advanced search
GET    /api/search/autocomplete     // Search autocomplete
GET    /api/search/recent           // Recent searches
```

### Audit Logs APIs

```javascript
// routes/auditLogs.js

GET    /api/audit-logs              // Get all logs (filtered, paginated)
GET    /api/audit-logs/user/:userId // Get logs for specific user
GET    /api/audit-logs/document/:docId // Get logs for document
GET    /api/audit-logs/department/:deptId // Get logs for department
GET    /api/audit-logs/export       // Export logs to CSV
```

### Notifications APIs

```javascript
// routes/notifications.js

GET    /api/notifications           // Get user notifications
GET    /api/notifications/unread    // Get unread count
PATCH  /api/notifications/:id/read  // Mark as read
PATCH  /api/notifications/read-all  // Mark all as read
DELETE /api/notifications/:id       // Delete notification
DELETE /api/notifications/clear-all // Clear all read notifications
```

### Dashboard/Statistics APIs

```javascript
// routes/dashboard.js

GET    /api/dashboard/stats         // Overall statistics
GET    /api/dashboard/recent-activity // Recent system activity
GET    /api/dashboard/my-documents  // User's recent documents
GET    /api/dashboard/storage       // Storage usage by department
GET    /api/dashboard/user-activity // User activity summary
```

---

## 4. BACKEND MIDDLEWARE IMPLEMENTATION

### Authentication Middleware

```javascript
// middleware/auth.js

const jwt = require('jsonwebtoken');
const User = require('../models/User');

exports.protect = async (req, res, next) => {
  let token;
  
  if (req.headers.authorization && req.headers.authorization.startsWith('Bearer')) {
    token = req.headers.authorization.split(' ')[1];
  }
  
  if (!token) {
    return res.status(401).json({ success: false, message: 'Not authorized' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = await User.findById(decoded.id).populate('department');
    
    if (!req.user || !req.user.isActive) {
      return res.status(401).json({ success: false, message: 'User not found or inactive' });
    }
    
    next();
  } catch (error) {
    return res.status(401).json({ success: false, message: 'Invalid token' });
  }
};
```

### Authorization Middleware

```javascript
// middleware/roleCheck.js

exports.authorize = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ 
        success: false, 
        message: 'Not authorized to access this resource' 
      });
    }
    next();
  };
};
```

### Permission Check Middleware

```javascript
// middleware/permissions.js

const Folder = require('../models/Folder');
const Document = require('../models/Document');

exports.checkFolderPermission = (requiredAccess) => {
  return async (req, res, next) => {
    try {
      const folder = await Folder.findById(req.params.id || req.body.folder);
      
      if (!folder) {
        return res.status(404).json({ success: false, message: 'Folder not found' });
      }
      
      if (req.user.role === 'superadmin') {
        req.folder = folder;
        return next();
      }
      
      const userPermission = folder.permissions.find(
        p => p.user && p.user.toString() === req.user._id.toString()
      );
      
      const rolePermission = folder.permissions.find(
        p => p.role === req.user.role
      );
      
      const permission = userPermission || rolePermission;
      
      if (!permission || !permission.access.includes(requiredAccess)) {
        return res.status(403).json({ 
          success: false, 
          message: 'Insufficient permissions' 
        });
      }
      
      req.folder = folder;
      next();
    } catch (error) {
      return res.status(500).json({ success: false, message: error.message });
    }
  };
};

exports.checkDocumentPermission = (requiredAccess) => {
  return async (req, res, next) => {
    try {
      const document = await Document.findById(req.params.id);
      
      if (!document) {
        return res.status(404).json({ success: false, message: 'Document not found' });
      }
      
      if (req.user.role === 'superadmin') {
        req.document = document;
        return next();
      }
      
      const userPermission = document.permissions.find(
        p => p.user && p.user.toString() === req.user._id.toString()
      );
      
      const rolePermission = document.permissions.find(
        p => p.role === req.user.role
      );
      
      let permission = userPermission || rolePermission;
      
      if (!permission) {
        const folder = await Folder.findById(document.folder);
        const folderUserPerm = folder.permissions.find(
          p => p.user && p.user.toString() === req.user._id.toString()
        );
        const folderRolePerm = folder.permissions.find(
          p => p.role === req.user.role
        );
        permission = folderUserPerm || folderRolePerm;
      }
      
      if (!permission || !permission.access.includes(requiredAccess)) {
        return res.status(403).json({ 
          success: false, 
          message: 'Insufficient permissions' 
        });
      }
      
      req.document = document;
      next();
    } catch (error) {
      return res.status(500).json({ success: false, message: error.message });
    }
  };
};
```

### Audit Logging Middleware

```javascript
// middleware/auditLog.js

const AuditLog = require('../models/AuditLog');

exports.logAction = (action, resourceType) => {
  return async (req, res, next) => {
    const originalSend = res.send;
    
    res.send = function(data) {
      if (res.statusCode >= 200 && res.statusCode < 300) {
        const logData = {
          user: req.user._id,
          action,
          resourceType,
          resourceId: req.params.id || req.body._id,
          resourceName: req.body.title || req.body.name,
          department: req.user.department,
          ipAddress: req.ip || req.connection.remoteAddress,
          userAgent: req.get('user-agent')
        };
        
        AuditLog.create(logData).catch(err => console.error('Audit log error:', err));
      }
      
      originalSend.call(this, data);
    };
    
    next();
  };
};
```

### File Upload Middleware

```javascript
// middleware/upload.js

const multer = require('multer');
const { S3Client } = require('@aws-sdk/client-s3');
const multerS3 = require('multer-s3');

const s3Client = new S3Client({
  region: process.env.AWS_REGION,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
  }
});

const fileFilter = (req, file, cb) => {
  const allowedTypes = ['application/pdf', 'application/msword', 
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
    'application/vnd.ms-excel',
    'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
    'image/jpeg', 'image/jpg', 'image/png', 'application/zip'];
  
  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error('Invalid file type. Only PDF, DOCX, XLSX, JPG, PNG, ZIP allowed'), false);
  }
};

const upload = multer({
  storage: multerS3({
    s3: s3Client,
    bucket: process.env.AWS_BUCKET_NAME,
    metadata: (req, file, cb) => {
      cb(null, { fieldName: file.fieldname });
    },
    key: (req, file, cb) => {
      const uniqueName = `${Date.now()}-${Math.round(Math.random() * 1E9)}-${file.originalname}`;
      cb(null, `documents/${uniqueName}`);
    }
  }),
  fileFilter,
  limits: {
    fileSize: 50 * 1024 * 1024 // 50MB max
  }
});

module.exports = upload;
```

### Error Handler Middleware

```javascript
// middleware/errorHandler.js

exports.errorHandler = (err, req, res, next) => {
  let error = { ...err };
  error.message = err.message;
  
  console.error(err);
  
  // Mongoose bad ObjectId
  if (err.name === 'CastError') {
    error.message = 'Resource not found';
    error.statusCode = 404;
  }
  
  // Mongoose duplicate key
  if (err.code === 11000) {
    error.message = 'Duplicate field value entered';
    error.statusCode = 400;
  }
  
  // Mongoose validation error
  if (err.name === 'ValidationError') {
    error.message = Object.values(err.errors).map(e => e.message).join(', ');
    error.statusCode = 400;
  }
  
  // JWT errors
  if (err.name === 'JsonWebTokenError') {
    error.message = 'Invalid token';
    error.statusCode = 401;
  }
  
  if (err.name === 'TokenExpiredError') {
    error.message = 'Token expired';
    error.statusCode = 401;
  }
  
  res.status(error.statusCode || 500).json({
    success: false,
    message: error.message || 'Server Error'
  });
};
```

---

## 5. FRONTEND IMPLEMENTATION

### 5.1 Authentication Pages

#### Login Page
**Components to Build:**
- [ ] Login form with username/email and password
- [ ] Form validation (required fields, email format)
- [ ] "Remember me" checkbox
- [ ] "Forgot password" link
- [ ] Show/hide password toggle
- [ ] Loading state during login
- [ ] Error message display
- [ ] Redirect to dashboard on success

#### Forgot Password Page
- [ ] Email input form
- [ ] Send reset link functionality
- [ ] Success message display
- [ ] Back to login link

#### Reset Password Page
- [ ] New password input (with confirmation)
- [ ] Password strength indicator
- [ ] Token validation from URL
- [ ] Success message and redirect

### 5.2 Dashboard Layout

#### Sidebar Navigation
**Menu Items:**
- [ ] Dashboard (home icon)
- [ ] My Documents (document icon)
- [ ] Folders (folder icon)
- [ ] Users (people icon) - Admin/Superadmin only
- [ ] Audit Logs (history icon) - Admin/Superadmin only
- [ ] Settings (gear icon)

**Features:**
- [ ] Collapsible sidebar
- [ ] Active menu highlighting
- [ ] Role-based menu visibility
- [ ] User info at bottom (avatar, name, role)

#### Top Navbar
- [ ] Logo and title
- [ ] Global search bar
- [ ] Notifications bell icon with badge (unread count)
- [ ] User profile dropdown
  - View Profile
  - Settings
  - Change Password
  - Logout

#### Dashboard Home Page
**Widgets:**
- [ ] Welcome message with user name
- [ ] Statistics cards:
  - Total Documents
  - My Documents
  - Storage Used
  - Recent Uploads (this month)
- [ ] Recent Activity Feed (last 10 actions)
- [ ] Quick Access to Frequently Used Folders
- [ ] Storage Usage Chart (by department) - Admin only

### 5.3 User Management (Admin/Superadmin)

#### Users List Page
**Features:**
- [ ] Data table with columns:
  - Username
  - Email
  - Role
  - Department
  - Status (Active/Inactive)
  - Last Login
  - Actions (Edit, Delete, Toggle Status)
- [ ] Search by username or email
- [ ] Filter by role, department, status
- [ ] Pagination (20 users per page)
- [ ] Add User button (top right)
- [ ] Bulk actions (optional)

#### Add/Edit User Modal
**Form Fields:**
- [ ] Username (required, unique)
- [ ] Email (required, unique, validated)
- [ ] Password (required for new, optional for edit)
- [ ] Confirm Password
- [ ] Role dropdown (Superadmin, Admin, Team Member, Member Bank)
- [ ] Department dropdown
- [ ] Active status toggle
- [ ] Form validation
- [ ] Submit and Cancel buttons

#### User Profile Page
- [ ] View own profile details
- [ ] Edit profile button
- [ ] Change password section
- [ ] Last login information
- [ ] Activity summary

### 5.4 Department Management (Superadmin)

#### Departments List Page
- [ ] Table with: Name, Code, Head, Status, Actions
- [ ] Add Department button
- [ ] Edit/Delete actions
- [ ] Toggle active status

#### Add/Edit Department Modal
- [ ] Name field
- [ ] Code field (uppercase, unique)
- [ ] Description textarea
- [ ] Department Head dropdown (users)
- [ ] Active status toggle

### 5.5 Folder Management

#### Folder Tree View
**Features:**
- [ ] Tree structure display (collapsible nodes)
- [ ] Icons for folders (different icons for levels)
- [ ] Folder name with document count
- [ ] Context menu on right-click:
  - Create Subfolder
  - Rename
  - Move
  - Set Permissions
  - Delete
  - View Details
- [ ] Click to view folder contents
- [ ] Breadcrumb navigation
- [ ] Search folders by name
- [ ] Filter by department

#### Create Folder Modal
**Form:**
- [ ] Folder name input
- [ ] Parent folder selector (tree dropdown)
- [ ] Department dropdown
- [ ] Auto-calculate path preview
- [ ] Set initial permissions
- [ ] Create button

#### Folder Permissions Modal
**Features:**
- [ ] List of users (searchable)
- [ ] List of roles
- [ ] Permission checkboxes for each:
  - View
  - Upload
  - Edit
  - Delete
  - Download
- [ ] Apply to subfolders checkbox
- [ ] Save button

#### Folder Details Page
**Sections:**
- [ ] Folder information (name, path, created by, created date)
- [ ] Subfolders list
- [ ] Documents list (with filters)
- [ ] Upload document button
- [ ] Folder statistics (total docs, total size)
- [ ] Edit folder button
- [ ] Delete folder button (with confirmation)

### 5.6 Document Management

#### Document List View
**Display Options:**
- [ ] Toggle between grid and table view
- [ ] Grid View: Cards with thumbnail, title, date, size
- [ ] Table View: Columns - Title, Type, Size, Uploaded By, Date, Actions

**Filters:**
- [ ] Sidebar filter panel:
  - Date range picker (upload date)
  - Uploader dropdown (all users)
  - File type checkboxes (PDF, DOCX, XLSX, Images, ZIP)
  - Tags multi-select
  - Department filter
  - Folder filter
- [ ] Clear all filters button
- [ ] Active filters display (chips)

**Actions:**
- [ ] Search bar with autocomplete
- [ ] Sort dropdown (Name, Date, Size)
- [ ] Upload button
- [ ] Bulk select checkbox
- [ ] Bulk actions: Download, Delete

#### Document Upload Modal
**Features:**
- [ ] Drag-and-drop zone (or click to browse)
- [ ] Multiple file upload support
- [ ] File preview list with:
  - File name
  - File size
  - Remove button
- [ ] Metadata form:
  - Title (auto-filled from filename, editable)
  - Tags input (comma-separated or chips)
  - Folder selector
- [ ] Upload progress bar (per file)
- [ ] Cancel upload button
- [ ] Success message with link to document

#### Document Detail Page
**Sections:**
- [ ] Document preview:
  - PDF viewer (with zoom, page navigation)
  - Image viewer
  - Download for other types
- [ ] Document information:
  - Title
  - File type and size
  - Uploaded by and date
  - Folder location (breadcrumb)
  - Tags (clickable)
  - Version number
- [ ] Action buttons:
  - Download
  - Edit metadata
  - Replace (upload new version)
  - Move to different folder
  - Delete (with confirmation)
  - Share (copy link)
- [ ] Version History section:
  - List of all versions
  - Download specific version
  - Restore version button
  - View changes description
- [ ] Permissions section:
  - List of users/roles with access
  - Edit permissions button
- [ ] Activity/Audit trail:
  - Who viewed, when
  - Who downloaded, when
  - Who edited, when

#### Document Viewer Component
**Features:**
- [ ] PDF viewer (using react-pdf or pdf.js)
- [ ] Zoom controls (+, -, fit)
- [ ] Page navigation (prev, next, page number)
- [ ] Fullscreen mode
- [ ] Download button
- [ ] Print button (optional)

#### Version History Modal
**Display:**
- [ ] Table with columns:
  - Version number
  - Uploaded by
  - Upload date
  - File size
  - Changes description
  - Actions (Download, Restore)
- [ ] Confirmation dialog for restore action
- [ ] Close button

### 5.7 Search & Filter

#### Advanced Search Page
**Features:**
- [ ] Large search input with placeholder
- [ ] Search button
- [ ] Advanced filters panel:
  - Date range picker (From - To)
  - Department dropdown
  - Folder selector
  - File type checkboxes
  - Tags multi-select (autocomplete)
  - Uploader dropdown
- [ ] Search results:
  - Grid/Table view toggle
  - Result count
  - Sort options (Relevance, Date, Name, Size)
  - Pagination
  - Result highlighting (search term)
- [ ] Save search button (optional)
- [ ] Export results button (CSV)

#### Global Search (Navbar)
**Features:**
- [ ] Search input with icon
- [ ] Auto-suggest dropdown (appears on typing)
- [ ] Show recent searches
- [ ] Show top results (max 5)
- [ ] "See all results" link to advanced search
- [ ] Keyboard shortcuts (Ctrl+K to focus)

### 5.8 Audit Logs (Admin Only)

#### Audit Logs Page
**Features:**
- [ ] Table with columns:
  - Timestamp
  - User
  - Action
  - Resource Type
  - Resource Name
  - Department
  - IP Address
- [ ] Filters:
  - Date range picker
  - User dropdown
  - Action dropdown (Upload, Download, View, Edit, Delete, Login, Logout)
  - Resource type dropdown (Document, Folder, User)
  - Department dropdown
- [ ] Search by resource name
- [ ] Pagination (50 logs per page)
- [ ] Export to CSV button
- [ ] Refresh button
- [ ] Auto-refresh toggle (every 30 seconds)

### 5.9 Notifications

#### Notifications Dropdown (Navbar)
**Features:**
- [ ] Bell icon with unread count badge
- [ ] Dropdown panel (max-height with scroll)
- [ ] List of notifications:
  - Icon based on type
  - Title and message
  - Timestamp (relative: "2 hours ago")
  - Read/Unread indicator
  - Click to mark as read and navigate
- [ ] "Mark all as read" button
- [ ] "View all notifications" link
- [ ] Empty state message

#### Notifications Page
**Features:**
- [ ] List of all notifications (paginated)
- [ ] Filter by type, read/unread
- [ ] Mark as read/unread toggle
- [ ] Delete notification button
- [ ] Clear all read notifications button
- [ ] Group by date (Today, Yesterday, This Week, Older)

### 5.10 Settings Page

#### Profile Settings Tab
- [ ] View and edit profile fields:
  - Username (read-only)
  - Email
  - Department (read-only for non-admin)
- [ ] Save changes button

#### Change Password Tab
- [ ] Current password input
- [ ] New password input
- [ ] Confirm new password input
- [ ] Password strength indicator
- [ ] Change password button

#### System Settings Tab (Superadmin only)
- [ ] File upload settings:
  - Max file size slider
  - Allowed file types checkboxes
- [ ] Session settings:
  - Session timeout duration
  - Auto-logout on inactivity
- [ ] Storage settings:
  - Storage quota per department
  - Low storage alerts
- [ ] Backup settings:
  - Backup frequency
  - Retention period
- [ ] Save settings button

---

## 6. KEY FEATURES IMPLEMENTATION

### 6.1 Role-Based Access Control (RBAC)

**Implementation Steps:**
- [ ] Define permission constants
  ```javascript
  const PERMISSIONS = {
    VIEW: 'view',
    UPLOAD: 'upload',
    EDIT: 'edit',
    DELETE: 'delete',
    DOWNLOAD: 'download'
  };
  
  const ROLES = {
    SUPERADMIN: 'superadmin',
    ADMIN: 'admin',
    TEAM_MEMBER: 'team_member',
    MEMBER_BANK: 'member_bank'
  };
  ```
- [ ] Create permission check utility functions
  ```javascript
  const hasPermission = (user, resource, requiredPermission) => {
    if (user.role === ROLES.SUPERADMIN) return true;
    
    const userPerm = resource.permissions.find(p => 
      p.user && p.user.toString() === user._id.toString()
    );
    
    const rolePerm = resource.permissions.find(p => 
      p.role === user.role
    );
    
    const permission = userPerm || rolePerm;
    return permission && permission.access.includes(requiredPermission);
  };
  ```
- [ ] Apply permissions in API endpoints
- [ ] Apply permissions in frontend (hide/show UI elements)
  ```javascript
  {hasPermission(user, folder, 'edit') && (
    <Button onClick={handleEdit}>Edit</Button>
  )}
  ```
- [ ] Implement permission inheritance (Folder → Subfolder → Document)
- [ ] Allow permission override at each level

### 6.2 Version Control

**Implementation Steps:**
- [ ] Auto-create DocumentVersion on file replacement
  ```javascript
  // When replacing document
  const newVersion = await DocumentVersion.create({
    document: document._id,
    version: document.version,
    fileUrl: document.fileUrl,
    fileKey: document.fileKey,
    fileSize: document.fileSize,
    uploadedBy: req.user._id,
    changes: req.body.changes || 'File replaced'
  });
  
  // Update document
  document.version += 1;
  document.fileUrl = newFileUrl;
  document.fileKey = newFileKey;
  await document.save();
  ```
- [ ] Store all versions with metadata
- [ ] Implement version restore functionality
  ```javascript
  const restoreVersion = async (documentId, versionId) => {
    const version = await DocumentVersion.findById(versionId);
    const document = await Document.findById(documentId);
    
    // Create version from current document
    await DocumentVersion.create({
      document: document._id,
      version: document.version,
      fileUrl: document.fileUrl,
      fileKey: document.fileKey,
      fileSize: document.fileSize,
      uploadedBy: req.user._id,
      changes: 'Backup before restore'
    });
    
    // Restore old version
    document.version += 1;
    document.fileUrl = version.fileUrl;
    document.fileKey = version.fileKey;
    document.fileSize = version.fileSize;
    await document.save();
  };
  ```
- [ ] Display version history in UI
- [ ] Allow downloading specific versions
- [ ] Show version comparison (optional)

### 6.3 Advanced Search

**Implementation Steps:**
- [ ] Create MongoDB text indexes
  ```javascript
  documentSchema.index({ title: 'text', tags: 'text' });
  ```
- [ ] Implement multi-field search query
  ```javascript
  const searchQuery = {
    $and: []
  };
  
  if (title) {
    searchQuery.$and.push({
      $text: { $search: title }
    });
  }
  
  if (tags && tags.length > 0) {
    searchQuery.$and.push({
      tags: { $in: tags }
    });
  }
  
  if (folder) {
    searchQuery.$and.push({ folder });
  }
  
  if (fileType && fileType.length > 0) {
    searchQuery.$and.push({
      fileType: { $in: fileType }
    });
  }
  
  if (uploadDateFrom || uploadDateTo) {
    const dateQuery = {};
    if (uploadDateFrom) dateQuery.$gte = new Date(uploadDateFrom);
    if (uploadDateTo) dateQuery.$lte = new Date(uploadDateTo);
    searchQuery.$and.push({ createdAt: dateQuery });
  }
  
  const documents = await Document.find(
    searchQuery.$and.length > 0 ? searchQuery : {}
  )
  .populate('uploadedBy', 'username')
  .populate('folder', 'name path')
  .sort({ [sortBy]: sortOrder })
  .skip((page - 1) * limit)
  .limit(limit);
  ```
- [ ] Implement autocomplete/suggestions
  ```javascript
  // Get unique tags for autocomplete
  const tags = await Document.distinct('tags', {
    tags: { $regex: query, $options: 'i' }
  });
  
  // Get matching titles
  const titles = await Document.find({
    title: { $regex: query, $options: 'i' }
  }, 'title').limit(5);
  ```
- [ ] Add pagination
- [ ] Highlight search terms in results (frontend)
- [ ] Implement search filters
- [ ] Cache recent searches (optional)

### 6.4 Audit Logging

**Implementation Steps:**
- [ ] Create audit log on every critical action
  ```javascript
  await AuditLog.create({
    user: req.user._id,
    action: 'upload',
    resourceType: 'document',
    resourceId: document._id,
    resourceName: document.title,
    department: req.user.department,
    ipAddress: req.ip,
    userAgent: req.get('user-agent'),
    details: {
      folder: document.folder,
      fileSize: document.fileSize
    }
  });
  ```
- [ ] Log these actions:
  - File uploads
  - File downloads
  - File views (optional, can be heavy)
  - File edits/renames
  - File deletions
  - File moves
  - User logins/logouts
  - Permission changes
  - Folder creation/deletion
- [ ] Capture metadata: User, Timestamp, IP, User Agent
- [ ] Make logs searchable and filterable
- [ ] Implement log retention policy (optional: TTL index)
- [ ] Export logs to CSV
- [ ] Display logs in admin dashboard

### 6.5 Notification System

**Implementation Steps:**
- [ ] Create notification on:
  - **Document upload in accessible folder:**
    ```javascript
    const foldersWithAccess = await Folder.find({
      _id: document.folder,
      'permissions.user': { $exists: true }
    }).populate('permissions.user');
    
    const usersToNotify = foldersWithAccess[0].permissions
      .filter(p => p.user && p.access.includes('view'))
      .map(p => p.user._id);
    
    const notifications = usersToNotify.map(userId => ({
      user: userId,
      type: 'document_upload',
      title: 'New Document Uploaded',
      message: `${req.user.username} uploaded "${document.title}"`,
      relatedDocument: document._id,
      relatedFolder: document.folder
    }));
    
    await Notification.insertMany(notifications);
    ```
  - **Access granted to folder/document:**
    ```javascript
    await Notification.create({
      user: grantedUserId,
      type: 'access_granted',
      title: 'Access Granted',
      message: `You have been granted access to "${resource.name}"`,
      relatedFolder: folderId
    });
    ```
  - **New version uploaded:**
    ```javascript
    await Notification.create({
      user: document.uploadedBy,
      type: 'version_update',
      title: 'Document Updated',
      message: `A new version of "${document.title}" has been uploaded`,
      relatedDocument: document._id
    });
    ```
- [ ] Implement real-time updates:
  - **Polling approach:** Frontend polls `/api/notifications/unread` every 30 seconds
  - **WebSocket approach (advanced):** Use Socket.io for real-time push
- [ ] Email notifications (optional):
  - Integrate with email service (SendGrid, AWS SES)
  - User preference toggle for email notifications
  - Daily digest
- [ ] Mark as read functionality
- [ ] Notification preferences per user
- [ ] Auto-delete old read notifications (TTL index)

### 6.6 Security Features

**Implementation Steps:**

#### JWT Authentication
- [ ] Generate JWT on login
  ```javascript
  const generateToken = (userId) => {
    return jwt.sign(
      { id: userId },
      process.env.JWT_SECRET,
      { expiresIn: process.env.JWT_EXPIRE || '7d' }
    );
  };
  ```
- [ ] Store token in httpOnly cookie (more secure) or localStorage
- [ ] Implement token refresh mechanism
- [ ] Verify token on every protected route

#### Password Security
- [ ] Hash passwords with bcrypt (salt rounds: 12)
- [ ] Implement password complexity requirements:
  - Minimum 8 characters
  - At least 1 uppercase letter
  - At least 1 lowercase letter
  - At least 1 number
  - At least 1 special character
- [ ] Password reset with email token
  ```javascript
  const crypto = require('crypto');
  
  // Generate reset token
  const resetToken = crypto.randomBytes(32).toString('hex');
  user.passwordResetToken = crypto
    .createHash('sha256')
    .update(resetToken)
    .digest('hex');
  user.passwordResetExpires = Date.now() + 3600000; // 1 hour
  await user.save();
  
  // Send email with resetToken (not hashed version)
  ```

#### MFA (Multi-Factor Authentication) - Optional
- [ ] Install speakeasy and qrcode libraries
- [ ] Generate MFA secret for user
- [ ] Display QR code for user to scan
- [ ] Verify TOTP code on login
- [ ] Backup codes generation

#### Auto Logout on Inactivity
- [ ] **Frontend:** Detect user inactivity
  ```javascript
  let inactivityTimer;
  const INACTIVITY_TIMEOUT = 30 * 60 * 1000; // 30 minutes
  
  const resetTimer = () => {
    clearTimeout(inactivityTimer);
    inactivityTimer = setTimeout(() => {
      // Auto logout
      localStorage.removeItem('token');
      window.location.href = '/login';
    }, INACTIVITY_TIMEOUT);
  };
  
  // Listen to user activity
  document.addEventListener('mousemove', resetTimer);
  document.addEventListener('keypress', resetTimer);
  document.addEventListener('click', resetTimer);
  ```
- [ ] **Backend:** Use short-lived JWT tokens (1-2 hours)
- [ ] Implement refresh token mechanism

#### Additional Security
- [ ] Input validation with express-validator
- [ ] Sanitize user inputs (prevent XSS)
- [ ] Use helmet.js for security headers
- [ ] Implement rate limiting
  ```javascript
  const rateLimit = require('express-rate-limit');
  
  const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100 // limit each IP to 100 requests per windowMs
  });
  
  app.use('/api/', limiter);
  ```
- [ ] CORS configuration (whitelist allowed origins)
- [ ] HTTPS enforcement in production
- [ ] SQL/NoSQL injection prevention (use parameterized queries)

### 6.7 File Storage (AWS S3)

**Implementation Steps:**

#### S3 Setup
- [ ] Create AWS account and S3 bucket
- [ ] Configure bucket CORS policy
  ```json
  [
    {
      "AllowedHeaders": ["*"],
      "AllowedMethods": ["GET", "PUT", "POST", "DELETE"],
      "AllowedOrigins": ["http://localhost:3000", "https://yourdomain.com"],
      "ExposeHeaders": []
    }
  ]
  ```
- [ ] Set bucket policy for private access
- [ ] Create IAM user with S3 access
- [ ] Get Access Key ID and Secret Access Key

#### Backend Integration
- [ ] Install AWS SDK v3
  ```bash
  npm install @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
  ```
- [ ] Configure S3 client
  ```javascript
  const { S3Client, PutObjectCommand, GetObjectCommand, DeleteObjectCommand } = require('@aws-sdk/client-s3');
  const { getSignedUrl } = require('@aws-sdk/s3-request-presigner');
  
  const s3Client = new S3Client({
    region: process.env.AWS_REGION,
    credentials: {
      accessKeyId: process.env.AWS_ACCESS_KEY_ID,
      secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
    }
  });
  ```
- [ ] Upload file to S3
  ```javascript
  const uploadToS3 = async (file, key) => {
    const params = {
      Bucket: process.env.AWS_BUCKET_NAME,
      Key: key,
      Body: file.buffer,
      ContentType: file.mimetype
    };
    
    await s3Client.send(new PutObjectCommand(params));
    
    return `https://${process.env.AWS_BUCKET_NAME}.s3.${process.env.AWS_REGION}.amazonaws.com/${key}`;
  };
  ```
- [ ] Generate signed URL for secure downloads
  ```javascript
  const getSignedDownloadUrl = async (key) => {
    const command = new GetObjectCommand({
      Bucket: process.env.AWS_BUCKET_NAME,
      Key: key
    });
    
    return await getSignedUrl(s3Client, command, { expiresIn: 3600 }); // 1 hour
  };
  ```
- [ ] Delete file from S3
  ```javascript
  const deleteFromS3 = async (key) => {
    const params = {
      Bucket: process.env.AWS_BUCKET_NAME,
      Key: key
    };
    
    await s3Client.send(new DeleteObjectCommand(params));
  };
  ```
- [ ] Organize files in folders
  ```javascript
  const key = `documents/${departmentId}/${Date.now()}-${filename}`;
  ```

#### Cleanup
- [ ] Delete S3 files when document is deleted
- [ ] Keep versions in S3 when document is replaced
- [ ] Implement S3 lifecycle policy (optional: archive old versions to Glacier)

### 6.8 Drag & Drop Functionality

**Implementation Steps:**

#### File Upload Drag & Drop
- [ ] Install react-dropzone
  ```bash
  npm install react-dropzone
  ```
- [ ] Implement dropzone component
  ```javascript
  import { useDropzone } from 'react-dropzone';
  
  const onDrop = useCallback((acceptedFiles) => {
    // Handle file upload
    acceptedFiles.forEach(file => {
      const formData = new FormData();
      formData.append('file', file);
      formData.append('title', file.name);
      formData.append('folder', selectedFolder);
      
      uploadDocument(formData);
    });
  }, [selectedFolder]);
  
  const { getRootProps, getInputProps, isDragActive } = useDropzone({
    onDrop,
    accept: {
      'application/pdf': ['.pdf'],
      'application/msword': ['.doc'],
      'application/vnd.openxmlformats-officedocument.wordprocessingml.document': ['.docx'],
      'application/vnd.ms-excel': ['.xls'],
      'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet': ['.xlsx'],
      'image/jpeg': ['.jpg', '.jpeg'],
      'image/png': ['.png'],
      'application/zip': ['.zip']
    },
    maxSize: 50 * 1024 * 1024 // 50MB
  });
  ```
- [ ] Visual feedback during drag operation
- [ ] Upload progress indicator
- [ ] Multiple file upload support

#### Folder Drag & Drop (Move Folder)
- [ ] Implement folder tree drag-and-drop
- [ ] Prevent dropping folder into itself or its children
- [ ] Update folder path on move
- [ ] Visual indicators (drag over, drop target)

### 6.9 PDF Preview

**Implementation Steps:**
- [ ] Install react-pdf
  ```bash
  npm install react-pdf
  ```
- [ ] Import PDF.js worker
  ```javascript
  import { Document, Page, pdfjs } from 'react-pdf';
  
  pdfjs.GlobalWorkerOptions.workerSrc = `//cdnjs.cloudflare.com/ajax/libs/pdf.js/${pdfjs.version}/pdf.worker.min.js`;
  ```
- [ ] Create PDF Viewer component
  ```javascript
  const PDFViewer = ({ fileUrl }) => {
    const [numPages, setNumPages] = useState(null);
    const [pageNumber, setPageNumber] = useState(1);
    const [scale, setScale] = useState(1.0);
    
    const onDocumentLoadSuccess = ({ numPages }) => {
      setNumPages(numPages);
    };
    
    return (
      <div>
        <div className="controls">
          <button onClick={() => setScale(scale - 0.1)}>Zoom Out


- [ ] Mark as read functionality
- [ ] Notification preferences per user
- [ ] Auto-delete old read notifications (TTL index)

### 6.6 Security Features

**Implementation Steps:**

#### JWT Authentication
- [ ] Generate JWT on login
  ```javascript
  const generateToken = (userId) => {
    return jwt.sign(
      { id: userId },
      process.env.JWT_SECRET,
      { expiresIn: process.env.JWT_EXPIRE || '7d' }
    );
  };
  ```
- [ ] Store token in httpOnly cookie (more secure) or localStorage
- [ ] Implement token refresh mechanism
- [ ] Verify token on every protected route

#### Password Security
- [ ] Hash passwords with bcrypt (salt rounds: 12)
- [ ] Implement password complexity requirements:
  - Minimum 8 characters
  - At least 1 uppercase letter
  - At least 1 lowercase letter
  - At least 1 number
  - At least 1 special character
- [ ] Password reset with email token
  ```javascript
  const crypto = require('crypto');
  
  // Generate reset token
  const resetToken = crypto.randomBytes(32).toString('hex');
  user.passwordResetToken = crypto
    .createHash('sha256')
    .update(resetToken)
    .digest('hex');
  user.passwordResetExpires = Date.now() + 3600000; // 1 hour
  await user.save();
  
  // Send email with resetToken (not hashed version)
  ```

#### MFA (Multi-Factor Authentication) - Optional
- [ ] Install speakeasy and qrcode libraries
- [ ] Generate MFA secret for user
- [ ] Display QR code for user to scan
- [ ] Verify TOTP code on login
- [ ] Backup codes generation

#### Auto Logout on Inactivity
- [ ] **Frontend:** Detect user inactivity
  ```javascript
  let inactivityTimer;
  const INACTIVITY_TIMEOUT = 30 * 60 * 1000; // 30 minutes
  
  const resetTimer = () => {
    clearTimeout(inactivityTimer);
    inactivityTimer = setTimeout(() => {
      // Auto logout
      localStorage.removeItem('token');
      window.location.href = '/login';
    }, INACTIVITY_TIMEOUT);
  };
  
  // Listen to user activity
  document.addEventListener('mousemove', resetTimer);
  document.addEventListener('keypress', resetTimer);
  document.addEventListener('click', resetTimer);
  ```
- [ ] **Backend:** Use short-lived JWT tokens (1-2 hours)
- [ ] Implement refresh token mechanism

#### Additional Security
- [ ] Input validation with express-validator
- [ ] Sanitize user inputs (prevent XSS)
- [ ] Use helmet.js for security headers
- [ ] Implement rate limiting
  ```javascript
  const rateLimit = require('express-rate-limit');
  
  const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100 // limit each IP to 100 requests per windowMs
  });
  
  app.use('/api/', limiter);
  ```
- [ ] CORS configuration (whitelist allowed origins)
- [ ] HTTPS enforcement in production
- [ ] SQL/NoSQL injection prevention (use parameterized queries)

### 6.7 File Storage (AWS S3)

**Implementation Steps:**

#### S3 Setup
- [ ] Create AWS account and S3 bucket
- [ ] Configure bucket CORS policy
  ```json
  [
    {
      "AllowedHeaders": ["*"],
      "AllowedMethods": ["GET", "PUT", "POST", "DELETE"],
      "AllowedOrigins": ["http://localhost:3000", "https://yourdomain.com"],
      "ExposeHeaders": []
    }
  ]
  ```
- [ ] Set bucket policy for private access
- [ ] Create IAM user with S3 access
- [ ] Get Access Key ID and Secret Access Key

#### Backend Integration
- [ ] Install AWS SDK v3
  ```bash
  npm install @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
  ```
- [ ] Configure S3 client
  ```javascript
  const { S3Client, PutObjectCommand, GetObjectCommand, DeleteObjectCommand } = require('@aws-sdk/client-s3');
  const { getSignedUrl } = require('@aws-sdk/s3-request-presigner');
  
  const s3Client = new S3Client({
    region: process.env.AWS_REGION,
    credentials: {
      accessKeyId: process.env.AWS_ACCESS_KEY_ID,
      secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
    }
  });
  ```
- [ ] Upload file to S3
  ```javascript
  const uploadToS3 = async (file, key) => {
    const params = {
      Bucket: process.env.AWS_BUCKET_NAME,
      Key: key,
      Body: file.buffer,
      ContentType: file.mimetype
    };
    
    await s3Client.send(new PutObjectCommand(params));
    
    return `https://${process.env.AWS_BUCKET_NAME}.s3.${process.env.AWS_REGION}.amazonaws.com/${key}`;
  };
  ```
- [ ] Generate signed URL for secure downloads
  ```javascript
  const getSignedDownloadUrl = async (key) => {
    const command = new GetObjectCommand({
      Bucket: process.env.AWS_BUCKET_NAME,
      Key: key
    });
    
    return await getSignedUrl(s3Client, command, { expiresIn: 3600 }); // 1 hour
  };
  ```
- [ ] Delete file from S3
  ```javascript
  const deleteFromS3 = async (key) => {
    const params = {
      Bucket: process.env.AWS_BUCKET_NAME,
      Key: key
    };
    
    await s3Client.send(new DeleteObjectCommand(params));
  };
  ```
- [ ] Organize files in folders
  ```javascript
  const key = `documents/${departmentId}/${Date.now()}-${filename}`;
  ```

#### Cleanup
- [ ] Delete S3 files when document is deleted
- [ ] Keep versions in S3 when document is replaced
- [ ] Implement S3 lifecycle policy (optional: archive old versions to Glacier)

### 6.8 Drag & Drop Functionality

**Implementation Steps:**

#### File Upload Drag & Drop
- [ ] Install react-dropzone
  ```bash
  npm install react-dropzone
  ```
- [ ] Implement dropzone component
  ```javascript
  import { useDropzone } from 'react-dropzone';
  
  const onDrop = useCallback((acceptedFiles) => {
    // Handle file upload
    acceptedFiles.forEach(file => {
      const formData = new FormData();
      formData.append('file', file);
      formData.append('title', file.name);
      formData.append('folder', selectedFolder);
      
      uploadDocument(formData);
    });
  }, [selectedFolder]);
  
  const { getRootProps, getInputProps, isDragActive } = useDropzone({
    onDrop,
    accept: {
      'application/pdf': ['.pdf'],
      'application/msword': ['.doc'],
      'application/vnd.openxmlformats-officedocument.wordprocessingml.document': ['.docx'],
      'application/vnd.ms-excel': ['.xls'],
      'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet': ['.xlsx'],
      'image/jpeg': ['.jpg', '.jpeg'],
      'image/png': ['.png'],
      'application/zip': ['.zip']
    },
    maxSize: 50 * 1024 * 1024 // 50MB
  });
  ```
- [ ] Visual feedback during drag operation
- [ ] Upload progress indicator
- [ ] Multiple file upload support

#### Folder Drag & Drop (Move Folder)
- [ ] Implement folder tree drag-and-drop
- [ ] Prevent dropping folder into itself or its children
- [ ] Update folder path on move
- [ ] Visual indicators (drag over, drop target)

### 6.9 PDF Preview

**Implementation Steps:**
- [ ] Install react-pdf
  ```bash
  npm install react-pdf
  ```
- [ ] Import PDF.js worker
  ```javascript
  import { Document, Page, pdfjs } from 'react-pdf';
  
  pdfjs.GlobalWorkerOptions.workerSrc = `//cdnjs.cloudflare.com/ajax/libs/pdf.js/${pdfjs.version}/pdf.worker.min.js`;
  ```
- [ ] Create PDF Viewer component
  ```javascript
  const PDFViewer = ({ fileUrl }) => {
    const [numPages, setNumPages] = useState(null);
    const [pageNumber, setPageNumber] = useState(1);
    const [scale, setScale] = useState(1.0);
    
    const onDocumentLoadSuccess = ({ numPages }) => {
      setNumPages(numPages);
    };
    
    return (
      <div>
        <div className="controls">
          <button onClick={() => setScale(scale - 0.1)}>Zoom Out</button>
          <button onClick={() => setScale(scale + 0.1)}>Zoom In</button>
          <button onClick={() => setPageNumber(pageNumber - 1)} disabled={pageNumber <= 1}>
            Previous
          </button>
          <span>Page {pageNumber} of {numPages}</span>
          <button onClick={() => setPageNumber(pageNumber + 1)} disabled={pageNumber >= numPages}>
            Next
          </button>
        </div>
        
        <Document
          file={fileUrl}
          onLoadSuccess={onDocumentLoadSuccess}
        >
          <Page pageNumber={pageNumber} scale={scale} />
        </Document>
      </div>
    );
  };
  ```
- [ ] Add zoom controls (+, -, fit width, fit page)
- [ ] Page navigation (prev, next, jump to page)
- [ ] Fullscreen mode
- [ ] Loading state
- [ ] Error handling (corrupted PDF, etc.)

---

## 7. NON-FUNCTIONAL REQUIREMENTS

### Performance Optimization

**Backend:**
- [ ] Add database indexes on frequently queried fields
  ```javascript
  // Already covered in schema definitions
  ```
- [ ] Implement pagination on all list endpoints
  ```javascript
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 20;
  const skip = (page - 1) * limit;
  
  const documents = await Document.find(query)
    .skip(skip)
    .limit(limit);
  
  const total = await Document.countDocuments(query);
  
  res.json({
    success: true,
    data: documents,
    pagination: {
      page,
      limit,
      total,
      pages: Math.ceil(total / limit)
    }
  });
  ```
- [ ] Use lean() for read-only queries (faster)
  ```javascript
  const documents = await Document.find(query).lean();
  ```
- [ ] Implement MongoDB aggregation for complex queries
- [ ] Enable compression middleware
  ```javascript
  const compression = require('compression');
  app.use(compression());
  ```
- [ ] Cache frequently accessed data (Redis - optional)

**Frontend:**
- [ ] Lazy load components
  ```javascript
  const Dashboard = lazy(() => import('./pages/Dashboard'));
  ```
- [ ] Implement virtual scrolling for large lists
- [ ] Optimize images (compress, use appropriate formats)
- [ ] Code splitting
- [ ] Memoize expensive calculations
  ```javascript
  const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
  ```
- [ ] Debounce search inputs
  ```javascript
  const debouncedSearch = useCallback(
    debounce((query) => searchDocuments(query), 500),
    []
  );
  ```
- [ ] Lazy load folder tree (load children on expand)
- [ ] API response time target: < 2 seconds

### Scalability

- [ ] Use cloud storage (S3/Azure) for files - ✅ Already planned
- [ ] MongoDB connection pooling
  ```javascript
  mongoose.connect(process.env.MONGODB_URI, {
    maxPoolSize: 10,
    minPoolSize: 5
  });
  ```
- [ ] Stateless backend (no session storage on server)
- [ ] Horizontal scaling support (load balancer ready)
- [ ] Handle 1000+ documents efficiently
- [ ] Support 100+ concurrent users
- [ ] Database sharding (if needed in future)

### Usability

**Responsive Design:**
- [ ] Mobile responsive layout (320px - 1920px+)
- [ ] Touch-friendly UI elements on mobile
- [ ] Collapsible sidebar on mobile
- [ ] Bottom navigation on mobile (optional)

**User Experience:**
- [ ] Clear navigation structure
- [ ] Loading spinners for async operations
- [ ] Progress indicators (file upload, page loading)
- [ ] Success/error toast notifications
- [ ] Confirmation dialogs for destructive actions
- [ ] Breadcrumb navigation
- [ ] Keyboard shortcuts (optional)
  - Ctrl+K: Focus search
  - Ctrl+U: Upload document
  - Esc: Close modals
- [ ] Tooltips and help text
- [ ] Empty states with helpful messages
- [ ] Error messages with actionable suggestions

### Availability & Reliability

**Monitoring:**
- [ ] Implement health check endpoint
  ```javascript
  app.get('/api/health', (req, res) => {
    res.status(200).json({ 
      status: 'OK', 
      timestamp: new Date().toISOString(),
      uptime: process.uptime()
    });
  });
  ```
- [ ] Setup error logging service (Sentry, LogRocket)
  ```bash
  npm install @sentry/node
  ```
- [ ] Log all errors to file/service
- [ ] Monitor server resources (CPU, RAM, Disk)
- [ ] Setup uptime monitoring (UptimeRobot, Pingdom)
- [ ] Configure alerts for downtime

**Error Handling:**
- [ ] Graceful error handling everywhere
- [ ] User-friendly error messages
- [ ] Automatic retry for failed uploads
- [ ] Fallback mechanisms
- [ ] 404 page for undefined routes
- [ ] 500 error page with support contact

**Target Uptime:** 99.9%

### Backup & Recovery

**Database Backup:**
- [ ] Setup automated daily MongoDB backups
  ```bash
  # Cron job for daily backup
  0 2 * * * mongodump --uri="mongodb://localhost:27017/dms_nafscob" --out=/backups/$(date +\%Y-\%m-\%d)
  ```
- [ ] Store backups in separate location (different server/region)
- [ ] 7-day backup retention
- [ ] Test restore procedure monthly
- [ ] Document restore steps

**File Backup:**
- [ ] S3 versioning enabled
- [ ] S3 cross-region replication (optional)
- [ ] Lifecycle policy for old versions

**Disaster Recovery Plan:**
- [ ] Document recovery procedures
- [ ] Maintain backup of configuration files
- [ ] Keep copy of environment variables
- [ ] Regular DR drills

---

## 8. TESTING

### Backend Testing

**Setup:**
```bash
npm install --save-dev jest supertest mongodb-memory-server
```

**Unit Tests:**
- [ ] Test authentication functions (login, register, JWT generation)
- [ ] Test user CRUD operations
- [ ] Test folder CRUD operations
- [ ] Test document CRUD operations
- [ ] Test permission checks
- [ ] Test search functionality
- [ ] Test audit logging
- [ ] Test file upload/download

**Example Test:**
```javascript
const request = require('supertest');
const app = require('../app');
const User = require('../models/User');

describe('Auth API', () => {
  it('should register a new user', async () => {
    const res = await request(app)
      .post('/api/auth/register')
      .send({
        username: 'testuser',
        email: 'test@example.com',
        password: 'Test@123',
        role: 'team_member'
      });
    
    expect(res.statusCode).toBe(201);
    expect(res.body.success).toBe(true);
    expect(res.body.data).toHaveProperty('token');
  });
  
  it('should login with correct credentials', async () => {
    // First create user
    await User.create({
      username: 'testuser',
      email: 'test@example.com',
      password: 'Test@123'
    });
    
    const res = await request(app)
      .post('/api/auth/login')
      .send({
        email: 'test@example.com',
        password: 'Test@123'
      });
    
    expect(res.statusCode).toBe(200);
    expect(res.body.success).toBe(true);
    expect(res.body).toHaveProperty('token');
  });
});
```

**Integration Tests:**
- [ ] Test complete user registration → login → access protected route flow
- [ ] Test document upload → version control → download flow
- [ ] Test folder creation → permission assignment → access control
- [ ] Test search across multiple filters
- [ ] Test audit log generation

**Testing Checklist:**
- [ ] Write tests for all API endpoints
- [ ] Test success scenarios
- [ ] Test error scenarios (invalid input, unauthorized access)
- [ ] Test edge cases
- [ ] Achieve 70%+ code coverage
- [ ] Run tests before every deployment

### Frontend Testing

**Setup:**
```bash
npm install --save-dev @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

**Component Unit Tests:**
- [ ] Test Login form (validation, submission)
- [ ] Test Document upload component
- [ ] Test Folder tree component
- [ ] Test Search component
- [ ] Test User form component
- [ ] Test Notifications component

**Example Test:**
```javascript
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import Login from './Login';

test('shows error message on invalid login', async () => {
  render(<Login />);
  
  const emailInput = screen.getByLabelText(/email/i);
  const passwordInput = screen.getByLabelText(/password/i);
  const submitButton = screen.getByRole('button', { name: /login/i });
  
  fireEvent.change(emailInput, { target: { value: 'invalid@example.com' } });
  fireEvent.change(passwordInput, { target: { value: 'wrongpassword' } });
  fireEvent.click(submitButton);
  
  await waitFor(() => {
    expect(screen.getByText(/invalid credentials/i)).toBeInTheDocument();
  });
});
```

**Integration Tests:**
- [ ] Test user login flow
- [ ] Test document upload and display
- [ ] Test folder navigation
- [ ] Test search functionality
- [ ] Test permission-based UI rendering

**E2E Tests (Optional - Cypress/Playwright):**
- [ ] Complete user journey: Login → Upload → Search → Download
- [ ] Admin workflow: Create user → Assign permissions → Verify access
- [ ] Test on different browsers (Chrome, Firefox, Safari)

### Security Testing

- [ ] Test JWT expiration and refresh
- [ ] Test unauthorized access attempts (access without token)
- [ ] Test role-based access (team member trying to access admin routes)
- [ ] Test SQL/NoSQL injection prevention
- [ ] Test XSS vulnerabilities (script injection in inputs)
- [ ] Test file upload restrictions (type, size)
- [ ] Test CORS policy
- [ ] Penetration testing (optional: hire security expert)

### User Acceptance Testing (UAT)

**Test with Different Roles:**
- [ ] Superadmin: All features accessible
- [ ] Admin: Department management, user management for dept
- [ ] Team Member: Upload, view, edit own documents
- [ ] Member Bank: Read-only access to shared documents

**Test Workflows:**
- [ ] Upload document → Assign to folder → Set permissions → Verify access
- [ ] Create folder hierarchy → Upload documents → Search and retrieve
- [ ] Replace document → Verify version history → Restore old version
- [ ] Grant user access → Verify notification → User accesses document
- [ ] Delete document → Verify audit log → Confirm permanent deletion

**Usability Testing:**
- [ ] Is navigation intuitive?
- [ ] Are error messages clear?
- [ ] Can users complete tasks without help?
- [ ] Is search finding relevant results?

---

## 9. DEPLOYMENT

### Prerequisites
- [ ] VPS or Cloud Server (AWS EC2, DigitalOcean, Linode, etc.)
- [ ] Domain name (optional but recommended)
- [ ] SSL certificate (Let's Encrypt - free)

### Backend Deployment

#### Server Setup (Ubuntu 20.04/22.04)
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js (v18 or later)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Install MongoDB
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
sudo apt update
sudo apt install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod

# Install Nginx
sudo apt install -y nginx

# Install PM2 (process manager)
sudo npm install -g pm2

# Install Git
sudo apt install -y git
```

#### Deploy Backend
```bash
# Clone repository
cd /var/www
sudo git clone <your-repo-url> dms-backend
cd dms-backend

# Install dependencies
npm install --production

# Create .env file
sudo nano .env
# Paste production environment variables

# Start with PM2
pm2 start server.js --name dms-backend
pm2 save
pm2 startup
```

**Checklist:**
- [ ] Setup Ubuntu server
- [ ] Install Node.js, MongoDB, Nginx
- [ ] Clone backend repository
- [ ] Configure production .env file
- [ ] Install dependencies
- [ ] Start application with PM2
- [ ] Configure PM2 to start on boot

#### Configure Nginx (Reverse Proxy)
```bash
sudo nano /etc/nginx/sites-available/dms-backend
```

```nginx
server {
    listen 80;
    server_name api.yourdomain.com;  # or server IP
    
    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        # Increase timeout for file uploads
        proxy_read_timeout 300;
        proxy_connect_timeout 300;
        proxy_send_timeout 300;
        
        # Increase max body size for file uploads
        client_max_body_size 50M;
    }
}
```

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/dms-backend /etc/nginx/sites-enabled/

# Test and reload Nginx
sudo nginx -t
sudo systemctl reload nginx
```

**Checklist:**
- [ ] Configure Nginx reverse proxy
- [ ] Set client_max_body_size for file uploads
- [ ] Test Nginx configuration
- [ ] Reload Nginx

#### Setup SSL with Let's Encrypt
```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Get SSL certificate
sudo certbot --nginx -d api.yourdomain.com

# Auto-renewal is configured by default
# Test renewal
sudo certbot renew --dry-run
```

**Checklist:**
- [ ] Install Certbot
- [ ] Generate SSL certificate
- [ ] Configure auto-renewal
- [ ] Test SSL (https://api.yourdomain.com)

#### Configure Firewall
```bash
# Allow SSH, HTTP, HTTPS
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

**Checklist:**
- [ ] Configure UFW firewall
- [ ] Allow necessary ports
- [ ] Enable firewall

### Frontend Deployment

#### Build React App
```bash
# On local machine or CI/CD
cd dms-frontend

# Update API URL in .env
REACT_APP_API_URL=https://api.yourdomain.com/api

# Build for production
npm run build
```

**Checklist:**
- [ ] Update .env with production API URL
- [ ] Build React app

#### Deploy Frontend (Option 1: Nginx)
```bash
# Copy build folder to server
scp -r build/* user@server:/var/www/dms-frontend/

# Or clone and build on server
cd /var/www
sudo git clone <frontend-repo-url> dms-frontend
cd dms-frontend
npm install
npm run build
```

**Configure Nginx for Frontend:**
```bash
sudo nano /etc/nginx/sites-available/dms-frontend
```

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    
    root /var/www/dms-frontend/build;
    index index.html;
    
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    # Cache static assets
    location /static/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
}
```

```bash
sudo ln -s /etc/nginx/sites-available/dms-frontend /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx

# Setup SSL
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

**Checklist:**
- [ ] Deploy frontend build to server
- [ ] Configure Nginx for frontend
- [ ] Setup SSL for frontend domain
- [ ] Test frontend access

#### Deploy Frontend (Option 2: Vercel/Netlify - Easier)
```bash
# Install Vercel CLI
npm install -g vercel

# Deploy
vercel --prod

# Or use Netlify
npm install -g netlify-cli
netlify deploy --prod
```

**Checklist:**
- [ ] Connect repository to Vercel/Netlify
- [ ] Configure build settings
- [ ] Add environment variables
- [ ] Deploy

### Database Setup

#### MongoDB Atlas (Cloud - Recommended)
- [ ] Create MongoDB Atlas account
- [ ] Create cluster (free tier available)
- [ ] Create database user
- [ ] Whitelist server IP
- [ ] Get connection string
- [ ] Update MONGODB_URI in backend .env
- [ ] Configure automated backups

#### Self-Hosted MongoDB
- [ ] Already installed on server
- [ ] Create database
  ```bash
  mongosh
  use dms_nafscob
  ```
- [ ] Create admin user
  ```javascript
  db.createUser({
    user: "dmsadmin",
    pwd: "strong_password",
    roles: [{ role: "readWrite", db: "dms_nafscob" }]
  })
  ```
- [ ] Enable authentication in mongod.conf
- [ ] Update connection string in .env

**Checklist:**
- [ ] Setup MongoDB (Atlas or self-hosted)
- [ ] Create database and user
- [ ] Configure connection string
- [ ] Test connection

### AWS S3 Setup

- [ ] Create AWS account
- [ ] Create S3 bucket (e.g., nafscob-dms-prod)
- [ ] Configure bucket settings:
  - Block all public access: ON
  - Versioning: Enabled
  - Encryption: AES-256
- [ ] Create IAM user for app
- [ ] Attach S3 policy to user
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "s3:PutObject",
          "s3:GetObject",
          "s3:DeleteObject",
          "s3:ListBucket"
        ],
        "Resource": [
          "arn:aws:s3:::nafscob-dms-prod/*",
          "arn:aws:s3:::nafscob-dms-prod"
        ]
      }
    ]
  }
  ```
- [ ] Get Access Key ID and Secret
- [ ] Update AWS credentials in backend .env
- [ ] Test file upload

**Checklist:**
- [ ] Create S3 bucket
- [ ] Configure bucket settings
- [ ] Create IAM user with S3 access
- [ ] Update credentials in backend
- [ ] Test upload/download

### Seed Initial Data

```bash
# On server
cd /var/www/dms-backend
node seeds/seedData.js
```

**Checklist:**
- [ ] Run seed script to create:
  - Initial departments
  - Superadmin user
  - Root folders for each department
- [ ] Verify seeded data in database

### Final Checks

- [ ] Test API endpoints (Postman/curl)
- [ ] Test frontend login
- [ ] Test file upload
- [ ] Test file download
- [ ] Test search
- [ ] Test permissions
- [ ] Check error logging
- [ ] Verify SSL certificates
- [ ] Test from different devices/browsers
- [ ] Performance test (response times)

---

## 10. SEED DATA & INITIAL SETUP

### Seed Script

Create `seeds/seedData.js`:

```javascript
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');
require('dotenv').config();

const Department = require('../models/Department');
const User = require('../models/User');
const Folder = require('../models/Folder');

const seedData = async () => {
  try {
    await mongoose.connect(process.env.MONGODB_URI);
    
    console.log('Connected to MongoDB. Starting seed...');
    
    // Clear existing data
    await Department.deleteMany({});
    await User.deleteMany({});
    await Folder.deleteMany({});
    
    // 1. Create Departments
    const departments = await Department.insertMany([
      { name: 'Human Resources', code: 'HR', description: 'HR Department', isActive: true },
      { name: 'Finance', code: 'FIN', description: 'Finance Department', isActive: true },
      { name: 'Information Technology', code: 'IT', description: 'IT Department', isActive: true },
      { name: 'Operations', code: 'OPS', description: 'Operations Department', isActive: true },
      { name: 'Legal', code: 'LEG', description: 'Legal Department', isActive: true },
      { name: 'Administration', code: 'ADMIN', description: 'Administration', isActive: true }
    ]);
    
    console.log('✓ Departments created');
    
    // 2. Create Superadmin
    const superadmin = await User.create({
      username: 'superadmin',
      email: 'superadmin@nafscob.org',
      password: 'Admin@123', // Will be hashed by pre-save hook
      role: 'superadmin',
      isActive: true
    });
    
    console.log('✓ Superadmin created');
    console.log(`   Username: superadmin`);
    console.log(`   Password: Admin@123`);
    
    //# Document Management System - Complete Implementation Guide

## Stack: Node.js + Express.js + MongoDB + React.js

---

## 📋 TABLE OF CONTENTS

1. [Project Setup & Architecture](#1-project-setup--architecture)
2. [Database Schema Design](#2-database-schema-design)
3. [Backend API Endpoints](#3-backend-api-endpoints)
4. [Backend Middleware](#4-backend-middleware-implementation)
5. [Frontend Implementation](#5-frontend-implementation)
6. [Key Features Implementation](#6-key-features-implementation)
7. [Non-Functional Requirements](#7-non-functional-requirements)
8. [Testing](#8-testing)
9. [Deployment](#9-deployment)
10. [Seed Data & Initial Setup](#10-seed-data--initial-setup)
11. [Development Best Practices](#11-development-best-practices)

---

## 1. PROJECT SETUP & ARCHITECTURE

### Notifications APIs

```javascript
// routes/notifications.js

GET    /api/notifications           // Get user notifications
GET    /api/notifications/unread    // Get unread count
PATCH  /api/notifications/:id/read  // Mark as read
PATCH  /api/notifications/read-all  // Mark all as read
DELETE /api/notifications/:id       // Delete notification
DELETE /api/notifications/clear-all // Clear all read notifications
```

**Implementation Checklist:**
- [ ] GET `/api/notifications` - Get user notifications (paginated)
- [ ] GET `/api/notifications/unread` - Get unread count
- [ ] PATCH `/api/notifications/:id/read` - Mark as read
- [ ] PATCH `/api/notifications/read-all` - Mark all as read
- [ ] DELETE `/api/notifications/:id` - Delete notification
- [ ] DELETE `/api/notifications/clear-all` - Clear all

### Dashboard/Statistics APIs

```javascript
// routes/dashboard.js

GET    /api/dashboard/stats         // Overall statistics
GET    /api/dashboard/recent-activity // Recent system activity
GET    /api/dashboard/my-documents  // User's recent documents
GET    /api/dashboard/storage       // Storage usage by department
GET    /api/dashboard/user-activity // User activity summary
```

**Implementation Checklist:**
- [ ] GET `/api/dashboard/stats` - Get dashboard statistics
- [ ] GET `/api/dashboard/recent-activity` - Get recent activity
- [ ] GET `/api/dashboard/my-documents` - Get user's documents
- [ ] GET `/api/dashboard/storage` - Storage usage stats
- [ ] GET `/api/dashboard/user-activity` - User activity stats

---

## 4. BACKEND MIDDLEWARE IMPLEMENTATION

### Authentication Middleware

```javascript
// middleware/auth.js

const jwt = require('jsonwebtoken');
const User = require('../models/User');

exports.protect = async (req, res, next) => {
  let token;
  
  if (req.headers.authorization && req.headers.authorization.startsWith('Bearer')) {
    token = req.headers.authorization.split(' ')[1];
  }
  
  if (!token) {
    return res.status(401).json({ success: false, message: 'Not authorized' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = await User.findById(decoded.id).populate('department');
    
    if (!req.user || !req.user.isActive) {
      return res.status(401).json({ success: false, message: 'User not found or inactive' });
    }
    
    next();
  } catch (error) {
    return res.status(401).json({ success: false, message: 'Invalid token' });
  }
};
```

**Implementation Checklist:**
- [ ] Create JWT verification middleware
- [ ] Check user active status
- [ ] Attach user to request object
- [ ] Handle token expiration

### Authorization Middleware

```javascript
// middleware/roleCheck.js

exports.authorize = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ 
        success: false, 
        message: 'Not authorized to access this resource' 
      });
    }
    next();
  };
};
```

**Implementation Checklist:**
- [ ] Create role-based authorization middleware
- [ ] Support multiple roles check
- [ ] Return 403 for unauthorized access

### Permission Check Middleware

```javascript
// middleware/permissions.js

const Folder = require('../models/Folder');
const Document = require('../models/Document');

exports.checkFolderPermission = (requiredAccess) => {
  return async (req, res, next) => {
    try {
      const folder = await Folder.findById(req.params.id || req.body.folder);
      
      if (!folder) {
        return res.status(404).json({ success: false, message: 'Folder not found' });
      }
      
      // Superadmin has all access
      if (req.user.role === 'superadmin') {
        req.folder = folder;
        return next();
      }
      
      // Check user-specific permissions
      const userPermission = folder.permissions.find(
        p => p.user && p.user.toString() === req.user._id.toString()
      );
      
      // Check role-based permissions
      const rolePermission = folder.permissions.find(
        p => p.role === req.user.role
      );
      
      const permission = userPermission || rolePermission;
      
      if (!permission || !permission.access.includes(requiredAccess)) {
        return res.status(403).json({ 
          success: false, 
          message: 'Insufficient permissions' 
        });
      }
      
      req.folder = folder;
      next();
    } catch (error) {
      return res.status(500).json({ success: false, message: error.message });
    }
  };
};

exports.checkDocumentPermission = (requiredAccess) => {
  return async (req, res, next) => {
    try {
      const document = await Document.findById(req.params.id);
      
      if (!document) {
        return res.status(404).json({ success: false, message: 'Document not found' });
      }
      
      // Superadmin has all access
      if (req.user.role === 'superadmin') {
        req.document = document;
        return next();
      }
      
      // Check document-level permissions first
      const userPermission = document.permissions.find(
        p => p.user && p.user.toString() === req.user._id.toString()
      );
      
      const rolePermission = document.permissions.find(
        p => p.role === req.user.role
      );
      
      let permission = userPermission || rolePermission;
      
      // If no document-level permission, inherit from folder
      if (!permission) {
        const folder = await Folder.findById(document.folder);
        const folderUserPerm = folder.permissions.find(
          p => p.user && p.user.toString() === req.user._id.toString()
        );
        const folderRolePerm = folder.permissions.find(
          p => p.role === req.user.role
        );
        permission = folderUserPerm || folderRolePerm;
      }
      
      if (!permission || !permission.access.includes(requiredAccess)) {
        return res.status(403).json({ 
          success: false, 
          message: 'Insufficient permissions' 
        });
      }
      
      req.document = document;
      next();
    } catch (error) {
      return res.status(500).json({ success: false, message: error.message });
    }
  };
};
```

**Implementation Checklist:**
- [ ] Create folder permission check middleware
- [ ] Create document permission check middleware
- [ ] Implement permission inheritance (folder → document)
- [ ] Handle superadmin access
- [ ] Check both user-specific and role-based permissions

### Audit Logging Middleware

```javascript
// middleware/auditLog.js

const AuditLog = require('../models/AuditLog');

exports.logAction = (action, resourceType) => {
  return async (req, res, next) => {
    // Store original send function
    const originalSend = res.send;
    
    res.send = function(data) {
      // Only log successful operations (2xx status codes)
      if (res.statusCode >= 200 && res.statusCode < 300) {
        const logData = {
          user: req.user._id,
          action,
          resourceType,
          resourceId: req.params.id || req.body._id,
          resourceName: req.body.title || req.body.name,
          department: req.user.department,
          ipAddress: req.ip || req.connection.remoteAddress,
          userAgent: req.get('user-agent')
        };
        
        AuditLog.create(logData).catch(err => console.error('Audit log error:', err));
      }
      
      originalSend.call(this, data);
    };
    
    next();
  };
};
```

**Implementation Checklist:**
- [ ] Create audit logging middleware
- [ ] Capture user, action, resource details
- [ ] Log IP address and user agent
- [ ] Only log successful operations
- [ ] Handle async logging without blocking response

### File Upload Middleware

```javascript
// middleware/upload.js

const multer = require('multer');
const { S3Client } = require('@aws-sdk/client-s3');
const multerS3 = require('multer-s3');

const s3Client = new S3Client({
  region: process.env.AWS_REGION,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
  }
});

const fileFilter = (req, file, cb) => {
  const allowedTypes = ['application/pdf', 'application/msword', 
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
    'application/vnd.ms-excel',
    'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
    'image/jpeg', 'image/jpg', 'image/png', 'application/zip'];
  
  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error('Invalid file type. Only PDF, DOCX, XLSX, JPG, PNG, ZIP allowed'), false);
  }
};

const upload = multer({
  storage: multerS3({
    s3: s3Client,
    bucket: process.env.AWS_BUCKET_NAME,
    metadata: (req, file, cb) => {
      cb(null, { fieldName: file.fieldname });
    },
    key: (req, file, cb) => {
      const uniqueName = `${Date.now()}-${Math.round(Math.random() * 1E9)}-${file.originalname}`;
      cb(null, `documents/${uniqueName}`);
    }
  }),
  fileFilter,
  limits: {
    fileSize: 50 * 1024 * 1024 // 50MB max
  }
});

module.exports = upload;
```

**Implementation Checklist:**
- [ ] Setup Multer with S3 storage
- [ ] Validate file types (PDF, DOCX, XLSX, JPG, PNG, ZIP)
- [ ] Set file size limit (50MB)
- [ ] Generate unique file names
- [ ] Store files in organized S3 structure

### Error Handler Middleware

```javascript
// middleware/errorHandler.js

exports.errorHandler = (err, req, res, next) => {
  let error = { ...err };
  error.message = err.message;
  
  // Log error for debugging
  console.error(err);
  
  // Mongoose bad ObjectId
  if (err.name === 'CastError') {
    error.message = 'Resource not found';
    error.statusCode = 404;
  }
  
  // Mongoose duplicate key
  if (err.code === 11000) {
    error.message = 'Duplicate field value entered';
    error.statusCode = 400;
  }
  
  // Mongoose validation error
  if (err.name === 'ValidationError') {
    error.message = Object.values(err.errors).map(e => e.message).join(', ');
    error.statusCode = 400;
  }
  
  // JWT errors
  if (err.name === 'JsonWebTokenError') {
    error.message = 'Invalid token';
    error.statusCode = 401;
  }
  
  if (err.name === 'TokenExpiredError') {
    error.message = 'Token expired';
    error.statusCode = 401;
  }
  
  res.status(error.statusCode || 500).json({
    success: false,
    message: error.message || 'Server Error'
  });
};
```

**Implementation Checklist:**
- [ ] Create global error handler
- [ ] Handle Mongoose errors (CastError, ValidationError, Duplicate)
- [ ] Handle JWT errors
- [ ] Log errors for debugging
- [ ] Return consistent error responses

---

## 5. FRONTEND IMPLEMENTATION

### 5.1 Authentication Pages

#### Login Page
**Components to Build:**
- [ ] Login form with username/email and password
- [ ] Form validation (required fields, email format)
- [ ] "Remember me" checkbox
- [ ] "Forgot password" link
- [ ] Show/hide password toggle
- [ ] Loading state during login
- [ ] Error message display
- [ ] Redirect to dashboard on success

#### Forgot Password Page
- [ ] Email input form
- [ ] Send reset link functionality
- [ ] Success message display
- [ ] Back to login link

#### Reset Password Page
- [ ] New password input (with confirmation)
- [ ] Password strength indicator
- [ ] Token validation from URL
- [ ] Success message and redirect

### 5.2 Dashboard Layout

#### Sidebar Navigation
**Menu Items:**
- [ ] Dashboard (home icon)
- [ ] My Documents (document icon)
- [ ] Folders (folder icon)
- [ ] Users (people icon) - Admin/Superadmin only
- [ ] Audit Logs (history icon) - Admin/Superadmin only
- [ ] Settings (gear icon)

**Features:**
- [ ] Collapsible sidebar
- [ ] Active menu highlighting
- [ ] Role-based menu visibility
- [ ] User info at bottom (avatar, name, role)

#### Top Navbar
- [ ] Logo and title
- [ ] Global search bar
- [ ] Notifications bell icon with badge (unread count)
- [ ] User profile dropdown
  - View Profile
  - Settings
  - Change Password
  - Logout

#### Dashboard Home Page
**Widgets:**
- [ ] Welcome message with user name
- [ ] Statistics cards:
  - Total Documents
  - My Documents
  - Storage Used
  - Recent Uploads (this month)
- [ ] Recent Activity Feed (last 10 actions)
- [ ] Quick Access to Frequently Used Folders
- [ ] Storage Usage Chart (by department) - Admin only

### 5.3 User Management (Admin/Superadmin)

#### Users List Page
**Features:**
- [ ] Data table with columns:
  - Username
  - Email
  - Role
  - Department
  - Status (Active/Inactive)
  - Last Login
  - Actions (Edit, Delete, Toggle Status)
- [ ] Search by username or email
- [ ] Filter by role, department, status
- [ ] Pagination (20 users per page)
- [ ] Add User button (top right)
- [ ] Bulk actions (optional)

#### Add/Edit User Modal
**Form Fields:**
- [ ] Username (required, unique)
- [ ] Email (required, unique, validated)
- [ ] Password (required for new, optional for edit)
- [ ] Confirm Password
- [ ] Role dropdown (Superadmin, Admin, Team Member, Member Bank)
- [ ] Department dropdown
- [ ] Active status toggle
- [ ] Form validation
- [ ] Submit and Cancel buttons

#### User Profile Page
- [ ] View own profile details
- [ ] Edit profile button
- [ ] Change password section
- [ ] Last login information
- [ ] Activity summary

### 5.4 Department Management (Superadmin)

#### Departments List Page
- [ ] Table with: Name, Code, Head, Status, Actions
- [ ] Add Department button
- [ ] Edit/Delete actions
- [ ] Toggle active status

#### Add/Edit Department Modal
- [ ] Name field
- [ ] Code field (uppercase, unique)
- [ ] Description textarea
- [ ] Department Head dropdown (users)
- [ ] Active status toggle

### 5.5 Folder Management

#### Folder Tree View
**Features:**
- [ ] Tree structure display (collapsible nodes)
- [ ] Icons for folders (different icons for levels)
- [ ] Folder name with document count
- [ ] Context menu on right-click:
  - Create Subfolder
  - Rename
  - Move
  - Set Permissions
  - Delete
  - View Details
- [ ] Click to view folder contents
- [ ] Breadcrumb navigation
- [ ] Search folders by name
- [ ] Filter by department

#### Create Folder Modal
**Form:**
- [ ] Folder name input
- [ ] Parent folder selector (tree dropdown)
- [ ] Department dropdown
- [ ] Auto-calculate path preview
- [ ] Set initial permissions
- [ ] Create button

#### Folder Permissions Modal
**Features:**
- [ ] List of users (searchable)
- [ ] List of roles
- [ ] Permission checkboxes for each:
  - View
  - Upload
  - Edit
  - Delete
  - Download
- [ ] Apply to subfolders checkbox
- [ ] Save button

#### Folder Details Page
**Sections:**
- [ ] Folder information (name, path, created by, created date)
- [ ] Subfolders list
- [ ] Documents list (with filters)
- [ ] Upload document button
- [ ] Folder statistics (total docs, total size)
- [ ] Edit folder button
- [ ] Delete folder button (with confirmation)

### 5.6 Document Management

#### Document List View
**Display Options:**
- [ ] Toggle between grid and table view
- [ ] Grid View: Cards with thumbnail, title, date, size
- [ ] Table View: Columns - Title, Type, Size, Uploaded By, Date, Actions

**Filters:**
- [ ] Sidebar filter panel:
  - Date range picker (upload date)
  - Uploader dropdown (all users)
  - File type checkboxes (PDF, DOCX, XLSX, Images, ZIP)
  - Tags multi-select
  - Department filter
  - Folder filter
- [ ] Clear all filters button
- [ ] Active filters display (chips)

**Actions:**
- [ ] Search bar with autocomplete
- [ ] Sort dropdown (Name, Date, Size)
- [ ] Upload button
- [ ] Bulk select checkbox
- [ ] Bulk actions: Download, Delete

#### Document Upload Modal
**Features:**
- [ ] Drag-and-drop zone (or click to browse)
- [ ] Multiple file upload support
- [ ] File preview list with:
  - File name
  - File size
  - Remove button
- [ ] Metadata form:
  - Title (auto-filled from filename, editable)
  - Tags input (comma-separated or chips)
  - Folder selector
- [ ] Upload progress bar (per file)
- [ ] Cancel upload button
- [ ] Success message with link to document

#### Document Detail Page
**Sections:**
- [ ] Document preview:
  - PDF viewer (with zoom, page navigation)
  - Image viewer
  - Download for other types
- [ ] Document information:
  - Title
  - File type and size
  - Uploaded by and date
  - Folder location (breadcrumb)
  - Tags (clickable)
  - Version number
- [ ] Action buttons:
  - Download
  - Edit metadata
  - Replace (upload new version)
  - Move to different folder
  - Delete (with confirmation)
  - Share (copy link)
- [ ] Version History section:
  - List of all versions
  - Download specific version
  - Restore version button
  - View changes description
- [ ] Permissions section:
  - List of users/roles with access
  - Edit permissions button
- [ ] Activity/Audit trail:
  - Who viewed, when
  - Who downloaded, when
  - Who edited, when

#### Document Viewer Component
**Features:**
- [ ] PDF viewer (using react-pdf or pdf.js)
- [ ] Zoom controls (+, -, fit)
- [ ] Page navigation (prev, next, page number)
- [ ] Fullscreen mode
- [ ] Download button
- [ ] Print button (optional)

#### Version History Modal
**Display:**
- [ ] Table with columns:
  - Version number
  - Uploaded by
  - Upload date
  - File size
  - Changes description
  - Actions (Download, Restore)
- [ ] Confirmation dialog for restore action
- [ ] Close button

### 5.7 Search & Filter

#### Advanced Search Page
**Features:**
- [ ] Large search input with placeholder
- [ ] Search button
- [ ] Advanced filters panel:
  - Date range picker (From - To)
  - Department dropdown
  - Folder selector
  - File type checkboxes
  - Tags multi-select (autocomplete)
  - Uploader dropdown
- [ ] Search results:
  - Grid/Table view toggle
  - Result count
  - Sort options (Relevance, Date, Name, Size)
  - Pagination
  - Result highlighting (search term)
- [ ] Save search button (optional)
- [ ] Export results button (CSV)

#### Global Search (Navbar)
**Features:**
- [ ] Search input with icon
- [ ] Auto-suggest dropdown (appears on typing)
- [ ] Show recent searches
- [ ] Show top results (max 5)
- [ ] "See all results" link to advanced search
- [ ] Keyboard shortcuts (Ctrl+K to focus)

### 5.8 Audit Logs (Admin Only)

#### Audit Logs Page
**Features:**
- [ ] Table with columns:
  - Timestamp
  - User
  - Action
  - Resource Type
  - Resource Name
  - Department
  - IP Address
- [ ] Filters:
  - Date range picker
  - User dropdown
  - Action dropdown (Upload, Download, View, Edit, Delete, Login, Logout)
  - Resource type dropdown (Document, Folder, User)
  - Department dropdown
- [ ] Search by resource name
- [ ] Pagination (50 logs per page)
- [ ] Export to CSV button
- [ ] Refresh button
- [ ] Auto-refresh toggle (every 30 seconds)

### 5.9 Notifications

#### Notifications Dropdown (Navbar)
**Features:**
- [ ] Bell icon with unread count badge
- [ ] Dropdown panel (max-height with scroll)
- [ ] List of notifications:
  - Icon based on type
  - Title and message
  - Timestamp (relative: "2 hours ago")
  - Read/Unread indicator
  - Click to mark as read and navigate
- [ ] "Mark all as read" button
- [ ] "View all notifications" link
- [ ] Empty state message

#### Notifications Page
**Features:**
- [ ] List of all notifications (paginated)
- [ ] Filter by type, read/unread
- [ ] Mark as read/unread toggle
- [ ] Delete notification button
- [ ] Clear all read notifications button
- [ ] Group by date (Today, Yesterday, This Week, Older)

### 5.10 Settings Page

#### Profile Settings Tab
- [ ] View and edit profile fields:
  - Username (read-only)
  - Email
  - Department (read-only for non-admin)
- [ ] Save changes button

#### Change Password Tab
- [ ] Current password input
- [ ] New password input
- [ ] Confirm new password input
- [ ] Password strength indicator
- [ ] Change password button

#### System Settings Tab (Superadmin only)
- [ ] File upload settings:
  - Max file size slider
  - Allowed file types checkboxes
- [ ] Session settings:
  - Session timeout duration
  - Auto-logout on inactivity
- [ ] Storage settings:
  - Storage quota per department
  - Low storage alerts
- [ ] Backup settings:
  - Backup frequency
  - Retention period
- [ ] Save settings button

---

## 6. KEY FEATURES IMPLEMENTATION

### 6.1 Role-Based Access Control (RBAC)

**Implementation Steps:**
- [ ] Define permission constants
  ```javascript
  const PERMISSIONS = {
    VIEW: 'view',
    UPLOAD: 'upload',
    EDIT: 'edit',
    DELETE: 'delete',
    DOWNLOAD: 'download'
  };
  
  const ROLES = {
    SUPERADMIN: 'superadmin',
    ADMIN: 'admin',
    TEAM_MEMBER: 'team_member',
    MEMBER_BANK: 'member_bank'
  };
  ```
- [ ] Create permission check utility functions
  ```javascript
  const hasPermission = (user, resource, requiredPermission) => {
    // Check if superadmin
    if (user.role === ROLES.SUPERADMIN) return true;
    
    // Check user-specific permissions
    const userPerm = resource.permissions.find(p => 
      p.user && p.user.toString() === user._id.toString()
    );
    
    // Check role-based permissions
    const rolePerm = resource.permissions.find(p => 
      p.role === user.role
    );
    
    const permission = userPerm || rolePerm;
    return permission && permission.access.includes(requiredPermission);
  };
  ```
- [ ] Apply permissions in API endpoints
- [ ] Apply permissions in frontend (hide/show UI elements)
  ```javascript
  {hasPermission(user, folder, 'edit') && (
    <Button onClick={handleEdit}>Edit</Button>
  )}
  ```
- [ ] Implement permission inheritance (Folder → Subfolder → Document)
- [ ] Allow permission override at each level

### 6.2 Version Control

**Implementation Steps:**
- [ ] Auto-create DocumentVersion on file replacement
  ```javascript
  // When replacing document
  const newVersion = await DocumentVersion.create({
    document: document._id,
    version: document.version,
    fileUrl: document.fileUrl,
    fileKey: document.fileKey,
    fileSize: document.fileSize,
    uploadedBy: req.user._id,
    changes: req.body.changes || 'File replaced'
  });
  
  // Update document
  document.version += 1;
  document.fileUrl = newFileUrl;
  document.fileKey = newFileKey;
  await document.save();
  ```
- [ ] Store all versions with metadata
- [ ] Implement version restore functionality
  ```javascript
  const restoreVersion = async (documentId, versionId) => {
    const version = await DocumentVersion.findById(versionId);
    const document = await Document.findById(documentId);
    
    // Create version from current document
    await DocumentVersion.create({
      document: document._id,
      version: document.version,
      fileUrl: document.fileUrl,
      fileKey: document.fileKey,
      fileSize: document.fileSize,
      uploadedBy: req.user._id,
      changes: 'Backup before restore'
    });
    
    // Restore old version
    document.version += 1;
    document.fileUrl = version.fileUrl;
    document.fileKey = version.fileKey;
    document.fileSize = version.fileSize;
    await document.save();
  };
  ```
- [ ] Display version history in UI
- [ ] Allow downloading specific versions
- [ ] Show version comparison (optional)

### 6.3 Advanced Search

**Implementation Steps:**
- [ ] Create MongoDB text indexes
  ```javascript
  documentSchema.index({ title: 'text', tags: 'text' });
  ```
- [ ] Implement multi-field search query
  ```javascript
  const searchQuery = {
    $and: []
  };
  
  if (title) {
    searchQuery.$and.push({
      $text: { $search: title }
    });
  }
  
  if (tags && tags.length > 0) {
    searchQuery.$and.push({
      tags: { $in: tags }
    });
  }
  
  if (folder) {
    searchQuery.$and.push({ folder });
  }
  
  if (fileType && fileType.length > 0) {
    searchQuery.$and.push({
      fileType: { $in: fileType }
    });
  }
  
  if (uploadDateFrom || uploadDateTo) {
    const dateQuery = {};
    if (uploadDateFrom) dateQuery.$gte = new Date(uploadDateFrom);
    if (uploadDateTo) dateQuery.$lte = new Date(uploadDateTo);
    searchQuery.$and.push({ createdAt: dateQuery });
  }
  
  const documents = await Document.find(
    searchQuery.$and.length > 0 ? searchQuery : {}
  )
  .populate('uploadedBy', 'username')
  .populate('folder', 'name path')
  .sort({ [sortBy]: sortOrder })
  .skip((page - 1) * limit)
  .limit(limit);
  ```
- [ ] Implement autocomplete/suggestions
  ```javascript
  // Get unique tags for autocomplete
  const tags = await Document.distinct('tags', {
    tags: { $regex: query, $options: 'i' }
  });
  
  // Get matching titles
  const titles = await Document.find({
    title: { $regex: query, $options: 'i' }
  }, 'title').limit(5);
  ```
- [ ] Add pagination
- [ ] Highlight search terms in results (frontend)
- [ ] Implement search filters
- [ ] Cache recent searches (optional)

### 6.4 Audit Logging

**Implementation Steps:**
- [ ] Create audit log on every critical action
  ```javascript
  // In controller or middleware
  await AuditLog.create({
    user: req.user._id,
    action: 'upload',
    resourceType: 'document',
    resourceId: document._id,
    resourceName: document.title,
    department: req.user.department,
    ipAddress: req.ip,
    userAgent: req.get('user-agent'),
    details: {
      folder: document.folder,
      fileSize: document.fileSize
    }
  });
  ```
- [ ] Log these actions:
  - File uploads
  - File downloads
  - File views (optional, can be heavy)
  - File edits/renames
  - File deletions
  - File moves
  - User logins/logouts
  - Permission changes
  - Folder creation/deletion
- [ ] Capture metadata: User, Timestamp, IP, User Agent
- [ ] Make logs searchable and filterable
- [ ] Implement log retention policy (optional: TTL index)
- [ ] Export logs to CSV
- [ ] Display logs in admin dashboard

### 6.5 Notification System

**Implementation Steps:**
- [ ] Create notification on:
  - **Document upload in accessible folder:**
    ```javascript
    const foldersWithAccess = await Folder.find({
      _id: document.folder,
      'permissions.user': { $exists: true }
    }).populate('permissions.user');
    
    const usersToNotify = foldersWithAccess[0].permissions
      .filter(p => p.user && p.access.includes('view'))
      .map(p => p.user._id);
    
    const notifications = usersToNotify.map(userId => ({
      user: userId,
      type: 'document_upload',
      title: 'New Document Uploaded',
      message: `${req.user.username} uploaded "${document.title}"`,
      relatedDocument: document._id,
      relatedFolder: document.folder
    }));
    
    await Notification.insertMany(notifications);
    ```
  - **Access granted to folder/document:**
    ```javascript
    await Notification.create({
      user: grantedUserId,
      type: 'access_granted',
      title: 'Access Granted',
      message: `You have been granted access to "${resource.name}"`,
      relatedFolder: folderId
    });
    ```
  - **New version uploaded:**
    ```javascript
    await Notification.create({
      user: document.uploadedBy,
      type: 'version_update',
      title: 'Document Updated',
      message: `A new version of "${document.title}" has been uploaded`,
      relatedDocument: document._id
    });
    ```
- [ ] Implement real-time updates:
  - **Polling approach:** Frontend polls `/api/notifications/unread` every 30 seconds
  - **WebSocket approach (advanced):** Use Socket.io for real-time push
- [ ] Email notifications (optional):
  - Integrate with email service (SendGrid, AWS SES)
  - User preference toggle for email notifications
  - Daily digest Backend Setup
- [ ] Initialize Node.js project with Express.js
  ```bash
  mkdir dms-backend && cd dms-backend
  npm init -y
  npm install express mongoose dotenv cors bcryptjs jsonwebtoken
  npm install multer multer-s3 @aws-sdk/client-s3
  npm install express-validator helmet morgan
  npm install --save-dev nodemon
  ```
- [ ] Setup MongoDB connection (local or Atlas)
- [ ] Configure environment variables (.env file)
  ```env
  PORT=5000
  MONGODB_URI=mongodb://localhost:27017/dms_nafscob
  JWT_SECRET=your-secret-key-change-in-production
  JWT_EXPIRE=7d
  AWS_ACCESS_KEY_ID=your-aws-key
  AWS_SECRET_ACCESS_KEY=your-aws-secret
  AWS_BUCKET_NAME=nafscob-dms
  AWS_REGION=ap-south-1
  NODE_ENV=development
  ```
- [ ] Setup JWT authentication middleware
- [ ] Configure CORS for frontend communication
- [ ] Setup file upload middleware (Multer)
- [ ] Configure AWS S3 or Azure Blob Storage SDK
- [ ] Setup password hashing (bcrypt)
- [ ] Configure express-validator for input validation
- [ ] Setup error handling middleware
- [ ] Setup logging (Morgan)
- [ ] Setup helmet for security headers

### Frontend Setup
- [ ] Initialize React.js project
  ```bash
  npx create-react-app dms-frontend
  cd dms-frontend
  npm install axios react-router-dom
  npm install @mui/material @emotion/react @emotion/styled
  npm install @mui/icons-material
  npm install react-hook-form yup @hookform/resolvers
  npm install react-toastify
  npm install react-dropzone
  npm install date-fns
  ```
- [ ] Setup React Router for navigation
- [ ] Configure Axios for API calls (create axios instance with interceptors)
- [ ] Setup state management (Context API or Redux Toolkit)
- [ ] Install UI library (Material-UI recommended)
- [ ] Setup protected route components
- [ ] Configure environment variables (.env)
  ```env
  REACT_APP_API_URL=http://localhost:5000/api
  ```
- [ ] Setup form validation library (React Hook Form + Yup)
- [ ] Configure toast notifications (react-toastify)

### Project Structure

**Backend:**
```
dms-backend/
├── config/
│   ├── db.js              # MongoDB connection
│   └── aws.js             # AWS S3 configuration
├── models/
│   ├── User.js
│   ├── Department.js
│   ├── Folder.js
│   ├── Document.js
│   ├── DocumentVersion.js
│   ├── AuditLog.js
│   └── Notification.js
├── middleware/
│   ├── auth.js            # JWT verification
│   ├── roleCheck.js       # Role-based access
│   ├── permissions.js     # Folder/document permissions
│   ├── upload.js          # File upload handling
│   ├── auditLog.js        # Auto-logging
│   └── errorHandler.js    # Global error handler
├── routes/
│   ├── auth.js
│   ├── users.js
│   ├── departments.js
│   ├── folders.js
│   ├── documents.js
│   ├── search.js
│   ├── auditLogs.js
│   └── notifications.js
├── controllers/
│   ├── authController.js
│   ├── userController.js
│   ├── departmentController.js
│   ├── folderController.js
│   ├── documentController.js
│   ├── searchController.js
│   ├── auditLogController.js
│   └── notificationController.js
├── utils/
│   ├── s3Upload.js        # S3 upload helper
│   ├── generateToken.js   # JWT helper
│   └── logger.js          # Custom logger
├── seeds/
│   └── seedData.js        # Initial data seeding
├── .env
├── .gitignore
├── server.js              # Entry point
└── package.json
```

**Frontend:**
```
dms-frontend/
├── public/
├── src/
│   ├── api/
│   │   └── axios.js       # Axios instance
│   ├── components/
│   │   ├── common/
│   │   │   ├── Sidebar.js
│   │   │   ├── Navbar.js
│   │   │   ├── Loader.js
│   │   │   └── ConfirmDialog.js
│   │   ├── folders/
│   │   │   ├── FolderTree.js
│   │   │   ├── CreateFolder.js
│   │   │   └── FolderPermissions.js
│   │   ├── documents/
│   │   │   ├── DocumentList.js
│   │   │   ├── DocumentUpload.js
│   │   │   ├── DocumentViewer.js
│   │   │   └── VersionHistory.js
│   │   └── users/
│   │       ├── UserList.js
│   │       └── UserForm.js
│   ├── pages/
│   │   ├── auth/
│   │   │   ├── Login.js
│   │   │   └── ForgotPassword.js
│   │   ├── Dashboard.js
│   │   ├── MyDocuments.js
│   │   ├── FoldersPage.js
│   │   ├── UsersPage.js
│   │   ├── AuditLogsPage.js
│   │   └── SettingsPage.js
│   ├── context/
│   │   ├── AuthContext.js
│   │   └── NotificationContext.js
│   ├── hooks/
│   │   ├── useAuth.js
│   │   └── usePermissions.js
│   ├── utils/
│   │   ├── constants.js
│   │   └── helpers.js
│   ├── App.js
│   ├── index.js
│   └── .env
└── package.json
```

---

## 2. DATABASE SCHEMA DESIGN (MongoDB)

### Understanding the Folder Hierarchy

**IMPORTANT:** There are NO separate Category or Sub-category collections. Everything is organized using the **Folders** collection with parent-child relationships.

```
Department (folder, level 0, parentFolder: null)
  └── Category (folder, level 1, parentFolder: dept_id)
      └── Sub-category (folder, level 2, parentFolder: category_id)
          └── Sub-sub-category (folder, level 3, parentFolder: subcategory_id)
              └── Documents (stored in folders)
```

**Example Structure:**
```
HR Department (Folder)
├── Certificates (Folder = Category)
│   ├── Training Certificates (Folder = Sub-category)
│   │   ├── 2024 (Folder = Sub-sub-category)
│   │   └── 2025 (Folder = Sub-sub-category)
│   └── Audit Certificates (Folder = Sub-category)
├── Circulars (Folder = Category)
│   ├── 2024 (Folder = Sub-category)
│   └── 2025 (Folder = Sub-category)
└── Policies (Folder = Category)
```

### Collections to Create

#### 1. Departments Collection

```javascript
const departmentSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    unique: true,
    trim: true
  },
  code: {
    type: String,
    required: true,
    unique: true,
    uppercase: true,
    trim: true
  },
  description: {
    type: String,
    trim: true
  },
  head: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  },
  isActive: {
    type: Boolean,
    default: true
  },
  createdBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }
}, {
  timestamps: true
});

// Indexes
departmentSchema.index({ code: 1 });
departmentSchema.index({ isActive: 1 });
```

**Required Seed Data:**
```javascript
[
  { name: "Human Resources", code: "HR", description: "HR Department", isActive: true },
  { name: "Finance", code: "FIN", description: "Finance Department", isActive: true },
  { name: "Information Technology", code: "IT", description: "IT Department", isActive: true },
  { name: "Operations", code: "OPS", description: "Operations Department", isActive: true },
  { name: "Legal", code: "LEG", description: "Legal Department", isActive: true },
  { name: "Administration", code: "ADMIN", description: "Administration", isActive: true }
]
```

#### 2. Users Collection

```javascript
const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,
    unique: true,
    trim: true,
    lowercase: true
  },
  email: {
    type: String,
    required: true,
    unique: true,
    trim: true,
    lowercase: true
  },
  password: {
    type: String,
    required: true,
    select: false  // Don't return password in queries by default
  },
  role: {
    type: String,
    enum: ['superadmin', 'admin', 'team_member', 'member_bank'],
    default: 'team_member'
  },
  department: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Department'
  },
  isActive: {
    type: Boolean,
    default: true
  },
  mfaEnabled: {
    type: Boolean,
    default: false
  },
  mfaSecret: {
    type: String,
    select: false
  },
  lastLogin: {
    type: Date
  },
  passwordResetToken: String,
  passwordResetExpires: Date
}, {
  timestamps: true
});

// Indexes
userSchema.index({ email: 1 });
userSchema.index({ username: 1 });
userSchema.index({ department: 1 });
userSchema.index({ role: 1, isActive: 1 });

// Hash password before saving
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  this.password = await bcrypt.hash(this.password, 12);
  next();
});

// Method to compare password
userSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};
```

**Required Seed Data:**
```javascript
[
  {
    username: "superadmin",
    email: "superadmin@nafscob.org",
    password: "Admin@123", // Will be hashed
    role: "superadmin",
    isActive: true
  },
  {
    username: "hradmin",
    email: "hr@nafscob.org",
    password: "Admin@123",
    role: "admin",
    department: "HR_DEPT_ID", // Reference to HR department
    isActive: true
  }
]
```

#### 3. Folders Collection

**This is the CORE collection that handles Department → Category → Sub-category hierarchy**

```javascript
const folderSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true
  },
  parentFolder: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Folder',
    default: null  // null means root level (Department folder)
  },
  department: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Department',
    required: true
  },
  path: {
    type: String,
    required: true  // e.g., "/HR/Certificates/Training"
  },
  level: {
    type: Number,
    default: 0  // 0=Department, 1=Category, 2=Sub-category, etc.
  },
  createdBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  permissions: [{
    user: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User'
    },
    role: {
      type: String,
      enum: ['superadmin', 'admin', 'team_member', 'member_bank']
    },
    access: [{
      type: String,
      enum: ['view', 'upload', 'edit', 'delete', 'download']
    }]
  }],
  isActive: {
    type: Boolean,
    default: true
  }
}, {
  timestamps: true
});

// Indexes
folderSchema.index({ department: 1 });
folderSchema.index({ parentFolder: 1 });
folderSchema.index({ path: 1 });
folderSchema.index({ 'permissions.user': 1 });
folderSchema.index({ 'permissions.role': 1 });

// Compound index for hierarchy queries
folderSchema.index({ department: 1, parentFolder: 1 });
```

**Required Seed Data (Example for HR Department):**
```javascript
// Root folder (Department level)
{
  name: "HR Department",
  parentFolder: null,
  department: "HR_DEPT_ID",
  path: "/HR",
  level: 0,
  createdBy: "SUPERADMIN_ID",
  permissions: [
    { role: "superadmin", access: ["view", "upload", "edit", "delete", "download"] },
    { role: "admin", access: ["view", "upload", "edit", "delete", "download"] }
  ]
}

// Category folders
{
  name: "Certificates",
  parentFolder: "HR_FOLDER_ID",
  department: "HR_DEPT_ID",
  path: "/HR/Certificates",
  level: 1,
  createdBy: "SUPERADMIN_ID"
}

{
  name: "Circulars",
  parentFolder: "HR_FOLDER_ID",
  department: "HR_DEPT_ID",
  path: "/HR/Circulars",
  level: 1,
  createdBy: "SUPERADMIN_ID"
}

// Sub-category folders
{
  name: "Training Certificates",
  parentFolder: "CERTIFICATES_FOLDER_ID",
  department: "HR_DEPT_ID",
  path: "/HR/Certificates/Training",
  level: 2,
  createdBy: "SUPERADMIN_ID"
}

{
  name: "2024",
  parentFolder: "CIRCULARS_FOLDER_ID",
  department: "HR_DEPT_ID",
  path: "/HR/Circulars/2024",
  level: 2,
  createdBy: "SUPERADMIN_ID"
}
```

#### 4. Documents Collection

```javascript
const documentSchema = new mongoose.Schema({
  title: {
    type: String,
    required: true,
    trim: true
  },
  originalFileName: {
    type: String,
    required: true
  },
  fileUrl: {
    type: String,
    required: true  // S3/Azure URL
  },
  fileKey: {
    type: String,
    required: true  // S3 object key for deletion
  },
  fileType: {
    type: String,
    enum: ['pdf', 'docx', 'xlsx', 'jpg', 'png', 'zip'],
    required: true
  },
  fileSize: {
    type: Number,
    required: true  // in bytes
  },
  folder: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Folder',
    required: true
  },
  department: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Department',
    required: true
  },
  tags: [{
    type: String,
    trim: true,
    lowercase: true
  }],
  uploadedBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  version: {
    type: Number,
    default: 1
  },
  permissions: [{
    user: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User'
    },
    role: {
      type: String,
      enum: ['superadmin', 'admin', 'team_member', 'member_bank']
    },
    access: [{
      type: String,
      enum: ['view', 'edit', 'delete', 'download']
    }]
  }],
  metadata: {
    type: Map,
    of: String  // Flexible key-value pairs for any additional data
  },
  isActive: {
    type: Boolean,
    default: true
  }
}, {
  timestamps: true
});

// Indexes
documentSchema.index({ folder: 1 });
documentSchema.index({ department: 1 });
documentSchema.index({ uploadedBy: 1 });
documentSchema.index({ tags: 1 });
documentSchema.index({ title: 'text', tags: 'text' });  // Text search
documentSchema.index({ createdAt: -1 });
documentSchema.index({ 'permissions.user': 1 });
documentSchema.index({ 'permissions.role': 1 });

// Compound indexes
documentSchema.index({ folder: 1, isActive: 1 });
documentSchema.index({ department: 1, createdAt: -1 });
```

#### 5. DocumentVersions Collection

```javascript
const documentVersionSchema = new mongoose.Schema({
  document: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Document',
    required: true
  },
  version: {
    type: Number,
    required: true
  },
  fileUrl: {
    type: String,
    required: true
  },
  fileKey: {
    type: String,
    required: true
  },
  fileSize: {
    type: Number,
    required: true
  },
  uploadedBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  changes: {
    type: String,
    trim: true
  }
}, {
  timestamps: true
});

// Indexes
documentVersionSchema.index({ document: 1, version: -1 });
documentVersionSchema.index({ uploadedBy: 1 });
```

#### 6. AuditLogs Collection

```javascript
const auditLogSchema = new mongoose.Schema({
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  action: {
    type: String,
    enum: ['upload', 'download', 'view', 'edit', 'delete', 'rename', 'move', 'login', 'logout', 'create_folder', 'delete_folder', 'update_permissions'],
    required: true
  },
  resourceType: {
    type: String,
    enum: ['document', 'folder', 'user', 'department', 'system'],
    required: true
  },
  resourceId: {
    type: mongoose.Schema.Types.ObjectId
  },
  resourceName: {
    type: String
  },
  department: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Department'
  },
  ipAddress: {
    type: String
  },
  userAgent: {
    type: String
  },
  details: {
    type: Map,
    of: mongoose.Schema.Types.Mixed
  },
  timestamp: {
    type: Date,
    default: Date.now
  }
}, {
  timestamps: false
});

// Indexes
auditLogSchema.index({ user: 1, timestamp: -1 });
auditLogSchema.index({ action: 1, timestamp: -1 });
auditLogSchema.index({ resourceType: 1, resourceId: 1 });
auditLogSchema.index({ department: 1, timestamp: -1 });
auditLogSchema.index({ timestamp: -1 });

// TTL index - auto-delete logs older than 2 years (optional)
auditLogSchema.index({ timestamp: 1 }, { expireAfterSeconds: 63072000 });
```

#### 7. Notifications Collection

```javascript
const notificationSchema = new mongoose.Schema({
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  type: {
    type: String,
    enum: ['document_upload', 'access_granted', 'version_update', 'folder_shared'],
    required: true
  },
  title: {
    type: String,
    required: true
  },
  message: {
    type: String,
    required: true
  },
  relatedDocument: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Document'
  },
  relatedFolder: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Folder'
  },
  isRead: {
    type: Boolean,
    default: false
  },
  readAt: {
    type: Date
  }
}, {
  timestamps: true
});

// Indexes
notificationSchema.index({ user: 1, isRead: 1, createdAt: -1 });
notificationSchema.index({ user: 1, createdAt: -1 });
notificationSchema.index({ createdAt: -1 });

// TTL index - auto-delete read notifications older than 30 days
notificationSchema.index({ readAt: 1 }, { expireAfterSeconds: 2592000, partialFilterExpression: { isRead: true } });
```

---

## 3. BACKEND API ENDPOINTS

### Authentication APIs

```javascript
// routes/auth.js

POST   /api/auth/register           // Register new user (Superadmin only)
POST   /api/auth/login              // User login with JWT
POST   /api/auth/logout             // Logout
POST   /api/auth/refresh-token      // Refresh JWT token
POST   /api/auth/forgot-password    // Send password reset email
POST   /api/auth/reset-password     // Reset password with token
GET    /api/auth/me                 // Get current user info
PUT    /api/auth/update-profile     // Update own profile
PUT    /api/auth/change-password    // Change own password
```

**Implementation Checklist:**
- [ ] POST `/api/auth/register` - Register new user (Superadmin only)
- [ ] POST `/api/auth/login` - User login with JWT
- [ ] POST `/api/auth/logout` - Logout and invalidate token
- [ ] POST `/api/auth/refresh-token` - Refresh JWT token
- [ ] POST `/api/auth/forgot-password` - Password reset request
- [ ] POST `/api/auth/reset-password` - Reset password with token
- [ ] GET `/api/auth/me` - Get current user info
- [ ] PUT `/api/auth/update-profile` - Update profile
- [ ] PUT `/api/auth/change-password` - Change password

### Department Management APIs

```javascript
// routes/departments.js

GET    /api/departments             // Get all departments
GET    /api/departments/:id         // Get department by ID
POST   /api/departments             // Create new department (Superadmin)
PUT    /api/departments/:id         // Update department (Superadmin)
DELETE /api/departments/:id         // Delete department (Superadmin)
PATCH  /api/departments/:id/toggle  // Toggle active status
GET    /api/departments/:id/stats   // Get department statistics
```

**Implementation Checklist:**
- [ ] GET `/api/departments` - Get all departments
- [ ] GET `/api/departments/:id` - Get department by ID
- [ ] POST `/api/departments` - Create department
- [ ] PUT `/api/departments/:id` - Update department
- [ ] DELETE `/api/departments/:id` - Delete department
- [ ] PATCH `/api/departments/:id/toggle` - Toggle status
- [ ] GET `/api/departments/:id/stats` - Get stats

### User Management APIs

```javascript
// routes/users.js

GET    /api/users                   // Get all users (paginated)
GET    /api/users/:id               // Get user by ID
POST   /api/users                   // Create new user
PUT    /api/users/:id               // Update user
DELETE /api/users/:id               // Delete user
PATCH  /api/users/:id/toggle        // Toggle active status
GET    /api/users/department/:deptId // Get users by department
GET    /api/users/role/:role        // Get users by role
```

**Implementation Checklist:**
- [ ] GET `/api/users` - Get all users with pagination
- [ ] GET `/api/users/:id` - Get user by ID
- [ ] POST `/api/users` - Create new user
- [ ] PUT `/api/users/:id` - Update user details
- [ ] DELETE `/api/users/:id` - Delete user
- [ ] PATCH `/api/users/:id/toggle` - Activate/Deactivate user
- [ ] GET `/api/users/department/:deptId` - Get users by department
- [ ] GET `/api/users/role/:role` - Get users by role

### Folder Management APIs

```javascript
// routes/folders.js

GET    /api/folders                 // Get all folders (tree structure)
GET    /api/folders/tree            // Get folder tree by department
GET    /api/folders/:id             // Get folder by ID with contents
POST   /api/folders                 // Create new folder
PUT    /api/folders/:id             // Update folder (rename, move)
DELETE /api/folders/:id             // Delete folder
POST   /api/folders/:id/permissions // Set folder permissions
GET    /api/folders/:id/breadcrumb  // Get folder path hierarchy
GET    /api/folders/:id/children    // Get immediate children of folder
POST   /api/folders/:id/move        // Move folder to different parent
```

**Implementation Checklist:**
- [ ] GET `/api/folders` - Get all folders
- [ ] GET `/api/folders/tree` - Get folder tree structure
- [ ] GET `/api/folders/:id` - Get folder details with contents
- [ ] POST `/api/folders` - Create new folder
- [ ] PUT `/api/folders/:id` - Update folder (rename)
- [ ] DELETE `/api/folders/:id` - Delete folder (with contents check)
- [ ] POST `/api/folders/:id/permissions` - Assign permissions
- [ ] GET `/api/folders/:id/breadcrumb` - Get breadcrumb path
- [ ] GET `/api/folders/:id/children` - Get child folders
- [ ] POST `/api/folders/:id/move` - Move folder

### Document Management APIs

```javascript
// routes/documents.js

GET    /api/documents               // Get all documents (filtered)
GET    /api/documents/:id           // Get document by ID
POST   /api/documents/upload        // Upload new document
PUT    /api/documents/:id           // Update document metadata
DELETE /api/documents/:id           // Delete document
POST   /api/documents/:id/replace   // Replace document (new version)
GET    /api/documents/:id/versions  // Get version history
POST   /api/documents/:id/restore   // Restore old version
GET    /api/documents/:id/download  // Download document
POST   /api/documents/:id/permissions // Set document permissions
GET    /api/documents/folder/:folderId // Get documents in folder
POST   /api/documents/:id/move      // Move document to different folder
POST   /api/documents/bulk-delete   // Bulk delete documents
```

**Implementation Checklist:**
- [ ] GET `/api/documents` - Get all documents with filters
- [ ] GET `/api/documents/:id` - Get document by ID
- [ ] POST `/api/documents/upload` - Upload new document
- [ ] PUT `/api/documents/:id` - Update document metadata
- [ ] DELETE `/api/documents/:id` - Delete document
- [ ] POST `/api/documents/:id/replace` - Replace with new version
- [ ] GET `/api/documents/:id/versions` - Get version history
- [ ] POST `/api/documents/:id/restore` - Restore old version
- [ ] GET `/api/documents/:id/download` - Download document (signed URL)
- [ ] POST `/api/documents/:id/permissions` - Set permissions
- [ ] GET `/api/documents/folder/:folderId` - Get docs in folder
- [ ] POST `/api/documents/:id/move` - Move document
- [ ] POST `/api/documents/bulk-delete` - Bulk delete

### Search & Filter APIs

```javascript
// routes/search.js

GET    /api/search/documents        // Advanced search
GET    /api/search/autocomplete     // Search autocomplete
GET    /api/search/recent           // Recent searches (user-specific)
```

**Query Parameters for /api/search/documents:**
```
?title=xxx
&tags=tag1,tag2
&uploader=userId
&department=deptId
&folder=folderId
&fileType=pdf,docx
&uploadDateFrom=2024-01-01
&uploadDateTo=2024-12-31
&page=1
&limit=20
&sortBy=createdAt
&sortOrder=desc
```

**Implementation Checklist:**
- [ ] GET `/api/search/documents` - Search with multiple filters
- [ ] GET `/api/search/autocomplete` - Auto-suggest for search

### Audit Logs APIs

```javascript
// routes/auditLogs.js

GET    /api/audit-logs              // Get all logs (filtered, paginated)
GET    /api/audit-logs/user/:userId // Get logs for specific user
GET    /api/audit-logs/document/:docId // Get logs for document
GET    /api/audit-logs/department/:deptId // Get logs for department
GET    /api/audit-logs/export       // Export logs to CSV
```

**Query Parameters:**
```
?action=upload,download
&resourceType=document,folder
&userId=xxx
&department=xxx
&dateFrom=2024-01-01
&dateTo=2024-12-31
&page=1
&limit=50
```

**Implementation Checklist:**
- [ ] GET `/api/audit-logs` - Get all audit logs with filters
- [ ] GET `/api/audit-logs/user/:userId` - Get logs by user
- [ ] GET `/api/audit-logs/document/:docId` - Get logs for document
- [ ] GET `/api/audit-logs/department/:deptId` - Get dept logs
- [ ] GET `/api/audit-logs/export` - Export to CSV

###
