# 📁 Document Management System (DMS)

A secure, modular, and role-based **Document Management System (DMS)** designed for internal file storage, version control, and access management — built using **React, Node.js, MongoDB, and AWS S3**.

---

## 🚀 Overview

The DMS allows organizations to:

* Upload and manage files in structured folders
* Assign access permissions (view/upload/download/delete)
* Maintain version history for each file
* Search and audit all activities
* Support member-level restricted access

---

## 🧩 Core Features (Phase 1 MVP)

### 🔐 Authentication & Roles

* JWT-based login
* Roles: `superadmin`, `admin`, `member`, `bank`
* Role-based access and visibility control

### 📁 Folder & Subfolder Management

* Create, rename, move, delete folders
* Infinite folder nesting supported
* Role-based visibility and permission settings

### 📄 File Upload & Metadata

* Supported formats: PDF, DOCX, XLSX, JPG, PNG, ZIP, MP4
* Metadata: title, tags, description, type, expiry (optional)
* Upload new versions of existing files

### 🔄 Version Control

* Every upload generates a new version (v1, v2, v3, …)
* Old versions stored securely in S3
* Option to view/download/restore previous versions

### 🧑‍🤝‍🧑 Role-Based Access Control (RBAC)

* Folder/file-level permissions
* Visibility levels: `public`, `private`, `restricted`
* Member Banks → read-only access

### 🧾 Audit Logs

* Tracks file uploads, downloads, views, version changes, deletions
* Filter logs by user, action, date, or folder

### 🔍 Search & Filters

* Search by title, tag, type, uploader
* Filter by date, department, or visibility

---

## ⚙️ Tech Stack

| Layer      | Technology                      |
| ---------- | ------------------------------- |
| Frontend   | React.js + Tailwind + ShadCN/UI |
| Backend    | Node.js + Express               |
| Database   | MongoDB                         |
| Storage    | AWS S3                          |
| Auth       | JWT + Role-Based Access         |
| Deployment | VPS / AWS EC2                   |

---

## 🧱 Project Structure

```
dms/
│
├── backend/
│   ├── src/
│   │   ├── config/
│   │   │   └── db.js
│   │   │   └── s3.js
│   │   ├── models/
│   │   │   ├── User.js
│   │   │   ├── Folder.js
│   │   │   ├── File.js
│   │   │   ├── FileVersion.js
│   │   │   ├── Log.js
│   │   ├── routes/
│   │   │   ├── auth.js
│   │   │   ├── folders.js
│   │   │   ├── files.js
│   │   │   ├── versions.js
│   │   ├── middleware/
│   │   │   └── auth.js
│   │   └── server.js
│   ├── .env
│   └── package.json
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   │   ├── Dashboard.jsx
│   │   │   ├── FileManager.jsx
│   │   │   ├── UploadModal.jsx
│   │   │   ├── VersionHistoryModal.jsx
│   │   │   ├── Login.jsx
│   │   ├── context/
│   │   ├── hooks/
│   │   ├── App.jsx
│   └── package.json
│
└── README.md
```

---

## ⚙️ Environment Variables

Create `.env` file inside `/backend`:

```
PORT=5000
MONGO_URI=mongodb+srv://<username>:<password>@cluster.mongodb.net/dms
JWT_SECRET=supersecurekey
AWS_ACCESS_KEY_ID=<your-key>
AWS_SECRET_ACCESS_KEY=<your-secret>
AWS_REGION=ap-south-1
AWS_BUCKET_NAME=dms-storage
```

---

## 🧩 Folder & File Schema (Simplified)

### **File Document**

```js
{
  _id: ObjectId,
  title: String,
  folderId: ObjectId,
  mimeType: String,
  visibility: String, // "public" | "private" | "restricted"
  currentVersion: Number,
  uploadedBy: ObjectId,
  tags: [String],
  size: Number,
  createdAt: Date,
  updatedAt: Date
}
```

### **File Version Document**

```js
{
  _id: ObjectId,
  fileId: ObjectId,
  versionNumber: Number,
  storageKey: String, // AWS S3 key
  size: Number,
  mimeType: String,
  uploadedBy: ObjectId,
  uploadedAt: Date
}
```

---

## 📤 Upload Logic

1. When a new file is uploaded:

   * Create record in `files`
   * Add version in `fileVersions`
2. When uploading a new version:

   * Find existing file
   * Increment `currentVersion`
   * Upload new file to S3
   * Save new `fileVersions` entry
   * Update parent file’s `currentVersion` + `updatedAt`
3. Older versions remain viewable and restorable

---

## 🪄 API Endpoints

### 🔑 Auth

| Method | Endpoint         | Description                   |
| ------ | ---------------- | ----------------------------- |
| POST   | `/auth/login`    | User login                    |
| POST   | `/auth/register` | Create user (superadmin only) |

### 📁 Folder

| Method | Endpoint       | Description        |
| ------ | -------------- | ------------------ |
| GET    | `/folders`     | Get all folders    |
| POST   | `/folders`     | Create folder      |
| PUT    | `/folders/:id` | Rename/move folder |
| DELETE | `/folders/:id` | Delete folder      |

### 📄 File

| Method | Endpoint        | Description        |
| ------ | --------------- | ------------------ |
| POST   | `/files/upload` | Upload new file    |
| GET    | `/files/:id`    | Get file details   |
| DELETE | `/files/:id`    | Move file to trash |

### 🔄 Version Control

| Method | Endpoint                          | Description         |
| ------ | --------------------------------- | ------------------- |
| POST   | `/files/:fileId/new-version`      | Upload new version  |
| GET    | `/files/:fileId/versions`         | Get all versions    |
| PUT    | `/files/:fileId/restore/:version` | Restore old version |

---

## 🧮 Suggested Collections

* `users` → user details + role
* `folders` → hierarchical structure
* `files` → file metadata + current version
* `fileVersions` → version history
* `logs` → action tracking
* `notices` → announcements (Phase 2)

---

## 💻 Frontend Flow

1. **Dashboard**

   * Show stats, recent files, and notices.

2. **Folder Tree**

   * Display nested folders on left sidebar.
   * Clicking folder loads its files on right panel.

3. **File Actions (⋮ menu)**

   * View Details
   * Download
   * Upload New Version
   * Delete / Move to Trash

4. **Upload New Version**

   * File picker opens.
   * On upload → call `/files/:fileId/new-version`
   * Refresh file list + version history modal.

5. **Version History Modal**

   * List versions: v1, v2, v3…
   * Show date, size, uploadedBy.
   * Buttons: “Download”, “Restore”.

---

## 🧠 Developer Notes

* Use **AWS SDK v3** for S3 uploads
* Use **Multer-S3** for handling file streams
* Store versions in `files/<fileId>/v<versionNumber>` format
* Use **Zustand** or **Redux Toolkit** for frontend state
* Implement global toast notifications for upload success/failure

---

## 🔒 Security Checklist

✅ JWT token verification middleware
✅ Role-based route protection
✅ File visibility validation before serving download
✅ S3 bucket private (use presigned URLs for downloads)
✅ Daily database backup cron

---

## 📦 Installation & Setup

### Backend

```bash
cd backend
npm install
npm run dev
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

---

## 📅 Roadmap

| Week | Task                               |
| ---- | ---------------------------------- |
| 1    | Setup backend, Mongo schemas, Auth |
| 2    | Folder CRUD + file uploads         |
| 3    | Version control + S3 integration   |
| 4    | RBAC + Search + Logs               |
| 5    | Frontend UI + API Integration      |
| 6    | Testing + Deployment               |

---

## 📈 Future Enhancements

* Blockchain-based document hashing
* Public document verification portal
* OCR for scanned PDFs
* PWA mobile version
* Multi-language UI

---

> ✅ With this setup, you can implement and scale your Document Management System step-by-step — including secure uploads, full version history, and granular access control.
