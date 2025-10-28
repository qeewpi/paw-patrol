# Security Checklist

- ✅ HTTPS enforced (Fly.io automatic)
- ✅ JWT tokens with expiration
- ✅ BCrypt password hashing (10 rounds)
- ✅ SQL injection prevention (JPA parameterized queries)
- ✅ XSS prevention (React escapes by default)
- ✅ CSRF protection (disabled for stateless JWT API)
- ✅ CORS configured (only allow frontend domain)
- ✅ Rate limiting (Spring Security + Fly.io)
- ✅ Input validation (Bean Validation + Zod)
- ✅ Role-based access control (USER, ADMIN, SUPER_ADMIN)
- ✅ Audit logging (all sensitive operations)
- ✅ Secrets management (Fly.io secrets, never in git)

---
