---
description: Initialize and run the idea pipeline — pick the next non-completed idea and drive it through research → PRD → build+QA → marketing → handover, looping across ideas until all are completed or killed
---

Start goaling: drive every non-completed idea in `docs/tasks.md` to a terminal state (`completed` or `killed`).

One-time init:

- No `docs/tasks.md` → tell the user to capture ideas first with `/add-idea`. Stop.
- Confirm the working root once via `AskUserQuestion` before any write: `Create idea-creator workspace and project repos under <cwd>?` Wrong place → stop so the user can `cd`.

Then hand to **`goal-orchestrate`** (fresh mode). It:

1. Picks the next idea — the one already `in-progress`, else the first `backlog` row.
2. Runs it through **`goal-pipeline`**: research → gate → scaffold → PRD → build + QA → marketing (business ideas only) → handover.
3. Logs each phase boundary to the diary (`goal-log`) and updates status/phase (`goal-backlog`).
4. Repeats until every idea is `completed` or `killed`.

Runs unattended. Pauses only at hard gates — real-money deploys, secret use, `git commit`/`push` — and on blockers it cannot resolve alone. Resume any time with `/continue-goaling`.
