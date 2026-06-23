---
name: goal-scaffold
description: >
  Create the per-idea project repo once research clears the gate — git init a
  fresh <slug>/ subdirectory, write a CLAUDE.md (≤200 lines, scoped to the idea
  and the chosen stack), and tailored .claude/rules for coding, security,
  compliance (only when the dossier flagged obligations), and code structure. No
  hooks. Never touches an existing repo, never auto-commits. Invoked by
  goal-pipeline after a GO verdict.
---

# Goal scaffold

Stand up the project's own git repo, scoped to this idea. Fresh dir only. Never auto-commit.

## Hard rules

- **Fresh subdir only.** `<slug>/` already exists → stop, tell the caller. Don't overwrite, don't `git init` an existing dir or the user's cwd.
- **git init the new dir only.** Run `git init <slug>` for the dir you just created. Never init or stage the user's working directory.
- **Never auto-commit or push.** Leave changes for the human to review with `git -C <slug> diff`.
- **No hooks.** Don't write `.claude/hooks/` or a hooks block.
- **No secrets.** Write `.env.example` with placeholder keys; add `.env` to `.gitignore`. Never write real secret values.
- **CLAUDE.md ≤ 200 lines.**

## Flow

Read `docs/research/<slug>/ARCHITECTURE.md` (stack: mobile → React Native + Expo + Cloudflare; web → Next.js + Tailwind + Hono + Drizzle + Cloudflare) and `COMPLIANCE.md` (any obligations?).

1. Create `<slug>/`; run `git init <slug>`.
2. Write `<slug>/.gitignore` (dependencies, build output, `.env`; keep `.goal-evidence/` committed by default — it's the proof-of-work for handover).
3. Write `<slug>/.env.example` (placeholder keys only).
4. Write `<slug>/CLAUDE.md` (≤200 lines):

```markdown
# <slug>

<one-line what this is, from the idea>

## Stack
<chosen stack from ARCHITECTURE.md>

## Rules
See `.claude/rules/`: coding, security, structure<, compliance if present>.

## Build
Phases tracked in `PRD.md`. Evidence per phase in `.goal-evidence/`.
```

5. Write `<slug>/.claude/rules/`:
   - `coding.md` — language/style for the stack; tests required per phase.
   - `security.md` — input validation, authn/z, no secrets in code, `.env` handling.
   - `structure.md` — directory layout + module boundaries for the stack.
   - `compliance.md` — **only if** `COMPLIANCE.md` flagged obligations; the concrete data-handling/retention/consent rules.

Tailor each rule file to this idea + stack — not boilerplate. Tell the caller the repo path. Next: `goal-prd`.
