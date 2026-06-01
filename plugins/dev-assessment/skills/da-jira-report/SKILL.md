---
name: da-jira-report
description: >
  Post a developer assessment to a Jira ticket comment — ONLY after explicit
  human approval. Detects a Jira comment-add MCP by description. Drafts a
  condensed comment from assessment.md (summary, complexity/effort, key risks,
  open questions, link to full report), shows it, and posts only on a yes via
  AskUserQuestion. No comment MCP → prints a paste-ready comment. Never
  auto-posts. Use when user says "post the assessment", "comment to jira",
  "report to ticket", after `da-assess` produced an assessment, or invokes
  `/dev-assessment`. Runs last.
---

# DA Jira report

Post the dev assessment to a Jira ticket comment. Only on explicit human approval. Never auto-post.

## Hard rules

- Never call non-MCP HTTP directly (no `curl`, no `fetch`). Jira access goes through MCP only.
- Identify Jira tools by their **description**, not name. Names vary across vendors.
- **Never post without explicit human confirmation.** Never auto-post.
- Missing comment-add MCP → print paste-ready comment, do not retry.

## Flow

### 1. Read assessment

Read `dev-assessments/<TICKET>/assessment.md`. Missing → run `da-assess` for `<TICKET>` first, then return. Take summary, complexity/effort, affected repos, risks, open questions.

### 2. Detect Jira comment tool

Scan loaded tool descriptions/schemas. Match by **description content**, not name.

Need a **comment-add tool** — description says it adds / posts a comment to a Jira issue/ticket.

Multiple candidates → prefer simplest signature (issue key + body). Zero qualify → step 3b.

### 3. Branch on detection

#### 3a. Comment tool detected

1. **Draft comment.** Condensed from the assessment — markdown body, let the comment-add MCP encode it:

   ```
   Dev assessment for <TICKET>.

   <one-line summary>

   Complexity / effort: <S | M | L | XL> — <why>

   Repos in scope: <repo paths>

   ## Key risks
   - <risk>

   ## Open questions
   - <question>

   Full assessment: dev-assessments/<TICKET>/assessment.md
   ```

2. **Confirm.** Show full draft. Ask via `AskUserQuestion` (yes/no): `Post this assessment to <TICKET>?`

   - **Yes** → call comment-add tool once. Print result + ticket URL.
   - **No** → print `Not posted.` Stop.

#### 3b. Comment tool missing

Do not post. Print drafted comment (same shape as 3a step 1). End with:

```
No Jira comment MCP detected — comment above is paste-ready.
```

Do not retry detection.
