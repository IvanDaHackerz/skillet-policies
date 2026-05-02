# Security Guidelines

These security guidelines must be followed by all skills to ensure safe and secure code generation.

---

## 1. Authentication & Authorization

### Never Hardcode Credentials
**❌ NEVER DO THIS**:
```javascript
const apiKey = 'sk-1234567890abcdef';
const dbPassword = 'mypassword123';
const jwtSecret = 'supersecret';
```

**✅ ALWAYS DO THIS**:
```javascript
const apiKey = process.env.API_KEY;
const dbPassword = process.env.DB_PASSWORD;
const jwtSecret = process.env.JWT_SECRET;
```

### Password Security
- **Hash passwords** with bcrypt (minimum 10 rounds)
- **Never store** plain text passwords
- **Never log** passwords or sensitive data
- **Implement** password strength requirements

```javascript
import bcrypt from 'bcrypt';

const hashPassword = async (password) => {
  const saltRounds = 10;
  return await bcrypt.hash(password, saltRounds);
};

const verifyPassword = async (password, hash) => {
  return await bcrypt.compare(password, hash);
};
```

### JWT Token Security
- Use strong secrets (min 32 characters)
- Set appropriate expiration times
- Implement token refresh mechanism
- Store tokens securely (httpOnly cookies)

```javascript
const token = jwt.sign(
  { userId: user.id },
  process.env.JWT_SECRET,
  { expiresIn: '1h' }
);
```

---

## 2. Input Validation & Sanitization

### Always Validate User Input
**Every** user input must be validated before processing.

```javascript
import { z } from 'zod';

const userSchema = z.object({
  email: z.string().email(),
  age: z.number().min(0).max(150),
  username: z.string().min(3).max(20).regex(/^[a-zA-Z0-9_]+$/)
});
```

### Sanitize HTML Input
Prevent XSS attacks by sanitizing HTML:

```javascript
import DOMPurify from 'isomorphic-dompurify';

const sanitizedContent = DOMPurify.sanitize(userInput);
```

### File Upload Security
- Validate file types (whitelist, not blacklist)
- Limit file sizes
- Scan for malware
- Store files outside web root
- Use random filenames

```javascript
const allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];
const maxSize = 5 * 1024 * 1024; // 5MB

if (!allowedTypes.includes(file.mimetype)) {
  throw new ValidationError('Invalid file type');
}

if (file.size > maxSize) {
  throw new ValidationError('File too large');
}
```

---

## 3. SQL Injection Prevention

### Use Parameterized Queries
**❌ NEVER DO THIS**:
```javascript
const query = `SELECT * FROM users WHERE email = '${userEmail}'`;
db.query(query);
```

**✅ ALWAYS DO THIS**:
```javascript
const query = 'SELECT * FROM users WHERE email = ?';
db.query(query, [userEmail]);
```

### Use ORM/Query Builders
Prefer ORMs that handle parameterization:

```javascript
// Sequelize
const user = await User.findOne({ where: { email: userEmail } });

// Prisma
const user = await prisma.user.findUnique({ where: { email: userEmail } });
```

---

## 4. Cross-Site Scripting (XSS) Prevention

### Escape Output
Always escape user-generated content before rendering:

```javascript
// React automatically escapes
<div>{userInput}</div>

// For dangerouslySetInnerHTML, sanitize first
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(html) }} />
```

### Content Security Policy (CSP)
Implement CSP headers:

```javascript
app.use((req, res, next) => {
  res.setHeader(
    'Content-Security-Policy',
    "default-src 'self'; script-src 'self' 'unsafe-inline'"
  );
  next();
});
```

---

## 5. Cross-Site Request Forgery (CSRF) Prevention

### Use CSRF Tokens
```javascript
import csrf from 'csurf';

const csrfProtection = csrf({ cookie: true });
app.use(csrfProtection);

app.get('/form', (req, res) => {
  res.render('form', { csrfToken: req.csrfToken() });
});
```

### SameSite Cookies
```javascript
res.cookie('token', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict'
});
```

---

## 6. Rate Limiting & DDoS Protection

### Implement Rate Limiting
```javascript
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests, please try again later'
});

app.use('/api/', limiter);
```

### Stricter Limits for Sensitive Endpoints
```javascript
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5, // 5 attempts per 15 minutes
  skipSuccessfulRequests: true
});

app.post('/api/auth/login', authLimiter, loginController);
```

---

## 7. Secure Headers

### Use Helmet.js
```javascript
import helmet from 'helmet';

app.use(helmet());
```

This sets multiple security headers:
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `X-XSS-Protection: 1; mode=block`
- `Strict-Transport-Security`

---

## 8. Error Handling & Information Disclosure

### Never Expose Stack Traces
**❌ NEVER DO THIS**:
```javascript
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.stack });
});
```

**✅ ALWAYS DO THIS**:
```javascript
app.use((err, req, res, next) => {
  logger.error(err.stack); // Log internally
  
  res.status(err.statusCode || 500).json({
    success: false,
    error: {
      message: err.isOperational ? err.message : 'Internal server error'
    }
  });
});
```

### Sanitize Error Messages
Don't expose sensitive information in error messages:

```javascript
// Bad
throw new Error(`User ${email} not found in database table users`);

// Good
throw new Error('User not found');
```

---

## 9. Dependency Security

### Keep Dependencies Updated
```bash
npm audit
npm audit fix
```

### Use Snyk or Dependabot
- Scan for vulnerabilities regularly
- Update dependencies promptly
- Review security advisories

### Lock Dependencies
Use `package-lock.json` or `yarn.lock`:
```bash
npm ci  # Use in CI/CD instead of npm install
```

---

## 10. Logging & Monitoring

### Log Security Events
```javascript
logger.info('User login attempt', { userId, ip: req.ip });
logger.warn('Failed login attempt', { email, ip: req.ip });
logger.error('Unauthorized access attempt', { userId, resource, ip: req.ip });
```

### Never Log Sensitive Data
**❌ NEVER LOG**:
- Passwords
- API keys
- Credit card numbers
- Personal identification numbers
- Session tokens

```javascript
// Bad
logger.info('User created', { user }); // May contain password

// Good
logger.info('User created', { userId: user.id, email: user.email });
```

---

## 11. API Security

### Use HTTPS
- **Always** use HTTPS in production
- Redirect HTTP to HTTPS
- Use HSTS headers

```javascript
if (process.env.NODE_ENV === 'production') {
  app.use((req, res, next) => {
    if (!req.secure) {
      return res.redirect('https://' + req.headers.host + req.url);
    }
    next();
  });
}
```

### CORS Configuration
Be specific with CORS origins:

```javascript
import cors from 'cors';

const corsOptions = {
  origin: process.env.ALLOWED_ORIGINS?.split(',') || 'http://localhost:3000',
  credentials: true,
  optionsSuccessStatus: 200
};

app.use(cors(corsOptions));
```

### API Versioning
```javascript
app.use('/api/v1', v1Routes);
app.use('/api/v2', v2Routes);
```

---

## 12. Session Security

### Secure Session Configuration
```javascript
import session from 'express-session';

app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: true, // HTTPS only
    httpOnly: true, // No JavaScript access
    maxAge: 3600000, // 1 hour
    sameSite: 'strict'
  }
}));
```

---

## 13. File System Security

### Prevent Path Traversal
```javascript
import path from 'path';

const sanitizePath = (userPath) => {
  const normalized = path.normalize(userPath);
  const basePath = path.resolve('./uploads');
  const fullPath = path.resolve(basePath, normalized);
  
  if (!fullPath.startsWith(basePath)) {
    throw new Error('Invalid path');
  }
  
  return fullPath;
};
```

### Secure File Permissions
```javascript
import fs from 'fs';

fs.writeFileSync('config.json', data, { mode: 0o600 }); // Owner read/write only
```

---

## 14. Regular Expression Security (ReDoS)

### Avoid Catastrophic Backtracking
**❌ DANGEROUS**:
```javascript
const regex = /^(a+)+$/;
```

**✅ SAFE**:
```javascript
const regex = /^a+$/;
```

### Use Timeout for Regex
```javascript
import safeRegex from 'safe-regex';

if (!safeRegex(userRegex)) {
  throw new Error('Unsafe regex pattern');
}
```

---

## 15. Environment Configuration

### .env File Security
```bash
# .env (NEVER commit this file)
NODE_ENV=production
DATABASE_URL=postgresql://user:pass@localhost:5432/db
JWT_SECRET=your-super-secret-key-min-32-chars
API_KEY=your-api-key
```

### .env.example
```bash
# .env.example (Safe to commit)
NODE_ENV=development
DATABASE_URL=postgresql://user:pass@localhost:5432/db
JWT_SECRET=change-me
API_KEY=your-api-key-here
```

### .gitignore
```
.env
.env.local
.env.*.local
*.key
*.pem
```

---

## Security Checklist for Skills

Before creating or approving a skill, verify:

### Authentication & Authorization
- ✅ No hardcoded credentials
- ✅ Passwords are hashed
- ✅ JWT tokens use strong secrets
- ✅ Environment variables for sensitive data

### Input Validation
- ✅ All user input is validated
- ✅ File uploads are restricted
- ✅ Input sanitization is implemented

### SQL & Injection Prevention
- ✅ Parameterized queries used
- ✅ No string concatenation in queries
- ✅ ORM/query builder used

### XSS Prevention
- ✅ Output is escaped
- ✅ CSP headers implemented
- ✅ HTML sanitization for user content

### Error Handling
- ✅ No stack traces exposed
- ✅ Generic error messages for users
- ✅ Detailed errors logged internally

### Dependencies
- ✅ Dependencies are up to date
- ✅ No known vulnerabilities
- ✅ Lock file is used

### Logging
- ✅ Security events are logged
- ✅ No sensitive data in logs
- ✅ Appropriate log levels used

### API Security
- ✅ HTTPS enforced
- ✅ CORS properly configured
- ✅ Rate limiting implemented
- ✅ Secure headers set

---

## Common Vulnerabilities to Avoid

1. **SQL Injection** - Use parameterized queries
2. **XSS** - Escape output, sanitize input
3. **CSRF** - Use CSRF tokens
4. **Authentication Bypass** - Validate sessions properly
5. **Insecure Direct Object References** - Check authorization
6. **Security Misconfiguration** - Use secure defaults
7. **Sensitive Data Exposure** - Encrypt data, use HTTPS
8. **Missing Function Level Access Control** - Check permissions
9. **Using Components with Known Vulnerabilities** - Update dependencies
10. **Unvalidated Redirects** - Validate redirect URLs

---

## Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Node.js Security Best Practices](https://nodejs.org/en/docs/guides/security/)
- [Express Security Best Practices](https://expressjs.com/en/advanced/best-practice-security.html)
- [npm Security Advisories](https://www.npmjs.com/advisories)

---

## Summary

Security is **everyone's responsibility**. All skills must:
- Follow these guidelines strictly
- Be reviewed for security issues
- Use secure coding practices
- Keep dependencies updated
- Log security events appropriately

**When in doubt, ask for a security review!**