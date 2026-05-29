---
name: qa-jira-report
description: >
  Post QA results to a Jira ticket comment — ONLY when QA passed, and only after
  explicit user confirmation. Detects a Jira comment-add MCP and a Jira
  attachment-upload MCP. Refuses to post on a FAIL verdict. Use when user says
  "post qa results", "comment qa to jira", "report qa to ticket", "log qa
  result", after `qa-execute` produced a report, or invokes `/feature-qa`. Runs
  after `qa-execute`. Always attaches per-scenario screenshots inline and always
  includes the scenario table in the comment.
---

# QA Jira report

Post QA result to Jira ticket comment. Only on PASS. Confirm before posting. Never auto-post. Always inline-attach screenshots. Always include scenario table.

## Hard rules

- Never call non-MCP HTTP directly (no `curl`, no `fetch`). Jira access goes through MCP only.
- Identify Jira tools by their **description**, not name. Names vary across vendors.
- **Post only on PASS.** FAIL verdict → never post.
- Never post without explicit user confirmation, even on PASS.
- **Inline image attachment mandatory.** PASS comment must reference every per-scenario screenshot as a Jira attachment. No attachment MCP → do not post.
- **Scenario table mandatory.** PASS comment must contain the Scenario / Expected result / Screenshot / Pass-Fail rows from `report.md`.
- Missing comment-add MCP OR attachment-upload MCP → print paste-ready comment + manual-attach list, do not retry.

## Flow

### 1. Read verdict + table

Read `qa-reports/<TICKET>/report.md`. Missing → run `qa-execute` for `<TICKET>` first, then return here. Take verdict (PASS / FAIL), scenario count, issues, and the full scenario table rows.

### 2. FAIL branch

Verdict FAIL → do not post. Print:

```
QA did not pass for <TICKET>. Not posting to Jira.
Failed scenarios: <list>
Issues: <list>
```

Stop. Fix issues, re-run `qa-execute` first.

### 3. Detect Jira tools

Verdict PASS → scan loaded tool descriptions/schemas. Match by **description content**, not name.

Need **both**:

- **Comment-add tool** — description says it adds / posts a comment to a Jira issue/ticket.
- **Attachment-upload tool** — description says it uploads / attaches a file to a Jira issue/ticket.

Multiple candidates → prefer simplest signature (issue key + body, or issue key + file path).

Zero qualify for either → step 4b.

### 4. Branch on detection

#### 4a. Both tools detected

1. **Upload screenshots.** For each row in the scenario table, upload `qa-reports/<TICKET>/screenshots/<n>-<slug>.png` via the attachment tool. One call per screenshot. Track attached filenames.

2. **Draft comment.** Markdown body — let the comment-add MCP encode it. Use Jira inline-image syntax `!<filename>!` in the Screenshot column so attached images render inline regardless of body markup:

   ```
   QA passed for <TICKET>.

   Scenarios run: <count> (positive, negative, regression, what-if). All passed, no issues found.

   | Scenario | Expected result | Screenshot | Pass/Fail |
   |---|---|---|---|
   | <plain words> | <plain words> | !<n>-<slug>.png! | Pass |
   | ... | ... | ... | ... |

   Full report: qa-reports/<TICKET>/report.md
   ```

3. **Confirm.** Show full draft (header + table + inline image refs). Ask via `AskUserQuestion` (yes/no): `Post this comment to <TICKET>?`

   - **Yes** → call comment-add tool once. Print result + ticket URL.
   - **No** → print `Not posted.` Note attachments already uploaded — tell user to remove manually if unwanted. Stop.

#### 4b. Comment tool OR attachment tool missing

Do not post. Print drafted comment (same shape as 4a step 2). End with:

```
No Jira <comment|attachment|comment+attachment> MCP detected — comment above is paste-ready. Attach these files manually before posting:
- qa-reports/<TICKET>/screenshots/<n>-<slug>.png
- ...
```

Do not retry detection.
