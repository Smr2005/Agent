# ProjectAgent - Complete Documentation

## ðŸ“‹ Project Overview

**ProjectAgent** is a WhatsApp-style direct messaging platform that connects students, developers, and admins with secure admin-controlled user registration. Students create projects assigned to developers through a queue system, featuring AI-powered message sanitization, direct messaging, and file sharing capabilities.

### Key Features:
- âœ… Admin-controlled user registration with temporary passwords
- âœ… One project per student constraint (enforced at database level)
- âœ… AI-powered message sanitization using Anthropic Claude API
- âœ… WhatsApp-style color-coded messaging interface
- âœ… File sharing with automatic format detection (50MB per file limit)
- âœ… Automatic contact information filtering from messages
- âœ… JWT-based secure authentication

### Tech Stack:
- **Frontend:** HTML, CSS, JavaScript (Vanilla)
- **Backend:** Python, FastAPI
- **Database:** PostgreSQL (via Replit/Neon)
- **AI:** Anthropic Claude API
- **Authentication:** JWT + bcrypt

---

## ðŸŽ¨ Frontend Architecture

### Folder Structure:
```
frontend/
â”œâ”€â”€ templates/
â”‚   â”œâ”€â”€ index.html              # Login page
â”‚   â”œâ”€â”€ change-password.html    # Password change page
â”‚   â”œâ”€â”€ admin.html              # Admin dashboard
â”‚   â”œâ”€â”€ student.html            # Student dashboard
â”‚   â””â”€â”€ developer.html          # Developer dashboard
â””â”€â”€ static/
    â””â”€â”€ js/
        â”œâ”€â”€ auth.js             # Authentication logic
        â”œâ”€â”€ change-password.js  # Password change logic
        â”œâ”€â”€ admin.js            # Admin panel functionality
        â”œâ”€â”€ student.js          # Student interface
        â””â”€â”€ developer.js        # Developer interface
```

### Key Components:

#### 1. **Authentication Flow (auth.js)**
- Handles login, token storage, role-based redirects
- Tokens stored in `localStorage` under key `token`
- User info stored: `userName`, `userId`, `role`
- Automatic redirect on invalid/expired tokens

#### 2. **Admin Dashboard (admin.html + admin.js)**
- View all users (students, developers, admins)
- Create new user accounts with temporary passwords
- User must change password on first login
- Display user roles and active status
- User management interface

#### 3. **Student Dashboard (student.html + student.js)**
- **Projects View:**
  - Create new project (ONE project per student maximum)
  - List all student projects with status
  - View project details (title, description, status, created date)
  
- **Messages View:**
  - View all projects in queue
  - Project chat interface (color-coded: blue for student messages)
  - Send messages to AI agent
  - Upload files (PDFs, images, Word docs, code files, etc.)
  - Download files with correct format
  - Display agent-sanitized messages

#### 4. **Developer Dashboard (developer.html + developer.js)**
- **Projects View:**
  - View assigned projects
  - Accept/reject project assignments
  - View project details
  
- **Messages View:**
  - Project chat interface (color-coded: green for developer messages)
  - View AI-sanitized student requests
  - Send updates to students
  - Upload files (will be visible to student)
  - Download files with correct format

### Message Flow in UI:
1. User sends message to AI agent
2. Frontend stores token in query parameter for downloads
3. AI processes message based on user role
4. Message displayed with color coding (blue=student, green=developer, gray=agent)
5. Files displayed with format and upload info
6. Download links include JWT token for authentication

### File Handling:
- Frontend sends file to `/messages/upload-project-file/{message_id}`
- Receives file info (ID, name, type, size, uploader)
- Downloads use query parameter: `/messages/download-file/{file_id}?token={jwt_token}`
- MIME types determine download format (not .bin)

---

## ðŸ”§ Backend Architecture

### Folder Structure:
```
backend/
â”œâ”€â”€ models/                 # Database models (SQLAlchemy)
â”‚   â”œâ”€â”€ user.py            # User model with roles
â”‚   â”œâ”€â”€ project.py         # Project model
â”‚   â”œâ”€â”€ message.py         # Project messages (agent routing)
â”‚   â”œâ”€â”€ direct_message.py  # Direct messages
â”‚   â””â”€â”€ file_attachment.py # File attachments
â”œâ”€â”€ routes/                # API endpoints
â”‚   â”œâ”€â”€ auth.py           # Authentication endpoints
â”‚   â”œâ”€â”€ admin.py          # Admin management endpoints
â”‚   â”œâ”€â”€ student.py        # Student endpoints
â”‚   â”œâ”€â”€ developer.py      # Developer endpoints
â”‚   â”œâ”€â”€ messaging.py      # Messaging & file upload
â”‚   â”œâ”€â”€ queue.py          # Project queue management
â”‚   â””â”€â”€ seed.py           # Database seeding
â”œâ”€â”€ services/             # Business logic
â”‚   â”œâ”€â”€ auth.py          # JWT creation/validation
â”‚   â”œâ”€â”€ ai_agent.py      # AI message processing
â”‚   â””â”€â”€ assignment.py    # Project assignment logic
â”œâ”€â”€ middleware/           # Authentication middleware
â”‚   â””â”€â”€ auth.py          # JWT validation
â”œâ”€â”€ main.py              # FastAPI app initialization
â””â”€â”€ database.py          # Database connection setup
```

### API Endpoints:

#### Authentication Routes (`/auth`)
- `POST /auth/login` - Login with email/password
- `POST /auth/change-password` - Change password (must change on first login)
- `GET /auth/admin-info` - Get admin information

#### Admin Routes (`/admin`)
- `GET /admin/users` - Get all users
- `POST /admin/create-user` - Create new user with temporary password
- `GET /admin/projects` - Get all projects
- `GET /admin/analytics` - Get system analytics

#### Student Routes (`/student`)
- `GET /student/projects` - Get student's projects
- `POST /student/project/create` - Create new project (ONE MAX per student)
- `GET /student/chat/{project_id}` - Get project chat messages
- `POST /student/chat/send` - Send message to project
- `GET /student/messages` - Get direct messages with admin

#### Developer Routes (`/developer`)
- `GET /developer/projects` - Get assigned projects
- `POST /developer/accept-project/{project_id}` - Accept project
- `POST /developer/reject-project/{project_id}` - Reject project
- `GET /developer/chat/{project_id}` - Get project chat messages
- `POST /developer/chat/send` - Send message to project

#### Messaging Routes (`/messages`)
- `POST /messages/upload-project-file/{message_id}` - Upload file to project message
- `GET /messages/download-file/{file_id}?token={jwt}` - Download file
- `GET /messages/conversation/{user_id}` - Get direct message conversation

#### Queue Routes (`/queue`)
- `GET /queue/projects` - Get projects waiting for developer
- `POST /queue/assign/{project_id}/{developer_id}` - Assign project to developer

---

## ðŸ—„ï¸ Database Schema

### PostgreSQL Tables and Structure:

#### 1. **users** Table
```sql
id          UUID PRIMARY KEY (auto-generated)
name        VARCHAR NOT NULL
email       VARCHAR UNIQUE NOT NULL
password_hash VARCHAR NOT NULL (bcrypt hashed)
role        ENUM('student', 'developer', 'admin') NOT NULL
active      BOOLEAN DEFAULT true
must_change_password BOOLEAN DEFAULT true
```

**Purpose:** Store all user accounts with roles and authentication

---

#### 2. **projects** Table
```sql
id              UUID PRIMARY KEY (auto-generated)
title           VARCHAR NOT NULL
description     TEXT NOT NULL
student_id      UUID FOREIGN KEY (users.id) NOT NULL
assigned_dev_id UUID FOREIGN KEY (users.id) NULLABLE
status          ENUM('open', 'assigned', 'in-progress', 'completed', 'queued') NOT NULL
created_at      TIMESTAMP DEFAULT now()
```

**Purpose:** Store project information and status

**Constraints:**
- One-to-one relationship: Each student can have only ONE project
- `student_id` is unique (enforced in application)

---

#### 3. **messages** Table
```sql
id                     UUID PRIMARY KEY (auto-generated)
from_role              VARCHAR NOT NULL ('student', 'developer', 'agent')
from_user_id           UUID FOREIGN KEY (users.id) NULLABLE
to_role                VARCHAR NOT NULL ('student', 'developer', 'agent')
to_user_id             UUID FOREIGN KEY (users.id) NULLABLE
project_id             UUID FOREIGN KEY (projects.id) NULLABLE
raw_text               TEXT NOT NULL (original message)
agent_sanitized_text   TEXT NULLABLE (AI-processed message)
created_at             TIMESTAMP DEFAULT now()
```

**Purpose:** Store all project chat messages with AI agent routing

**How it Works:**
- When student sends message: Creates 2 messages
  1. `student â†’ agent`
  2. `agent â†’ developer` (AI-sanitized)
- When developer sends message: Creates 2 messages
  1. `developer â†’ agent`
  2. `agent â†’ student` (AI-sanitized)
- Files attached to BOTH messages for visibility

---

#### 4. **direct_messages** Table
```sql
id              UUID PRIMARY KEY (auto-generated)
from_user_id    UUID FOREIGN KEY (users.id) NOT NULL
to_user_id      UUID FOREIGN KEY (users.id) NOT NULL
message_text    TEXT NOT NULL
is_read         BOOLEAN DEFAULT false
created_at      TIMESTAMP DEFAULT now()
```

**Purpose:** Store direct messages between users (e.g., student â†” admin)

---

#### 5. **file_attachments** Table
```sql
id                  UUID PRIMARY KEY (auto-generated)
message_id          UUID FOREIGN KEY (direct_messages.id) NULLABLE
project_message_id  UUID FOREIGN KEY (messages.id) NULLABLE
file_name           VARCHAR NOT NULL (original filename)
file_type           VARCHAR NOT NULL (extension: .jpg, .pdf, etc.)
file_size           INTEGER NOT NULL (bytes)
file_path           VARCHAR NOT NULL (server path)
uploaded_by         UUID FOREIGN KEY (users.id) NOT NULL
created_at          TIMESTAMP DEFAULT now()
```

**Purpose:** Store file metadata and references

**Dual Attachment:**
- Files are attached to BOTH project messages
- Student sees file from both their message AND agent-relayed message
- Developer sees same file for visibility

---

## ðŸ”„ Data Flow & Workflows

### 1. **User Registration & Login Flow**

```
Admin Creates User
    â†“
User assigned temporary password
    â†“
User logs in with temp password
    â†“
Forced to change password (must_change_password = true)
    â†“
Password updated, flag set to false
    â†“
User can now access dashboard
```

### 2. **Project Creation Flow (Student)**

```
Student creates project
    â†“
Constraint check: Student has NO existing project
    â†“
Project created with status = 'open'
    â†“
Project added to queue for developers
    â†“
Project appears in developer's available projects
```

### 3. **Message Routing with AI Agent**

#### Student sends message to project:
```
Student Message Input
    â†“
Filter contact info (emails, phones, social media)
    â†“
Create message: student â†’ agent
    â†“
AI Agent processes via Claude API
    â†“
Claude converts to technical instructions
    â†“
Create message: agent â†’ developer (sanitized text)
    â†“
Both messages stored in database
```

#### Developer sends message:
```
Developer Message Input
    â†“
Filter contact info
    â†“
Create message: developer â†’ agent
    â†“
AI Agent processes via Claude API
    â†“
Claude converts to student-friendly language
    â†“
Create message: agent â†’ student (sanitized text)
    â†“
Both messages stored in database
```

### 4. **File Upload & Download Flow**

#### Upload:
```
User selects file
    â†“
Frontend sends to /messages/upload-project-file/{message_id}
    â†“
Backend validates JWT token
    â†“
File saved to /backend/storage/ with UUID prefix
    â†“
FileAttachment record created with metadata
    â†“
Find corresponding message for other party
    â†“
ALSO attach file to that message
    â†“
Response: file info (ID, name, type, size, uploader)
```

#### Download:
```
User clicks download link
    â†“
GET /messages/download-file/{file_id}?token={jwt}
    â†“
Backend validates JWT token from query param
    â†“
Check file access permissions
    â†“
Look up MIME type based on file_type
    â†“
Set Content-Type header (e.g., image/jpeg, application/pdf)
    â†“
Set filename header
    â†“
Stream file to browser
    â†“
Browser downloads with correct extension
```

### 5. **AI Agent Message Processing**

#### Contact Info Filtering:
```
Email: name@domain.com        â†’ [EMAIL REMOVED]
Phone: +1-234-567-8900       â†’ [PHONE REMOVED]
Social: instagram.com/user   â†’ [SOCIAL MEDIA REMOVED]
WhatsApp: wa.me/phonenumber  â†’ [SOCIAL MEDIA REMOVED]
```

#### Message Transformation:

**Student to Developer:**
```
INPUT:  "Hi, I need a website for my business. 
         Can you call me at 555-1234?"

OUTPUT: "[Project: My Business Website]

Developer Instructions:
- Build a website for a business
- [PHONE REMOVED] - No direct contact

Project Description: Modern business website"
```

**Developer to Student:**
```
INPUT:  "We've implemented the homepage with 
         React components. Using Redux for state 
         management."

OUTPUT: "Update on your project 'My Business Website':

We've successfully created the main page of your website 
and set up the system to manage data effectively. 
The interface is responsive and ready for testing.

Next steps: We'll add the contact form and payment 
integration."
```

---

## ðŸ” Security Features

### 1. **Authentication**
- JWT tokens (24-hour expiration)
- Bcrypt password hashing
- HTTPBearer security scheme
- Forced password change on first login

### 2. **Authorization**
- Role-based access control (student, developer, admin)
- Middleware validates roles for protected routes
- Token required in Authorization header OR query parameter

### 3. **Data Protection**
- Contact info automatically filtered from all messages
- Students and developers never see each other's direct info
- AI agent acts as secure intermediary
- File access permissions validated before download

### 4. **Input Validation**
- Email validation (email-validator library)
- UUID validation for all IDs
- File size limits (50MB per file)
- Message length limits

---

## ðŸ“Š Key Constraints & Business Rules

### 1. **One Project Per Student**
- Implemented at backend validation level
- Checked in `POST /student/project/create`
- Returns error if student already has project
- Ensures clean queue management

### 2. **File Format Support**
Supported for upload and download:
- Documents: `.pdf`, `.doc`, `.docx`, `.xls`, `.xlsx`, `.ppt`, `.pptx`
- Images: `.jpg`, `.jpeg`, `.png`, `.gif`
- Code: `.py`, `.js`, `.ts`, `.java`, `.cpp`, `.c`, `.html`, `.css`, `.json`, `.xml`
- Others: `.txt`, `.zip`, `.csv`

### 3. **File Size Limit**
- Maximum 50MB per file
- Enforced at upload endpoint
- Checked against file content length

### 4. **Message Types**
Each message has `from_role` and `to_role`:
- `student â†’ agent` (original student message)
- `agent â†’ developer` (sanitized for developer)
- `developer â†’ agent` (original developer message)
- `agent â†’ student` (sanitized for student)

---

## ðŸš€ Deployment & Environment

### Environment Variables Required:
```
DATABASE_URL=postgresql://user:pass@host/dbname
SESSION_SECRET=your-secret-key-here
ANTHROPIC_API_KEY=sk-ant-...your-key...
```

### Storage:
- Files stored in `/backend/storage/` directory
- File naming: `{uuid}_{original_filename}`
- Path stored in database for future retrieval

### API Server:
- Runs on `http://0.0.0.0:8000`
- CORS enabled for all origins
- Static files served from `/frontend/static`

---

## ðŸ”§ Development Guide

### Running the Project:
```bash
python -m uvicorn backend.main:app --host 0.0.0.0 --port 8000
```

### Adding New Features:

**To add a new API endpoint:**
1. Create route in appropriate file (`backend/routes/`)
2. Add required models/fields to `backend/models/`
3. Use `get_db` dependency for database access
4. Apply `@require_role()` middleware for protection
5. Return FastAPI response models

**To add new database table:**
1. Create model in `backend/models/`
2. Inherit from `Base`
3. Run `create_tables.py` or use Alembic migrations

**To modify file types:**
1. Update MIME type dictionary in `backend/routes/messaging.py`
2. Add new extension â†’ MIME type mapping
3. Restart backend

---

## ðŸ“ Example Workflows

### Workflow 1: Student Creates Project and Uploads File

```
1. Student logs in â†’ sees dashboard
2. Clicks "Create Project"
3. Enters title: "E-commerce Website"
   Description: "Need a website to sell products"
4. Backend validates: Student has no project âœ…
5. Project created, status = 'open'
6. Project appears in queue for developers

7. Student enters project chat
8. Uploads file: product-list.csv
9. Backend:
   - Saves file to storage
   - Creates FileAttachment record
   - Finds agentâ†’developer message
   - Attaches file to BOTH messages

10. Developer logs in â†’ sees project
11. Enters project chat â†’ sees file
12. Clicks download â†’ file downloads as product-list.csv
```

### Workflow 2: Developer Sends Update

```
1. Developer types: "We've set up the database schema with tables 
   for products, orders, and customers. Using PostgreSQL."
   
2. Backend filters contact info (none found)

3. Creates messages:
   - developer â†’ agent (original)
   - agent â†’ student (sanitized)

4. AI Agent (Claude) converts to:
   "Update on your project 'E-commerce Website':
   
   We've successfully created the system to store your product data,
   customer information, and order details. Everything is ready for
   the next phase.
   
   Next: We'll add user registration and login."

5. Student logs in â†’ sees project chat
6. Sees professional update in simple language
7. Never sees developer's direct info
```

### Workflow 3: File Access Control

```
1. Student uploads: design.pptx
2. File stored at: /backend/storage/abc123_design.pptx
3. FileAttachment created:
   - file_type: .pptx
   - uploaded_by: student_uuid

4. Developer requests: /messages/download-file/file_uuid?token=jwt
5. Backend checks:
   - JWT valid? âœ…
   - User has access to this project? âœ…
   - File exists? âœ…
   - Look up MIME: .pptx â†’ application/vnd.openxmlformats-officedocument.presentationml.presentation

6. Return file with:
   - Content-Type: application/vnd.openxmlformats...
   - Content-Disposition: attachment; filename="design.pptx"

7. Browser downloads as design.pptx (NOT .bin!)
```

---

## ðŸ› Troubleshooting

### File downloads as .bin instead of correct format
**Solution:** File type stored with dot (`.jpg`) but lookup expected without dot (`jpg`). Fixed by using `.lstrip('.')`.

### "Not authenticated" on file download
**Solution:** Token must be passed as query parameter `?token=` because browser links don't support Authorization headers.

### File not visible to developer
**Solution:** File must be attached to BOTH messages (studentâ†’agent AND agentâ†’developer) for cross-visibility.

### Student can create multiple projects
**Solution:** Added validation in `POST /student/project/create` to check existing project count.

### AI agent not processing messages
**Solution:** Ensure `ANTHROPIC_API_KEY` environment variable is set. If missing, falls back to placeholder text.

---

**Last Updated:** November 27, 2025  
**Version:** 1.0.0  
**Status:** Documentation Ready
