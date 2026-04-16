---
name: security-review
description: Security-focused review of staged or branch changes before merging
license: MIT
compatibility: opencode
metadata:
  tags: security, review
---

## What I do

Audit pending changes for security vulnerabilities using a checklist derived from OWASP Top 10 and common secrets exposure patterns.

## Steps

1. Get the diff: `git diff main...HEAD` (or the relevant base branch)
2. Check each category below — cite `file:line` for every finding

### Checklist

**Secrets exposure**
- Hardcoded API keys, tokens, passwords, private keys, connection strings
- `.env` files or credential files committed
- Secrets logged via `console.log`, `print`, `logger.info`

**Injection**
- SQL built with string concatenation: `"SELECT * FROM users WHERE id = " + userId`
- Shell commands with user input: `exec("git clone " + repoUrl)`
- Template injection, LDAP injection, XPath injection

**Broken authentication**
- New routes added without auth middleware/guard
- JWT: missing signature verification, `alg: none` accepted, expiry not checked
- Session tokens not invalidated on logout

**Insecure direct object reference (IDOR)**
- User-supplied IDs used in DB queries without ownership check:
  `db.query("SELECT * FROM orders WHERE id = ?", [req.params.id])` — is `req.user.id` verified against the result?

**XSS**
- User input rendered directly into HTML: `innerHTML = userInput`, `dangerouslySetInnerHTML`
- Missing Content-Security-Policy header on new routes

**Dependency changes**
- New packages added: run `npm audit` / `pip-audit` / `cargo audit`
- Major version bumps: check if the new version has known CVEs

**Sensitive data exposure**
- PII (email, name, phone) returned in API responses where not needed
- Sensitive fields included in error messages or stack traces sent to clients
- Overly permissive CORS: `Access-Control-Allow-Origin: *` on authenticated endpoints

## Concrete vulnerable patterns

```typescript
// VULNERABLE — SQL injection
const user = await db.query(`SELECT * FROM users WHERE email = '${req.body.email}'`);

// SAFE
const user = await db.query('SELECT * FROM users WHERE email = ?', [req.body.email]);

// VULNERABLE — IDOR, no ownership check
const order = await Order.findById(req.params.id);
return res.json(order);

// SAFE
const order = await Order.findOne({ _id: req.params.id, userId: req.user.id });
if (!order) return res.status(404).json({ error: 'Not found' });

// VULNERABLE — secret in code
const stripe = new Stripe('sk_live_abc123...');

// SAFE
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);
```

## Output format

```
## Security Review — <branch or PR>

### Critical
- `src/api/users.ts:34` — SQL injection via string concatenation in email lookup.
  Replace with parameterized query. See example above.

### High
- `src/api/orders.ts:88` — IDOR: order fetched by ID without userId ownership check.

### Informational
- `src/api/products.ts:12` — CORS set to * — confirm this endpoint has no auth requirement.

### Clean
- No hardcoded secrets found
- No new deps with known CVEs (npm audit clean)
- Auth middleware applied to all new routes
```

## Rules

- Flag any hardcoded secret as Critical — even if it looks like a test key or placeholder
- Do not flag theoretical vulnerabilities — only flag real, exploitable patterns in the actual diff
- A "false positive" is better documented than silently skipped — note it in Informational with reasoning
- Always run a dep audit when `package.json`, `requirements.txt`, `go.mod`, or `Cargo.toml` changed
