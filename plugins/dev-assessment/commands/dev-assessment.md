---
description: Turn an existing Jira ticket into an engineering scoping assessment — check info sufficiency (ask the human when thin), scan the workspace, write caveman-lite assessment, post to Jira on your approval
argument-hint: <jira-ticket-number> [notes]
---

Build a developer (engineering) assessment for the Jira ticket in `$ARGUMENTS`.

First token of `$ARGUMENTS` = Jira ticket key. Remaining text (optional) = free-text notes — extra context to factor in (focus areas, constraints, target repos). No ticket → ask once: `Jira ticket number?` Stop until provided.

Run these skills in order for the ticket:

1. **`da-intake`** — detect Jira reader MCP by description, fetch ticket (no reader MCP → ask human to paste it). Check info sufficiency — clear intent, scope boundary, way to tell when done. Insufficient → **always** ask the human for clarity via `AskUserQuestion`. Do not assume. Do not proceed until resolved.
2. **`da-workspace`** — discover the workspace (parent of cwd / sibling repos), not just current repo. List candidate repos/projects, confirm/trim in-scope set with the human.
3. **`da-assess`** — analyze in-scope repos against ticket intent. Write caveman-lite assessment: summary, complexity/effort, affected code & repos, approach, risks, open questions.
4. **`da-jira-report`** — detect comment-add MCP, draft condensed comment, confirm via `AskUserQuestion`, post on approval only. No comment MCP → paste-ready comment.

Stop the chain if a step cannot proceed (no ticket, insufficient info unresolved, no in-scope code). Surface the blocker to the user.

All output under `dev-assessments/<ticket-key>/` (ticket key only, not the notes).
