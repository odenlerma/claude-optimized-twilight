---
name: vibe-refactor-plan
description: >
  Assess a file, directory, or git diff against a reusable clean-code ruleset and
  produce a prioritized, NON-destructive refactor plan — flag long functions,
  mixed responsibilities, duplication, magic numbers, dead code, poor naming,
  deep nesting, and wrong dependency direction. Outputs ordered, low-risk steps
  with rationale; does NOT edit code. Use when user says "refactor plan", "clean
  up this code", "how would you improve this structure", "tech-debt review", or
  after AI generates a large chunk of code. Planning only — pair with the user
  applying changes.
---

# Vibe refactor plan

Assess code against the clean-code ruleset. Produce an ordered refactor plan. No edits.

## Inputs

Target = file, directory, or git diff the user names. Default to the current diff when in a repo and none given. Load `${CLAUDE_PLUGIN_ROOT}/rules/clean-code.md` as the reference checklist.

## Flow

1. **Read the target.** Understand what it does before judging structure.
2. **Check against clean-code rules:**
   - Functions doing more than one job / too long (Rule 1–2).
   - Names that hide intent (Rule 3); dead code (Rule 4).
   - Real duplicated knowledge vs incidental similarity (Rule 5) — only flag the former.
   - Swallowed errors (Rule 6); missing/noisy comments (Rule 7); magic numbers/strings (Rule 8).
   - Inconsistent style vs the project (Rule 9); wrong dependency direction (Rule 10).
   - Hard-to-test side-effect tangles (Rule 11); hidden mutation (Rule 12).
   - Too many params / boolean flag params (Rule 13); deep nesting (Rule 14); leaky module boundaries (Rule 15).
3. **Sequence the work.** Order steps so each is small, independently shippable, and low-risk. Cheap high-value cleanups first; structural changes (dependency inversion, module splits) last, called out as higher-risk.

## Report

A prioritized, numbered plan. Each step:
- What to change and which rule it serves — `path:line` anchor.
- Why it matters (impact on readability/testability/maintenance).
- Risk level + suggested verification (run tests, diff review).

Skip subjective style preferences the project doesn't already follow. Note where behavior must stay identical (pure refactor) vs where tests are needed first. End with a one-line "start here" recommendation.

## Hard rules

- Planning only — never edit code. User applies steps (or asks you to, separately).
- Respect existing project conventions over generic ideals (Rule 9).
- No bug hunting — that's `vibe-security-audit` / a code review. Structure and clarity only.
