---
name: idea-assess
description: >
  Codebase-aware feasibility for an idea before ticketing. Maps the idea onto
  existing code, names features it may collide with and what could break, biases
  to reuse/extend over touching working code, and writes a developer assessment
  plus a test plan for dev and QA. Runs only when a codebase is present. Use
  after `idea-clarify`, when user says "assess the idea", "what would this
  touch", "dev assessment", "feasibility", or invokes `/idea-to-ticket` in a
  codebase. Runs before `idea-draft`.
---

# Idea assess

Map the idea onto the codebase. Find collisions. Prefer reuse over touching working code. Write dev assessment + test plan.

## Hard rules

- **Bias to reuse/extend, not rewrite.** Do not propose touching working code unless necessary. Unavoidable → flag the risk explicitly.
- Map idea to real code via Explore / Grep before assessing. No guessing about what exists.
- Always think possible scenarios and **what existing features the change may collide with**.
- No codebase → this skill should not run. Skip to `idea-draft`.
- Plain language. This is read by devs and QA later.

## Flow

### 1. Load intent

Read `ticket-drafts/<slug>/intent.md`. Missing → run `idea-clarify` first, then return.

### 2. Locate landing spot

Find where the idea lands in the code (Explore / Grep): owning module/screen, callers, shared components and utilities it would touch or duplicate. Note existing code that already does part of this.

### 3. Collision scan

List existing features the idea may affect or conflict with, and what could break. Mark each touch-point:

- **reuse** — existing code covers it, call it
- **extend** — add to existing code, low risk
- **modify-existing (risk)** — changes working behavior; name what could break

### 4. Write assessment

Write `ticket-drafts/<slug>/assessment.md`:

```markdown
# Dev assessment + test plan — <slug>

## Developer assessment
Approach: <plain, how to build it>
Files / areas: `path` — <role>
Reuse: <existing code to call/extend instead of writing new>
Risky touch-points (avoid if possible): `path` — <what could break>
Complexity: low | medium | high — <one line why>
Open technical questions: (none) | - <item>

## Test plan
### Positive
1. <do this> → <should see this>

### Negative
1. <do this> → <should see this>

### Regression — must not break
1. <existing feature> → <still works like before>

### What-if
1. <edge case> → <should see this>
```

Tell user the path. Next: run `idea-draft`.
