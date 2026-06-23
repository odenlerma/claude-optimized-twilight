---
description: Autonomous loop — drive an objective to working and verified with evidence, fixing and re-verifying until it passes or needs a human
argument-hint: <objective> [context]
---

Drive the objective in `$ARGUMENTS` to **verified with evidence** in the current project.

`$ARGUMENTS` = the objective (what must end up working). Text after a clear break (optional) = context — files, acceptance criteria, constraints. Empty → ask once: `What's the goal?` Stop until provided.

Run **`goal-run`**: implement → **`goal-verify`** (tests pass + app actually runs + behavior matches the acceptance criteria + an evidence artifact is captured) → on FAIL, read the failure, fix, re-verify (≤3 cycles; stop early on a repeated identical failure) → on ESCALATE (missing capability, secret, or ambiguous criterion), write the blocker and stop for the human.

Done only on PASS with an evidence artifact on disk. Never reports success without evidence. Runs standalone, and internally to drive each build phase under `/start-goal`.
