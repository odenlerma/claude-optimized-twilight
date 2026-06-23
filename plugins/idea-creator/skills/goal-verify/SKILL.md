---
name: goal-verify
description: >
  Evidence gate for one objective or build phase — confirm it actually works:
  tests pass, the app actually runs, observed behavior matches the acceptance
  criteria, and an artifact (screenshot via a browser MCP, or a run/HTTP log) is
  captured to the evidence dir. Returns PASS / FAIL / ESCALATE. No artifact → can
  not PASS. Called by goal-run each cycle; leans on the built-in `run` and
  `verify` skills.
---

# Goal verify

Prove it works — don't assert it. One artifact, or it doesn't pass.

## Hard rules

- **All four or it's not PASS:** tests green **and** the app actually runs **and** behavior matches the acceptance criteria **and** an artifact is captured.
- **No artifact → cannot PASS.** Evidence on disk, or it didn't happen.
- **Detect tools by description, not name.** A browser/screenshot MCP qualifies by what its description says it does. None present + criteria need a UI screenshot → ESCALATE (can't prove it); don't fake it.
- **Don't fix here.** Verify only — report the verdict; `goal-run` does the fixing.

## Flow

For the objective's acceptance criteria + evidence dir `<dir>`:

1. **Tests** — run the phase's tests. Red, or none where the phase clearly needs them → FAIL (note which).
2. **Run** — use the built-in **`run`** skill to actually launch the app (web: server boots + the new route/page responds; mobile: app builds + the new screen renders). Won't run → FAIL.
3. **Behavior** — use the built-in **`verify`** skill to confirm observed behavior matches each acceptance criterion. Mismatch → FAIL.
4. **Capture evidence** to `<dir>/`: a screenshot (UI, via the browser MCP) or a run-log / HTTP-response capture (API/CLI). Write `<dir>/verdict.txt` (PASS/FAIL + one line). No artifact possible (missing MCP/capability) → ESCALATE.

Return **PASS** (all four met, artifact written), **FAIL** (a criterion unmet but fixable — name it), or **ESCALATE** (can't obtain evidence / ambiguous criterion / hard gate).
