---
date: "2026-04-08T14:10:00"
wing: mempalace
room: lifecycle-design
hall: decisions
tags: [lifecycle, tiering, archival, 30-60-90]
summarized_from: .memory/active/mempalace/lifecycle-design/2026-04-08_14-10_tiering-strategy-decision.md
summarized_at: "2026-04-08T15:00:00"
---

## Decision: 30/60/90 day memory tiering strategy

- The real gap in AI memory is **lifecycle management**, not retrieval — neither MemPalace (22,000+ memories) nor Claude Code's Auto Memory has aging, archival, or cleanup
- **Tiered approach**: Active (0-30d, full verbatim, ~200-300 files) → Warm (30-60d, summarized) → Cold (60-90d, index entries in manifest.md) → Archive (90d+, permanent, never searched)
- **Rationale**: Keeps active search space small enough that "grep + read + reason" works without ChromaDB; directory structure replaces metadata filtering; tiering replaces scalable search
- **Test mode**: lifecycle.json supports minute-based thresholds to validate the full lifecycle quickly
