---
name: qa-jira-report
description: >
  Post QA results to a Jira ticket comment — ONLY when QA passed, and only after
  explicit user confirmation. Detects a Jira comment-add MCP. Refuses to post on
  a FAIL verdict. Use when user says "post qa results", "comment qa to jira",
  "report qa to ticket", "log qa result", after `qa-execute` produced a report,
  or invokes `/feature-qa`. Runs after `qa-execute`.
---

# QA Jira report

Post QA result to Jira ticket comment. Only on PASS. Confirm before posting. Never auto-post.

## Hard rules

- Never call non-MCP HTTP directly (no `curl`, no `fetch`). Jira access goes through MCP only.
- Identify Jira comment-add tool by its **description**, not its name. Names vary across vendors.
- **Post only on PASS.** FAIL verdict → never post.
- Never post without explicit user confirmation, even on PASS.
- No comment tool detected → print paste-ready comment, do not retry.

## Flow

### 1. Read verdict

Read `qa-reports/<TICKET>/report.md`. Missing → run `qa-execute` for `<TICKET>` first, then return here. Take verdict (PASS / FAIL), scenario count, issues.

### 2. FAIL branch

Verdict FAIL → do not post. Print:

```
QA did not pass for <TICKET>. Not posting to Jira.
Failed scenarios: <list>
Issues: <list>
```

Stop. Fix issues, re-run `qa-execute` first.

### 3. Detect Jira comment-add tool

Verdict PASS → scan loaded tool descriptions/schemas. Match by **description content**, not name.

Tool qualifies if its description says it adds / posts a comment to a Jira issue/ticket.

Zero qualify → step 4b. Multiple → prefer simplest add-comment signature (issue key + body).

### 4. Branch on detection

#### 4a. Comment tool detected

Draft comment (caveman lite, plain words):

```
QA passed for <TICKET>.
- Scenarios run: <count> (positive, negative, regression, what-if)
- All passed, no issues found.
- Evidence + full report: qa-reports/<TICKET>/report.md
```

Show draft. Ask via `AskUserQuestion` (yes/no): `Post this comment to <TICKET>?`

- **Yes** → call comment-add tool once. Print result + ticket URL.
- **No** → print `Not posted.` Stop.

#### 4b. No comment tool

Print the drafted comment. End with:

```
No Jira comment MCP detected — comment above is paste-ready.
```

Do not retry detection.
