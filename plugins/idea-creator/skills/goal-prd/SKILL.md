---
name: goal-prd
description: >
  Write the phased PRD for one idea at <slug>/PRD.md — break the work into
  numbered phases, each with acceptance criteria and a status checkbox. Business
  ideas: phase 1 is an MVP that is already profitable, even if thin. Productivity
  tools: ship fully working, phase by phase. Invoked by goal-pipeline after
  goal-scaffold; the phases drive goal-build and resume.
---

# Goal PRD

Turn the idea + research into a phased build plan. Phase 1 must stand on its own.

## Hard rules

- **Phase 1 stands alone.** Business idea → phase 1 is an already-profitable MVP (thin is fine). Productivity tool → phase 1 is fully working for its core use, even if narrow.
- **Every phase is verifiable.** Each states acceptance criteria checkable by running the app — the contract `goal-verify` checks against.
- **Checkbox per phase.** `goal-build` ticks them; resume reads them.
- **Ground in research.** Stack from `ARCHITECTURE.md`, constraints from `TECH-FEASIBILITY.md`, the phase-1 milestone from `SUMMARY.md`.

## Flow

Write `<slug>/PRD.md`:

```markdown
# PRD — <slug>

## Goal
<what done looks like>

## Stack
<from ARCHITECTURE.md>

## Phases

- [ ] **Phase 1 — <name>** (the standalone milestone)
  - Scope: <smallest shippable slice>
  - Acceptance criteria:
    - <checkable: "POST /split returns correct shares for 3 people">
  - Evidence: <artifact that proves it — screenshot, run log, test output>

- [ ] **Phase 2 — <name>**
  - Scope: …
  - Acceptance criteria: …
  - Evidence: …
```

Number phases in build order, smallest-viable first. Tell the caller the path + phase count. Next: `goal-build`.
