---
name: goal-gate
description: >
  Apply the build/kill go-no-go for one idea from its research SUMMARY.md.
  Auto-GO when the summary clears the bar (business: a real profit path; tool:
  clear usefulness + cross-platform; no unresolved compliance blocker) AND
  phase-1 needs no paid deploy. Auto-KILL a clear loser with a written rationale,
  setting the idea killed. Escalate to the human only when phase-1 needs a paid
  deploy/secret or the call is genuinely close. Invoked by goal-pipeline after
  goal-research.
---

# Goal gate

Decide build vs kill. Don't pay to ship a loss. Record why, either way.

## Hard rules

- **Evidence-based.** Decide from `SUMMARY.md`, not optimism. Payment ≠ profit.
- **Kill is cheap — surface it.** A kill only saves money. Auto-kill clear losers, but write a prominent rationale so the human sees it on review.
- **GO authorizes spend.** Auto-GO only when the bar is clear AND phase-1 reaches its milestone with no real-money deploy. Otherwise escalate before spending.
- **Record the rationale every time**, GO or KILL, in `SUMMARY.md` and the diary.

## Flow

1. Read `docs/research/<slug>/SUMMARY.md`: recommendation, decisive factor, risks, "needs paid deploy for phase 1".

2. Decide:
   - **KILL** — summary recommends KILL, or a business idea shows no profit path, or an unresolved compliance blocker. → `goal-backlog` set `killed`; append a `## Gate decision: KILL` block with the rationale to `SUMMARY.md`; log to diary; return KILL.
   - **GO (auto)** — bar cleared (business: profit path positive; tool: useful + cross-platform; no compliance blocker) AND phase-1 needs no paid deploy. → append `## Gate decision: GO`; log; return GO.
   - **ESCALATE** — phase-1 needs a paid deploy/secret, or GO vs KILL is genuinely close. → present `SUMMARY.md`'s verdict + cost + top risks via `AskUserQuestion` (`Build <slug>, or kill it?`). Record the human's call + reason; return it.

3. Return the verdict to `goal-pipeline`.
