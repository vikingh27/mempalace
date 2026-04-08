---
date: "2026-04-08T14:00:00"
wing: mempalace
room: memory-philosophy
hall: discoveries
tags: [claude-code, semantic-search, vector-db, chromadb]
source: session
---

## Discovery: Claude Code's file reading IS semantic search

Key insight from our analysis: Claude Code reading files and reasoning about
them is functionally superior to ChromaDB vector search for most practical
use cases.

Vector search (ChromaDB) finds mathematical embedding distance. Claude Code
reads files and actually understands them — handles ambiguity, cross-references
across files, follows reasoning threads, and provides answers not just chunks.

A skill like `/search-decision` that tells Claude to grep through structured
memory files, read the hits, and synthesize is **richer** than what ChromaDB
returns. ChromaDB gives you the 5 closest text chunks. Claude gives you an
answer with reasoning.

**Where it breaks down**: Only at scale. At 500 files, Claude handles it
easily. At 5,000 files, grep returns too many hits to triage. At 22,000+
(MemPalace's benchmark scale), vector search earns its keep because it returns
top-5 in milliseconds without reading anything.

**Conclusion**: For a typical developer across a few projects (hundreds to
low thousands of memory files), Claude Code + structured files + skills is
not just "good enough" — it's arguably better because you get reasoning,
not just retrieval.
