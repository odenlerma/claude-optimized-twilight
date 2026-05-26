---
description: Ingest a meeting transcript into the wiki — extract people, projects, topics, decisions, and action items, then update all relevant pages
argument-hint: <transcript-path>
---

Ingest transcript at `$ARGUMENTS` (`.txt`/`.md`/`.vtt`/`.srt` from Zoom, Granola, Otter, Meet).

Steps:

1. If `$ARGUMENTS` empty, ask user for path. Stop until provided.
2. Verify `wiki/CLAUDE.md` exists in cwd. If not, print `No wiki here. Run /wiki-init first.` and stop.
3. Read `wiki/CLAUDE.md` to load schema, page templates, length budgets into context.
4. Read transcript at `$ARGUMENTS`.
5. Derive `YYYY-MM-DD` (transcript header → filename → file mtime, in that order) and kebab-case `<slug>` from meeting title or first agenda line.
6. If `$ARGUMENTS` not already inside `raw/`, copy to `raw/YYYY-MM-DD-<slug>.<ext>`. Never modify copy afterward.
7. Extract entities from transcript:
   - **Attendees** — dedupe against existing `wiki/people/*.md` by `name` and `aliases`. New attendees → new person page.
   - **Projects/initiatives** — dedupe against `wiki/projects/*.md`. Create new project pages as needed.
   - **Topics** — recurring concepts warranting own page. Dedupe against `wiki/topics/*.md`.
   - **Decisions** — statements of form "we will…", "agreed to…", "decided…".
   - **Action items** — owner + verb phrase; flag unowned with `### Unassigned`.
8. Read `wiki/index.md` to confirm which entity pages already exist. Reuse by name or alias.
9. Write meeting page at `wiki/meetings/YYYY-MM-DD-<slug>.md` using meeting template from `wiki/CLAUDE.md`. Respect ~80-word Summary cap. Drop empty sections.
10. For each new person/project/topic, create page with appropriate template. Set `last_updated_by: meeting-wiki` and today's `last_updated`.
11. For each **existing** person/project/topic page touched by this transcript:
    - Append 1-line entry to `## Recent mentions` (or equivalent section): `- YYYY-MM-DD — [[meetings/YYYY-MM-DD-<slug>]] — <1-line context>`.
    - Prune oldest entries beyond cap defined in `wiki/CLAUDE.md`.
    - For project pages, append to `## Recent decisions` if applicable.
    - Update `last_updated`. If page's existing `last_updated_by` not `meeting-wiki`, **append rather than rewrite** existing sections, flag this file in diff summary as `(preserved hand edits)`.
12. Append one bullet per decision to `wiki/decisions.md`:
    `- YYYY-MM-DD — <decision> — [[meetings/YYYY-MM-DD-<slug>]] — projects: [[projects/<slug>]]`
13. Append open action items to `wiki/todos.md` under `### [[people/<owner-slug>]]` headings (create heading if missing). Unowned items go under `### Unassigned`.
14. Update `wiki/index.md`:
    - Add any new entity pages (alphabetized in their section).
    - Add new meeting at top of `## Meetings (most recent first)`, keep section sorted reverse-chronologically.
15. Append one line to `wiki/log.md`:
    `- YYYY-MM-DD — ingested raw/<file> — created N pages, touched M`
16. Print the diff summary block (use exactly this shape, with real file names and short reasons):

    ```
    Wiki updated.
      Created:
        + meetings/<file>.md   (new meeting)
        + people/<file>.md      (first mention)
      Updated:
        ~ people/<file>.md      (+1 recent mention, +1 open action)
        ~ projects/<file>.md    (+1 decision)
        ~ decisions.md          (+N entries)
        ~ todos.md              (+N actions for <owner>)
        ~ index.md, log.md
    Review: `git -C wiki diff` for full detail.
    ```

    Use `+` for created files, `~` for updated. One short reason per file. Mark any page whose body was preserved due to hand edits as `~ <file> (preserved hand edits)`.

17. If meeting produced action items or actionable work, suggest `/wiki-ticketize $ARGUMENTS` as next step. Do not auto-invoke.

18. **Do not run git commit.** User reviews and commits.
