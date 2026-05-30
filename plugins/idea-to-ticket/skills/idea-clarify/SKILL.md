---
name: idea-clarify
description: >
  Turn a raw idea into a sharp intent and acceptance criteria before any ticket
  is drafted. Interrogates vague input, names contradictions, pushes back — no
  sycophancy. Optionally checks a Jira reader MCP for duplicate tickets. Use when
  user says "idea to ticket", "make a ticket for", "ticketize this idea", "draft
  a jira ticket", "new feature ticket", gives a rough idea to file, or invokes
  `/idea-to-ticket`. Runs first, before `idea-assess`.
---

# Idea clarify

Turn a fuzzy idea into a sharp intent + acceptance criteria. Interrogate first. Do not draft from vagueness.

## Hard rules

- **Interrogate vague requests.** Do not build a ticket from a fuzzy idea. Resolve unknowns first.
- **Disagree when something is off.** Flag contradictions before acting — never proceed past them silently. No sycophancy. No flattery filler.
- **Never silently overwrite.** Draft dir already exists → flag it, ask before reusing.
- Never call non-MCP HTTP directly (no `curl`, no `fetch`). Jira access goes through MCP only.
- Identify Jira reader tool by its **description**, not its name. Names vary across vendors.

## Flow

### 1. Resolve idea

`$ARGUMENTS` = free-text idea. Remaining text after a clear break (optional) = **notes**: constraints, scope, target area. Empty → ask once: `What's the idea?` Stop until provided.

Derive `<slug>` = kebab-case short title from the idea (e.g. `csv-export-reports`). All output lives under `ticket-drafts/<slug>/`.

### 2. Interrogate

Read the idea hard. Surface, via `AskUserQuestion`, anything that blocks a clean ticket:

- **Vague** — "make reports better": what report, better how, for whom?
- **Contradictory** — idea conflicts with stated notes or with itself → name the contradiction, ask which wins.
- **Scope-creep** — idea bundles several unrelated asks → note it, ask what's in scope now.
- **Unstated success** — no way to tell when it's done → ask what "done" looks like.

Resolve unknowns before writing intent. Do not assume. Do not flatter the idea.

### 3. Optional duplicate check

Jira reader MCP present (description says it fetches / searches / reads Jira issues) → search for existing tickets covering this idea. Likely duplicate found → surface key + summary, ask whether to continue or stop. No reader MCP → skip silently.

### 4. Write intent

Existing `ticket-drafts/<slug>/` → flag, ask before overwriting. Then write `ticket-drafts/<slug>/intent.md`:

```markdown
# Intent — <slug>

Intent: <one-line plain summary of what to build and why>
Users / surface: <who uses it, where it appears>

## Notes
- (none) | - <user-supplied notes>

## Acceptance criteria
- <checkable condition — "CSV downloads with all visible columns">

## Open questions
- (none) | - <unresolved item>
```

Tell user the path. Next: run `idea-assess` if a codebase is present, else `idea-draft`.
