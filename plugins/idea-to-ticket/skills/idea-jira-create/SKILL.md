---
name: idea-jira-create
description: >
  Create Jira ticket(s) from drafts — ONLY after explicit user confirmation.
  Never auto-creates. Detects a Jira create-issue MCP and a comment-add MCP by
  description. On confirm, creates one issue per draft, then posts the developer
  assessment + test plan as a comment when a comment MCP exists. No MCP → prints
  paste-ready ticket(s) and comment(s). Use after `idea-draft`, when user says
  "create the tickets", "file to jira", "push to jira", "create in jira", or
  invokes `/idea-to-ticket`. Runs last.
---

# Idea Jira create

Create ticket(s) in Jira. Only on explicit confirmation. Never auto-create. Post dev assessment + test plan as a comment when possible.

## Hard rules

- **Never create or comment without explicit user confirmation.** Never auto-create.
- Never call non-MCP HTTP directly (no `curl`, no `fetch`). Jira access goes through MCP only.
- Identify Jira tools by their **description**, not name. Names vary across vendors.
- No create-issue MCP → print paste-ready ticket(s) + comment(s). Do not retry. Do not pick a near-miss tool.

## Flow

### 1. Read drafts

Read `ticket-drafts/<slug>/tickets.md`. Missing → run `idea-draft` first, then return. Take each ticket (title, description, acceptance criteria) and its comment body.

### 2. Detect Jira tools

Scan loaded tool descriptions/schemas. Match by **description content**, not name.

- **Create-issue tool** — description says it creates an issue/story/ticket in Jira, or takes project/board key + summary to create one.
- **Comment-add tool** (optional) — description says it adds / posts a comment to a Jira issue/ticket.

Multiple create candidates → prefer simplest signature (project key + summary + description). Zero create candidates → step 4.

### 3. Create branch (create-issue MCP detected)

1. Ask required inputs once: `Jira project / board key?` and `Which drafts to create? (numbers, or "all")`. Re-ask only missing ones. Zero selected → print `Nothing created.` Stop.
2. Show selected drafts in one block. Confirm via `AskUserQuestion` (yes/no): `Create these N tickets in <KEY>?`
   - **No** → print `Nothing created.` Stop.
   - **Yes** → call create-issue tool once per selected ticket. Print resulting Jira keys + URLs.
3. **Comment per ticket.** Ticket has a non-empty comment body (dev assessment + test plan) AND comment-add MCP present → show the comment body, confirm via `AskUserQuestion` (yes/no): `Post dev assessment + test plan as a comment on <KEY>?`
   - **Yes** → call comment-add tool once. Print result.
   - **No** → print `Comment not posted for <KEY>.`
   No comment-add MCP → print the comment body paste-ready per ticket.

### 4. No create-issue MCP

Do not create. Print each ticket paste-ready (title, description, acceptance criteria) and, below it, the comment body paste-ready:

```
No Jira create-issue MCP detected — drafts below are paste-ready.

--- Ticket 1 ---
Title: <...>
Description: <...>
Acceptance criteria:
- <...>

Comment (paste after creating):
<dev assessment + test plan, or "(none — no codebase)">
```

Do not retry detection.
