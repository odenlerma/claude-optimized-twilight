---
name: goal-backlog
description: >
  Read, append to, and update the idea backlog at docs/tasks.md — the ledger of
  every idea with status (backlog|in-progress|completed|killed), category
  (productivity-tool|business-idea), slug, and current pipeline phase. Scaffolds
  the docs/ workspace on first use. Owns all status transitions and the
  one-idea-in-progress invariant. Use when adding an idea, picking the next idea
  to work, or changing an idea's status/phase, or when the user invokes
  `/add-idea`, `/start-goal`, or `/continue-goaling`.
---

# Goal backlog

Own `docs/tasks.md` — the idea ledger. Every read or change to an idea's status, phase, or notes goes through here.

## Hard rules

- **One active idea.** At most one row may be `in-progress`. Never set a second.
- **Status flow:** `backlog → in-progress → completed`, or `backlog → in-progress → killed`. `completed` and `killed` are terminal — never reopen.
- **Never delete rows.** Edit status/phase/notes in place. History lives in the diary (`goal-log`).
- **Slug is the join key.** `docs/research/<slug>/` and the `<slug>/` repo derive from it. Keep it stable once set; keep it unique in the table.

## Flow

### 1. Scaffold workspace (first use)

`docs/` missing → create `docs/`, `docs/research/`, `docs/notes/`, then write `docs/tasks.md`:

```markdown
# Idea backlog

Statuses: `backlog` · `in-progress` · `completed` · `killed`
Categories: `productivity-tool` · `business-idea`

| ID | Slug | Description | Category | Status | Phase | Notes |
|----|------|-------------|----------|--------|-------|-------|
```

### 2. Add an idea

Append one row. `ID` = next integer. `Slug` = kebab-case short title (unique). `Status` = `backlog`. `Phase` = `—`. `Notes` = user notes; for an enhancement, include `enhances <slug>`.

### 3. Pick the next idea

Return the `in-progress` row if one exists; else the first `backlog` row by `ID`. None of either → report "all ideas completed or killed".

### 4. Update status / phase

Move a row through the flow. Start an idea: `backlog → in-progress`, set `Phase`. Advance: update `Phase` along `research → gate → scaffold → prd → build → marketing → handover`. Finish: `in-progress → completed`; gate kill: `in-progress → killed`.

Report the change in one line.
