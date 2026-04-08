# AI Memory Has a Garbage Collection Problem

## The Problem Nobody Talks About

Every AI memory system I've looked at — MCP servers, vector databases, Auto Memory — focuses on the same two things: **storage** and **retrieval**. Save everything. Search it later.

Nobody asks: *what happens after 6 months?*

I analyzed [MemPalace](https://github.com/igorls/mempalace), one of the most impressive open-source AI memory projects out there. It scores 96.6% on LongMemEval with zero API calls. Genuinely great engineering. But it has 22,000+ memories with **zero lifecycle policy**. No TTL. No archival. No tiering. It stores everything forever — which is exactly why it needs ChromaDB and vector search to cope with the growing haystack.

Claude Code's Auto Memory has the same pattern. It grows until it hits the line limit, and then... what?

**The gap isn't retrieval. It's decay.**

---

## What I Built

A file-based memory system using **4 Claude Code skills, 1 hook, and a CLAUDE.md protocol**. No vector database. No MCP server. No external dependencies. Just structured markdown files and Claude Code's native capabilities.

### Directory Structure

```
.memory/
├── active/                    # Full verbatim memories (0-30 days)
│   └── <wing>/                # Project grouping (e.g., "mempalace")
│       ├── _index.md          # Wing overview
│       └── <room>/            # Topic grouping (e.g., "memory-philosophy")
│           └── 2026-04-08_14-30_skills-over-mcp.md
├── warm/                      # Summarized memories (30-60 days)
├── cold/                      # Index entries only (60-90 days)
│   └── manifest.md            # One-line per memory
├── archive/                   # Permanent deep storage (90+ days)
├── diary/                     # Session summaries
└── lifecycle.json             # Tiering config with test mode
```

### The 30/60/90 Tiering Flow

```
Active (full verbatim)
    │
    ├── 30 days ──> Warm (3-5 bullet summary, original → archive)
    │                 │
    │                 ├── 60 days ──> Cold (one-line in manifest.md)
    │                 │                 │
    │                 │                 ├── 90 days ──> Purge from index
    │                 │                 │               (archive copy stays forever)
```

### The 4 Skills

| Skill | Purpose |
|-------|---------|
| `/save-memory` | File a memory with wing/room/hall classification and YAML frontmatter |
| `/recall` | Search indexes → grep keywords → read matches → synthesize a cited answer |
| `/memory-cleanup` | Run the lifecycle engine — age, summarize, archive, purge |
| `/memory-status` | Dashboard: tier sizes, wings, rooms, aging alerts |

### The Auto-Save Hook

A bash script on Claude Code's `Stop` event. Every 12 human messages, it blocks the stop and tells Claude to save important context:

```bash
SAVE_INTERVAL=12

# Count human messages in session transcript
EXCHANGE_COUNT=$(python3 - "$TRANSCRIPT_PATH" <<'PYEOF'
import json, sys
count = 0
with open(sys.argv[1]) as f:
    for line in f:
        entry = json.loads(line)
        msg = entry.get('message', {})
        if isinstance(msg, dict) and msg.get('role') == 'user':
            count += 1
print(count)
PYEOF
)

# Block and remind to save
if [ "$SINCE_LAST" -ge "$SAVE_INTERVAL" ]; then
    cat << 'HOOKJSON'
{
  "decision": "block",
  "reason": "MEMORY CHECKPOINT. Use /save-memory to file any important
             decisions or discoveries. Then continue."
}
HOOKJSON
fi
```

Zero tokens. Bash script runs locally. Claude spends a few seconds organizing memories it already has in context.

---

## Why Skills Over MCP?

This was a deliberate architectural choice, not a shortcut.

| | MCP Server (19 tools) | Native Skills (4 skills) |
|---|---|---|
| **Startup cost** | ~3,054 tokens (all schemas loaded every session) | ~300-600 tokens (one-line descriptions only) |
| **Token efficiency** | 1x baseline | **5-10x cheaper** |
| **Tool access** | Only what the server implements | ALL of Claude's built-in tools |
| **Flexibility** | Rigid JSON schemas | Natural language instructions + full reasoning |

MCP dumps every tool schema into your context window at session start, whether you use the tools or not. Skills load a one-line description; full instructions are fetched **on-demand** only when invoked. For a memory system with 4 commands, that's the difference between ~3,000 tokens burned every session vs ~400.

And skills can use `Read`, `Grep`, `Edit`, `Bash`, `Write` — everything Claude already has. An MCP server can only do what its code implements. A skill that says "grep for keywords, read the matches, synthesize an answer" leverages Claude's full reasoning capability. An MCP tool that does keyword search returns... keyword matches.

---

## Live Session Demo

I ran the full test sequence in a single session to validate every component. Here's exactly what happened.

### Step 1: `/memory-status` — The Dashboard

Started by checking the current state:

```
## Memory Palace Status

### Tiers
| Tier    | Files | Description              |
|---------|-------|--------------------------|
| Active  | 5     | Full verbatim, 0-30 days |
| Warm    | 0     | Summarized, 30-60 days   |
| Cold    | 0     | Index only, 60-90 days   |
| Archive | 0     | Permanent deep storage   |
| Diary   | 1     | Session diary entries    |

### Active Wings
- **mempalace** (5 memories)
  - memory-philosophy: 3 memories (latest: 2026-04-08)
  - lifecycle-design: 1 memory (latest: 2026-04-08)
  - mempalace-analysis: 1 memory (latest: 2026-04-08)

### Lifecycle Config
- Active threshold: 30 days | Warm: 60 days | Cold: 90 days
- Test mode: OFF

### Aging Soon
- No memories approaching any threshold (all are from today)
```

5 active memories, nothing in warm/cold/archive. Clean starting state.

### Step 2: `/recall why did we decide against MCP?` — Retrieval With Citations

Asked it a natural language question about a past decision. Here's the actual output:

```
## Decision: Native Claude Code Skills over MCP

Source: .memory/active/mempalace/memory-philosophy/2026-04-08_14-30_skills-over-mcp.md

The decision came down to three factors:

### 1. Token cost
MCP loads all tool schemas at startup — MemPalace's 19 tools cost ~3,054 tokens
every session. Native skills only load descriptions (~300-600 tokens), with full
content fetched on-demand. Skills are 5-10x cheaper on the context window.

### 2. Capability
MCP tools are limited to what the server implements. Skills can use all of
Claude's built-in tools (Read, Grep, Bash, Write, Edit).

### 3. Claude Code already IS semantic search
Your key observation was:
> "Cloud Code can achieve that without ChromaDB in my understanding."

This is correct for sub-10K memory files — Claude reading structured files
and reasoning over them is effectively semantic search, without needing
a vector database.
```

This is not keyword matching. Not nearest-embedding retrieval. Claude searched the indexes, grepped for "MCP", read the matching file, and **synthesized a reasoned answer with direct citations**. That's what "Claude Code IS semantic search" means in practice.

### Step 3: `/recall top 3 decisions` — Multi-Document Synthesis

Asked a broader query that required finding and ranking across multiple files:

```
## Top 3 Decisions

### 1. 30/60/90 Day Memory Tiering Strategy
Source: .memory/active/mempalace/lifecycle-design/2026-04-08_14-10_tiering-strategy-decision.md

The real gap in AI memory isn't retrieval — it's lifecycle management.
MemPalace has 22,000+ memories with zero lifecycle policy; Claude Code's
Auto Memory grows until it hits 200 lines.

> Active (0-30d, full verbatim) → Warm (30-60d, summarized)
> → Cold (60-90d, index entries) → Archive (90d+, permanent deep storage)

### 2. Native Claude Code Skills over MCP
Source: .memory/active/mempalace/memory-philosophy/2026-04-08_14-30_skills-over-mcp.md

Skills are 5-10x cheaper on the context window. Skills can use all of
Claude's built-in tools; MCP tools are limited to what the server implements.

### 3. Honest Novelty Assessment — Lifecycle Is the Gap
Source: .memory/active/mempalace/memory-philosophy/2026-04-08_14-40_novelty-assessment.md

The individual components are not novel. What's genuinely unaddressed:
> "Everyone building AI memory focuses on storage and retrieval.
>  Nobody focuses on lifecycle and decay."
```

It found all 3 decision-tagged files, read them, ranked by significance, and presented a coherent summary with verbatim quotes. No vector database needed.

### Step 4: Enable Test Mode

Edited one line in `lifecycle.json`:

```json
"test_mode": {
    "enabled": true,          // ← flipped from false
    "active_max_minutes": 5,  // instead of 30 days
    "warm_max_minutes": 10,   // instead of 60 days
    "cold_max_minutes": 15    // instead of 90 days
}
```

This lets you validate the full lifecycle pipeline in minutes instead of waiting months.

### Step 5: `/memory-cleanup` — The Lifecycle Engine

Ran cleanup with test mode active. All 5 memories were older than 5 minutes, so all were eligible for Active → Warm transition.

**What happened for each memory:**

1. Read the full verbatim file
2. Wrote a **summary** to `.memory/warm/` — condensed to 3-5 bullet points, preserving exact quotes and specific numbers
3. Moved the original to `.memory/archive/` (never deleted)
4. Updated the wing index

**Transition report:**

```
### Transitions: Active → Warm (5 memories)

| Memory                        | Wing      | Room               |
|-------------------------------|-----------|--------------------|
| claude-code-is-semantic-search| mempalace | memory-philosophy  |
| tiering-strategy-decision     | mempalace | lifecycle-design   |
| skills-over-mcp               | mempalace | memory-philosophy  |
| novelty-assessment            | mempalace | memory-philosophy  |
| mempalace-substance-vs-hype   | mempalace | mempalace-analysis |

### Tier Summary (Before → After)
| Tier    | Before | After |
|---------|--------|-------|
| Active  | 5      | 0     |
| Warm    | 0      | 5     |
| Cold    | 0      | 0     |
| Archive | 0      | 5     |
```

### What Summarization Actually Looks Like

Here's a real before/after from the session — the MCP decision memory:

**BEFORE — Active tier (full verbatim, 38 lines):**

```markdown
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
- /save-memory — file memories with wing/room/hall classification
- /recall — grep + read + synthesize from memory files
- /memory-cleanup — lifecycle tiering (the novel piece)
- /memory-status — overview dashboard
- Stop hook — auto-reminds to save every 12 messages
- CLAUDE.md — memory protocol loaded every session
```

**AFTER — Warm tier (summarized, 4 bullets):**

```markdown
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

- **Token cost**: MCP (19 tools) costs ~3,054 tokens at startup; native skills
  cost ~300-600 tokens — skills are 5-10x cheaper on context window
- **Capability**: Skills can use ALL of Claude's tools (Read, Grep, Bash, Write,
  Edit); MCP tools limited to what the server implements
- User's key observation: "Cloud Code can achieve that without ChromaDB" —
  correct for sub-10K files, since Claude reading structured files IS semantic
  search with reasoning on top
- **Architecture chosen**: /save-memory, /recall, /memory-cleanup, /memory-status,
  stop hook, CLAUDE.md protocol
```

The summary preserves the exact quote, the specific token numbers (3,054 vs 300-600), and the decision rationale. Context is compressed ~70%, not lost. And the original is safe in `archive/` if you ever need the full version.

---

## An Honest Assessment

I want to be transparent: **the individual components here are not novel.**

- File-based storage — standard
- Directory hierarchy — standard
- 30/60/90 tiering — decades-old data lifecycle management
- Auto-save hooks — MemPalace already ships these

Claude Code already has Auto Memory, CLAUDE.md, Hooks (26 event types), Skills, and the `/memory` command. The team hasn't built a full memory palace because Auto Memory solves 80% of the need, and MCP is their answer for the power-user 20%.

**What's genuinely new here is applying lifecycle discipline to AI memory.** Nobody — not MemPalace, not Auto Memory, not Mem0 — has aging, summarization, and archival built in. They all accumulate forever.

The fix isn't a product. It's a practice, codified into skills.

---

## What This Means for Claude Code

A few observations for the community and team:

**1. Skills are underrated.**
They're 5-10x cheaper than MCP on token budget and more flexible. For tasks where Claude's built-in tools are sufficient (and they usually are), skills are the better primitive. I think more people should be building with skills before reaching for MCP.

**2. CLAUDE.md + Skills + Hooks is a full application framework.**
The protocol layer (CLAUDE.md) defines session behavior. The skill layer handles on-demand commands. The hook layer handles automation. This combination built a complete memory management system with zero external dependencies.

**3. Memory lifecycle is a real gap.**
As people use Claude Code for longer projects across more sessions, the "just accumulate everything" approach will hit scaling walls. Even at hundreds of files, some form of aging and summarization keeps the context window efficient. Would a built-in lifecycle feature for Auto Memory be useful?

**4. The test mode pattern is worth copying.**
`lifecycle.json` supports minute-based thresholds so you can validate the full pipeline in minutes instead of months. More configs should ship with a fast-validation mode.

---

## Try It

The full implementation: [vikingh27/mempalace](https://github.com/vikingh27/mempalace)

```
.claude/skills/         — All 4 skill definitions
.memory/                — The tiered directory structure
hooks/memory_save_hook.sh — Auto-save hook
CLAUDE.md               — The memory protocol
.memory/lifecycle.json  — Tiering config
```

Quick test sequence:
```bash
git clone https://github.com/vikingh27/mempalace
cd mempalace
# Open with Claude Code, then:
/memory-status                        # see the dashboard
/recall <any question>                # test retrieval
# edit .memory/lifecycle.json → set test_mode.enabled: true
/memory-cleanup                       # watch memories tier
/memory-status                        # see the after state
```

---

## Discussion

I'd love to hear from others:

- How are you handling memory growth over time?
- Has anyone else hit the "Auto Memory just accumulates" wall?
- Would a built-in lifecycle/aging feature for Auto Memory be useful?
- What other systems have you built with Skills + Hooks + CLAUDE.md?

The gap is real. The fix doesn't require new infrastructure — just discipline applied to what already exists.
