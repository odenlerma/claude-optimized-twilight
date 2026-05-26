---
description: Quick overview of the wiki — counts, recent meetings, open TODOs, active projects
---

Print compact overview of wiki. No prose preamble.

Steps:

1. Verify `wiki/CLAUDE.md` exists. If not, print `No wiki here. Run /wiki-init first.` and stop.
2. Count files under `wiki/meetings/`, `wiki/people/`, `wiki/projects/`, `wiki/topics/`.
3. List 5 most recent meetings (sort by filename date prefix, descending).
4. Count unchecked `- [ ]` items in `wiki/todos.md`.
5. List project pages where frontmatter `status: active`.
6. Print this exact block:

   ```
   Wiki status (<cwd>/wiki/)
     Meetings: <N>     People: <N>
     Projects: <N>     Topics: <N>
     Open TODOs: <N>

   Recent meetings:
     - YYYY-MM-DD — [[meetings/...]]
     ...

   Active projects:
     - [[projects/...]]
     ...
   ```

   Drop "Recent meetings" or "Active projects" sub-block if its list is empty.
