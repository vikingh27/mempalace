---
date: "2026-04-08T14:30:00"
wing: mempalace
room: memory-philosophy
hall: decisions
tags: [skills, mcp, tokens, architecture]
source: session
---

## Decision: Claude Code skills over MCP server for memory management

We evaluated using MemPalace as an MCP server vs building native Claude Code
skills. Decision: native skills.

**Token comparison:**
- MemPalace MCP (19 tools): ~3,054 tokens at startup (all schemas loaded)
- Native skills (6 skills): ~300-600 tokens (only descriptions load, full
  content on-demand when invoked)
- Skills are 5-10x cheaper on context window

**Capability comparison:**
- MCP gives structured tool schemas that Claude knows how to call automatically
- Skills give instructions that Claude follows, but with more flexibility
- Skills can use ALL of Claude's tools (Read, Grep, Bash, Write, Edit)
- MCP tools are limited to what the server implements

**The user's key observation:** "Cloud Code can achieve that without ChromaDB
in my understanding." This is correct for sub-10K memory files. Claude Code
reading structured files IS semantic search — with full reasoning on top.

**Architecture chosen:**
- `/save-memory` — file memories with wing/room/hall classification
- `/recall` — grep + read + synthesize from memory files
- `/memory-cleanup` — lifecycle tiering (the novel piece)
- `/memory-status` — overview dashboard
- Stop hook — auto-reminds to save every 12 messages
- CLAUDE.md — memory protocol loaded every session
