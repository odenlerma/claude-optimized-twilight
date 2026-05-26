---
name: jira-ticketize
description: >
  Convert meeting transcript action items and actionable work into Jira ticket
  drafts — with explicit human approval at every gate. Never auto-creates. Use
  when user says "create jira tickets", "ticketize action items", "ticketize
  this meeting", "file tickets from meeting", "convert meeting to jira", "make
  tickets from transcript", drops a transcript and asks for Jira tickets, or
  invokes `/wiki-ticketize`. Typically runs after `/wiki-ingest`.
---

# Jira ticketize

Turn transcript work items into draft Jira tickets. Human approves at every gate. Never auto-create.

## Hard rules

- Never write to `wiki/`. Candidate list and drafts stay ephemeral.
- Never create ticket without explicit `create` confirmation from user.
- Never call non-MCP HTTP API directly (no `curl`, no `fetch`).
- Identify Jira create-issue tool by its **description**, not its name. MCP server names vary across vendors.
- If no tool's description matches creating Jira issue/story/ticket, go to no-MCP branch. Do not guess. Do not pick near-miss tool.
- Never instruct user to install MCP. Detection only.

## Flow

### 1. Resolve transcript

Take `$ARGUMENTS` as transcript path. If empty, ask once: `Transcript path?` Stop until provided.

Derive `YYYY-MM-DD` (transcript header → filename → file mtime) and kebab-case `<slug>` (meeting title or first agenda line) — same derivation as `/wiki-ingest`. If `wiki/meetings/YYYY-MM-DD-<slug>.md` exists, read it to align owner spellings with what `/wiki-ingest` filed. No match → proceed from transcript alone.

### 2. Extract candidates

Scan transcript for:

- **Action items** — owner + verb phrase ("Jane will fix login bug").
- **Actionable work** — concrete work mentioned, even without owner ("we need to migrate Postgres", "auth bug needs fixing", "dashboard needs new metrics").

Exclude:

- Pure decisions (live in `wiki/decisions.md`).
- Discussion topics, opinions, scheduling.
- Already-completed work.

### 3. Approval gate (always)

Print numbered list. One line per candidate:

```
1. <short verb phrase> — owner: <name or "unassigned">
2. ...
```

Ask via `AskUserQuestion` (single yes/no): `Proceed to create Jira tickets from any of these?`

- **No** → print `Skipped. No tickets drafted.` Stop. Do not touch `wiki/`.
- **Yes** → continue to step 4.

### 4. Detect Jira create-issue tool

Scan currently loaded tool descriptions/schemas. Match by **description content**, not name.

Tool qualifies if its description:

- Says it creates issue/story/ticket in Jira, or
- Mentions Jira issue keys / project keys as required input for issue creation.

MCP server names vary (e.g. `atlassian-rovo`, `jira-cloud`, vendor- or org-specific names). Substring match on tool name unreliable — always check description text.

If multiple tools qualify, prefer simplest "create issue" signature (title/summary + description + project/board + optional assignee/team).

If zero qualify → go to step 5b. Do not retry. Do not pick near-miss.

### 5. Branch on detection

#### 5a. Jira MCP detected

Ask three required inputs in one message (all required to proceed):

```
Three inputs required:
1. Jira board / project key?
2. Default team? (used when context doesn't imply team)
3. Which tasks to ticketize? (numbers from list above, e.g. "1, 3, 5", or "all")
```

If user reply omits any of the three, re-ask only missing ones. Do not proceed until all three answered.

For task selection, also accept `AskUserQuestion` multi-select against step 3 list when free-text parsing ambiguous.

If user selects zero tasks → print `Skipped. No tickets drafted.` Stop.

Draft ticket per selected task:

```
Title: <imperative verb + object, ≤60 chars>
Description: <≤3 sentences, caveman lite, meeting context>
Acceptance criteria:
- <checkable condition>
- <checkable condition>
Team: <suggested from context, fallback to default team>
```

Show all drafts in one block. Ask: `Create these N tickets? Reply 'create' to confirm, or edit inline.`

- On `create` → call detected MCP tool once per ticket. Print resulting Jira keys + URLs.
- On edits → apply edits, re-show drafts, re-ask.
- On `cancel` or "no" → print `Skipped. No tickets created.` Stop.

#### 5b. No Jira MCP detected

Print drafted tickets (same four-field structure, caveman lite). End with:

```
No Jira MCP detected — drafts above are paste-ready.
```

Do not write anywhere. Do not retry detection.

## Caveman lite for ticket fields

- **Titles** — imperative verb + object. "Fix auth bug" not "We need to fix the authentication bug". ≤60 chars.
- **Descriptions** — ≤3 sentences. Drop `a/an/the`, `just`, `basically`, `really`, hedging. Keep technical terms exact. Include 1-line meeting context: source meeting + date.
- **Acceptance criteria** — 2–5 bullets. Each bullet checkable. "Login succeeds with valid creds" not "User should be able to login".
- **Team** — short label (`auth`, `platform`, `billing`). Not "the authentication team".

## Why a separate skill

`/wiki-ingest` already files action items into `wiki/todos.md` and meeting pages. Ticketization is downstream, optional, routes through different system (Jira MCP). Separate means: wiki stays clean if user says no, MCP detection lives in one place, command callable independently of ingest.
