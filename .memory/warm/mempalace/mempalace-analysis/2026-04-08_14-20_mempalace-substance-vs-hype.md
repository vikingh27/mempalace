---
date: "2026-04-08T14:20:00"
wing: mempalace
room: mempalace-analysis
hall: discoveries
tags: [mempalace, benchmarks, aaak, evaluation]
summarized_from: .memory/active/mempalace/mempalace-analysis/2026-04-08_14-20_mempalace-substance-vs-hype.md
summarized_at: "2026-04-08T15:00:00"
---

## Discovery: MemPalace — what's real vs what's hype

- **Genuinely valuable**: Raw text philosophy (store verbatim, don't summarize — gets 96.6% on LongMemEval with zero API calls), local-first/zero API, conversation parsers for multiple formats, temporal knowledge graph, honest team
- **Overstated**: AAAK "30x lossless" compression actually drops accuracy from 96.6% to 84.2%; "+34% palace boost" is standard ChromaDB metadata filtering; NLP extraction is regex-based (16 patterns), not neural; contradiction detection code exists but isn't wired in
- **Honest framing**: MemPalace is "ChromaDB + the right design choices" — real value is the philosophy (keep everything, search semantically), not the architecture
- **Critical gap**: Zero memory lifecycle management — no TTL, no archival, no tiering. Stores everything forever, which is why they need vector search
