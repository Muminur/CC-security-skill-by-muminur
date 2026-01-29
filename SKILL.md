# Security Review Skill

## When to Use
Activate this skill when reviewing code, writing security tests, auditing API endpoints, or performing any security-related analysis on a web application (especially Next.js/Node.js).

## Security Checklist for Every API Route

### 1. Authentication & Authorization
- [ ] Every non-public endpoint requires valid session token
- [ ] Admin endpoints verify admin role (not just authentication)
- [ ] Admin email matching is case-insensitive but prevents suffix attacks (e.g., `hacker+admin@test.com`)
- [ ] Session tokens are HTTP-only, Secure, SameSite cookies
- [ ] JWT tokens have correct expiry (magic link: 15min, session: 7 days)
- [ ] Token type validation (prevent session token used as magic link and vice versa)
- [ ] Tampered/expired/invalid tokens return 401 without information leakage

### 2. IDOR (Insecure Direct Object Reference) Prevention
- [ ] All database queries filter by authenticated userId
- [ ] Users cannot access other users' data by manipulating IDs
- [ ] userId comes from session token, never from request body/params
- [ ] ObjectId parameters validated with regex: `/^[0-9a-f]{24}$/`

### 3. Injection Prevention
- [ ] **NoSQL Injection**: Reject MongoDB operators (`$gt`, `$ne`, `$regex`, `$where`, `$expr`) in user input
- [ ] **XSS**: Strip `<script>`, `<iframe>`, `javascript:` protocol, event handlers (`onclick`, `onerror`, etc.)
- [ ] **Command Injection**: Reject `&&`, `|`, `;`, backticks in user input
- [ ] **ReDoS**: Escape regex special characters with `escapeRegex()` before using in queries
- [ ] **Prototype Pollution**: Reject `__proto__`, `constructor`, `prototype` in object keys

### 4. Input Validation Patterns
- **Symbols**: Uppercase alphanumeric, 2-20 chars, ending in USDT: `/^[A-Z]{2,10}USDT$/`
- **ObjectIds**: Exactly 24 hex characters: `/^[0-9a-f]{24}$/`
- **Transaction hashes**: Exactly 64 hex characters
- **TRC20 addresses**: T + 33 alphanumeric: `/^T[a-zA-Z0-9]{33}$/`
- **Emails**: Validate format, normalize to lowercase
- **API keys**: Minimum 64 characters, encrypt with AES-256-GCM before storage
- **Numeric fields**: Reject NaN, Infinity, negative zero; validate ranges with Zod

### 5. Type Coercion Attack Prevention
- [ ] Validate type BEFORE parsing (don't rely on `parseFloat` alone)
- [ ] `parseFloat("100; DROP TABLE")` returns `100` - use Zod schema validation instead
- [ ] Arrays, booleans, null can bypass `Number()` - check `typeof` first
- [ ] Objects like `{$gt: 0}` can bypass numeric checks - validate with Zod

### 6. Rate Limiting
- [ ] Authentication endpoints: 5 attempts/IP
- [ ] Trading endpoints: 10 requests/user/minute
- [ ] Settings changes: Rate limited per user
- [ ] Public endpoints (ticker, health): IP-based rate limiting
- [ ] Payment/subscription submissions: 3 requests/hour
- [ ] Return `429` with `Retry-After` and `X-RateLimit-*` headers

### 7. Financial Security (Trading Apps)
- [ ] Never trust client-supplied amounts for payments (use server-side tier config)
- [ ] Target distributions must sum to 100% (with floating point tolerance)
- [ ] Prevent double-sell/double-execute with status checks
- [ ] Validate position sizing bounds (min/max)
- [ ] Handle integer overflow in price * quantity calculations
- [ ] Division by zero returns safe error, not Infinity

### 8. Encryption & Secrets
- [ ] API keys encrypted with AES-256-GCM (salt.iv.authTag.ciphertext format)
- [ ] Keys never returned in plaintext (only masked preview like `abc...xyz`)
- [ ] Encryption produces different ciphertext each time (random IV/salt)
- [ ] Tampered ciphertext detected via auth tag verification
- [ ] Sensitive tokens encrypted immediately on receipt, cleared on stop

### 9. Error Handling Security
- [ ] No stack traces in production error responses
- [ ] No internal database structure exposed in errors
- [ ] No sensitive data (credentials, paths, DB names) in error messages
- [ ] Generic error messages for auth failures (don't reveal whether email exists)
- [ ] Consistent error response format: `{ error: string, success: false }`

### 10. Race Condition Prevention
- [ ] Use MongoDB transactions (`session.withTransaction()`) for atomic operations
- [ ] Prevent duplicate resource creation (connections, subscriptions, trades)
- [ ] Use pending state tracking before resource creation
- [ ] Guaranteed cleanup in `finally` blocks

### 11. Resource Management
- [ ] WebSocket/SSE connections have cleanup on disconnect
- [ ] Memory-bounded caches (TTL + max entries + cleanup interval)
- [ ] Processed message deduplication uses TTL-based Map, not unbounded Set
- [ ] Database query timeouts (maxTimeMS) to prevent DoS
- [ ] Pagination limits enforced (e.g., 1-100 per page)
- [ ] Unbounded queries must have `.limit()` to prevent memory exhaustion

### 12. Page/Frontend Security
- [ ] Admin pages verify admin role before rendering
- [ ] Protected pages redirect to /login if unauthenticated
- [ ] User-supplied data HTML-escaped before rendering
- [ ] Sensitive fields (API keys, passwords) use masked display
- [ ] URL parameters validated (e.g., `/oco/[orderListId]` must be numeric)
- [ ] Forms disable autocomplete on sensitive fields
- [ ] Destructive actions require confirmation

## Test Writing Patterns

### Security Test Structure
```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';

describe('Route Security Tests', () => {
  // 1. Authentication
  describe('Authentication Requirements', () => {
    it('should reject unauthenticated requests', async () => {
      // No session cookie -> 401
    });
    it('should reject expired tokens', async () => {});
    it('should reject tampered tokens', async () => {});
  });

  // 2. Input Validation
  describe('Input Validation', () => {
    it('should reject XSS payloads', async () => {});
    it('should reject NoSQL injection', async () => {});
    it('should reject invalid ObjectIds', async () => {});
  });

  // 3. IDOR Prevention
  describe('IDOR Prevention', () => {
    it('should filter by authenticated userId', async () => {});
    it('should prevent cross-user access', async () => {});
  });

  // 4. Rate Limiting
  describe('Rate Limiting', () => {
    it('should return 429 when limit exceeded', async () => {});
    it('should include rate limit headers', async () => {});
  });

  // 5. Combined Attacks
  describe('Combined Attack Scenarios', () => {
    it('should handle XSS + NoSQL injection combo', async () => {});
  });
});
```

### Common XSS Payloads to Test
```typescript
const XSS_PAYLOADS = [
  '<script>alert("xss")</script>',
  '<img src=x onerror=alert(1)>',
  '<iframe src="javascript:alert(1)">',
  'javascript:alert(document.cookie)',
  '<svg onload=alert(1)>',
  '"><script>alert(1)</script>',
];
```

### Common NoSQL Injection Payloads
```typescript
const NOSQL_PAYLOADS = [
  { $gt: '' },
  { $ne: null },
  { $regex: '.*' },
  { $where: 'function(){return true}' },
  { $expr: { $eq: [1, 1] } },
];
```

## Security Gaps to Always Flag
1. Public endpoints without rate limiting
2. Missing `.limit()` on database queries
3. `parseFloat`/`Number()` without type checking
4. Error responses containing stack traces or internal paths
5. ObjectId parameters without `/^[0-9a-f]{24}$/` validation
6. User input used directly in regex without `escapeRegex()`
7. Sensitive data returned in API responses (unmasked keys, tokens)
8. Missing `finally` block for resource cleanup
9. Unbounded in-memory collections (Sets, Maps) without TTL
10. Missing CSRF protection on state-changing endpoints
