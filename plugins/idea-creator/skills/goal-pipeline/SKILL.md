---
name: goal-pipeline
description: >
  Run one idea through the full pipeline: research → build/kill gate → per-idea
  repo scaffold → phased PRD → phased build + QA → marketing (business ideas) →
  human handover. Writes a write-ahead diary entry at every phase boundary and
  updates the idea's status/phase. Stops the idea on a kill verdict or an
  unresolved blocker. Invoked by goal-orchestrate per idea, not directly by the
  user.
---

# Goal pipeline

Sequence one idea's phases. Gate first — don't build a loser. Log every boundary.

## Hard rules

- **Write-ahead logging.** Before each phase, append a diary entry "entering <phase>" via `goal-log`, and set the idea's `phase` via `goal-backlog`.
- **Gate is mandatory.** Never scaffold or build before `goal-gate` returns GO.
- **Kill stops cleanly.** Kill verdict → set `killed`, log the rationale, return. No repo, no build.
- **Resume-safe.** Each phase checks whether its output already exists (dossiers, repo, PRD, evidence) and skips redoing finished work.

## Flow

For the active idea (`<slug>`, `<category>`):

1. **research** — run **`goal-research`** → dossiers + `docs/research/<slug>/SUMMARY.md`.
2. **gate** — run **`goal-gate`** on `SUMMARY.md`. KILL → set `killed`, log rationale, return. GO → continue.
3. **scaffold** — run **`goal-scaffold`** → the `<slug>/` git repo, `.claude/` rules, `CLAUDE.md`.
4. **prd** — run **`goal-prd`** → `<slug>/PRD.md` with numbered phases.
5. **build** — run **`goal-build`** → implement + QA each PRD phase to verified-with-evidence.
6. **marketing** — `business-idea` only → spawn the **`goal-marketing-strategist`** agent → `<slug>/MARKETING.md`. Productivity tools skip this phase.
7. **handover** — run **`goal-handover`** → `<slug>/HANDOVER.md` + communication plan.
8. Set status `completed` (`goal-backlog`); log done.

Any phase escalates a blocker it can't resolve → log it, return to `goal-orchestrate` (which decides whether to park the idea and move on).
