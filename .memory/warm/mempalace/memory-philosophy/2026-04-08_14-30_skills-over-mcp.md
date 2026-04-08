---
date: "2026-04-08T14:30:00"
wing: mempalace
room: memory-philosophy
hall: decisions
tags: [skills, mcp, tokens, architecture]
summarized_from: .memory/active/mempalace/memory-philosophy/2026-04-08_14-30_skills-over-mcp.md
summarized_at: "2026-04-08T15:00:00"
---

## Decision: Claude Code skills over MCP server for memory management

- **Token cost**: MCP (19 tools) costs ~3,054 tokens at startup; native skills (6 skills) cost ~300-600 tokens — skills are 5-10x cheaper on context window
- **Capability**: Skills can use ALL of Claude's tools (Read, Grep, Bash, Write, Edit); MCP tools limited to what the server implements
- User's key observation: "Cloud Code can achieve that without ChromaDB" — correct for sub-10K files, since Claude reading structured files IS semantic search with reasoning on top
- **Architecture chosen**: `/save-memory`, `/recall`, `/memory-cleanup`, `/memory-status`, stop hook (auto-reminds every 12 messages), CLAUDE.md protocol
