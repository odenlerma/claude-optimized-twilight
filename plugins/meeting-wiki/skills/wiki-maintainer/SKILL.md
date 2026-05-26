---
name: wiki-maintainer
description: >
  Disciplined maintainer of a markdown "meeting wiki" — turns meeting transcripts
  into an evolving, interlinked knowledge base of people, projects, topics,
  decisions, and action items. Use when the user says "meeting wiki", "meeting
  notes", "ingest transcript", "team wiki", "meeting knowledge base", "summarize
  this meeting into the wiki", drops a Zoom/Granola/Otter/Meet transcript and
  asks Claude to file it, or invokes any `/wiki-*` slash command (`/wiki-init`,
  `/wiki-ingest`, `/wiki-query`, `/wiki-lint`, `/wiki-status`).
---

# Wiki maintainer

Maintain markdown wiki in `<cwd>/wiki/`. Wiki layout, page templates, length budgets defined in `<cwd>/wiki/CLAUDE.md` — **read that file first on every invocation if it exists**. If absent, tell user to run `/wiki-init` and stop.

## Architecture

Three layers:

1. `<cwd>/raw/` — immutable transcripts. Never modify. Source of truth.
2. `<cwd>/wiki/` — LLM-owned markdown pages. Edit freely, stay concise.
3. `<cwd>/wiki/CLAUDE.md` — schema for future Claude sessions. Update only when conventions change.

Cross-references use Obsidian-style `[[wikilinks]]`. Slugs lowercase-kebab. Dates ISO `YYYY-MM-DD`.

## Three operations

### Ingest (`/wiki-ingest <transcript-path>`)

1. Verify `wiki/CLAUDE.md` exists. If not, tell user to run `/wiki-init`.
2. Read `wiki/CLAUDE.md` to load schema and budgets into context.
3. Read transcript at given path.
4. Derive `YYYY-MM-DD` (transcript header → filename → file mtime, in that order) and kebab-case `<slug>` from meeting title or first agenda line.
5. If transcript not already inside `raw/`, copy to `raw/YYYY-MM-DD-<slug>.<ext>`. Never modify copy afterward.
6. Extract entities:
   - **Attendees** — dedupe against existing `wiki/people/*.md` by `name` and `aliases` frontmatter.
   - **Projects/initiatives** mentioned.
   - **Topics** — recurring concepts warranting own page.
   - **Decisions** — statements of form "we will…", "agreed to…", "decided…".
   - **Action items** — owner + verb phrase; flag unowned with `### Unassigned`.
7. Read `wiki/index.md` and existing entity directories. Reuse existing pages by name or alias. Create new pages only when no match.
8. Write meeting page at `wiki/meetings/YYYY-MM-DD-<slug>.md` using meeting template from `wiki/CLAUDE.md`.
9. For each referenced person/project/topic page:
   - Append 1-line entry to `## Recent mentions` with date + meeting wikilink + 1-line context.
   - Prune oldest entries beyond per-section cap.
   - Update `last_updated`, set `last_updated_by: meeting-wiki` in frontmatter.
   - If `last_updated_by` missing or not `meeting-wiki`, treat body as hand-edited: **append** new content; do not rewrite existing sections.
10. Append one bullet per decision to `wiki/decisions.md`.
11. Append open action items to `wiki/todos.md` under appropriate owner heading (or `### Unassigned`).
12. Update `wiki/index.md` — add any new entity pages (alphabetized); re-sort meetings reverse-chronologically.
13. Append one line to `wiki/log.md`: `- YYYY-MM-DD — ingested raw/<file> — created N pages, touched M`.
14. Print diff summary block (see below).
15. If meeting produced action items or actionable work, suggest `/wiki-ticketize <same-transcript-path>` as next step. Do not auto-invoke.
16. **Do not run git commit.** User reviews and commits.

### Query (`/wiki-query <question>`)

1. Verify `wiki/CLAUDE.md` exists.
2. Read `wiki/index.md` first to map territory.
3. Pick at most **5 candidate pages** whose titles or topics match question. Read them.
4. Synthesize short answer (≤ 8 sentences). Cite every claim with `[[wikilinks]]` reference. Quote at most one short sentence per source.
5. If question requires sources you didn't read, say so explicitly and stop — don't guess.
6. Ask once (one line, no `AskUserQuestion`): "Save this as a topic page? (y/N)". If yes, create `wiki/topics/<slug>.md` with answer in topic template; append to `index.md` and `log.md`.

### Lint (`/wiki-lint`)

Report only. Never auto-fix.

1. Verify `wiki/CLAUDE.md` exists.
2. Inventory every `.md` under `wiki/` with its frontmatter `type`, `last_updated`, outgoing wikilinks.
3. Run these checks:
   - **Missing pages** — wikilinks whose target file doesn't exist.
   - **Orphans** — pages not linked from `index.md` or any other page (four flat logs exempt).
   - **Stale projects** — `type: project` with `status: active` and no mention in any meeting in last 60 days.
   - **Contradictions** — same fact stated differently across pages (heuristic: conflicting `> Note` annotations).
   - **Missing cross-refs** — meeting attendee with no `people/` page; project mentioned in meeting but absent from `index.md`.
   - **Budget violations** — any page exceeding section caps in `wiki/CLAUDE.md`.
4. Print findings grouped by category. Each finding one line: `<file> — <category> — <short reason>`.
5. End with: `Run /wiki-ingest, /wiki-init, or edit by hand to fix. Re-run /wiki-lint when done.`.

## Auto-edit posture

Edit without confirmation. At **end of every `/wiki-ingest`**, print exactly this block:

```
Wiki updated.
  Created:
    + meetings/2025-05-20-q2-planning.md   (new meeting)
    + people/john-smith.md                  (first mention)
  Updated:
    ~ people/jane-doe.md                    (+1 recent mention, +1 open action)
    ~ projects/q2-launch.md                 (+1 decision)
    ~ decisions.md                          (+1 entry)
    ~ todos.md                              (+2 actions for jane-doe, +1 unassigned)
    ~ index.md, log.md
Review: `git -C wiki diff` for full detail.
```

Use `+` for created, `~` for updated. One short reason in parentheses per file. Mark preserved hand edits as `~ <file> (preserved hand edits)`.

## Conflict handling

- **Contradiction with existing prose**: do not overwrite. Append `> Note (YYYY-MM-DD): per [[meetings/...]], <new statement>.` directly under contradicted line.
- **Same action item in two meetings**: keep one entry in `todos.md`, list both meetings in `from:` field.
- **Ambiguous attendee name** (e.g. "Jane" with two `jane-*` pages): leave `> Note` on meeting page, skip auto-linking until user disambiguates.

## Conciseness — hard rules

- Meeting summary: max ~80 words, one paragraph.
- Bullet lists capped per `wiki/CLAUDE.md` budgets.
- Never quote transcript verbatim in wiki pages. Paraphrase.
- Never add watermarks, "generated by" lines, emoji, signatures.
- Drop empty sections — no `## Notes\n_None_`.

## Editability

- Every page has `last_updated` and `last_updated_by` in frontmatter. Auto-edits set `last_updated_by: meeting-wiki`.
- Heading order fixed by template. Ingest never reorders or renames headings; new content lands under existing headings.
- If page's `last_updated_by` not `meeting-wiki`, treat body as hand-edited, prefer append-only updates. Flag in diff summary as `(preserved hand edits)`.
- Wiki is git repo. User reviews with `git -C wiki diff`, commits/reverts deliberately. **Never auto-commit.**
