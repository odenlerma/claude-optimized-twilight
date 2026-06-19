---
name: goal-research
description: >
  Run the research phase for one idea — spawn the dedicated research agents in
  parallel by category (market, usefulness, architecture, compliance,
  feasibility), each writing one dossier to docs/research/<slug>/, then
  synthesize SUMMARY.md: reconcile contradictions across the dossiers, list key
  risks, and state an explicit go/no-go recommendation with the decisive factor.
  Invoked by goal-pipeline as the research-gate front of the pipeline.
---

# Goal research

Gate the idea on evidence before any build spend. Spawn role agents in parallel, then synthesize.

## Hard rules

- **Agents do the digging.** Spawn each research role as its own agent (isolated context, parallel). Don't research inline.
- **Cite sources.** Each dossier cites what it found (agents use WebSearch/WebFetch). Drop unverifiable claims.
- **Synthesis reconciles, doesn't average.** Dossiers conflict (architect says buildable, feasibility says a rate-limited API kills it) → name the decisive factor in `SUMMARY.md`.
- **Skip finished work.** A dossier already on disk for this slug → don't re-spawn that agent (resume-safe).

## Flow

### 1. Spawn agents (parallel, by category)

All write to `docs/research/<slug>/`. Pass each agent the idea description, notes, category, and slug.

- **business-idea** → `goal-market-analyst`, `goal-architect`, `goal-compliance-analyst`, `goal-feasibility-analyst`.
- **productivity-tool** → `goal-usefulness-analyst`, `goal-architect`, `goal-compliance-analyst`, `goal-feasibility-analyst`.

Dossiers: `MARKET-ANALYSIS.md` / `USEFULNESS.md` / `ARCHITECTURE.md` / `COMPLIANCE.md` / `TECH-FEASIBILITY.md`.

### 2. Synthesize SUMMARY.md

Read every dossier. Write `docs/research/<slug>/SUMMARY.md`:

```markdown
# Summary — <slug>

## What it is
<1–2 lines>

## Findings
- Market / Usefulness: <takeaway>
- Architecture: <mobile | web> + <stack>
- Compliance: <obligations, or none>
- Feasibility: <buildable? key constraint>

## Cost vs value
<business: recurring infra vs projected revenue → profit verdict. tool: build effort vs reuse value.>

## Key risks
- <risk + why it matters>

## Go / no-go
Recommendation: **GO** | **KILL**
Decisive factor: <the one thing that drove it>
Phase-1 milestone: <smallest verifiable win; for business, the profitable MVP>
Needs paid deploy for phase 1: yes | no
```

Tell the caller the verdict + path. Next: `goal-gate`.
