---
name: cr-review
description: >
  Unbiased, impact-driven code review of a feature's changed code plus related
  context. Reviews across seven impact dimensions — performance, security,
  responsiveness, error handling, user experience, reusable components, code
  structure — with each finding tied to the feature intent, its real-world
  impact, a severity (Blocker/Major/Minor), and a suggested direction. Skips
  per-line nits, formatting, and generic by-the-book standards. Produces a
  PASS / CHANGES REQUESTED verdict and writes review.md. Use when user says
  "review the code", "code review this", "do the review", after a scope exists,
  or invokes `/feature-code-review`. Runs after `cr-scope`, before
  `cr-jira-report`.
---

# CR review

Review a feature's code. Unbiased. Impact-driven. Group findings by what they affect, not by file or line.

## Hard rules

- **No per-line / per-block nits.** No formatting, naming-convention, import-order, line-length, or "add a comment here" findings.
- **No generic by-the-book standards.** No finding that cites a rule without a concrete impact. If it does not change behavior, performance, safety, or how a human reads/uses the code, drop it.
- **Group by impact dimension, not by file.** Every finding ties back to the feature intent in `scope.md`.
- **Optimize for usability + readability + real impact.** Readability findings are about whether a maintainer can follow the code, not whether it matches a textbook structure.
- **Unbiased.** Judge against intent and real-world effect, not personal style or framework fashion. State impact, not preference.
- Skip any dimension that has nothing meaningful to say. Empty is better than filler.

## Flow

### 1. Load scope

Read `code-reviews/<TICKET>/scope.md`. Missing → run `cr-scope` for `<TICKET>` first, then return here. Take intent, acceptance criteria, changed files, related context.

### 2. Read the code

Read the changed code and the related-context files. Understand what the feature does and how it fits the surrounding code.

### 3. Review across seven dimensions

For each dimension, look for meaningful impact. Skip if nothing real.

- **Performance** — N+1 queries, repeated work, blocking calls on hot paths, unbounded payloads, missing pagination/caching, needless re-renders.
- **Security** — injection, missing authz/authn checks, unsafe input handling, secrets in code, unsafe deserialization, leaking data in responses/logs.
- **Responsiveness** — UI behavior across viewports/devices, async states (loading/disabled/empty), work blocking the main thread, layout that breaks on small/large screens.
- **Error handling** — unhandled failures, swallowed exceptions, no user feedback on failure, missing edge cases from the acceptance criteria, partial-failure states.
- **User experience** — does the flow match intent, clear feedback, sensible defaults, accessibility, confusing or dead-end states.
- **Reusable components** — duplication of existing components/utilities, reinventing something the codebase already has (use related-context files), or new code that should be extracted for reuse.
- **Code structure** — can a maintainer follow it? Tangled control flow, misleading names, hidden side effects, logic in the wrong layer. Readability and maintainability — not textbook patterns.

### 4. Score each finding

Each finding gets a severity:

- **Blocker** — ships a bug, security hole, or breaks the acceptance criteria. Must fix before merge.
- **Major** — real impact on users, performance, or maintainability; should fix before merge.
- **Minor** — worth improving, not blocking.

State **impact** for every finding: what it does to the user or system, tied to intent. Give a **suggested direction**, not a prescriptive line edit.

### 5. Verdict

- **CHANGES REQUESTED** — any Blocker or Major finding.
- **PASS** — no Blocker and no Major. Minors allowed.

### 6. Write review

Write `code-reviews/<TICKET>/review.md`:

```markdown
# Code review — <TICKET>

Verdict: PASS | CHANGES REQUESTED
Summary: <one-line plain summary of the review>

## Findings by impact

### Performance
- **[Major]** `file:line` — <impact on user/system, tied to intent>. Direction: <suggested direction>.

### Security
- (none) | - **[Blocker]** `file:line` — <impact>. Direction: <...>.

### Responsiveness
### Error handling
### User experience
### Reusable components
### Code structure
<omit any dimension with no meaningful finding>

## Strengths
- <what the change does well, worth keeping>
```

Tell user report path + verdict. PASS → next step `cr-jira-report` for `<TICKET>`. CHANGES REQUESTED → fix findings, do not report to Jira.
