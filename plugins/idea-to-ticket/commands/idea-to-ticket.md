---
description: Turn a raw idea into well-formed Jira ticket(s) — interrogate the idea, assess the codebase, draft caveman-lite tickets, create on your approval
argument-hint: <idea text> [notes]
---

Turn the idea in `$ARGUMENTS` into Jira ticket(s).

`$ARGUMENTS` = free-text idea (plus optional notes — constraints, scope, target area). Empty → ask once: `What's the idea?` Stop until provided.

Detect codebase: cwd is a git repo or holds source files → `codebase: yes`, else `codebase: no`. Record it.

Run these skills in order:

1. **`idea-clarify`** — interrogate the idea. Push back on vagueness, name contradictions, no sycophancy. Capture intent + acceptance criteria. Stay curious — ask before assuming.
2. **`idea-assess`** — **only if `codebase: yes`.** Map idea onto existing code, name features it may collide with, bias to reuse/extend over touching working code. Write developer assessment + test plan. Skip cleanly if `codebase: no`.
3. **`idea-draft`** — caveman-lite ticket(s): title, description, acceptance criteria. Idea spans separate work → propose flat split, confirm before drafting.
4. **`idea-jira-create`** — create in Jira only on explicit confirmation. Never auto-create. Post dev assessment + test plan as a comment when a comment MCP exists, else paste-ready.

Stop the chain if a step cannot proceed (no idea, unresolved contradiction). Surface the blocker to the user.

All output under `ticket-drafts/<slug>/` (`<slug>` = kebab-case short title from the idea).
