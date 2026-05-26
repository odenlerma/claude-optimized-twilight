---
description: Scaffold a meeting wiki in the current working directory
---

Scaffold `wiki/` and `raw/` in user's current working directory.

Steps:

1. If `wiki/CLAUDE.md` already exists, print `Wiki already initialized at <cwd>/wiki/. Use /wiki-status to see contents.` and stop.
2. Create directories: `wiki/`, `wiki/meetings/`, `wiki/people/`, `wiki/projects/`, `wiki/topics/`, `raw/`.
3. Write `wiki/CLAUDE.md` with exactly this content:

   ```markdown
   # Wiki — schema and conventions

   This directory is an LLM-maintained knowledge wiki built from meeting transcripts
   in `../raw/`. When invoked via the `meeting-wiki` plugin, follow these rules.

   ## Layers

   1. `../raw/` — immutable transcripts. Never edit.
   2. `wiki/` — LLM-owned markdown pages. Edit freely, but stay concise.
   3. This file — the schema. Update only when conventions change.

   ## Page types

   - `meetings/YYYY-MM-DD-<slug>.md` — one per meeting
   - `people/<slug>.md` — one per person
   - `projects/<slug>.md` — one per initiative
   - `topics/<slug>.md` — one per recurring concept
   - `decisions.md`, `todos.md` — flat append-only logs
   - `index.md`, `log.md` — catalog and chronological log

   ## Linking

   Use `[[meetings/2025-05-20-q2-planning]]`, `[[people/jane-doe]]`, etc.
   Always relative to `wiki/`.

   ## Frontmatter (every wiki page)

   ```yaml
   ---
   type: meeting | person | project | topic
   last_updated: YYYY-MM-DD
   last_updated_by: meeting-wiki
   ---
   ```

   Meeting pages add `source:` (path to raw transcript). See meeting template.

   ## Length budgets (HARD CAPS)

   | Page type | Section | Cap |
   |---|---|---|
   | meeting | Summary | 1 paragraph, ~80 words |
   | meeting | Notes | ≤5 bullets, drop if empty |
   | person | Bio | ≤5 lines |
   | person | Recent mentions | 10 entries (FIFO prune) |
   | person | Open actions | 10 entries |
   | project | Status | 1 paragraph |
   | project | Recent decisions | 10 entries |
   | project | Open action items | 10 entries |
   | topic | Body | ≤3 paragraphs |

   `decisions.md`, `todos.md`, `log.md` are append-only and unbounded.
   `index.md` is unbounded but reverse-chron for meetings, alphabetical for entities.

   ## Page templates

   ### Meeting — `meetings/YYYY-MM-DD-<slug>.md`

   ```markdown
   ---
   type: meeting
   date: YYYY-MM-DD
   title: <one-line title>
   attendees: [[people/jane-doe]], [[people/john-smith]]
   projects: [[projects/q2-launch]]
   topics: [[topics/billing]]
   source: ../../raw/YYYY-MM-DD-<slug>.<ext>
   last_updated: YYYY-MM-DD
   last_updated_by: meeting-wiki
   ---

   # <Title>

   ## Summary
   <One short paragraph, ~80 words max.>

   ## Decisions
   - <decision> ([[decisions]])

   ## Action items
   - [ ] <action> — owner: [[people/jane-doe]] — due: YYYY-MM-DD

   ## Notes
   - <≤5 bullets of context. Skip section if nothing.>
   ```

   ### Person — `people/<slug>.md`

   ```markdown
   ---
   type: person
   name: Jane Doe
   aliases: [jane, j.doe]
   last_updated: YYYY-MM-DD
   last_updated_by: meeting-wiki
   ---

   # Jane Doe

   <≤5-line bio. Hand-editable.>

   ## Recent mentions
   - YYYY-MM-DD — [[meetings/YYYY-MM-DD-<slug>]] — <1-line context>

   ## Open actions
   - [ ] <action> — from [[meetings/YYYY-MM-DD-<slug>]]
   ```

   ### Project — `projects/<slug>.md`

   ```markdown
   ---
   type: project
   name: Q2 Launch
   status: active | paused | done
   last_updated: YYYY-MM-DD
   last_updated_by: meeting-wiki
   ---

   # Q2 Launch

   ## Status
   <One short paragraph.>

   ## Recent decisions
   - YYYY-MM-DD — <decision> — [[meetings/...]]

   ## Open action items
   - [ ] <action> — [[people/...]]

   ## Related
   - People: [[people/...]]
   - Topics: [[topics/...]]
   ```

   ### Topic — `topics/<slug>.md`

   ```markdown
   ---
   type: topic
   name: Billing
   last_updated: YYYY-MM-DD
   last_updated_by: meeting-wiki
   ---

   # Billing

   <≤3 paragraphs of synthesized understanding.>

   ## Mentioned in
   - [[meetings/YYYY-MM-DD-<slug>]]
   ```

   ## Editability rules

   - Never delete human-typed lines. If contradicted by a new transcript, append a
     `> Note (YYYY-MM-DD): per [[meetings/...]], <new statement>.` line beneath.
   - Never reformat existing sections — keep headings stable and in template order.
   - A page's body is "hand-edited" if `last_updated_by` is missing or not
     `meeting-wiki`. In that case, append new content rather than rewrite.

   ## Auto-edit posture

   The plugin edits without asking. After each ingest it prints a concise diff
   summary listing created/updated files with one-line reasons. The wiki is a git
   repo; review with `git -C wiki diff`. The plugin never auto-commits.

   ## Operations

   - `/wiki-ingest <path>` — file transcript into wiki.
   - `/wiki-query <question>` — ask question against wiki.
   - `/wiki-ticketize <path>` — draft Jira tickets from transcript work items (report-only unless Jira MCP present).
   - `/wiki-lint` — health check (report-only).
   - `/wiki-status` — quick overview.
   ```

4. Write `wiki/index.md`:

   ```markdown
   # Wiki index

   ## Meetings (most recent first)

   ## People

   ## Projects

   ## Topics
   ```

5. Write `wiki/log.md` (substitute today's date):

   ```markdown
   # Event log

   Append-only. One line per ingest.

   - YYYY-MM-DD — wiki initialized
   ```

6. Write `wiki/decisions.md`:

   ```markdown
   # Decisions log

   Append-only. Prefix every line with the ISO date.
   ```

7. Write `wiki/todos.md`:

   ```markdown
   # Open action items

   Aggregated from meeting pages on ingest. Check off `- [ ]` items here or in source meeting page. Removal is manual — no operation deletes them.

   ## By owner

   ### Unassigned
   ```

8. If `<cwd>` not git repo (no `.git/` directory), tell user: `Run \`git init\` so future ingests are diffable.` Do not run it yourself.

9. Print tree of what was created, then:

   ```
   Drop transcripts into `raw/` and run `/wiki-ingest raw/<file>`.
   ```
