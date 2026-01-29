# CC Security Review Skill

A comprehensive security review skill for Claude Code that provides automated security checklist validation during code reviews.

## Installation

### macOS / Linux

```bash
mkdir -p ~/.claude/skills/security-review
cp SKILL.md ~/.claude/skills/security-review/SKILL.md
```

### Windows (PowerShell)

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills\security-review"
Copy-Item SKILL.md "$env:USERPROFILE\.claude\skills\security-review\SKILL.md"
```

### Windows (CMD)

```cmd
mkdir "%USERPROFILE%\.claude\skills\security-review"
copy SKILL.md "%USERPROFILE%\.claude\skills\security-review\SKILL.md"
```

## What It Does

When reviewing code, this skill activates a 12-category security checklist covering:

| Category | What It Checks |
|----------|---------------|
| **Authentication & Authorization** | Session tokens, JWT validation, admin role verification |
| **IDOR Prevention** | User isolation, userId from session not request body |
| **Injection Prevention** | NoSQL, XSS, command injection, ReDoS, prototype pollution |
| **Input Validation** | Symbols, ObjectIds, transaction hashes, addresses, numerics |
| **Type Coercion Attacks** | parseFloat pitfalls, array/boolean/null bypass vectors |
| **Rate Limiting** | Per-user, per-IP, endpoint-specific limits |
| **Financial Security** | Amount tampering, double-execution, precision attacks |
| **Encryption & Secrets** | AES-256-GCM, key masking, IV randomness |
| **Error Handling** | No stack traces, no internal paths, generic auth errors |
| **Race Conditions** | MongoDB transactions, pending state tracking, cleanup |
| **Resource Management** | TTL caches, query limits, connection cleanup |
| **Page/Frontend Security** | Admin gates, URL param validation, data masking |

## Includes

- Security checklist for every API route
- Test writing patterns with example structure
- Common XSS and NoSQL injection payloads for testing
- Top 10 security gaps to always flag during review

## Tech Stack Focus

Optimized for **Next.js / Node.js / MongoDB / TypeScript** applications but applicable to most web APIs.

## Author

**Muminur** - [GitHub](https://github.com/Muminur)

## License

MIT
