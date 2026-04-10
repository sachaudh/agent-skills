# Security Checklist

Quick reference for frontend security in React/TypeScript applications. Use alongside the `security-and-hardening` skill. Focused on StackRox UI concerns -- browser-side vulnerabilities, auth token handling, and PatternFly component security.

## Table of Contents

- [Pre-Commit Checks](#pre-commit-checks)
- [Authentication](#authentication)
- [Authorization](#authorization)
- [Input Validation](#input-validation)
- [React/Frontend-Specific](#reactfrontend-specific)
- [Security Headers (Backend)](#security-headers-backend)
- [CORS Configuration (Backend)](#cors-configuration-backend)
- [Data Protection (Backend)](#data-protection-backend)
- [Dependency Security](#dependency-security)
- [Error Handling](#error-handling)
- [OWASP Top 10 Quick Reference](#owasp-top-10-quick-reference)

## Pre-Commit Checks

- [ ] No secrets in code (`git diff --cached | grep -i "password\|secret\|api_key\|token"`)
- [ ] `.gitignore` covers: `.env`, `.env.local`, `*.pem`, `*.key`
- [ ] `.env.example` uses placeholder values (not real secrets)

## Authentication

- [ ] Auth tokens stored in httpOnly cookies (never localStorage or sessionStorage)
- [ ] Apollo Client uses `credentials: 'same-origin'` for automatic cookie inclusion
- [ ] Apollo error link handles `UNAUTHENTICATED` errors with redirect to login
- [ ] `client.clearStore()` called on logout to purge cached authenticated data
- [ ] Password fields use `type="password"` and appropriate `autocomplete` attributes
- [ ] Password fields cleared after form submission
- [ ] Login form does not send credentials in URL query parameters
- [ ] _Backend:_ Session cookies set with `httpOnly`, `secure`, `sameSite: 'lax'`
- [ ] _Backend:_ Rate limiting on login endpoint (<=10 attempts per 15 minutes)

## Authorization

- [ ] Every protected endpoint checks authentication
- [ ] Every resource access checks ownership/role (prevents IDOR)
- [ ] Admin endpoints require admin role verification
- [ ] API keys scoped to minimum necessary permissions
- [ ] JWT tokens validated (signature, expiration, issuer)

## Input Validation

- [ ] All forms use Formik + Yup for validation (no ad-hoc validation logic)
- [ ] Yup schemas define allowlists for enum-like fields (`oneOf`)
- [ ] String lengths constrained with `min()` / `max()`
- [ ] Numeric ranges validated with `min()` / `max()`
- [ ] GraphQL mutations use typed input variables (not string interpolation)
- [ ] Client-side validation mirrors GraphQL schema constraints
- [ ] URLs validated before use in `href` or `src` (allowlist protocols: `http:`, `https:`)
- [ ] React auto-escaping not bypassed (no `dangerouslySetInnerHTML` without DOMPurify)
- [ ] URL params parsed with type checking before use in queries

## React/Frontend-Specific

- [ ] No `dangerouslySetInnerHTML` without DOMPurify sanitization
- [ ] No `eval()`, `new Function()`, or dynamic script injection
- [ ] User-controlled values never used in `style` props without sanitization (CSS injection)
- [ ] Sensitive data cleared from React state on component unmount (`useEffect` cleanup)
- [ ] Auth-protected routes use route guards, not conditional rendering alone
- [ ] Error boundaries catch rendering errors without exposing internal details to users
- [ ] No secrets or API keys in client-side code (check `import.meta.env` usage)
- [ ] GraphQL error responses handled without exposing server internals to users
- [ ] File downloads validated and sanitized before triggering browser download

## Security Headers (Backend)

```
Content-Security-Policy: default-src 'self'; script-src 'self'
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 0  (disabled, rely on CSP)
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

## CORS Configuration (Backend)

```typescript
// Restrictive (recommended)
cors({
  origin: ['https://yourdomain.com', 'https://app.yourdomain.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
})

// NEVER use in production:
cors({ origin: '*' })  // Allows any origin
```

## Data Protection (Backend)

- [ ] Sensitive fields excluded from API responses (`passwordHash`, `resetToken`, etc.)
- [ ] Sensitive data not logged (passwords, tokens, full CC numbers)
- [ ] PII encrypted at rest (if required by regulation)
- [ ] HTTPS for all external communication
- [ ] Database backups encrypted

## Dependency Security

```bash
# Audit dependencies
npm audit

# Fix automatically where possible
npm audit fix

# Check for critical vulnerabilities
npm audit --audit-level=critical

# Keep dependencies updated
npx npm-check-updates
```

## Error Handling

```typescript
// Production: generic error, no internals
res.status(500).json({
  error: { code: 'INTERNAL_ERROR', message: 'Something went wrong' }
});

// NEVER in production:
res.status(500).json({
  error: err.message,
  stack: err.stack,         // Exposes internals
  query: err.sql,           // Exposes database details
});
```

## OWASP Top 10 Quick Reference

| # | Vulnerability | Prevention |
|---|---|---|
| 1 | Broken Access Control | Auth checks on every endpoint, ownership verification |
| 2 | Cryptographic Failures | HTTPS, strong hashing, no secrets in code |
| 3 | Injection | Parameterized queries, input validation |
| 4 | Insecure Design | Threat modeling, spec-driven development |
| 5 | Security Misconfiguration | Security headers, minimal permissions, audit deps |
| 6 | Vulnerable Components | `npm audit`, keep deps updated, minimal deps |
| 7 | Auth Failures | Strong passwords, rate limiting, session management |
| 8 | Data Integrity Failures | Verify updates/dependencies, signed artifacts |
| 9 | Logging Failures | Log security events, don't log secrets |
| 10 | SSRF | Validate/allowlist URLs, restrict outbound requests |
