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

**SIMPLIFIED STRUCTURE:** Department → Folder → Subfolder → Document

The system uses a simple parent-child relationship:
- **Departments** are root-level containers
- **Folders** can have parent folders (creating nested structure)
- **Documents** belong to folders
- Navigation is tree-based: Click department → Fetch its folders → Click folder → Fetch its contents (subfolders & documents)

```
Department (parentFolder: null)
  └── Folder (parentFolder: null, department: dept_id)
      └── Subfolder (parentFolder: folder_id, department: dept_id)
          └── Sub-subfolder (parentFolder: subfolder_id, department: dept_id)
              └── Documents (folder: sub_subfolder_id, department: dept_id)
```

**Example Structure:**
```
HR Department
├── Certificates (Folder, parentFolder: null)
│   ├── Training Certificates (Folder, parentFolder: Certificates_id)
│   │   ├── 2024 (Folder, parentFolder: Training_id)
│   │   └── 2025 (Folder, parentFolder: Training_id)
│   └── Audit Certificates (Folder, parentFolder: Certificates_id)
├── Circulars (Folder, parentFolder: null)
│   ├── 2024 (Folder, parentFolder: Circulars_id)
│   └── 2025 (Folder, parentFolder: Circulars_id)
└── Policies (Folder, parentFolder: null)
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
departmentSchema.index({ name: 1 });
departmentSchema.index({ isActive: 1 });
```

**Required Seed Data:**
```javascript
[
  { name: "Human Resources", description: "HR Department", isActive: true },
  { name: "Finance", description: "Finance Department", isActive: true },
  { name: "Information Technology", description: "IT Department", isActive: true },
  { name: "Operations", description: "Operations Department", isActive: true },
  { name: "Legal", description: "Legal Department", isActive: true },
  { name: "Administration", description: "Administration", isActive: true }
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
folderSchema.index({ parentFolder: 1 });
folderSchema.index({ department: 1 });
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

GET    /api/folders/department/:deptId        // Get root folders of department
GET    /api/folders/:id/children              // Get child folders and documents
GET    /api/folders/:id                       // Get folder details
POST   /api/folders                           // Create new folder
PUT    /api/folders/:id                       // Update folder
DELETE /api/folders/:id                       // Delete folder
POST   /api/folders/:id/permissions           // Set folder permissions
GET    /api/folders/:id/breadcrumb            // Get breadcrumb (parent chain)
POST   /api/folders/:id/move                  // Move folder to different parent
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
- [ ] Table with: Name, Head, Status, Actions
- [ ] Add Department button
- [ ] Edit/Delete actions
- [ ] Toggle active status

#### Add/Edit Department Modal
- [ ] Name field
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
- [ ] Breadcrumb navigation using parentFolder chain
- [ ] Search folders by name
- [ ] Filter by department

#### Create Folder Modal
**Form:**
- [ ] Folder name input
- [ ] Parent folder selector (tree dropdown)
- [ ] Department dropdown
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
- [ ] Folder information (name, created by, created date)
- [ ] Parent folder (breadcrumb)
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
  - Folder location (breadcrumb using parentFolder)
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
  .populate('folder', 'name')
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
  const titles = await
