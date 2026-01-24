---
name: security-review
description: Security review checklist for web applications and APIs. Use when reviewing code for security issues, before deploying to production, or when implementing auth/payments/sensitive features. Covers OWASP Top 10, authentication patterns, secrets management, input validation, and common vulnerabilities.
---

# Security Review

Systematic security review for web applications and APIs.

## When to Use

- Before merging PRs that touch auth, payments, or user data
- When implementing new API endpoints
- Before production deployments
- When reviewing third-party integrations
- After adding new dependencies

---

## Quick Checklist

Run through this for every security-sensitive change:

### Authentication & Authorization
- [ ] Auth checks on every protected endpoint (not just frontend)
- [ ] Session tokens are httpOnly, secure, sameSite
- [ ] Password hashing uses bcrypt/argon2 (not MD5/SHA1)
- [ ] Rate limiting on login/signup endpoints
- [ ] Account lockout after failed attempts
- [ ] Proper logout (invalidate session server-side)

### Input Validation
- [ ] All user input validated on server (never trust client)
- [ ] SQL queries use parameterized statements (no string concat)
- [ ] File uploads validate type, size, and sanitize filename
- [ ] URLs validated before fetch/redirect (no SSRF)
- [ ] JSON/XML parsers configured to prevent XXE

### Output Encoding
- [ ] HTML output escaped (XSS prevention)
- [ ] JSON responses have correct Content-Type
- [ ] User content in HTML uses DOMPurify or equivalent
- [ ] No `dangerouslySetInnerHTML` with user content

### Secrets Management
- [ ] No secrets in code or git history
- [ ] API keys in environment variables
- [ ] Different secrets per environment
- [ ] Secrets rotated periodically
- [ ] `.env` files in `.gitignore`

### Headers & Transport
- [ ] HTTPS everywhere (no mixed content)
- [ ] CSP header configured
- [ ] CORS restricted to known origins
- [ ] Security headers set (X-Frame-Options, X-Content-Type-Options)

---

## OWASP Top 10 Reference

| Risk | What to Check |
|------|---------------|
| **Injection** | Parameterized queries, input validation |
| **Broken Auth** | Session management, password policies |
| **Sensitive Data** | Encryption at rest/transit, data minimization |
| **XXE** | Disable external entities in XML parsers |
| **Broken Access Control** | Auth checks on every resource |
| **Misconfiguration** | Default creds, error messages, debug mode |
| **XSS** | Output encoding, CSP |
| **Insecure Deserialization** | Validate/sign serialized data |
| **Vulnerable Components** | Dependency scanning, updates |
| **Logging Gaps** | Audit logs, no sensitive data in logs |

---

## Common Vulnerabilities by Context

### API Endpoints
```typescript
// BAD: No auth check
app.get('/api/users/:id', (req, res) => {
  return db.getUser(req.params.id);
});

// GOOD: Auth + authorization check
app.get('/api/users/:id', authenticate, (req, res) => {
  if (req.user.id !== req.params.id && !req.user.isAdmin) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  return db.getUser(req.params.id);
});
```

### Database Queries
```typescript
// BAD: SQL injection
const user = await db.query(`SELECT * FROM users WHERE id = ${id}`);

// GOOD: Parameterized query
const user = await db.query('SELECT * FROM users WHERE id = $1', [id]);
```

### User Input in HTML
```tsx
// BAD: XSS vulnerability
<div dangerouslySetInnerHTML={{ __html: userComment }} />

// GOOD: Sanitize first
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userComment) }} />

// BETTER: Don't use dangerouslySetInnerHTML at all
<div>{userComment}</div>
```

### Redirect URLs
```typescript
// BAD: Open redirect
res.redirect(req.query.returnUrl);

// GOOD: Validate against allowlist
const allowedHosts = ['myapp.com', 'app.myapp.com'];
const url = new URL(req.query.returnUrl, 'https://myapp.com');
if (!allowedHosts.includes(url.host)) {
  return res.redirect('/');
}
res.redirect(url.toString());
```

---

## Secrets Scanning

Before committing, check for leaked secrets:

```bash
# Quick grep for common patterns
grep -rE "(api[_-]?key|secret|password|token)\s*[:=]" --include="*.ts" --include="*.js" --include="*.py"

# Check .env files aren't staged
git status | grep -E "\.env"
```

**Common secret patterns to avoid committing:**
- API keys: `sk-`, `pk_`, `AKIA`
- Passwords in connection strings
- Private keys (RSA, SSH)
- JWT secrets
- OAuth client secrets

---

## Dependency Security

```bash
# JavaScript/TypeScript
npm audit
npx audit-ci --moderate

# Python
pip-audit
safety check

# General
snyk test
```

**Before adding a dependency:**
- Check last update date (abandoned = risky)
- Check open security issues
- Check download count (popular = more eyes)
- Prefer well-maintained alternatives

---

## Security Review Output

When reviewing, document findings:

```markdown
## Security Review: [Feature/PR]

### Critical (block merge)
- [ ] Issue: [description]
  - Location: [file:line]
  - Fix: [recommendation]

### High (fix before deploy)
- [ ] Issue: [description]
  - Location: [file:line]
  - Fix: [recommendation]

### Medium (fix soon)
- [ ] Issue: [description]

### Notes
- [Observations, recommendations for future]
```

---

## Anti-Patterns

| Anti-Pattern | Why Dangerous | Fix |
|--------------|---------------|-----|
| **Security through obscurity** | Attackers will find it | Use proper auth/encryption |
| **Client-side only validation** | Easily bypassed | Always validate server-side |
| **Rolling your own crypto** | Subtle bugs = broken | Use established libraries |
| **Storing passwords in plaintext** | Breach = all accounts compromised | Hash with bcrypt/argon2 |
| **Catching exceptions silently** | Hides security failures | Log and alert |
| **Trusting user-provided IDs** | IDOR attacks | Verify ownership server-side |
