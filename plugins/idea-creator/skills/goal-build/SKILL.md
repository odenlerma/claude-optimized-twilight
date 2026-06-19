---
name: goal-build
description: >
  Build the project phase by phase from <slug>/PRD.md — for each unchecked phase,
  drive it to verified-with-evidence via goal-run, capture the artifact, tick the
  PRD checkbox, and log the boundary. Phased manual QA is built in: a phase isn't
  done until it works and is proven working. Invoked by goal-pipeline after
  goal-prd.
---

# Goal build

Build phase by phase. Each phase done only when verified with evidence. Never skip ahead.

## Hard rules

- **One phase at a time, in order.** Don't start phase N+1 until phase N is verified.
- **Verified = evidence on disk.** A phase completes only when `goal-run` returns PASS with an artifact in `<slug>/.goal-evidence/phase-<N>/`.
- **Tick the box.** On PASS, change `- [ ]` to `- [x]` for that phase in `PRD.md` — a resume signal.
- **Hard gates stay gated.** A phase needing a real-money deploy, a secret, or a `git commit` → escalate; don't do it autonomously.

## Flow

Read `<slug>/PRD.md`. For each unchecked phase, in order:

1. Log `entering build phase-N/implementing` (`goal-log`, write-ahead).
2. Run **`goal-run`** with the objective = the phase scope + acceptance criteria + the evidence type to capture; evidence dir `<slug>/.goal-evidence/phase-<N>/`.
3. On the result:
   - **PASS** → tick the PRD checkbox; log `phase-N/verified` with the evidence path; go to the next phase.
   - **ESCALATE** → log the blocker (`phase-N/blocked`); return to `goal-pipeline` (the human resolves it).
4. All phases checked → build done. Return to `goal-pipeline` (next: marketing or handover).

Phased QA is built in — `goal-run`'s verify step (`goal-verify`) runs the app and captures evidence for every phase. That is the manual QA, per phase.
