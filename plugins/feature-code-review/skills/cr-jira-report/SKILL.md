---
name: cr-jira-report
description: >
  Post a code-review result to a Jira ticket comment — ONLY when the review
  verdict is PASS, and only after explicit user confirmation. Detects a Jira
  comment-add MCP. Refuses to post on a CHANGES REQUESTED verdict. Comment carries
  the verdict, the dimensions reviewed, findings grouped by impact dimension with
  severity, and a link to the full report. Use when user says "post the review",
  "comment review to jira", "report review to ticket", after `cr-review` produced
  a report, or invokes `/feature-code-review`. Runs after `cr-review`.
---

# CR Jira report

Post code-review result to Jira ticket comment. Only on PASS. Confirm before posting. Never auto-post.

## Hard rules

- Never call non-MCP HTTP directly (no `curl`, no `fetch`). Jira access goes through MCP only.
- Identify Jira tools by their **description**, not name. Names vary across vendors.
- **Post only on PASS.** CHANGES REQUESTED verdict → never post.
- Never post without explicit user confirmation, even on PASS.
- **Findings table mandatory.** PASS comment must carry the findings grouped by dimension + severity from `review.md` (Minors and notes included).
- Missing comment-add MCP → print paste-ready comment, do not retry.

## Flow

### 1. Read verdict + findings

Read `code-reviews/<TICKET>/review.md`. Missing → run `cr-review` for `<TICKET>` first, then return here. Take verdict (PASS / CHANGES REQUESTED), summary, findings by dimension + severity, strengths.

### 2. CHANGES REQUESTED branch

Verdict CHANGES REQUESTED → do not post. Print:

```
Review did not pass for <TICKET>. Not posting to Jira.
Blockers/Majors: <list>
```

Stop. Fix findings, re-run `cr-review` first.

### 3. Detect Jira comment tool

Verdict PASS → scan loaded tool descriptions/schemas. Match by **description content**, not name.

Need a **comment-add tool** — description says it adds / posts a comment to a Jira issue/ticket.

Multiple candidates → prefer simplest signature (issue key + body). Zero qualify → step 4b.

### 4. Branch on detection

#### 4a. Comment tool detected

1. **Draft comment.** Markdown body — let the comment-add MCP encode it:

   ```
   Code review passed for <TICKET>. Approved.

   <one-line summary>

   Dimensions reviewed: performance, security, responsiveness, error handling, user experience, reusable components, code structure.

   ## Findings
   ### <dimension>
   - [<severity>] <file:line> — <impact>. Direction: <...>.

   (Carry remaining Minor findings / notes. Omit dimensions with none.)

   Full report: code-reviews/<TICKET>/review.md
   ```

2. **Confirm.** Show full draft. Ask via `AskUserQuestion` (yes/no): `Post this review to <TICKET>?`

   - **Yes** → call comment-add tool once. Print result + ticket URL.
   - **No** → print `Not posted.` Stop.

#### 4b. Comment tool missing

Do not post. Print drafted comment (same shape as 4a step 1). End with:

```
No Jira comment MCP detected — comment above is paste-ready.
```

Do not retry detection.
