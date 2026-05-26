---
description: Health check the wiki — contradictions, stale claims, orphan pages, missing cross-refs, budget violations
---

Lint wiki. Report only; never auto-fix.

Steps:

1. Verify `wiki/CLAUDE.md` exists. If not, print `No wiki here. Run /wiki-init first.` and stop.
2. Build inventory: every `.md` under `wiki/` with its frontmatter `type`, `last_updated`, outgoing wikilinks.
3. Run these checks and collect findings:
   - **Missing pages** — any wikilink whose target file doesn't exist.
   - **Orphans** — any page not linked from `index.md` or any other page (decisions.md, todos.md, log.md, index.md, CLAUDE.md exempt).
   - **Stale projects** — `type: project` with `status: active` and no mention in any meeting in last 60 days.
   - **Contradictions** — same fact stated differently across pages (heuristic: conflicting `> Note (YYYY-MM-DD): ...` annotations).
   - **Missing cross-refs** — meeting attendee with no `people/` page; project mentioned in meeting but absent from `index.md`.
   - **Budget violations** — pages exceeding section caps defined in `wiki/CLAUDE.md`.
4. Print findings grouped by category. Each finding one line:
   `<file> — <category> — <short reason>`
5. If nothing found, print: `Wiki is clean.`
6. End with: `Run /wiki-ingest, /wiki-init, or edit by hand to fix. Re-run /wiki-lint when done.`
