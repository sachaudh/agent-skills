---
name: security-and-hardening
description: Hardens frontend code against vulnerabilities in React/TypeScript applications. Use when handling user input, authentication tokens, data rendering, or external integrations. Use when building any feature that accepts untrusted data, manages sessions, or renders dynamic content in a StackRox UI context.
---

# Security and Hardening

## Overview

Security-first development practices for React/TypeScript frontend applications. Treat every external input as hostile, every auth token as sacred, and every rendered value as potentially malicious. Security isn't a phase -- it's a constraint on every component that touches user data, authentication, or external systems.

## When to Use

- Building anything that accepts user input
- Implementing authentication or authorization
- Storing or transmitting sensitive data
- Integrating with external APIs or services
- Adding file uploads, webhooks, or callbacks
- Handling payment or PII data

## The Three-Tier Boundary System

### Always Do (No Exceptions)

- **Validate all external input** at the system boundary (form handlers, URL params, GraphQL responses)
- **Use Formik + Yup** for form validation -- never write ad-hoc validation logic
- **Encode output** to prevent XSS (React auto-escapes JSX; never bypass it)
- **Use HTTPS** for all external communication
- **Set security headers** (CSP, HSTS, X-Frame-Options, X-Content-Type-Options)
- **Use httpOnly, secure, sameSite cookies** for auth tokens
- **Run `npm audit`** (or equivalent) before every release
- **Sanitize before `dangerouslySetInnerHTML`** -- always use DOMPurify, never raw HTML strings

### Ask First (Requires Human Approval)

- Adding new authentication flows or changing auth logic
- Storing new categories of sensitive data (PII, payment info)
- Adding new external service integrations
- Changing CORS configuration
- Adding file upload handlers
- Modifying rate limiting or throttling
- Granting elevated permissions or roles

### Never Do

- **Never commit secrets** to version control (API keys, passwords, tokens)
- **Never log sensitive data** (passwords, tokens, PII)
- **Never trust client-side validation** as a security boundary
- **Never disable security headers** for convenience
- **Never use `eval()` or `dangerouslySetInnerHTML`** with unsanitized user data
- **Never store auth tokens in localStorage or sessionStorage** -- use httpOnly cookies
- **Never expose stack traces** or internal error details to users
- **Never store sensitive data in React state** that persists across routes without cleanup

## OWASP Top 10 Prevention

### 1. Injection (SQL, NoSQL, OS Command)

```typescript
// BAD: SQL injection via string concatenation
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// GOOD: Parameterized query
const user = await db.query('SELECT * FROM users WHERE id = $1', [userId]);

// GOOD: ORM with parameterized input
const user = await prisma.user.findUnique({ where: { id: userId } });
```

### 2. Broken Authentication

```typescript
// Password hashing
import { hash, compare } from 'bcrypt';

const SALT_ROUNDS = 12;
const hashedPassword = await hash(plaintext, SALT_ROUNDS);
const isValid = await compare(plaintext, hashedPassword);

// Session management
app.use(session({
  secret: process.env.SESSION_SECRET,  // From environment, not code
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,     // Not accessible via JavaScript
    secure: true,       // HTTPS only
    sameSite: 'lax',    // CSRF protection
    maxAge: 24 * 60 * 60 * 1000,  // 24 hours
  },
}));
```

### 3. Cross-Site Scripting (XSS)

React auto-escapes JSX expressions by default. XSS vulnerabilities in React come from explicitly bypassing this protection.

```typescript
// BAD: dangerouslySetInnerHTML with raw user input
return <div dangerouslySetInnerHTML={{ __html: userInput }} />;

// BAD: Setting href with user-controlled protocol
return <a href={userProvidedUrl}>Link</a>;  // javascript: protocol executes

// GOOD: React auto-escaping (default behavior)
return <div>{userInput}</div>;

// GOOD: Sanitize when dangerouslySetInnerHTML is unavoidable
import DOMPurify from 'dompurify';
return <div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />;

// GOOD: Validate URL protocols
function isSafeUrl(url: string): boolean {
  try {
    const parsed = new URL(url);
    return ['http:', 'https:'].includes(parsed.protocol);
  } catch {
    return false;
  }
}
```

**React-specific XSS vectors to watch for:**
- `dangerouslySetInnerHTML` -- always sanitize with DOMPurify
- `href` and `src` attributes with user input -- validate protocol
- Server-rendered content injected into client hydration
- CSS injection via `style` props with user-controlled values

### CSP for the StackRox Embedded Console

The StackRox UI runs as an embedded console. CSP must account for:

```
Content-Security-Policy:
  default-src 'self';
  script-src 'self';
  style-src 'self' 'unsafe-inline';    # PatternFly requires inline styles
  img-src 'self' data:;                # Data URIs for icons
  connect-src 'self' wss:;             # WebSocket for live updates
  font-src 'self';
  frame-ancestors 'self';              # Prevent clickjacking in embedded context
```

When adding new external resources (CDN fonts, analytics, etc.), update the CSP rather than weakening it with wildcards.

### 4. Broken Access Control

```typescript
// Always check authorization, not just authentication
app.patch('/api/tasks/:id', authenticate, async (req, res) => {
  const task = await taskService.findById(req.params.id);

  // Check that the authenticated user owns this resource
  if (task.ownerId !== req.user.id) {
    return res.status(403).json({
      error: { code: 'FORBIDDEN', message: 'Not authorized to modify this task' }
    });
  }

  // Proceed with update
  const updated = await taskService.update(req.params.id, req.body);
  return res.json(updated);
});
```

### 5. Security Misconfiguration

```typescript
// Security headers (use helmet for Express)
import helmet from 'helmet';
app.use(helmet());

// Content Security Policy
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
    styleSrc: ["'self'", "'unsafe-inline'"],  // Tighten if possible
    imgSrc: ["'self'", 'data:', 'https:'],
    connectSrc: ["'self'"],
  },
}));

// CORS — restrict to known origins
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || 'http://localhost:3000',
  credentials: true,
}));
```

### 6. Sensitive Data Exposure

```typescript
// Never return sensitive fields in API responses
function sanitizeUser(user: UserRecord): PublicUser {
  const { passwordHash, resetToken, ...publicFields } = user;
  return publicFields;
}

// Use environment variables for secrets
const API_KEY = process.env.STRIPE_API_KEY;
if (!API_KEY) throw new Error('STRIPE_API_KEY not configured');
```

## Input Validation Patterns

### Formik + Yup Schema Validation

Use Formik for form state and Yup for schema validation. This is the standard pattern for StackRox frontend forms.

```typescript
import { Formik, Form, Field, ErrorMessage } from 'formik';
import * as Yup from 'yup';

const CreatePolicySchema = Yup.object({
  name: Yup.string()
    .required('Policy name is required')
    .min(1)
    .max(200)
    .trim(),
  description: Yup.string().max(2000),
  severity: Yup.string()
    .oneOf(['LOW', 'MEDIUM', 'HIGH', 'CRITICAL'])
    .required('Severity is required'),
  categories: Yup.array()
    .of(Yup.string().required())
    .min(1, 'Select at least one category'),
});

type CreatePolicyValues = Yup.InferType<typeof CreatePolicySchema>;

function CreatePolicyForm() {
  const initialValues: CreatePolicyValues = {
    name: '',
    description: '',
    severity: 'MEDIUM',
    categories: [],
  };

  return (
    <Formik
      initialValues={initialValues}
      validationSchema={CreatePolicySchema}
      onSubmit={(values) => {
        // values are typed and validated by Yup
        createPolicy(values);
      }}
    >
      <Form>
        <Field name="name" />
        <ErrorMessage name="name" component="div" />
        {/* ... */}
      </Form>
    </Formik>
  );
}
```

### GraphQL Input Validation

GraphQL typed variables provide a layer of validation, but always validate on the client side before sending mutations:

```typescript
// Yup schema mirrors the GraphQL input type
const UpdateDeploymentSchema = Yup.object({
  id: Yup.string().required(),
  replicas: Yup.number().min(0).max(100).required(),
  namespace: Yup.string().required(),
});

// Validate before executing the mutation
const validatedInput = await UpdateDeploymentSchema.validate(formValues);
await updateDeployment({ variables: { input: validatedInput } });
```

## Secure Auth Token Handling in React

StackRox uses token-based authentication. The frontend must handle tokens without exposing them to XSS attacks.

### Token Storage

```typescript
// BAD: Storing tokens in localStorage (accessible via XSS)
localStorage.setItem('authToken', token);

// BAD: Storing tokens in React state (lost on refresh, tempting to persist unsafely)
const [token, setToken] = useState(response.token);

// GOOD: httpOnly cookies set by the backend (not accessible via JavaScript)
// The frontend doesn't handle the token at all -- it's sent automatically by the browser
```

### Apollo Client Token Refresh

```typescript
import { ApolloClient, InMemoryCache, createHttpLink, from } from '@apollo/client';
import { onError } from '@apollo/client/link/error';

const errorLink = onError(({ graphQLErrors, networkError }) => {
  if (graphQLErrors) {
    for (const err of graphQLErrors) {
      if (err.extensions?.code === 'UNAUTHENTICATED') {
        // Redirect to login -- do NOT attempt to refresh tokens client-side
        // unless the backend provides a secure refresh endpoint with httpOnly cookies
        window.location.href = '/login';
      }
    }
  }
});

const httpLink = createHttpLink({
  uri: '/api/graphql',
  credentials: 'same-origin',  // Include httpOnly cookies automatically
});

const client = new ApolloClient({
  link: from([errorLink, httpLink]),
  cache: new InMemoryCache(),
});
```

### Sensitive Data Cleanup

```typescript
// Clean up sensitive data when components unmount or users navigate away
useEffect(() => {
  return () => {
    // Clear any sensitive data from component state
    setFormData(initialValues);
  };
}, []);

// Clear Apollo cache on logout
async function handleLogout() {
  await client.clearStore();
  window.location.href = '/login';
}
```

## Triaging npm audit Results

Not all audit findings require immediate action. Use this decision tree:

```
npm audit reports a vulnerability
├── Severity: critical or high
│   ├── Is the vulnerable code reachable in your app?
│   │   ├── YES --> Fix immediately (update, patch, or replace the dependency)
│   │   └── NO (dev-only dep, unused code path) --> Fix soon, but not a blocker
│   └── Is a fix available?
│       ├── YES --> Update to the patched version
│       └── NO --> Check for workarounds, consider replacing the dependency, or add to allowlist with a review date
├── Severity: moderate
│   ├── Reachable in production? --> Fix in the next release cycle
│   └── Dev-only? --> Fix when convenient, track in backlog
└── Severity: low
    └── Track and fix during regular dependency updates
```

**Key questions:**
- Is the vulnerable function actually called in your code path?
- Is the dependency a runtime dependency or dev-only?
- Is the vulnerability exploitable given your deployment context (e.g., a server-side vulnerability in a client-only app)?

When you defer a fix, document the reason and set a review date.

## Backend Reference

The following are backend concerns. The StackRox frontend does not implement these directly, but frontend engineers should understand them when debugging auth or API issues.

### Rate Limiting (Backend)

Rate limiting is enforced server-side. The frontend should handle 429 (Too Many Requests) responses gracefully:

```typescript
// Handle rate limiting in Apollo error link
if (networkError && 'statusCode' in networkError && networkError.statusCode === 429) {
  // Show user-friendly message, do NOT auto-retry aggressively
  showAlert('Too many requests. Please wait a moment and try again.');
}
```

### Password Hashing (Backend)

Password hashing (bcrypt/scrypt/argon2) is a backend responsibility. The frontend should:
- Never send passwords in URL query parameters
- Clear password fields after form submission
- Use `type="password"` and `autocomplete="current-password"` attributes

## Secrets Management

```
.env files:
  ├── .env.example  → Committed (template with placeholder values)
  ├── .env          → NOT committed (contains real secrets)
  └── .env.local    → NOT committed (local overrides)

.gitignore must include:
  .env
  .env.local
  .env.*.local
  *.pem
  *.key
```

**Always check before committing:**
```bash
# Check for accidentally staged secrets
git diff --cached | grep -i "password\|secret\|api_key\|token"
```

## Security Review Checklist

```markdown
### Authentication
- [ ] Passwords hashed with bcrypt/scrypt/argon2 (salt rounds ≥ 12)
- [ ] Session tokens are httpOnly, secure, sameSite
- [ ] Login has rate limiting
- [ ] Password reset tokens expire

### Authorization
- [ ] Every endpoint checks user permissions
- [ ] Users can only access their own resources
- [ ] Admin actions require admin role verification

### Input
- [ ] All user input validated at the boundary
- [ ] SQL queries are parameterized
- [ ] HTML output is encoded/escaped

### Data
- [ ] No secrets in code or version control
- [ ] Sensitive fields excluded from API responses
- [ ] PII encrypted at rest (if applicable)

### Infrastructure
- [ ] Security headers configured (CSP, HSTS, etc.)
- [ ] CORS restricted to known origins
- [ ] Dependencies audited for vulnerabilities
- [ ] Error messages don't expose internals
```
## See Also

For detailed security checklists and pre-commit verification steps, see `references/security-checklist.md`.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "This is an internal tool, security doesn't matter" | Internal tools get compromised. Attackers target the weakest link. |
| "We'll add security later" | Security retrofitting is 10x harder than building it in. Add it now. |
| "No one would try to exploit this" | Automated scanners will find it. Security by obscurity is not security. |
| "The framework handles security" | Frameworks provide tools, not guarantees. You still need to use them correctly. |
| "It's just a prototype" | Prototypes become production. Security habits from day one. |

## Red Flags

- User input passed directly to database queries, shell commands, or HTML rendering
- Secrets in source code or commit history
- API endpoints without authentication or authorization checks
- Missing CORS configuration or wildcard (`*`) origins
- No rate limiting on authentication endpoints
- Stack traces or internal errors exposed to users
- Dependencies with known critical vulnerabilities

## Verification

After implementing security-relevant code:

- [ ] `npm audit` shows no critical or high vulnerabilities
- [ ] No secrets in source code or git history
- [ ] All user input validated at system boundaries
- [ ] Authentication and authorization checked on every protected endpoint
- [ ] Security headers present in response (check with browser DevTools)
- [ ] Error responses don't expose internal details
- [ ] Rate limiting active on auth endpoints
