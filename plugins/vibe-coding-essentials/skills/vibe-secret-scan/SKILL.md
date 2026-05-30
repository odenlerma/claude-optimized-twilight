---
name: vibe-secret-scan
description: >
  Scan a path, file, or staged git diff for hardcoded secrets before they reach
  history — run gitleaks when installed, else fall back to grep patterns for API
  keys, tokens, private keys, passwords, and connection strings. Reports each hit
  as file:line with the matched pattern category and a remediation step (move to
  env, rotate if already committed). Read-only — never edits or commits. Use when
  user says "scan for secrets", "check for hardcoded secrets/keys", "any secrets
  in here", before a commit, or after a batch of AI-generated changes.
---

# Vibe secret scan

Find hardcoded secrets before commit. Read-only. Report, never edit.

## Scope

Default target: staged changes (`git diff --cached --name-only`) when in a git repo with staged files. Else the path the user names (file or dir). No git, no path → scan working tree files under cwd, skip ignored paths (`.git`, `node_modules`, build dirs, vendored deps).

## Flow

1. **Prefer gitleaks.** `command -v gitleaks` → run `gitleaks detect --no-git --source=<path>` (or `--staged` for staged scan). Parse findings.
2. **Grep fallback** when gitleaks absent. Search for:
   - Assignment patterns: `(password|passwd|secret|api[_-]?key|token|access[_-]?key|client[_-]?secret)\s*[:=]\s*['"][^'"]{8,}`
   - Private keys: `-----BEGIN (RSA|EC|OPENSSH|PGP|PRIVATE) KEY-----`
   - Cloud/provider tokens: AWS `AKIA[0-9A-Z]{16}`, generic `gh[pousr]_[A-Za-z0-9]{20,}`, `xox[baprs]-` Slack tokens
   - Connection strings with inline credentials: `://[^:]+:[^@]+@`
3. **Filter noise.** Skip obvious placeholders (`xxx`, `changeme`, `your-key-here`, `example`, `.env.example` files, test fixtures clearly marked). Note what was filtered.

## Report

For each finding: `path:line` — category — short snippet (redact the secret value, show first/last 2 chars). Then one remediation line:
- Move value to environment / secrets manager; reference at runtime.
- Already committed → **rotate the secret** (history keeps it); add file to `.gitignore`.

End with a count and a one-line verdict (clean / N findings). No findings → say so plainly. Tools absent → state grep-only fallback ran, recommend installing gitleaks for depth.

## Hard rules

- No network calls. Local tools and grep only.
- Never print a full secret value — redact.
- Never edit or commit. Reporting only.
