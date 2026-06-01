---
name: da-intake
description: >
  Read a Jira ticket and decide if it carries enough to scope engineering work.
  Detects a Jira reader MCP by description; no reader MCP → asks the human to
  paste the ticket. Scores the ticket sufficient / insufficient on intent, scope
  boundary, and definition of done. Insufficient → ALWAYS interrogates the human
  via AskUserQuestion before anything is assessed — never assumes. Captures
  ticket facts + clarified intent for downstream skills. Use when user says "dev
  assessment", "assess this ticket", "scope this ticket", "engineering
  assessment", gives a Jira ticket key to scope, or invokes `/dev-assessment`.
  Runs first, before `da-workspace`.
---

# DA intake

Read the ticket. Decide if it is enough to scope engineering work. Not enough → ask the human. Never assume.

## Hard rules

- **Insufficient ticket → always ask.** Thin / vague / contradictory ticket → interrogate the human via `AskUserQuestion` before any assessment. Do not assume. Do not draft from a fuzzy ticket.
- **Disagree when something is off.** Flag contradictions before acting — never proceed past them silently. No sycophancy. No flattery filler.
- Never call non-MCP HTTP directly (no `curl`, no `fetch`). Jira access goes through MCP only.
- Identify Jira reader tool by its **description**, not its name. Names vary across vendors.
- **Never silently overwrite.** `dev-assessments/<TICKET>/` already exists → flag it, ask before reusing.

## Flow

### 1. Resolve ticket key

`$ARGUMENTS` first token = Jira ticket key (e.g. `PROJ-123`). Remaining text (optional) = **notes**: focus areas, constraints, target repos. No key → ask once: `Jira ticket number?` Stop until provided.

All output lives under `dev-assessments/<TICKET>/`.

### 2. Fetch ticket

Scan loaded tool descriptions/schemas. Match by **description content**, not name.

- **Reader tool** — description says it fetches / gets / reads / searches Jira issues by key.

Reader tool detected → fetch the ticket. Capture: summary, description, acceptance criteria, type, linked issues, attachments.

No reader tool → ask the human to paste the ticket: summary, description, acceptance criteria. Do not retry detection. Do not pick a near-miss tool.

### 3. Sufficiency check

Score the ticket against three tests:

- **Intent** — is it clear what to build and why?
- **Scope boundary** — is what's in / out of scope clear?
- **Definition of done** — acceptance criteria, or some checkable way to tell it's complete?

All three clear → **sufficient**. Any unclear → **insufficient**.

### 4. Clarify gate (insufficient)

Insufficient → **always** surface the specific gaps via `AskUserQuestion`. Do not proceed until resolved:

- **Vague** — "improve search": what surface, better how, for whom?
- **No definition of done** — no acceptance criteria → ask what "done" looks like.
- **Contradictory** — ticket conflicts with notes or with itself → name the contradiction, ask which wins.
- **Scope-creep** — ticket bundles several unrelated asks → ask what's in scope now.

Resolve unknowns before writing intent. Do not assume. Do not flatter the ticket.

### 5. Write intent

Existing `dev-assessments/<TICKET>/` → flag, ask before overwriting. Then write `dev-assessments/<TICKET>/intent.md`:

```markdown
# Intent — <TICKET>

Intent: <one-line plain summary of what to build and why>
Scope: <in scope / out of scope>

## Notes
- (none) | - <user-supplied notes>

## Acceptance criteria
- <checkable condition>

## Resolved questions
- (none) | - <question> → <answer from human>
```

Tell user the path. Next: run `da-workspace`.
