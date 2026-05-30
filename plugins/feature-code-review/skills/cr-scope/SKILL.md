---
name: cr-scope
description: >
  Scope a feature for code review from a Jira ticket. Detects a Jira reader MCP,
  fetches the ticket, captures the feature intent and acceptance criteria, then
  builds a change inventory — git diff of the feature branch vs base, widened to
  the related code the change touches (callers, shared components, reused
  utilities). Asks when intent is unclear, then writes scope.md. Use when user
  says "code review", "review this feature", "feature code review", "review jira",
  "scope the review for", gives a Jira ticket number to review, or invokes
  `/feature-code-review`. Runs before `cr-review`.
---

# CR scope

Scope a feature for review. Anchor everything to ticket intent. Stay curious — ask when unclear.

## Hard rules

- Never call non-MCP HTTP directly (no `curl`, no `fetch`). Jira access goes through MCP only.
- Identify Jira reader tool by its **description**, not its name. MCP server names vary across vendors.
- No tool description matches reading Jira issues → go to paste fallback. Do not guess. Do not pick near-miss tool.
- Scope is the diff **plus** the code it touches. Reuse and structure findings need context, not just changed lines.
- Capture **intent**, not just code. A review judges code against what the feature must do.

## Flow

### 1. Resolve ticket

First whitespace-delimited token of `$ARGUMENTS` = Jira ticket number/key. Remaining text (optional) = free-text **notes**: extra context to factor into this scope and review (focus areas, constraints, scope). No ticket → ask once: `Jira ticket number?` Stop until provided.

Slug `<TICKET>` = the ticket key (e.g. `PROJ-123`), notes excluded. All output lives under `code-reviews/<TICKET>/`.

### 2. Detect Jira reader tool

Scan currently loaded tool descriptions/schemas. Match by **description content**, not name.

Tool qualifies if its description:

- Says it fetches / gets / searches / reads a Jira issue/ticket, or
- Takes a Jira issue key as input to return issue fields.

Names vary (`atlassian-rovo`, `jira-cloud`, org-specific). Substring match on name unreliable — read description text.

Zero qualify → step 3b. Multiple qualify → prefer simplest get-issue-by-key signature.

### 3. Branch on detection

#### 3a. Jira reader detected

Fetch ticket. Read: summary, description, acceptance criteria, status, relevant comments, linked tickets.

#### 3b. No Jira reader

Ask user to paste ticket details:

```
No Jira MCP detected. Paste ticket details:
1. Summary / title
2. Description
3. Acceptance criteria
```

Stop until provided. Do not retry detection.

### 4. Capture intent

From the ticket, write the feature intent in plain words: what it must do, who uses it, the user-facing surface, and the acceptance criteria the code must satisfy. This is the yardstick for the review.

Notes present → fold them into the yardstick (focus area named → weight it in review; constraint named → treat as criterion).

### 5. Build change inventory

Find the code that implements the feature.

1. **Diff first.** Detect base branch (`main`, `master`, or the repo default). Unclear or multiple candidates → ask user once which base to diff against. Run `git diff <base>...HEAD --stat` then per-file diffs. List changed files + hunks.
2. **Widen for context.** For the changed code, locate closely-related existing code via Explore / Grep: callers of changed functions, shared components/screens it renders into, utilities it reuses or duplicates. Reuse and structure findings need this.
3. **Fallback.** No diff / no feature branch → locate feature files from intent (Explore / Grep) and inventory those instead. Note that scope is file-based, not diff-based.

### 6. Stay curious

Intent vague, acceptance criteria thin, or feature surface ambiguous → ask user with `AskUserQuestion` before writing scope. Resolve unknowns first. Do not assume.

### 7. Write scope

Write `code-reviews/<TICKET>/scope.md`:

```markdown
# Review scope — <TICKET>

Intent: <one-line plain summary of what the feature must do>
Users / surface: <who uses it, where it appears>

## Notes
- (none) | - <user-supplied notes>

## Acceptance criteria
- <criterion the code must satisfy>

## Changed files (diff vs <base>)
- `path/to/file` — <what changed, plain>

## Related context (not changed, needed for review)
- `path/to/file` — <caller / shared component / reused utility>

## Open questions
- (none) | - <unresolved item>
```

Tell user scope path. Next step: run `cr-review` for `<TICKET>`.
