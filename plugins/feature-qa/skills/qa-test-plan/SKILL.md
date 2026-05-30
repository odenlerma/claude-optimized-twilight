---
name: qa-test-plan
description: >
  Build a MANUAL QA test plan from a Jira ticket. Detects a Jira reader MCP,
  fetches the ticket, studies the codebase, asks questions when requirements are
  unclear, then writes a plan covering positive, negative, regression
  ("works today, must not break"), and what-if edge scenarios. Use when user says
  "feature qa", "qa this ticket", "qa jira", "test plan for", "build qa plan",
  gives a Jira ticket number to QA, or invokes `/feature-qa`. Runs before
  `qa-execute`.
---

# QA test plan

Build manual QA test plan for a Jira ticket. Stay curious — ask when unclear. Output plain language, no jargon.

## Hard rules

- Never call non-MCP HTTP directly (no `curl`, no `fetch`). Jira access goes through MCP only.
- Identify Jira reader tool by its **description**, not its name. MCP server names vary across vendors.
- No tool description matches reading Jira issues → go to paste fallback. Do not guess. Do not pick near-miss tool.
- Always think "what if". Cover scenarios that should change AND scenarios that must stay unchanged.

## Flow

### 1. Resolve ticket

First whitespace-delimited token of `$ARGUMENTS` = Jira ticket number/key. Remaining text (optional) = free-text **notes**: extra context to factor into this plan (focus areas, constraints, scope). No ticket → ask once: `Jira ticket number?` Stop until provided.

Slug `<TICKET>` = the ticket key (e.g. `PROJ-123`), notes excluded. All output lives under `qa-reports/<TICKET>/`.

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

### 4. Analyze codebase

Locate feature code and current behavior. Use Explore / Grep / Read. Map:

- Where the requirement lands in code (files, components, routes, screens).
- How feature behaves today.
- Nearby features that share code or screens — candidates for regression.

### 5. Stay curious

Requirement vague, acceptance criteria thin, or scenario ambiguous → ask user with `AskUserQuestion` before writing plan. Resolve unknowns first. Do not assume.

### 6. Build test plan

Cover four scenario kinds:

- **Positive** — feature works as required, valid inputs, happy path.
- **Negative** — bad input, missing data, wrong permission, error handling.
- **Regression** — features that worked before and must not break (use step 4 nearby-feature map).
- **What-if** — edge cases, limits, odd order of steps, empty/huge values, interrupted flows.

Each scenario: one line, plain words, what a tester does + what should happen. No technical terms.

Notes present → factor them into scenario selection (focus area named → weight those scenarios; scope limit named → respect it).

### 7. Write plan

Write `qa-reports/<TICKET>/test-plan.md`:

```markdown
# QA test plan — <TICKET>

Requirement: <one-line plain summary>

## Notes
- (none) | - <user-supplied notes>

## Positive
1. <do this> → <should see this>

## Negative
1. <do this> → <should see this>

## Regression — must not break
1. <do this> → <still works like before>

## What-if
1. <edge case> → <should see this>

## Open questions
- (none) | - <unresolved item>
```

Tell user plan path. Next step: run `qa-execute` for `<TICKET>`.
