---
name: idea-draft
description: >
  Assemble caveman-lite Jira ticket draft(s) from a clarified idea — title,
  description, acceptance criteria. Proposes a flat split into N tickets when the
  idea spans separate work, and confirms before drafting. Keeps the developer
  assessment and test plan separate (they become a comment, not the description).
  Use after `idea-clarify` (and `idea-assess` when a codebase is present), when
  user says "draft the ticket", "write the ticket", "split into tickets", or
  invokes `/idea-to-ticket`. Runs before `idea-jira-create`.
---

# Idea draft

Assemble ticket draft(s). Caveman lite. Split flat when the idea spans separate work.

## Hard rules

- Caveman lite for every ticket field (see below). Concise, exact, no filler.
- Description carries title/desc/AC only. Dev assessment + test plan stay **separate** — they become the Jira comment, not the description.
- Split is **flat** — independent tickets, no epic/story hierarchy. Confirm before drafting.

## Flow

### 1. Load inputs

Read `ticket-drafts/<slug>/intent.md`. Read `ticket-drafts/<slug>/assessment.md` if present (codebase run).

### 2. Split decision

Idea spans independent pieces of work → propose a numbered flat list, confirm via `AskUserQuestion`: `Split into these N tickets?` One idea = one ticket otherwise. Each ticket gets its own draft block.

### 3. Draft each ticket

```
Title: <imperative verb + object, ≤60 chars>
Description: <≤3 sentences. Drop a/an/the, just, basically, really, hedging. Keep technical terms exact.>
Acceptance criteria:
- <checkable condition>
- <checkable condition>
```

If `assessment.md` present, build a per-ticket **comment body** (dev assessment + the relevant test-plan scenarios) — kept separate from the description.

### 4. Write drafts

Write `ticket-drafts/<slug>/tickets.md`:

```markdown
# Ticket drafts — <slug>

## Ticket 1
Title: <...>
Description: <...>
Acceptance criteria:
- <...>

### Comment (dev assessment + test plan)
<dev assessment + test-plan scenarios for this ticket, or "(none — no codebase)">

## Ticket 2
...
```

Tell user the path. Next: run `idea-jira-create`.

## Caveman lite for ticket fields

- **Titles** — imperative verb + object. "Add CSV export to reports" not "We need to add a CSV export feature to the reports page". ≤60 chars.
- **Descriptions** — ≤3 sentences. Drop `a/an/the`, `just`, `basically`, `really`, hedging. Keep technical terms exact.
- **Acceptance criteria** — 2–5 bullets. Each checkable. "CSV downloads with all visible columns" not "User should be able to export".
