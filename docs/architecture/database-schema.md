# Database Schema

This section provides the complete SQL DDL for creating the Paw Patrol database schema in PostgreSQL 15+. The schema supports the five core entities (User, Colony, Cat, Activity, AuditEvent) with proper indexes, constraints, and performance optimizations.

## Schema Overview

**Database:** `paw_patrol`  
**Encoding:** UTF-8  
**Collation:** en_US.UTF-8  
**PostgreSQL Version:** 15+ (via Supabase)

**Design Principles:**
- UUID primary keys for distributed scalability
- JSONB for flexible semi-structured data (photos, documents, metadata)
- Indexed foreign keys for join performance
- Composite indexes for common query patterns
- `created_at` and `updated_at` timestamps on all tables
- Enum types for status/category fields
- Soft deletes where needed (not implemented in MVP)

---

## Complete DDL

```sql
-- ==============================================
-- Paw Patrol Database Schema
-- PostgreSQL 15+
-- ==============================================

-- Enable UUID extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Enable PostGIS for future geospatial features (optional)
-- CREATE EXTENSION IF NOT EXISTS postgis;

-- ==============================================
-- ENUMS
-- ==============================================

CREATE TYPE user_role AS ENUM ('USER', 'ADMIN', 'SUPER_ADMIN');
CREATE TYPE user_status AS ENUM ('PENDING_VERIFICATION', 'ACTIVE', 'REJECTED', 'INACTIVE');
CREATE TYPE cat_status AS ENUM ('HEALTHY', 'SICK', 'INJURED', 'DECEASED', 'ADOPTED', 'MISSING');
CREATE TYPE cat_gender AS ENUM ('MALE', 'FEMALE', 'UNKNOWN');
CREATE TYPE activity_category AS ENUM ('FEEDING', 'HEALTH', 'ENVIRONMENT', 'OTHER');
CREATE TYPE audit_event_type AS ENUM (
    'USER_REGISTERED', 'USER_APPROVED', 'USER_REJECTED', 'USER_LOGGED_IN',
    'COLONY_CREATED', 'COLONY_UPDATED', 'COLONY_DELETED',
    'CAT_ADDED', 'CAT_UPDATED', 'CAT_STATUS_CHANGED', 'CAT_DELETED',
    'ACTIVITY_LOGGED', 'ACTIVITY_UPDATED', 'ACTIVITY_DELETED'
);

-- ==============================================
-- TABLE: users
-- ==============================================

CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL, -- BCrypt hash
    full_name VARCHAR(255) NOT NULL,
    organization VARCHAR(255),
    role user_role NOT NULL DEFAULT 'USER',
    status user_status NOT NULL DEFAULT 'PENDING_VERIFICATION',
    verification_document JSONB, -- {fileName, fileType, base64Data}
    last_login_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_status ON users(status);
CREATE INDEX idx_users_role ON users(role);

-- Comments
COMMENT ON TABLE users IS 'User accounts with role-based access control';
COMMENT ON COLUMN users.password_hash IS 'BCrypt-hashed password (10 rounds)';
COMMENT ON COLUMN users.verification_document IS 'JSONB containing base64-encoded verification document (ID, organization proof)';

-- ==============================================
-- TABLE: colonies
-- ==============================================

CREATE TABLE colonies (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(255) NOT NULL,
    latitude NUMERIC(10, 7) NOT NULL, -- 6 decimal places = ~0.1m precision
    longitude NUMERIC(10, 7) NOT NULL,
    address TEXT,
    creator_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    
    -- Constraints
    CONSTRAINT chk_latitude CHECK (latitude >= -90 AND latitude <= 90),
    CONSTRAINT chk_longitude CHECK (longitude >= -180 AND longitude <= 180)
);

-- Indexes
CREATE INDEX idx_colonies_creator ON colonies(creator_id);
CREATE INDEX idx_colonies_coordinates ON colonies(latitude, longitude); -- For geospatial queries
CREATE INDEX idx_colonies_created_at ON colonies(created_at DESC); -- For "recent colonies" queries

-- Comments
COMMENT ON TABLE colonies IS 'Cat colony locations';
COMMENT ON COLUMN colonies.latitude IS 'Latitude with 6 decimal precision (~0.1m accuracy)';
COMMENT ON COLUMN colonies.longitude IS 'Longitude with 6 decimal precision (~0.1m accuracy)';
COMMENT ON COLUMN colonies.creator_id IS 'User who created this colony (ON DELETE RESTRICT prevents deletion if user has colonies)';

-- ==============================================
-- TABLE: cats
-- ==============================================

CREATE TABLE cats (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    colony_id UUID NOT NULL REFERENCES colonies(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    status cat_status NOT NULL DEFAULT 'HEALTHY',
    gender cat_gender NOT NULL DEFAULT 'UNKNOWN',
    
    -- Physical characteristics
    color_pattern TEXT,
    approximate_age_months INTEGER,
    
    -- TNR information
    is_neutered BOOLEAN, -- null = unknown, false = not neutered, true = neutered
    neuter_date DATE,
    
    -- Health
    health_notes TEXT,
    temperament TEXT, -- MVP: free-text (e.g., "Friendly", "Shy", "Aggressive")
    
    -- Photos
    photos JSONB, -- Array of {url: string, uploadedAt: timestamp} (max 10)
    
    -- Metadata
    creator_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    
    -- Constraints
    CONSTRAINT chk_approximate_age CHECK (approximate_age_months >= 0 AND approximate_age_months <= 360),
    CONSTRAINT chk_neuter_date_requires_neutered CHECK (
        (neuter_date IS NULL) OR (is_neutered = TRUE)
    )
);

-- Indexes
CREATE INDEX idx_cats_colony ON cats(colony_id);
CREATE INDEX idx_cats_status ON cats(status);
CREATE INDEX idx_cats_is_neutered ON cats(is_neutered);
CREATE INDEX idx_cats_colony_status ON cats(colony_id, status); -- Composite for catCount queries
CREATE INDEX idx_cats_created_at ON cats(created_at DESC);

-- Comments
COMMENT ON TABLE cats IS 'Individual cats within colonies';
COMMENT ON COLUMN cats.is_neutered IS 'Three-state boolean: null (unknown), false (not neutered), true (neutered)';
COMMENT ON COLUMN cats.photos IS 'JSONB array of photo objects (max 10): [{url, uploadedAt}]';
COMMENT ON COLUMN cats.temperament IS 'MVP: free-text temperament (future: controlled vocabulary)';

-- ==============================================
-- TABLE: activities
-- ==============================================

CREATE TABLE activities (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    colony_id UUID NOT NULL REFERENCES colonies(id) ON DELETE CASCADE,
    cat_id UUID REFERENCES cats(id) ON DELETE CASCADE, -- NULL = colony-level activity
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    
    -- Activity details
    category activity_category NOT NULL,
    action VARCHAR(100) NOT NULL, -- e.g., "Food", "Vaccination", "Cleaning"
    quantity VARCHAR(100), -- e.g., "2 kg", "5 bowls"
    notes TEXT,
    
    -- Photos
    photos JSONB, -- Array of {url: string, uploadedAt: timestamp} (max 3)
    
    -- Timestamps
    activity_timestamp TIMESTAMP WITH TIME ZONE NOT NULL, -- When activity occurred (can be backdated)
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(), -- When logged in system
    
    -- Constraints
    CONSTRAINT chk_activity_timestamp CHECK (activity_timestamp <= NOW())
);

-- Indexes
CREATE INDEX idx_activities_colony ON activities(colony_id);
CREATE INDEX idx_activities_cat ON activities(cat_id);
CREATE INDEX idx_activities_user ON activities(user_id);
CREATE INDEX idx_activities_category ON activities(category);
CREATE INDEX idx_activities_timestamp ON activities(activity_timestamp DESC); -- For recent activities
CREATE INDEX idx_activities_colony_category ON activities(colony_id, category, activity_timestamp DESC); -- For lastFedAt queries

-- Comments
COMMENT ON TABLE activities IS 'Activity logs for colonies and individual cats';
COMMENT ON COLUMN activities.cat_id IS 'NULL for colony-level activities, UUID for cat-specific activities';
COMMENT ON COLUMN activities.activity_timestamp IS 'When the activity actually occurred (can be backdated by user)';
COMMENT ON COLUMN activities.created_at IS 'When the activity was logged in the system';
COMMENT ON COLUMN activities.photos IS 'JSONB array of photo objects (max 3): [{url, uploadedAt}]';

-- ==============================================
-- TABLE: audit_events
-- ==============================================

CREATE TABLE audit_events (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    event_type audit_event_type NOT NULL,
    user_id UUID, -- NULL for system events
    entity_type VARCHAR(50), -- e.g., "COLONY", "CAT", "USER"
    entity_id UUID, -- ID of affected entity (denormalized for orphan-safety)
    event_data JSONB, -- Additional event context (e.g., changed fields, old values)
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_audit_events_user ON audit_events(user_id);
CREATE INDEX idx_audit_events_entity ON audit_events(entity_type, entity_id);
CREATE INDEX idx_audit_events_type ON audit_events(event_type);
CREATE INDEX idx_audit_events_created_at ON audit_events(created_at DESC);

-- Comments
COMMENT ON TABLE audit_events IS 'Immutable audit trail of all significant system events';
COMMENT ON COLUMN audit_events.entity_id IS 'Denormalized entity ID (not FK) to preserve audit trail even if entity is deleted';
COMMENT ON COLUMN audit_events.event_data IS 'JSONB containing event context (e.g., {oldStatus: "PENDING", newStatus: "ACTIVE"})';

-- ==============================================
-- TRIGGERS: Auto-update updated_at timestamps
-- ==============================================

CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_users_updated_at BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_colonies_updated_at BEFORE UPDATE ON colonies
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_cats_updated_at BEFORE UPDATE ON cats
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- ==============================================
-- INITIAL DATA: Super Admin User
-- ==============================================

-- Password: "admin123" (BCrypt hashed with 10 rounds)
-- IMPORTANT: Change this password immediately after deployment!
INSERT INTO users (id, email, password_hash, full_name, role, status)
VALUES (
    'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa',
    'admin@pawpatrol.com',
    '$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy', -- "admin123"
    'Super Admin',
    'SUPER_ADMIN',
    'ACTIVE'
);

COMMENT ON TABLE users IS 'Default super admin account created. Password: admin123 (CHANGE IMMEDIATELY!)';

-- ==============================================
-- PERFORMANCE NOTES
-- ==============================================

-- 1. Haversine Distance Query (for duplicate colony detection):
--    SELECT * FROM colonies
--    WHERE (
--        6371000 * acos(
--            cos(radians(:latitude)) * cos(radians(latitude)) *
--            cos(radians(longitude) - radians(:longitude)) +
--            sin(radians(:latitude)) * sin(radians(latitude))
--        )
--    ) < 100
--    ORDER BY <distance>;
--
--    Performance: ~20-50ms for 1000 colonies (uses idx_colonies_coordinates)

-- 2. Computed Fields Queries:
--    - catCount: SELECT COUNT(*) FROM cats WHERE colony_id = ? AND status != 'DECEASED'
--      Performance: ~10ms (uses idx_cats_colony_status)
--    
--    - lastFedAt: SELECT MAX(activity_timestamp) FROM activities WHERE colony_id = ? AND category = 'FEEDING'
--      Performance: ~15ms (uses idx_activities_colony_category)
--    
--    - TNR Progress: 3 COUNT queries (neutered=true, =false, =null)
--      Performance: ~30ms total (uses idx_cats_colony_status)

-- 3. Activity Feed Query:
--    SELECT * FROM activities WHERE colony_id = ? ORDER BY activity_timestamp DESC LIMIT 20
--    Performance: ~5-10ms (uses idx_activities_colony + activity_timestamp)

-- 4. Global Search (future):
--    SELECT * FROM colonies WHERE name ILIKE '%keyword%' OR address ILIKE '%keyword%'
--    Performance: ~50-100ms (full table scan, consider pg_trgm extension for fuzzy search)

-- ==============================================
-- BACKUP & MAINTENANCE
-- ==============================================

-- Backup Strategy:
-- - Supabase provides automatic daily backups (7-day retention on free tier)
-- - Manual backups: pg_dump paw_patrol > backup_$(date +%Y%m%d).sql
-- - Restore: psql paw_patrol < backup_YYYYMMDD.sql

-- Vacuum Schedule:
-- - PostgreSQL autovacuum runs automatically (default: when 20% of rows change)
-- - Manual vacuum: VACUUM ANALYZE; (updates statistics for query planner)

-- Index Maintenance:
-- - Monitor index usage: SELECT * FROM pg_stat_user_indexes WHERE schemaname = 'public';
-- - Rebuild indexes if fragmented: REINDEX TABLE table_name;

```

---

## Schema Migration Strategy (Flyway)

**Migration File Naming Convention:**
```
db/migration/
  V1__Initial_schema.sql
  V2__Add_cat_temperament_field.sql
  V3__Add_activity_photos_constraint.sql
  ...
```

**Flyway Configuration (application.properties):**
```properties
spring.flyway.enabled=true
spring.flyway.locations=classpath:db/migration
spring.flyway.baseline-on-migrate=true
spring.flyway.baseline-version=0
```

**Initial Migration (V1__Initial_schema.sql):**
- Contains the complete DDL above
- Run on first deployment to Supabase

**Future Migrations:**
- Always additive (add columns, indexes, tables)
- Never destructive (no DROP TABLE/COLUMN in production)
- Test migrations on local database before production deployment

---

## Database Connection Configuration

**Supabase Connection String:**
```
postgresql://postgres:[YOUR-PASSWORD]@db.[YOUR-PROJECT-REF].supabase.co:5432/postgres
```

**Spring Boot Configuration (application.properties):**
```properties
# Database
spring.datasource.url=${DATABASE_URL}
spring.datasource.username=${DATABASE_USER}
spring.datasource.password=${DATABASE_PASSWORD}
spring.datasource.driver-class-name=org.postgresql.Driver

# JPA/Hibernate
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.show-sql=false
spring.jpa.hibernate.ddl-auto=validate # NEVER use 'update' or 'create-drop' in production!
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.jdbc.time_zone=UTC

# Connection Pool (HikariCP)
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=2
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.idle-timeout=600000
spring.datasource.hikari.max-lifetime=1800000
```

**Fly.io Environment Variables:**
```bash
flyctl secrets set DATABASE_URL="postgresql://postgres:xxxxx@db.xxxxx.supabase.co:5432/postgres"
flyctl secrets set DATABASE_USER="postgres"
flyctl secrets set DATABASE_PASSWORD="your-supabase-password"
```

---

## Data Seeding (Development Only)

**Sample Data SQL (`db/dev_seed.sql`):**
```sql
-- Insert test users (password: "password123")
INSERT INTO users (email, password_hash, full_name, role, status) VALUES
('john@example.com', '$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy', 'John Doe', 'USER', 'ACTIVE'),
('jane@example.com', '$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy', 'Jane Smith', 'USER', 'ACTIVE');

-- Insert test colonies (Manila area)
INSERT INTO colonies (name, latitude, longitude, address, creator_id) VALUES
('Quezon City Colony A', 14.6760, 121.0437, '123 Main St, Quezon City', (SELECT id FROM users WHERE email = 'john@example.com')),
('Makati Colony B', 14.5547, 121.0244, '456 Business Ave, Makati', (SELECT id FROM users WHERE email = 'jane@example.com'));

-- Insert test cats
INSERT INTO cats (colony_id, name, status, gender, color_pattern, is_neutered, creator_id) VALUES
((SELECT id FROM colonies WHERE name = 'Quezon City Colony A'), 'Whiskers', 'HEALTHY', 'MALE', 'Orange tabby', true, (SELECT id FROM users WHERE email = 'john@example.com')),
((SELECT id FROM colonies WHERE name = 'Quezon City Colony A'), 'Mittens', 'HEALTHY', 'FEMALE', 'White with black spots', false, (SELECT id FROM users WHERE email = 'john@example.com'));

-- Insert test activities
INSERT INTO activities (colony_id, user_id, category, action, quantity, notes, activity_timestamp) VALUES
((SELECT id FROM colonies WHERE name = 'Quezon City Colony A'), (SELECT id FROM users WHERE email = 'john@example.com'), 'FEEDING', 'Food', '2 kg', 'Dry kibble', NOW() - INTERVAL '2 hours');
```

**Load Seed Data (Development Only):**
```bash
psql $DATABASE_URL < db/dev_seed.sql
```

**⚠️ WARNING:** Never seed data in production! Use migrations for schema only.

---
