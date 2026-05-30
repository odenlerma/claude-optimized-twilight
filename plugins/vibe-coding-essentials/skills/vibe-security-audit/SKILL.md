---
name: vibe-security-audit
description: >
  Audit a file, directory, or git diff against a reusable software-security
  ruleset — run semgrep and bandit when installed, then read the code for
  injection (SQL/command/XSS), broken authn/authz, missing input validation,
  hardcoded secrets, weak crypto, and error/log leakage. Produces ranked findings
  (severity + location + concrete fix), not per-line style nits. Read-only — never
  edits. Use when user says "security audit", "security review", "is this code
  safe", "check for vulnerabilities", or after generating security-sensitive code.
  Broader than vibe-secret-scan (full ruleset, deeper reasoning).
---

# Vibe security audit

Audit code against the security ruleset. Find real vulnerabilities. Read-only.

## Inputs

Target = file, directory, or git diff the user names. Default to the current diff (`git diff` vs base, or staged) when in a repo and none given. Load the ruleset from `${CLAUDE_PLUGIN_ROOT}/rules/security.md` as the reference checklist.

## Flow

1. **Run tools if present** (`command -v`, all optional, `|| true`):
   - `semgrep --config=auto <target>` — broad static analysis.
   - `bandit <file>` on Python files.
   - Treat tool output as leads, not gospel — confirm each by reading the code.
2. **Read against the ruleset.** For each security.md rule, check the target:
   - Secrets hardcoded (Rule 1–2) — defer deep secret hunt to `vibe-secret-scan`.
   - Unvalidated external input (Rule 3).
   - Injection: string-built SQL/NoSQL (Rule 4), shell/eval on input (Rule 5), unescaped output / XSS sinks (Rule 6).
   - AuthZ: object-level checks present and server-side (Rule 7); least privilege (Rule 8).
   - Insecure defaults, error/log leakage, fail-open (Rule 9–10).
   - Weak crypto / plaintext passwords (Rule 11), missing TLS/cert validation (Rule 12).
   - Risky/outdated deps (Rule 13), missing rate limits (Rule 14), insecure randomness (Rule 15).
3. **Confirm before reporting.** Trace whether untrusted input actually reaches the sink. Drop theoretical findings with no real path.

## Report

Rank by severity (Critical / High / Medium / Low). Each finding:
- `path:line` — rule violated — one-line impact (what an attacker does).
- Concrete fix direction (parameterize this query / move secret to env / add authz check on owner).

Skip: formatting, naming, generic by-the-book notes with no exploit path. End with a summary count and a one-line verdict. Clean → say so. Tools absent → note manual-review-only and recommend installing semgrep/bandit.

## Hard rules

- No network calls. Local tools + code reading only.
- Confirm exploitability before flagging — no speculative noise.
- Never edit. Reporting only; user applies fixes.
