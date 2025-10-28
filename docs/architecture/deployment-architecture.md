# Deployment Architecture

## Fly.io Deployment Configuration

**fly.toml:**

```toml
app = "paw-patrol"
primary_region = "sin" # Singapore (closest to Philippines)

[build]
  dockerfile = "Dockerfile"

[env]
  PORT = "8080"
  SPRING_PROFILES_ACTIVE = "prod"

[[services]]
  internal_port = 8080
  protocol = "tcp"
  auto_stop_machines = true
  auto_start_machines = true
  min_machines_running = 0 # Scale to zero when idle

  [[services.ports]]
    port = 80
    handlers = ["http"]
    force_https = true

  [[services.ports]]
    port = 443
    handlers = ["tls", "http"]

  [[services.http_checks]]
    interval = "30s"
    timeout = "5s"
    grace_period = "10s"
    method = "GET"
    path = "/actuator/health"

[mounts]
  source = "paw_patrol_photos"
  destination = "/app/photos"
```

**Multi-Stage Dockerfile:**

```dockerfile
# Stage 1: Build frontend
FROM node:20-alpine AS frontend-builder
WORKDIR /build
COPY package*.json ./
COPY frontend/package*.json ./frontend/
COPY shared/package*.json ./shared/
RUN npm ci --production=false
COPY frontend ./frontend
COPY shared ./shared
RUN npm run build -w frontend

# Stage 2: Build backend
FROM maven:3.9-eclipse-temurin-17 AS backend-builder
WORKDIR /build
COPY backend/pom.xml ./
RUN mvn dependency:go-offline
COPY backend/src ./src
RUN mvn package -DskipTests

# Stage 3: Runtime
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Copy backend JAR
COPY --from=backend-builder /build/target/*.jar app.jar

# Copy frontend build (served as static files by Spring Boot)
COPY --from=frontend-builder /build/frontend/dist ./public

# Create photos directory
RUN mkdir -p /app/photos

# Set memory limits
ENV JAVA_OPTS="-Xmx256m -Xms128m"

EXPOSE 8080

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

---

## Environment Variables

**Required Secrets (set via `flyctl secrets set`):**

```bash
# Database
flyctl secrets set DATABASE_URL="postgresql://postgres:xxxxx@db.xxxxx.supabase.co:5432/postgres"
flyctl secrets set DATABASE_USER="postgres"
flyctl secrets set DATABASE_PASSWORD="your-supabase-password"

# JWT
flyctl secrets set JWT_SECRET="your-256-bit-secret-key-here"
flyctl secrets set JWT_EXPIRATION="604800000" # 7 days in ms
flyctl secrets set JWT_REFRESH_EXPIRATION="2592000000" # 30 days

# Google Maps
flyctl secrets set GOOGLE_MAPS_API_KEY="AIzaSyYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY"

# Frontend (public)
flyctl secrets set VITE_API_BASE_URL="https://paw-patrol.fly.dev/api"
flyctl secrets set VITE_GOOGLE_MAPS_API_KEY="AIzaSyXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
flyctl secrets set VITE_GOOGLE_MAPS_MAP_ID="your-map-id"
```

---

## Deployment Workflow

**Initial Deployment:**

```bash
# 1. Install Fly CLI
brew install flyctl # or: curl -L https://fly.io/install.sh | sh

# 2. Login
flyctl auth login

# 3. Launch app (creates fly.toml)
flyctl launch --name paw-patrol --region sin --no-deploy

# 4. Create volume for photos
flyctl volumes create paw_patrol_photos --region sin --size 3

# 5. Set secrets (all of the above)
flyctl secrets set DATABASE_URL="..."
# ... (set all secrets)

# 6. Deploy
flyctl deploy

# 7. Verify
flyctl status
flyctl logs
```

**Continuous Deployment:**

```bash
# Deploy new version
flyctl deploy

# Rollback if needed
flyctl releases
flyctl releases rollback <version>
```

---
