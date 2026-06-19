---
description: Resume goaling from the diary — reconstruct where we left off and carry on until all ideas are completed or killed
---

Resume goaling from the diary `docs/notes/goaling-log.md`.

No diary or no `docs/tasks.md` → tell the user to run `/start-goal` first. Stop.

Hand to **`goal-orchestrate`** (resume mode). It reconstructs state — active idea, pipeline phase, build sub-phase, last verified evidence, open blockers — from the diary, `docs/tasks.md`, and the project's research/PRD/evidence files, **reconciling against filesystem ground truth** (files on disk win over the diary on any conflict).

Resolve any open blocker first, then re-enter `goal-pipeline` at the reconstructed point and continue the cross-idea loop until every idea is `completed` or `killed`.
