---
name: goal-handover
description: >
  Write the human handover for a built project at <slug>/HANDOVER.md — what
  shipped, how to run and deploy it, the evidence trail, open items, and a
  communication plan for the developer taking it over (including how to drive
  Claude Code remotely, citing the bundled remote-control reference). Invoked by
  goal-pipeline as the last phase, after build (and marketing for business ideas).
---

# Goal handover

Hand the project to a developer. Make it runnable, reviewable, and continuable without you.

## Hard rules

- **Point at evidence.** Reference the `.goal-evidence/` artifacts so the receiver can trust each phase.
- **Don't auto-commit/deploy.** List the exact commands for the human to run; don't run `git commit`/`push` or a real deploy yourself.
- **Cite the remote-control reference**, don't restate it: `${CLAUDE_PLUGIN_ROOT}/reference/remote-control.md`.

## Flow

Write `<slug>/HANDOVER.md`:

```markdown
# Handover — <slug>

## What shipped
<phases completed, from PRD.md, with evidence links>

## Run it
<exact commands: install, run locally, test>

## Deploy it
<steps; flag every real-money / secret step as human-gated — never auto-run>

## Evidence
<links to .goal-evidence/phase-*/ artifacts>

## Open items
<unchecked PRD phases, known issues, escalated blockers>

## Communication plan
- Status: where progress lives (this repo's PRD checkboxes + the goaling diary).
- Cadence: when to check in / what triggers a ping.
- Remote control: to steer Claude Code on this project from a phone, see
  `${CLAUDE_PLUGIN_ROOT}/reference/remote-control.md` (official Remote Control
  first; SSH-over-Tailscale fallback).
```

Business idea → also link `<slug>/MARKETING.md`. Tell the caller the path. Next: `goal-pipeline` marks the idea `completed`.
