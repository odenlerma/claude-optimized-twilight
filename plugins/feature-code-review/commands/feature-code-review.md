---
description: Unbiased, impact-driven code review of a feature from a Jira ticket — scope the change, review by impact, post on PASS after you confirm
argument-hint: <jira-ticket-number> [notes]
---

Run full code review for Jira ticket in `$ARGUMENTS`.

First token of `$ARGUMENTS` = Jira ticket key. Remaining text (optional) = free-text notes — extra context to factor into review (focus areas, constraints, scope). No ticket → ask once: `Jira ticket number?` Stop until provided.

Run these skills in order for the ticket:

1. **`cr-scope`** — detect Jira reader MCP, fetch ticket, capture feature intent + acceptance criteria. Build change inventory: git diff vs base branch, then widen to related code the change touches (callers, shared components, reused utilities). Stay curious — ask user if intent unclear before continuing.
2. **`cr-review`** — unbiased review of changed code + context across seven impact dimensions (performance, security, responsiveness, error handling, user experience, reusable components, code structure). No per-line nits, no generic standards. Verdict: PASS or CHANGES REQUESTED.
3. **`cr-jira-report`** — on PASS only, draft comment, confirm with user, post to ticket. On CHANGES REQUESTED, do not post; report findings to fix.

Stop the chain if a step cannot proceed (no ticket, no diff/feature code, unresolved questions). Surface the blocker to the user.

All output under `code-reviews/<ticket-key>/` (ticket key only, not the notes).
