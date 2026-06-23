---
description: Add an idea to the backlog — capture description, notes, category, status in docs/tasks.md; supports enhancement ideas linked to an existing project
argument-hint: <idea description> [notes]
---

Add the idea in `$ARGUMENTS` to the backlog.

`$ARGUMENTS` = free-text idea description. Text after a clear break (optional) = **notes** — constraints, target users, scope. Empty → ask once: `What's the idea?` Stop until provided.

Run **`goal-backlog`** to append the idea to `docs/tasks.md`:

- Derive a kebab-case `slug` from the description (the join key to `docs/research/<slug>/` and the `<slug>/` project repo).
- Set `status: backlog`, `phase: —`.
- Pick `category`: `business-idea` (aims to earn money) or `productivity-tool` (developer/personal utility). Genuinely ambiguous → ask once via `AskUserQuestion`.
- Enhancement of an existing project (notes name a built `<slug>`, or user says "add to `<project>`") → record `enhances <slug>` in notes; same flow.

`docs/` missing → `goal-backlog` scaffolds it first. Report the row added + its slug. Nothing else runs — building ideas is `/start-goal`.
