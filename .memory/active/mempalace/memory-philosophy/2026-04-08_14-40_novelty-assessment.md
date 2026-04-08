---
date: "2026-04-08T14:40:00"
wing: mempalace
room: memory-philosophy
hall: decisions
tags: [novelty, honesty, auto-memory, lifecycle]
source: session
---

## Decision: Honest novelty assessment — lifecycle is the gap, not the system

User challenged: "Is this actually novel? If so, why isn't Claude Code's team
already doing it?"

**Honest answer: The individual components are NOT novel.**
- File-based storage → standard
- Directory hierarchy → standard
- 30/60/90 tiering → decades-old data lifecycle management
- Knowledge graphs → MemPalace already does this
- Auto-save hooks → MemPalace already ships these

**What Claude Code already has:**
- Auto Memory (MEMORY.md + topic files at ~/.claude/projects/<project>/memory/)
- CLAUDE.md (persistent instructions, survives compaction)
- Hooks (26 event types, can block and inject context)
- Skills (on-demand, cheap on tokens)
- /memory command (view, edit, toggle auto-memory)

**Why Claude Code team hasn't built a full memory palace:**
1. Auto Memory solves 80% of the need for most users
2. The remaining 20% is deeply personal — how you organize differs from others
3. MCP is their answer for power users (enable the ecosystem, not build everything)
4. A full memory system is a product, not a feature (MemPalace is 15+ files, 6000+ lines)

**The one genuinely unaddressed gap:**
Everyone building AI memory focuses on storage and retrieval. Nobody focuses on
lifecycle and decay. Neither MemPalace nor Auto Memory has tiering, archival,
or cleanup. Both just accumulate. The fix is a practice, not a product — but
the practice can be codified into a skill.

**What we're actually building:** A lifecycle discipline layer on top of Claude
Code's existing capabilities. Not a novel system. An integration of existing
patterns applied to a real, unaddressed gap.
