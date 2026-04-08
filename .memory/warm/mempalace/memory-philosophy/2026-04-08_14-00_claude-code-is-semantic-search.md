---
date: "2026-04-08T14:00:00"
wing: mempalace
room: memory-philosophy
hall: discoveries
tags: [claude-code, semantic-search, vector-db, chromadb]
summarized_from: .memory/active/mempalace/memory-philosophy/2026-04-08_14-00_claude-code-is-semantic-search.md
summarized_at: "2026-04-08T15:00:00"
---

## Discovery: Claude Code's file reading IS semantic search

- Claude Code reading files + reasoning is functionally superior to ChromaDB vector search for most use cases — it handles ambiguity, cross-references, and provides answers with reasoning, not just text chunks
- A `/recall` skill using grep + read + synthesize is **richer** than ChromaDB's top-5 nearest embeddings
- **Breaks down at scale**: works well at 500 files, manageable at 5,000, but at 22,000+ (MemPalace scale) vector search earns its keep
- **Conclusion**: For typical developer use (hundreds to low thousands of memory files), Claude Code + structured files + skills is "arguably better because you get reasoning, not just retrieval"
