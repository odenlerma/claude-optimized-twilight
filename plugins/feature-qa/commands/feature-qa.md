---
description: Run manual QA for a Jira ticket — build a test plan, execute it with screenshot evidence, post results on pass
argument-hint: <jira-ticket-number> [notes]
---

Run full manual QA for Jira ticket in `$ARGUMENTS`.

First token of `$ARGUMENTS` = Jira ticket key. Remaining text (optional) = free-text notes — extra context to factor into QA (focus areas, constraints, scope). No ticket → ask once: `Jira ticket number?` Stop until provided.

Run these skills in order for the ticket:

1. **`qa-test-plan`** — fetch ticket, analyze codebase, build test plan (positive, negative, regression, what-if). Stay curious — ask user if any requirement or scenario is unclear before continuing.
2. **`qa-execute`** — run plan by hand through a browser MCP, screenshot each scenario, write report with pass/fail verdict.
3. **`qa-jira-report`** — on PASS only, draft comment, confirm with user, post to ticket. On FAIL, do not post; report failures.

Stop the chain if a step cannot proceed (no plan, no browser MCP, unresolved questions). Surface the blocker to the user.

All output under `qa-reports/<ticket-key>/` (ticket key only, not the notes).
