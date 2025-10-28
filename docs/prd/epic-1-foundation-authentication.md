# Epic 1: Foundation & Authentication

**Epic Goal:** Establish the foundational project infrastructure including monorepo setup, database configuration, deployment pipeline, and complete user authentication system with registration, login, password reset, and admin verification workflow. This epic delivers a deployable application where users can register, await approval, and authenticated users can access a basic landing page, providing the security foundation for all subsequent features.

### Story 1.1: Project Initialization & Repository Setup

As a **developer**,
I want to initialize the monorepo with backend and frontend directories and configure version control,
so that I have a structured foundation for development with proper Git workflow.

**Acceptance Criteria:**

1. Repository created with `/backend` and `/frontend` directories at root level
2. Backend initialized as Spring Boot 3.x project with Java 17+ using Spring Initializr (or manual setup) with dependencies: Spring Web, Spring Security, Spring Data JPA, PostgreSQL Driver, Spring Validation
3. Frontend initialized as Vite + React project with Tailwind CSS configured
4. `.gitignore` files configured for both backend (Maven/Gradle artifacts, IDE files) and frontend (node_modules, build artifacts)
5. Root-level README.md with project description, setup instructions, and technology stack documentation
6. Environment variable templates created (`.env.example` for frontend, `application.properties.example` for backend) documenting required configuration
7. Git repository initialized with initial commit containing project structure

---

### Story 1.2: Database Setup & Configuration

As a **developer**,
I want to configure PostgreSQL database connection and create initial schema with audit trail support,
so that the application can persist data with proper tracking of changes.

**Acceptance Criteria:**

1. Docker Compose file created for local PostgreSQL 15+ database (eliminates local installation requirement)
2. Backend configured to connect to PostgreSQL via environment variables (database URL, username, password)
3. JPA/Hibernate configured with proper dialect for PostgreSQL
4. Abstract base entity class created with audit fields: `id` (UUID), `createdBy`, `createdAt`, `updatedBy`, `updatedAt`
5. Soft delete support implemented via `deleted` boolean field and `deletedAt` timestamp in base entity
6. Database migration tool (Flyway or Liquibase) configured for schema version control
7. Initial migration script creates necessary database schema structure
8. Application successfully starts and connects to database with connection pool configured

---

### Story 1.3: User Entity & Role-Based Access Control Foundation

As a **developer**,
I want to create User entity with role management and password security,
so that the system can support different user types with appropriate access controls.

**Acceptance Criteria:**

1. `User` entity created extending base audit entity with fields: email (unique), passwordHash, name, verificationStatus (PENDING/APPROVED/REJECTED), roles (collection)
2. `Role` enum created with values: REGULAR_USER, ADMIN, SUPER_ADMIN
3. User entity supports multiple roles via `@ElementCollection` or join table
4. `VerificationDocument` entity created with fields: userId (foreign key), filename, filepath, uploadedAt
5. Database constraints enforce email uniqueness (case-insensitive)
6. JPA repositories created for User and VerificationDocument entities
7. Password field excluded from JSON serialization (security best practice)
8. Database migration script creates User and VerificationDocument tables with proper indexes

---

### Story 1.4: User Registration with Document Upload

As a **new user**,
I want to register with my email, password, name, and verification documents,
so that I can request access to the platform.

**Acceptance Criteria:**

1. POST `/api/v1/auth/register` endpoint accepts: email, password, name, and multipart file uploads (verification documents)
2. Password validation enforces minimum 8 characters, at least 1 uppercase, 1 lowercase, 1 number
3. Email validation enforces valid email format
4. Password hashed using BCrypt (via Spring Security) before database storage
5. Uploaded files validated: whitelist image MIME types (JPEG, PNG, PDF), max 5MB per file, max 5 files total
6. Files stored in local filesystem with UUID-prefixed filenames to prevent collisions
7. User created with `verificationStatus = PENDING` and default role `REGULAR_USER`
8. VerificationDocument records created linking files to user
9. Returns 201 Created with user ID and message "Registration submitted. Awaiting admin approval."
10. Returns 400 Bad Request for validation failures with specific error messages
11. Returns 409 Conflict if email already exists

---

### Story 1.5: Pending Approval Status Page

As a **newly registered user**,
I want to see a pending approval status page after registration,
so that I understand my account is awaiting verification.

**Acceptance Criteria:**

1. Frontend registration form component created with fields: email, password, confirm password, name, file upload (multiple files)
2. Form validation matches backend rules (password strength, email format, file size/type)
3. File upload UI shows selected files with ability to remove before submission
4. On successful registration (201 response), user redirected to pending approval status page
5. Pending approval page displays: "Your registration is under review", "Admins will review your verification documents within 48 hours", user's email address
6. Pending approval page has no navigation options except logout/return to login
7. Login attempt for pending user returns 403 Forbidden with message "Account pending approval"
8. Error messages displayed for registration failures (duplicate email, validation errors)

---

### Story 1.6: User Login with JWT Token Generation

As a **verified user**,
I want to log in with my email and password,
so that I can access the platform securely.

**Acceptance Criteria:**

1. POST `/api/v1/auth/login` endpoint accepts email and password in request body
2. Endpoint validates user exists, password matches (BCrypt comparison), and verificationStatus = APPROVED
3. JWT token generated with 30-day expiration containing: userId, email, roles
4. JWT signed with secret key from environment variable
5. Returns 200 OK with JWT token and user info (id, email, name, roles) on successful authentication
6. Returns 401 Unauthorized for invalid credentials with message "Invalid email or password"
7. Returns 403 Forbidden for pending/rejected users with appropriate message
8. Frontend login page created with email and password fields
9. On successful login (200 response), JWT token stored in memory or sessionStorage (not localStorage for security)
10. User redirected to map page (placeholder page for now showing "Welcome [name]" and logout button)
11. Error messages displayed for login failures

---

### Story 1.7: JWT Authentication Middleware & Protected Routes

As a **developer**,
I want to implement JWT authentication middleware for protected API endpoints,
so that only authenticated users can access secure resources.

**Acceptance Criteria:**

1. Spring Security filter chain configured to validate JWT tokens on protected endpoints
2. JWT validation checks: signature validity, expiration, token structure
3. Authenticated user details (userId, email, roles) extracted from valid JWT and made available in request context
4. All `/api/v1/**` endpoints require authentication EXCEPT: `/auth/register`, `/auth/login`, `/auth/forgot-password`, `/auth/reset-password`
5. Returns 401 Unauthorized for missing or invalid JWT tokens
6. Frontend HTTP client (Axios interceptor or Fetch wrapper) configured to attach JWT token to all API requests via Authorization header
7. Frontend handles 401 responses by clearing stored token and redirecting to login page
8. CORS configuration allows frontend domain with credentials

---

### Story 1.8: Password Reset Workflow

As a **user**,
I want to reset my password via email verification,
so that I can regain access if I forget my password.

**Acceptance Criteria:**

1. POST `/api/v1/auth/forgot-password` endpoint accepts email address
2. If email exists, generate password reset token (UUID) with 1-hour expiration and store in database (PasswordResetToken entity: token, userId, expiresAt)
3. Send password reset email via SendGrid (or SMTP relay) with reset link containing token
4. Returns 200 OK with message "If email exists, reset instructions sent" (prevent email enumeration)
5. POST `/api/v1/auth/reset-password` endpoint accepts token and new password
6. Validates token exists, not expired, and new password meets strength requirements
7. Updates user password (BCrypt hash) and deletes used token
8. Returns 200 OK with message "Password reset successful"
9. Returns 400 Bad Request for invalid/expired token
10. Frontend "Forgot Password" page with email input
11. Frontend "Reset Password" page (accessed via email link) with new password and confirm password fields
12. Success messages displayed for password reset completion

---

### Story 1.9: Admin Verification Panel - View Pending Users

As an **admin**,
I want to view a list of pending user registration applications,
so that I can review and approve/reject new users.

**Acceptance Criteria:**

1. GET `/api/v1/admin/users/pending` endpoint returns list of users with `verificationStatus = PENDING`
2. Endpoint requires authenticated user with ADMIN or SUPER_ADMIN role (403 Forbidden otherwise)
3. Response includes: userId, email, name, registrationDate, verification document count
4. Frontend admin panel page created (accessible only to admin users)
5. Admin panel displays pending users in table/list format with: email, name, date registered, document count
6. Each user row has "Review" button navigating to document viewer
7. Admin navigation added to bottom sheet menu (visible only to users with ADMIN or SUPER_ADMIN role)
8. Non-admin users attempting to access admin routes receive 403 error and redirect

---

### Story 1.10: Admin Verification Panel - Review & Approve/Reject

As an **admin**,
I want to review verification documents and approve or reject user applications,
so that only legitimate rescuers gain platform access.

**Acceptance Criteria:**

1. GET `/api/v1/admin/users/{userId}/verification-documents` endpoint returns list of document metadata with download URLs
2. GET `/api/v1/admin/verification-documents/{documentId}/download` endpoint streams document file (requires ADMIN role)
3. POST `/api/v1/admin/users/{userId}/approve` endpoint updates verificationStatus to APPROVED (requires ADMIN role)
4. POST `/api/v1/admin/users/{userId}/reject` endpoint updates verificationStatus to REJECTED with optional rejection reason (requires ADMIN role)
5. Both approve/reject endpoints return 200 OK with updated user status
6. Frontend document viewer page displays user info and verification documents (images shown inline, PDFs with download button)
7. Viewer page has "Approve" and "Reject" buttons with confirmation dialogs
8. Reject dialog includes optional text area for rejection reason
9. On approve/reject success, admin redirected back to pending users list
10. Success/error messages displayed for approval/rejection actions
11. Email notification sent to user on approval (optional for MVP, can be manual initially)

---

### Story 1.11: Deployment Configuration & Initial Deploy

As a **developer**,
I want to deploy the application to Render with proper environment configuration,
so that the application is accessible for testing and development.

**Acceptance Criteria:**

1. Render account created with PostgreSQL database provisioned (free tier)
2. Backend deployed as Render Web Service with environment variables configured: DATABASE_URL, JWT_SECRET, SENDGRID_API_KEY (or SMTP credentials)
3. Frontend deployed as Render Static Site with API URL configured
4. Database migrations run automatically on backend deployment
5. HTTPS enabled by default (Render provides this automatically)
6. Health check endpoint GET `/api/v1/health` returns 200 OK (used by Render for monitoring)
7. Frontend successfully communicates with backend API
8. Admin user manually seeded in database for initial testing (via SQL script or backend admin creation endpoint)
9. Application accessible via Render URLs (backend and frontend)
10. Deployment documented in README with URLs and deployment process

---

**Epic 1 Complete:** At this point, users can register, await approval, login after approval, and access a basic authenticated landing page. Admins can review and approve/reject applications. The foundation is ready for building colony and cat management features in Epic 2.
