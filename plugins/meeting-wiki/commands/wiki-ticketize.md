---
description: Convert meeting transcript action items and actionable work into Jira ticket drafts — with explicit human approval at every gate. Routes through installed Jira MCP if present, otherwise prints paste-ready drafts.
argument-hint: <transcript-path>
---

Load `jira-ticketize` skill and run against `$ARGUMENTS`.

Steps:

1. If `$ARGUMENTS` empty, ask user for transcript path. Stop until provided.
2. Derive `YYYY-MM-DD` (transcript header → filename → file mtime) and kebab-case `<slug>` (meeting title or first agenda line) — same derivation as `/wiki-ingest`. If `wiki/meetings/YYYY-MM-DD-<slug>.md` exists, read it to align owner spellings with prior `/wiki-ingest`. No match → proceed from transcript alone.
3. Extract candidates from transcript:
   - **Action items** — owner + verb phrase.
   - **Actionable work** — concrete work mentioned, even without owner (bugs, migrations, follow-ups).
   - Exclude pure decisions, discussion topics, scheduling, completed work.
4. Print numbered candidate list (one line each: short verb phrase + owner or "unassigned").
5. Ask via `AskUserQuestion` (yes/no): `Proceed to create Jira tickets from any of these?`
   - **No** → print `Skipped. No tickets drafted.` Stop. Do not touch `wiki/`.
   - **Yes** → continue.
6. Detect Jira create-issue tool by scanning currently loaded tool **descriptions** (not names — MCP server names vary). Tool qualifies if its description says it creates issue/story/ticket in Jira, or requires Jira project/issue keys.
7. Branch:
   - **MCP detected** — ask three required inputs in one message:
     1. Jira board / project key?
     2. Default team?
     3. Which tasks to ticketize? (numbers, or `all`)
     Re-ask any missing inputs. Draft ticket per selected task with caveman-lite **title, description, acceptance criteria, team** (per-ticket suggestion, fallback to default). Show drafts. Ask: `Create these N tickets? Reply 'create' to confirm, or edit inline.` On `create`, call detected MCP tool once per ticket. Print Jira keys + URLs.
   - **No MCP detected** — print drafted tickets (same four-field caveman-lite structure). End with `No Jira MCP detected — drafts above are paste-ready.` Do not write anywhere.

Hard rules:

- Never write to `wiki/`. Candidate list and drafts stay ephemeral.
- Never create ticket without explicit `create` confirmation.
- Never call non-MCP HTTP API directly.
- Identify Jira tool by description, not name.
- If no tool qualifies, take no-MCP branch. Do not guess.
