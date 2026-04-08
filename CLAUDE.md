# Memory Protocol

This repository has a file-based memory palace with two locations:
- **Project memory** at `.memory/` — decisions specific to this project
- **Shared memory** at `~/.memory/` — cross-project decisions accessible from any repo

## On Session Start

1. Read `.memory/active/*/_index.md` files to know what project wings/rooms exist
2. Read `~/.memory/active/*/_index.md` files to know what shared wings/rooms exist
3. Read `.memory/diary/` for the most recent diary entry to understand what happened last session

## Before Answering About Past Decisions

**Always check memory before guessing.** Search both locations:
- Grep `.memory/active/` and `.memory/warm/` for project memories
- Grep `~/.memory/active/` and `~/.memory/warm/` for shared memories
Read matching files. Cite your sources. Label results as [project] or [shared].

## During Conversation

When important decisions, discoveries, or preferences emerge, note them mentally. Use `/save-memory` to file them before the session ends. Consider whether they are project-specific or shared (cross-project).

## Before Session End

Save any unfiled decisions or insights using `/save-memory`. Write a diary entry to `.memory/diary/YYYY-MM-DD.md` summarizing what was discussed and decided.

## Available Skills

- `/save-memory` — File a memory with wing/room/hall classification (project or shared scope)
- `/recall` — Search both project and shared memory for past decisions and context
- `/memory-cleanup` — Run lifecycle tiering on both locations
- `/memory-status` — Show dashboard for both project and shared memory

## Memory Philosophy

- **Store verbatim, not summaries.** Raw context with reasoning > extracted bullet points.
- **Lifecycle matters.** Active memories age into warm (summarized), cold (indexed), and archive. Run `/memory-cleanup` periodically.
- **Scope matters.** Project-specific decisions stay in `.memory/`. Cross-cutting decisions (design systems, conventions, preferences) go to `~/.memory/`.
- **Structure aids retrieval.** Wing (project/topic) → Room (subtopic) → Hall (type) narrows search space so grep + read is effective without vector search.

## Build & Test

```bash
pip install -e .
pytest
```
