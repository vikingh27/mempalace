---
date: "2026-04-08T14:20:00"
wing: mempalace
room: mempalace-analysis
hall: discoveries
tags: [mempalace, benchmarks, aaak, evaluation]
source: session
---

## Discovery: MemPalace — what's real vs what's hype

### What's genuinely valuable:
1. **Raw text philosophy** — store verbatim, don't summarize. This is the core
   insight. Challenges the "extract facts with LLMs" orthodoxy. Gets 96.6% on
   LongMemEval with zero API calls.
2. **Local-first, zero API** — runs entirely on your machine, real privacy.
3. **Conversation parsers** — handles Claude Code JSONL, ChatGPT JSON, Slack,
   Cursor formats. Genuine convenience.
4. **Temporal knowledge graph** — SQLite-backed, tracks "what was true when."
5. **Honest team** — corrected overclaims publicly within 48 hours of launch.

### What's overstated:
1. **AAAK compression** — claimed "30x lossless" but it's lossy and DROPS
   accuracy from 96.6% to 84.2% (12.4 point regression).
2. **"+34% palace boost"** — actually just standard ChromaDB metadata filtering.
3. **"Highest score ever"** — only on one specific benchmark (LongMemEval).
4. **NLP extraction** — all regex/keyword based (16 patterns), not neural.
5. **Contradiction detection** — code exists but isn't wired in.

### The honest framing:
MemPalace is "ChromaDB + the right design choices." The 96.6% comes from raw
ChromaDB + good defaults (all-MiniLM-L6-v2 embeddings). The palace metaphor
is excellent UX but not a technical moat. The real value is the philosophy
(keep everything, search semantically) not the architecture.

### Critical gap we identified:
MemPalace has ZERO memory lifecycle management. No TTL, no archival, no
tiering, no retention policies. It stores everything forever. This is why
they need vector search — the haystack grows without bound.
