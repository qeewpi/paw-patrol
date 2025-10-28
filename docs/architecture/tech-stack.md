# Tech Stack

This is the DEFINITIVE technology selection for the entire project. All development must use these exact technologies and versions.

| Category | Technology | Version | Purpose | Rationale |
|----------|-----------|---------|---------|-----------|
| **Frontend Language** | TypeScript | 5.x | Type-safe frontend development | Catches errors at compile-time, better IDE support, shares types with backend via shared package |
| **Frontend Framework** | React | 18.x | UI component library | Industry standard, excellent ecosystem, aligns with PRD requirement for Vite + React |
| **Build Tool** | Vite | 5.x | Frontend build and dev server | Instant server start, lightning-fast HMR with native ESM, Rollup-based production builds with automatic code splitting and tree-shaking, PRD-specified |
| **UI Component Library** | Untitled UI | Latest | Pre-built components | PRD-specified, Tailwind-based, mobile-first design system reduces custom CSS |
| **CSS Framework** | Tailwind CSS | 3.x | Utility-first styling | Rapid prototyping, mobile-first defaults, integrates with Untitled UI, small bundle size |
| **State Management** | TanStack Query (React Query) | 5.x | Server state management | Powerful async state management with caching, optimistic updates (via onMutate/onError/onSettled), automatic cache invalidation, and request deduplication; local UI state via React hooks |
| **Routing** | React Router | 6.x | Client-side routing | Industry standard for React SPAs, supports nested routes, lazy loading |
| **Form Handling** | React Hook Form | 7.x | Form state and validation | Performant (uncontrolled), excellent DX, built-in validation, small bundle size |
| **API Client** | Axios | 1.x | HTTP client | Interceptors for JWT auth, request/response transforms, better error handling than fetch |
| **Backend Language** | Java | 17 LTS | Backend development | PRD requirement, LTS support until 2029, excellent Spring Boot ecosystem |
| **Backend Framework** | Spring Boot | 3.x | Backend framework | PRD requirement, comprehensive ecosystem, rapid development, excellent documentation |
| **API Style** | REST | - | API architecture | Simple, well-understood, excellent Spring Boot support, no GraphQL complexity needed |
| **ORM** | Spring Data JPA / Hibernate | 6.x | Database ORM | Standard Spring stack, reduces boilerplate, type-safe queries, automatic schema generation |
| **Database** | PostgreSQL | 15+ | Primary database | PRD requirement, relational model suits domain, excellent spatial support (PostGIS future), Supabase 500MB free tier |
| **Authentication** | Spring Security + JWT | 6.x | Auth framework | Industry standard for Spring, handles password hashing (BCrypt), JWT token generation/validation |
| **File Storage** | Fly.io Persistent Volume | - | Photo storage | 3GB free persistent storage, survives deploys/restarts, included with Fly.io |
| **Mapping** | Google Maps Platform | - | Map rendering & geocoding | PRD requirement, Maps JavaScript API for frontend, Geocoding API for backend, \$200/month free credit |
| **Maps React Library** | @vis.gl/react-google-maps | Latest | React wrapper for Google Maps | Official vis.gl library, modern API with hooks (useMap), supports AdvancedMarker API, better TypeScript support than @react-google-maps/api |
| **Frontend Testing** | Vitest + Testing Library | Latest | Unit/integration tests | Blazing fast Vite-native test framework with instant HMR, Jest-compatible API, React Testing Library for component tests |
| **Backend Testing** | JUnit 5 + Mockito | Latest | Unit/integration tests | Industry-standard Spring testing stack with @SpringBootTest for integration tests, MockMvc for REST controller testing, Mockito for service layer mocking |
| **E2E Testing** | Playwright | Latest | End-to-end tests | Fast, reliable, multi-browser, better than Cypress for this stack |
| **API Documentation** | Springdoc OpenAPI | 2.x | API spec generation | Generates OpenAPI 3.0 from Spring controllers, enables TypeScript codegen |
| **Type Validation** | Zod | 3.x | Runtime type validation | Validates API responses match TypeScript types, prevents drift |
| **Bundler** | Vite (built-in) | 5.x | Production bundling | Included with Vite, Rollup-based, tree-shaking, code splitting |
| **Monorepo Tool** | npm workspaces | Built-in | Package management | Zero additional dependencies, sufficient for 3-package monorepo |
| **Containerization** | Docker | Latest | Backend packaging | Required for Fly.io deployment, simple Dockerfile for Spring Boot |
| **CI/CD** | GitHub Actions | - | Automation | Free for public repos, Fly.io deployment via GitHub Actions |
| **Hosting - Frontend** | Fly.io Static | - | Frontend hosting | Served from same Fly.io app as backend, HTTPS, global Anycast network |
| **Hosting - Backend** | Fly.io Machines | - | Backend hosting | Free 2,340 GB-hrs/month (3 VMs), persistent volumes, Singapore region |
| **Hosting - Database** | Supabase PostgreSQL | - | Database hosting | Free 500MB tier, 50K MAU, no time limit, connection pooling |
| **Monitoring** | Fly.io Logs | - | Basic logging | Built-in log aggregation and viewing (free), access via `fly logs` CLI, real-time streaming, sufficient for MVP |
| **Error Tracking** | Console logging | - | Error capture | MVP uses console logs; add Sentry post-validation if needed |

---

## Deployment Setup Guide

**Why Fly.io + Supabase over Render:**
-  **3x more free runtime:** 2,340 GB-hours vs 750 hours  
-  **Persistent photo storage:** 3GB volume (Render has ephemeral filesystem)
-  **No database expiration:** Supabase free tier permanent (Render expires after 90 days)
-  **Better performance:** Fly.io machines faster cold starts
-  **Trade-off:** Requires Dockerfile (~10 lines) vs Render's auto-detect

### Initial Setup Steps

**1. Fly.io Backend Setup:**
```bash
# Install flyctl CLI
# Windows: scoop install flyctl
# Mac: brew install flyctl

# Login and launch app
fly auth login
fly launch --name paw-patrol-api --region sin

# Create persistent volume for photos (3GB free)
fly volumes create paw_patrol_photos --region sin --size 3

# Deploy
fly deploy
```

**2. Supabase Database Setup:**
```bash
# Create project at https://supabase.com
# Select Singapore region
# Note connection string from Settings > Database

# Add to backend application.properties:
# spring.datasource.url=jdbc:postgresql://db.xxx.supabase.co:5432/postgres
# spring.datasource.username=postgres
# spring.datasource.password=<your-password>
```

**3. Environment Variables:**
```bash
# Set secrets for production
fly secrets set GOOGLE_MAPS_API_KEY=<your-key>
fly secrets set DATABASE_URL=<supabase-connection-string>
```

### Configuration Files Needed

**fly.toml** (project root):
```toml
app = "paw-patrol"
primary_region = "sin"

[build]
  dockerfile = "Dockerfile"

[env]
  PORT = "8080"

[http_service]
  internal_port = 8080
  force_https = true
  auto_stop_machines = true
  auto_start_machines = true
  min_machines_running = 0

[[mounts]]
  source = "paw_patrol_photos"
  destination = "/app/photos"

# Serve frontend static files from backend container
[[statics]]
  guest_path = "/app/frontend/dist"
  url_prefix = "/"
```

**Dockerfile** (project root):
```dockerfile
# Multi-stage build: Build frontend, then package with backend
# Stage 1: Build Vite frontend
FROM node:18-alpine AS frontend-build
WORKDIR /frontend
COPY frontend/package*.json ./
RUN npm ci --production=false
COPY frontend/ ./
RUN npm run build

# Stage 2: Package Spring Boot backend with frontend dist
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Copy pre-built backend JAR (build via mvn package before docker build)
COPY backend/target/*.jar app.jar

# Copy frontend production build from stage 1
COPY --from=frontend-build /frontend/dist /app/frontend/dist

# Expose Spring Boot port
EXPOSE 8080

# Run Spring Boot application
CMD ["java", "-Xmx256m", "-jar", "app.jar"]
```

---
