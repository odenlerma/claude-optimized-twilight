---
name: goal-orchestrate
description: >
  Drive the whole backlog: pick the next non-completed idea, run it through
  goal-pipeline, then advance to the next, until every idea is completed or
  killed. In resume mode, reconstruct where a prior session left off from the
  diary + tasks.md + the project's files, reconciling against filesystem ground
  truth. Runs unattended, pausing only at hard money/secret/commit gates and
  unresolvable blockers. Use when the user invokes `/start-goal` (fresh) or
  `/continue-goaling` (resume).
---

# Goal orchestrate

Outer loop over the backlog. Pick → run pipeline → advance → stop when all ideas terminal.

## Hard rules

- **One idea at a time.** Honor `goal-backlog`'s single-`in-progress` invariant.
- **Filesystem wins.** On any conflict between the diary and files on disk (dossiers, PRD checkboxes, evidence artifacts), trust the files.
- **Don't deadlock.** An idea stuck on the same blocker after 2 escalations with no human input → leave it `in-progress` with the blocker logged and move to the next backlog idea. The human resumes it later.
- **Unattended, with hard stops.** Proceed without asking, except at the gates `goal-pipeline` enforces (deploy / secret / commit) and genuine blockers.

## Flow

### Fresh mode (`/start-goal`)

1. Read `docs/tasks.md` via `goal-backlog`. Empty → "add ideas with /add-idea", stop. All rows terminal → "all ideas done", stop.
2. Pick the next idea (in-progress first, else first backlog). Set it `in-progress`.
3. Run **`goal-pipeline`** for it to a terminal state.
4. Loop to step 1.

### Resume mode (`/continue-goaling`)

1. Read `docs/tasks.md`. Find the `in-progress` idea.
   - **Zero** → no active idea; go to fresh mode step 2.
   - **More than one** → drift. Stop, show both, ask the human which to resume. Never guess.
2. Parse the diary tail for that slug (`goal-log`): `phase`, `sub`, `state`, `evidence`, `blocker`.
3. **Reconcile against ground truth** (anti-drift):
   - `phase: research` → check `docs/research/<slug>/`. All category dossiers + `SUMMARY.md` present → advance to `gate`. Missing → re-run only the missing parts.
   - `phase: build, sub: phase-N/verified` → confirm the artifact at `evidence:` exists and phase-N tests still pass. Good → phase N+1. Missing/red → redo phase N.
   - `phase: build, sub: phase-N/verifying` → always re-verify phase N from scratch (a half-captured verify is not evidence).
   - Cross-check build progress against `<slug>/PRD.md` phase checkboxes. Two-of-three agreement (diary, evidence files, PRD checkboxes) wins; full disagreement → escalate.
4. Open `blocker:` non-empty → present it to the human first. Don't silently retry past a known blocker.
5. Re-enter **`goal-pipeline`** at the reconciled phase, then continue the cross-idea loop (fresh mode steps).
