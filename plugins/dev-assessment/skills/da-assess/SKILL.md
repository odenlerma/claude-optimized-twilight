---
name: da-assess
description: >
  Engineering scoping of a Jira ticket across the in-scope workspace repos. Maps
  ticket intent onto real code (Explore/Grep), names affected files/modules and
  cross-repo dependencies, biases to reuse/extend over touching working code,
  and writes a caveman-lite developer assessment: summary, complexity/effort
  (S/M/L/XL), affected code & repos, approach, risks/unknowns, open questions.
  Read-only — assesses, never edits. Use after `da-workspace`, when user says
  "assess it", "scope the work", "what would this touch", "effort estimate", or
  invokes `/dev-assessment`. Runs before `da-jira-report`.
---

# DA assess

Scope the engineering work for the ticket across in-scope repos. Find what it touches. Prefer reuse over touching working code. Write caveman-lite assessment.

## Hard rules

- **Bias to reuse/extend, not rewrite.** Do not propose touching working code unless necessary. Unavoidable → flag the risk explicitly.
- Map intent to real code via Explore / Grep across **every in-scope repo** before assessing. No guessing about what exists.
- **Cross-repo aware.** Name dependencies and contracts that cross repo boundaries (shared APIs, schemas, packages).
- Read-only. Assess, never edit.
- **Caveman lite.** Assessment text drops articles (a/an/the), filler (just/really/basically/simply), and hedging. Keep technical terms exact. Straight to the point.

## Flow

### 1. Load intent + scope

Read `dev-assessments/<TICKET>/intent.md` and `dev-assessments/<TICKET>/scope.md`. Either missing → run `da-intake` / `da-workspace` first, then return.

### 2. Locate landing spots

For each in-scope repo, find where the ticket lands (Explore / Grep): owning module/screen, callers, shared components and utilities it would touch or duplicate. Note existing code that already does part of this.

### 3. Touch-point + collision scan

List code the change affects and existing features it may collide with. Mark each touch-point:

- **reuse** — existing code covers it, call it
- **extend** — add to existing code, low risk
- **modify-existing (risk)** — changes working behavior; name what could break

Note cross-repo touch-points separately — a change in repo A that forces a change in repo B.

### 4. Rate complexity/effort

Rate **S / M / L / XL** with a one-line why. Factor: number of repos touched, modify-existing risk, cross-repo coordination, unknowns.

### 5. Write assessment

Write `dev-assessments/<TICKET>/assessment.md` — caveman lite, short sections:

```markdown
# Dev assessment — <TICKET>

## Summary
<1-2 lines — what work the ticket needs>

## Complexity / effort
<S | M | L | XL> — <one line why>

## Affected code & repos
### `<repo path>`
- `path` — <role> — reuse | extend | modify-existing (risk)

## Approach
- <step — ≤5 bullets>

## Risks / unknowns
- (none) | - <risk — what could break, cross-repo coordination>

## Open questions
- (none) | - <unresolved technical item>
```

Tell user the path. Next: run `da-jira-report` to post on approval.
