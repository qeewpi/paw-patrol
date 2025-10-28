# Technical Assumptions

### Repository Structure: Monorepo

**Decision:** Single repository with `/backend` and `/frontend` directories

**Rationale:**
- Solo developer project - simplifies dependency management and version control
- Easier to maintain consistency between API contracts and frontend implementations
- Single deployment pipeline for coordinated releases
- Natural fit for tightly coupled backend/frontend in MVP phase
- Can split to polyrepo later if needed for scale or team separation

### Service Architecture

**Architecture Pattern:** Monolithic backend

**Rationale:**
- 1-month MVP timeline requires fastest path to deployment
- Expected scale (10-20 colonies, 5-15 cats each) well within monolith capabilities
- Simpler debugging and testing than microservices
- Reduced operational complexity (single deployment unit)
- Database transactions easier to manage within monolithic boundary
- Can refactor to microservices post-MVP if scaling demands require it

**Technology Stack:**

**Backend:**
- **Language:** Java 17+
- **Framework:** Spring Boot 3.x
  - Spring Web (REST APIs)
  - Spring Security (authentication/authorization with JWT)
  - Spring Data JPA (ORM with Hibernate)
  - Spring Validation (input validation)
- **Database:** PostgreSQL 15+
  - Simple lat/long decimal fields for geolocation (no PostGIS complexity for MVP)
  - JSONB columns for flexible metadata if needed
- **Authentication:** JWT tokens with 30-day expiration, BCrypt password hashing
- **File Storage:** Local filesystem for MVP (photos stored on Render persistent disk), migrate to cloud object storage (AWS S3, Cloudflare R2) post-MVP if needed

**Frontend:**
- **Build Tool:** Vite
- **Framework:** React 18+
- **Language:** JavaScript (TypeScript optional post-MVP if type safety becomes priority)
- **Styling:** Tailwind CSS
- **Component Library:** Untitled UI (pre-built components for rapid development)
- **State Management:** React Context API or Zustand (lightweight, avoid Redux complexity)
- **Mapping:** Google Maps JavaScript API integration
- **HTTP Client:** Axios or Fetch API
- **Form Handling:** React Hook Form (lightweight form validation)

**Deployment:**
- **Platform:** Render
  - Backend: Render Web Service (auto-deploy from Git)
  - Frontend: Render Static Site
  - Database: Render Managed PostgreSQL
- **Target Tier:** Free tier initially, upgrade to paid tier ($25-50/month) if resource limits exceeded
- **HTTPS:** Provided automatically by Render (Let's Encrypt certificates)
- **CI/CD:** Render's built-in Git integration (auto-deploy on push to main branch)

**External Services:**
- **Google Maps Platform:**
  - Maps JavaScript API (interactive map rendering)
  - Geocoding API (address → coordinates conversion)
  - Places API (location search and autocomplete)
  - Target: Stay within $200/month free tier credit during pilot
- **Email Service (for password reset):** SendGrid free tier or Render's SMTP relay

### Testing Requirements

**MVP Testing Strategy:** Unit testing only with manual testing convenience methods

**Rationale:**
- 1-month timeline prioritizes feature delivery over comprehensive test coverage
- Solo developer can manually test critical paths efficiently
- Full testing pyramid (unit + integration + E2E) deferred to post-MVP
- Risk mitigation: Aggressive manual testing before pilot launch

**Testing Tools:**
- **Backend:** JUnit 5, Mockito (Spring Boot Test starter included)
- **Frontend:** Vitest (Vite's native test runner), React Testing Library (optional)
- **Manual Testing Aids:**
  - Postman collection for API testing
  - Seed data scripts for populating test colonies/cats
  - Admin user creation script for bypassing verification during development

**Post-MVP Testing Goals:**
- Integration tests for critical API endpoints (auth, colony CRUD, activity logging)
- E2E tests for key user journeys (Playwright or Cypress)
- Contract testing between frontend and backend (Pact or OpenAPI validation)

### Additional Technical Assumptions and Requests

**Security:**
- All API endpoints require authentication except registration, login, and password reset
- CORS configuration restricted to frontend domain (prevent unauthorized API access)
- SQL injection prevention via JPA parameterized queries
- File upload validation: whitelist image MIME types, max file size enforcement (5MB)
- XSS prevention via React's automatic escaping and Content Security Policy headers
- Rate limiting on authentication endpoints (prevent brute force attacks)

**Data Modeling:**
- Soft deletes for cats and colonies (preserve referential integrity, support audit trail)
- Audit columns on all entities: created_by, created_at, updated_by, updated_at
- Activity logs immutable (no UPDATE/DELETE operations) for MVP audit trail integrity
- Photo storage: filename with UUID prefix to prevent collisions, metadata stored in database

**Performance Optimization:**
- Database indexes on frequently queried fields: colony coordinates, cat status, activity timestamps
- Pagination for activity logs (default 50 items per page)
- Image optimization: resize/compress uploads to max 1920px width, 80% JPEG quality
- Lazy loading for photo galleries (load thumbnails first, full resolution on demand)
- Map marker clustering for dense colony areas (Google Maps MarkerClusterer)

**Development Workflow:**
- Git branching: main branch for production, feature branches for development
- Code style: Google Java Style Guide (backend), Prettier + ESLint (frontend)
- Environment variables for secrets (database credentials, JWT secret, API keys)
- Local development: Docker Compose for PostgreSQL database (avoid local installation)

**API Design:**
- RESTful conventions: GET (read), POST (create), PUT (update), DELETE (remove)
- JSON request/response format
- Standardized error responses with status codes and messages
- API versioning via URL prefix `/api/v1/` (prepare for future breaking changes)

**Browser Compatibility:**
- Target: Last 2 versions of Chrome, Firefox, Safari, Edge (mobile and desktop)
- Polyfills only if critical feature requires them (prefer modern browser APIs)
- No IE11 support (outdated, adds development complexity)

**Monitoring and Logging (MVP):**
- Backend: SLF4J with Logback (Spring Boot default)
- Log levels: ERROR for failures, WARN for concerning events, INFO for key operations
- Frontend: Console logging for errors (user-facing error messages for failures)
- Post-MVP: Centralized logging (Sentry, LogRocket) and application monitoring (Render metrics, New Relic)
