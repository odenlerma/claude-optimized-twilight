---
name: goal-log
description: >
  Append to and parse the goaling diary at docs/notes/goaling-log.md — the
  dated, append-only, caveman record of pipeline progress. Each entry carries a
  fixed parseable header (idea, phase, sub-phase, state, evidence, blocker) plus
  free caveman bullets, so a later session resumes from it. Use at every pipeline
  phase boundary, on any blocker or escalation, and when the user invokes
  `/log-goaling` or `/continue-goaling`.
---

# Goal log

Own `docs/notes/goaling-log.md` — the diary. Append-only, dated, caveman. The last entry is always a truthful "where we are".

## Hard rules

- **Append only.** Never edit or delete a past entry — even a wrong one. Correct by appending a newer entry.
- **Header is the resume contract.** Every entry starts with the fixed header + the three key lines below. Keep them exact so resume can parse them.
- **Caveman bullets.** Drop articles and filler. Record only what matters: what changed, what blocks, what's next.
- **Write-ahead.** Log "entering phase X" before doing X, so a crash mid-phase still leaves a correct marker.

## Entry format

```
## <YYYY-MM-DD HH:MM> — idea: <slug> — phase: <phase> — sub: <subphase>/<substate>
state: <in-progress|completed|killed|blocked>
evidence: <relative path, or —>
blocker: <one line, or none>
- did: <what got done>
- doing: <in flight>
- next: <next step>
```

`<phase>` ∈ research · gate · scaffold · prd · build · marketing · handover.
`<subphase>/<substate>` for build = `phase-<N>/<implementing|verifying|verified|blocked>`; other phases use `—/—`.

## Flow

### Append

Run `date '+%Y-%m-%d %H:%M'` for the timestamp. Fill the header from current state. Write 1–3 caveman bullets. Fold in any `$ARGUMENTS` note. Diary missing → create it with a `# Goaling log` title line first.

### Parse (resume — called by `goal-orchestrate`)

Read the file. Find the **last** entry whose `idea:` matches the active slug. Return its `phase`, `sub`, `state`, `evidence`, `blocker` for reconciliation. No entry for the slug → treat as fresh (no prior progress).
