# Memory Protocol

This repository has a file-based memory palace at `.memory/` that stores
decisions, discoveries, and context across sessions.

## On Session Start

1. Read `.memory/active/*/_index.md` files to know what wings/rooms exist
2. Read `.memory/diary/` for the most recent diary entry to understand what happened last session

## Before Answering About Past Decisions

**Always check memory before guessing.** Use Grep to search `.memory/active/` and `.memory/warm/` for relevant keywords. Read matching files. Cite your sources.

## During Conversation

When important decisions, discoveries, or preferences emerge, note them mentally. Use `/save-memory` to file them before the session ends.

## Before Session End

Save any unfiled decisions or insights using `/save-memory`. Write a diary entry to `.memory/diary/YYYY-MM-DD.md` summarizing what was discussed and decided.

## Available Skills

- `/save-memory` — File a memory with wing/room/hall classification
- `/recall` — Search memory files for past decisions and context
- `/memory-cleanup` — Run lifecycle tiering (active → warm → cold → archive)
- `/memory-status` — Show memory palace overview

## Memory Philosophy

- **Store verbatim, not summaries.** Raw context with reasoning > extracted bullet points.
- **Lifecycle matters.** Active memories age into warm (summarized), cold (indexed), and archive. Run `/memory-cleanup` periodically.
- **Structure aids retrieval.** Wing (project) → Room (topic) → Hall (type) narrows search space so grep + read is effective without vector search.

## Build & Test

```bash
pip install -e .
pytest
```
