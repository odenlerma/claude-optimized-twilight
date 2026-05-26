---
description: Answer a question using the wiki — reads index first, drills into relevant pages, cites sources
argument-hint: <question>
---

Answer question `$ARGUMENTS` using wiki.

Steps:

1. If `$ARGUMENTS` empty, ask user for question. Stop until provided.
2. Verify `wiki/CLAUDE.md` exists. If not, print `No wiki here. Run /wiki-init first.` and stop.
3. Read `wiki/index.md` to map territory.
4. Pick **at most 5** candidate pages whose titles or topics match question. Read them.
5. Synthesize short answer (≤ 8 sentences). Cite every claim with `[[wikilinks]]` reference. Quote at most one short sentence per source.
6. If question requires sources you didn't read, say so explicitly and stop — do not guess.
7. Ask once (one line, no `AskUserQuestion`): `Save this as a topic page? (y/N)`.
   - If `y`: create `wiki/topics/<slug>.md` using topic template. Set `last_updated_by: meeting-wiki`. Append new page to `wiki/index.md` under `## Topics`. Append one line to `wiki/log.md`: `- YYYY-MM-DD — saved query as topics/<slug>.md`.
   - If `n` or no reply: stop.
