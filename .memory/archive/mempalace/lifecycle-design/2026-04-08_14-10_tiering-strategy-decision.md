---
date: "2026-04-08T14:10:00"
wing: mempalace
room: lifecycle-design
hall: decisions
tags: [lifecycle, tiering, archival, 30-60-90]
source: session
---

## Decision: 30/60/90 day memory tiering strategy

We identified that the REAL gap in AI memory systems is not retrieval but
lifecycle management. Neither MemPalace nor Claude Code's Auto Memory handles
aging, archival, or cleanup. Both just accumulate forever.

MemPalace has 22,000+ memories with zero lifecycle policy. Claude Code's
Auto Memory grows until it hits 200 lines and then behavior is unclear.
The user's document-generation approach also grows without bound.

**Agreed approach — tiered lifecycle:**

- **Active (0-30 days)**: Full verbatim files. Claude searches these directly.
  Target: ~200-300 files max. Grep + read is instant at this scale.
- **Warm (30-60 days)**: Summarized. Each memory condensed to key decisions,
  outcomes, rationale in 3-5 bullets. Originals moved to archive/.
- **Cold (60-90 days)**: Index entries only. One-line per memory in manifest.md.
  Knowledge graph retains entity relationships.
- **Archive (90+ days)**: Deep storage. Originals kept permanently but never
  searched. Only KG facts survive in active use.

**Rationale**: This keeps the active search space small enough that Claude
Code's native intelligence (grep + read + reason) handles retrieval without
needing ChromaDB or any vector database. The directory structure replaces
metadata filtering. The tiering replaces the need for scalable search.

**Test approach**: lifecycle.json has a test_mode with minute-based thresholds
so we can validate the full lifecycle in minutes, not months.
