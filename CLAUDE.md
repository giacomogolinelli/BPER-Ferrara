# BPER Ferrara — Claude Rules

## Knowledge System
This project uses a wiki-first knowledge system. Knowledge lives in `docs/wiki/`, not in chat history.

## Session Protocol (mandatory)

### Session Start (auto, no confirmation needed)
0. Run `python3 scripts/version_check.py` — checks for updates, silent if current, prints notice if outdated
1. Read `docs/wiki/index.md` — get the full page list
2. Read `docs/wiki/current-status.md` — know where things stand
3. Read `docs/wiki/log.md` — understand recent session history
4. Read additional wiki pages **only as needed** for the current task (don't read everything)

### During Work
- New decision made → update the relevant wiki page immediately
- **Any file mentioned, received, referenced, or saved — including screenshots, customer attachments, chat images, debug captures, PDFs, Excel, CAD files, archives → check `manifests/raw_sources.csv` first. Not listed → register it before proceeding.** For batches, run `python3 scripts/ingest_raw.py` instead of hand-editing rows one by one. This is the single most commonly skipped step.
- Task completed → update `docs/wiki/current-status.md`

### Session End
1. Append one line to `docs/wiki/log.md`: date, topic, key outcomes
2. Update `docs/wiki/current-status.md` with latest state
3. If context is running low → generate `CONTINUATION-SUMMARY.md`

### Consistency
- If `current-status.md` conflicts with other wiki pages → trust the more specific page, then fix `current-status.md`
- If `log.md` is missing entries from before your session → don't guess, only append your own
- If two wiki pages contradict each other → flag it to the user, resolve before proceeding

### Token Budget
Normal operations are cheap. The wiki is designed for incremental updates, not bulk reads.
- **Session start**: read 3 files (index, status, log). ~2K tokens.
- **During work**: read specific pages one at a time, only when needed.
- **Never read all wiki pages upfront.**

Full audit and recompilation are disaster recovery, not normal workflow.

### Frontmatter
Every wiki page (except index.md and log.md) must start with YAML frontmatter:
```yaml
---
title: Page Title
source: where this info came from (raw file path, URL, or "session")
source_hash: a1b2c3d4e5f67890
compiled_at: 2026-05-15T00:00:00+00:00
compiled_from: []
created: 2026-05-15
tags: [relevant, tags]
status: current
---
```

### Rules
- compile-first: don't just answer, write conclusions into wiki pages
- writeback is mandatory: if you learned something durable, it goes in the wiki
- raw files stay outside Git (in `raw/`), only manifests go in
- all non-code files are "raw" — PDFs, spreadsheets, images, screenshots, attachments
- every wiki page must have frontmatter (title + source + created)
