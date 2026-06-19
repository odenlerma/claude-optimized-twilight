---
name: goal-run
description: >
  Autonomous verify-until-done engine for one objective — implement it, then loop
  goal-verify (tests + run + behavior + evidence), fixing and re-verifying on FAIL
  until it PASSes, with caps so it never spins. The behavior the `/goal` command
  exposes, and the per-phase driver goal-build uses. Returns PASS (with an
  evidence artifact) or ESCALATE (a blocker a human must clear). Use when the user
  invokes `/goal` or says "drive this to done", "keep going until it works".
---

# Goal run

Drive one objective to verified-with-evidence. Implement → verify → fix → re-verify. Stop on PASS or a real blocker.

## Hard rules

- **No fake done.** Return PASS only when `goal-verify` confirms it with an artifact on disk. Never declare success from reading code alone.
- **Bounded.** At most **3** fix→verify cycles per objective. Two consecutive identical failures → stop early (stuck, not converging).
- **Escalate the un-fixable.** Missing capability (browser MCP, secret, deploy credential) or an ambiguous criterion → ESCALATE with the exact question; don't grind.
- **Hard gates stay gated.** Never deploy real money, use a secret, or `git commit`/`push` autonomously — escalate instead.

## Flow

1. **Implement** the objective — write/edit code toward the acceptance criteria.
2. **Verify** — run **`goal-verify`** with the acceptance criteria + evidence dir.
   - **PASS** → return PASS + the evidence path.
   - **FAIL** → read the failure. Actionable (red test, runtime error, wrong behavior) → fix it, increment the cycle counter, go to step 2. Cycle cap hit, or identical failure twice → ESCALATE with the accumulated failure log.
   - **ESCALATE** → write the blocker to the diary (`goal-log`), return ESCALATE.

Standalone (`/goal`): the objective is `$ARGUMENTS`; pick a sensible evidence dir (e.g. `.goal-evidence/<objective-slug>/`). Under `goal-build`: objective + evidence dir come from the PRD phase.
