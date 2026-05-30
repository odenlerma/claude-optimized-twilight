# Software security rules

Reusable across any software project. Applies when writing or editing code.
Load via `/vibe-security-rules`.

## 1. Never hardcode secrets
No API keys, tokens, passwords, private keys, connection strings in source. Read from environment or secrets manager at runtime. Hardcoded secret leaks moment code is shared, logged, or pushed. Ship `.env.example` with blank placeholders, never real values.

## 2. Keep secrets out of version control and logs
Add secret files (`.env`, key files, dumps) to `.gitignore`. Never log credentials, tokens, full request bodies, or session IDs. Redact before logging. Secret in git history stays forever — rotate, do not just delete.

## 3. Validate all external input
Treat everything from outside as hostile — request params, headers, files, env, third-party responses. Validate type, length, range, format against allowlist. Reject on fail. Allowlist (permit known-good) beats denylist (block known-bad); denylist always misses cases.

## 4. Parameterize queries — never build SQL by string concat
Use prepared statements / parameterized queries / ORM bindings for every database call. String-built SQL from user input is top injection vector. Same rule for NoSQL query objects: never inject raw user input into query structure.

## 5. Never pass user input to a shell or eval
No `exec`/`system`/`eval`/`os.system` on strings containing user input. Command injection gives full host control. Use library APIs with argument arrays (no shell), or strict allowlist of permitted operations.

## 6. Encode output for its context (stop XSS)
Escape data for sink it lands in — HTML body, HTML attribute, JS, URL, SQL each need different encoding. Use framework auto-escaping; do not disable it. Never `innerHTML`/`dangerouslySetInnerHTML` with untrusted data. Set Content-Security-Policy.

## 7. Authenticate, then authorize, on every request
Authentication answers "who"; authorization answers "allowed to do this". Check authorization server-side on every protected action, against resource owner — not just menu visibility. Most common bug: object-level authz miss (user A reads user B's record by guessing an ID).

## 8. Apply least privilege everywhere
Every account, token, service, DB user, API key gets minimum scope it needs and nothing more. Read-only when reads suffice. Scope tokens narrow. Short TTLs. Blast radius of a compromise = privileges granted.

## 9. Secure defaults — opt in to risk, not out of it
Default config closed: deny by default, features off, verbose errors off, debug off in production. Safe path is default path. Insecure mode must be explicit, conscious choice, never fallback.

## 10. Fail closed and handle errors without leaking
On error, deny access — do not fall through to "allow". Return generic messages to clients; full stack traces, SQL errors, file paths, versions go to server logs only. Leaked internals hand attackers a map.

## 11. Hash passwords, encrypt sensitive data
Store passwords only as salted slow hashes (bcrypt/scrypt/argon2) — never plaintext, never fast hashes (MD5/SHA1). Encrypt sensitive data at rest. Use vetted crypto libraries; never roll your own crypto or invent your own auth scheme.

## 12. Enforce transport security
All traffic over TLS — no plaintext HTTP for anything carrying data or credentials. Verify certificates; never disable cert validation to "make it work". Set `Secure`, `HttpOnly`, `SameSite` on cookies. Redirect HTTP to HTTPS.

## 13. Keep dependencies current and minimal
Fewer dependencies = smaller attack surface. Pin versions, audit regularly (`npm audit`, `pip-audit`, Dependabot), patch known CVEs promptly. Vet new packages — typosquats and abandoned libs are real. Do not add a dependency for a one-liner.

## 14. Rate-limit and guard against abuse
Limit auth attempts, expensive endpoints, and resource-heavy operations. Prevents brute force, credential stuffing, denial of service. Cap upload sizes, payload sizes, and recursion/pagination depth.

## 15. Generate secure randomness for security values
Tokens, session IDs, password-reset codes, nonces use a cryptographically secure RNG (`secrets`, `crypto.randomBytes`) — never `Math.random`/`rand()`. Predictable values are guessable, and guessable security tokens are no security.
