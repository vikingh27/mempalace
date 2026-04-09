# How I'm Managing Memory Lifecycle in Claude Code (Inspired by MemPalace)

## How It Started

I've been using Claude Code heavily across sessions, and my workflow for keeping track of things was... manual. I'd ask Claude to create documents saving key discussions and decisions from each session. Then in the next session, I'd reference those documents to recall what I'd decided. I was tracking dates and timestamps myself to keep everything organized. When I added features or completed tasks, I'd ask Claude to mark those items as done so the docs stayed current.

It worked. But it didn't scale. The documents kept growing. Finding the right decision from three weeks ago meant scrolling through walls of text. There was no aging, no cleanup, no structure beyond "here's another document."

Then I looked at [MemPalace](https://github.com/igorls/mempalace) — genuinely impressive. It scores 96.6% on LongMemEval with zero API calls. Great engineering, great philosophy (store raw text, don't summarize). But it requires ChromaDB — a vector database, external dependencies, an MCP server.

That got me thinking: **what if I could manage memory using just Claude Code's native capabilities?** Skills, hooks, and a CLAUDE.md protocol — no vector database, no MCP server, no external dependencies. Just structured markdown files and Claude's built-in tools.

So I started researching. And what I found is that the real gap isn't storage or retrieval — it's **lifecycle management**. MemPalace has 22,000+ memories with zero lifecycle policy. Claude Code's Auto Memory grows until it hits the line limit. They all just accumulate forever. Nobody's tackling decay.

I'm sharing what I've built so far — it's working for me, but it's very much a v1. I'd love to hear how others are handling this, and whether there's a better approach I'm missing.

---

## What I Built

A file-based memory system using **4 Claude Code skills, 1 hook, and a CLAUDE.md protocol**. No vector database. No MCP server. No external dependencies.

The system has two memory locations — **project-scoped** and **shared** — so decisions can stay local to a repo or be accessible across all projects.

### Directory Structure

```
<repo>/.memory/                     # PROJECT — specific to this repo
├── active/                         # Full verbatim memories (0-30 days)
│   └── <wing>/                     # Topic grouping
│       ├── _index.md               # Wing overview
│       └── <room>/                 # Subtopic
│           └── 2026-04-08_14-30_skills-over-mcp.md
├── warm/                           # Summarized memories (30-60 days)
├── cold/                           # Index entries only (60-90 days)
│   └── manifest.md
├── archive/                        # Permanent deep storage (90+ days)
├── diary/                          # Session summaries
└── lifecycle.json                  # Tiering config

~/.memory/                          # SHARED — accessible from any project
├── active/
│   └── design-system/              # Cross-project decisions
│       └── color-tokens/
├── warm/
├── cold/
│   └── manifest.md
└── archive/
```

### Why Two Locations?

This was a real problem I hit: I made a design-system decision in project-1, then started working in project-2 which uses the same design system. I needed that decision to be available in project-2 without copy-pasting files around.

The solution is simple:
- **Project memory** (`<repo>/.memory/`) — decisions specific to this codebase (API design, database schema, feature architecture). Travels with the repo, can be shared with a team via git.
- **Shared memory** (`~/.memory/`) — decisions that apply across projects (design systems, coding conventions, infrastructure patterns, personal preferences). Accessible from any project.

At save time, the `/save-memory` skill determines scope — either by asking, or by inferring from the topic. Design-system decisions default to shared. Database schema decisions default to project.

### The 30/60/90 Tiering Flow

This is the core of the lifecycle management:

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

- **Active (0-30 days)**: Full verbatim text. Claude searches these directly with grep + read. Target: ~200-300 files max.
- **Warm (30-60 days)**: Summarized to 3-5 bullet points. Key decisions, exact quotes, and specific numbers preserved. Originals moved to archive.
- **Cold (60-90 days)**: One-line entry in a manifest file. Just enough to know it existed.
- **Archive (90+ days)**: Permanent storage. Never auto-deleted. Available if you ever need the full original.

The idea is to keep the active search space small enough that Claude's native reasoning (grep + read + think) handles retrieval without needing a vector database. At sub-10K files, Claude reading structured files IS semantic search — with full reasoning on top.

### The 4 Skills

| Skill | Purpose |
|-------|---------|
| `/save-memory` | File a memory with wing/room/hall classification, choose project or shared scope |
| `/recall` | Search both project and shared memory, synthesize a cited answer labeled [project] or [shared] |
| `/memory-cleanup` | Run lifecycle on both locations — age, summarize, archive, purge |
| `/memory-status` | Dashboard showing both locations: tier sizes, wings, rooms, aging alerts |

### The Auto-Save Hook

A bash script on Claude Code's `Stop` event. Every 12 human messages, it blocks the stop and reminds Claude to save important context:

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

Zero extra tokens. Bash script runs locally. The hook actually fired during my test session — it blocked, I saved the hybrid architecture decision, and the session continued normally.

### Global Setup

For the skills and protocol to work in every project, they live at the global level:

```
~/.claude/
├── skills/
│   ├── save-memory/SKILL.md
│   ├── recall/SKILL.md
│   ├── memory-cleanup/SKILL.md
│   └── memory-status/SKILL.md
├── CLAUDE.md                   # Memory protocol (loads every session)
├── settings.json               # Hook config (fires every session)
└── memory_save_hook.sh         # Auto-save script
```

This means `/recall`, `/save-memory`, `/memory-status`, and `/memory-cleanup` work from any project, in any session, without any project-level setup.

---

## Why Skills Over MCP?

I initially considered building this as an MCP server (that's what MemPalace does). But the token math pushed me toward skills:

| | MCP Server (19 tools) | Native Skills (4 skills) |
|---|---|---|
| **Startup cost** | ~3,054 tokens (all schemas loaded every session) | ~300-600 tokens (one-line descriptions only) |
| **Token efficiency** | 1x baseline | **5-10x cheaper** |
| **Tool access** | Only what the server implements | ALL of Claude's built-in tools |
| **Flexibility** | Rigid JSON schemas | Natural language instructions + full reasoning |

MCP dumps every tool schema into your context window at session start, whether you use the tools or not. Skills load a one-line description; full instructions are fetched **on-demand** only when invoked.

And skills can use `Read`, `Grep`, `Edit`, `Bash`, `Write` — everything Claude already has. A skill that says "grep for keywords, read the matches, synthesize an answer" leverages Claude's full reasoning. An MCP tool that does keyword search returns... keyword matches.

---

## Live Demo: Lifecycle In Action

I ran the full test sequence in a single session. Here's what happened.

### Lifecycle Tiering Test

Started with 5 active memories, enabled test mode (minute-based thresholds instead of 30-day waits), and ran `/memory-cleanup`:

**Before:**
```
| Tier    | Files | Description              |
|---------|-------|--------------------------|
| Active  | 5     | Full verbatim, 0-30 days |
| Warm    | 0     | Summarized, 30-60 days   |
| Cold    | 0     | Index only, 60-90 days   |
| Archive | 0     | Permanent deep storage   |
```

**After `/memory-cleanup`:**
```
| Tier    | Before | After |
|---------|--------|-------|
| Active  | 5      | 0     |
| Warm    | 0      | 5     |
| Cold    | 0      | 0     |
| Archive | 0      | 5     |
```

Each memory was summarized to 3-5 bullets preserving exact quotes and numbers, then the original was moved to archive. Here's a real before/after:

**Before (active — full verbatim, 38 lines):**
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

**The user's key observation:** "Claude Code can achieve that without ChromaDB
in my understanding." This is correct for sub-10K memory files...
```

**After (warm — summarized, 4 bullets):**
```markdown
---
date: "2026-04-08T14:30:00"
wing: mempalace
room: memory-philosophy
hall: decisions
tags: [skills, mcp, tokens, architecture]
summarized_from: .memory/active/mempalace/memory-philosophy/...
summarized_at: "2026-04-08T15:00:00"
---

## Decision: Claude Code skills over MCP server for memory management

- **Token cost**: MCP (19 tools) costs ~3,054 tokens at startup; native skills
  cost ~300-600 tokens — skills are 5-10x cheaper
- **Capability**: Skills can use ALL of Claude's tools; MCP limited to server
- User's observation: "Claude Code can achieve that without ChromaDB" — correct
  for sub-10K files
- **Architecture chosen**: /save-memory, /recall, /memory-cleanup, /memory-status,
  stop hook, CLAUDE.md protocol
```

~70% compression. The exact quote, the specific numbers, and the decision rationale are preserved.

### `/recall` Test — Retrieval With Citations

Asked a natural language question: *"Why did we decide against MCP?"*

```
## Decision: Native Claude Code Skills over MCP

Source: .memory/active/mempalace/memory-philosophy/2026-04-08_14-30_skills-over-mcp.md

The decision came down to three factors:

### 1. Token cost
MCP loads all tool schemas at startup — 19 tools cost ~3,054 tokens every
session. Skills load descriptions only (~300-600 tokens). 5-10x cheaper.

### 2. Capability
Skills can use all of Claude's built-in tools. MCP is limited to what
the server implements.

### 3. Claude Code already IS semantic search
> "Claude Code can achieve that without ChromaDB in my understanding."

Correct for sub-10K memory files — Claude reading structured files and
reasoning over them is effectively semantic search without a vector database.
```

Not a keyword match. A synthesized, reasoned answer with source citations.

### Cross-Project Memory Test

This was the test I was most excited about. I created a second repo (`webapp-project`) with its own project-specific memory, then verified the scoping works:

| # | Test | Result |
|---|------|--------|
| 1 | From webapp-project, search for shared "design system" memory | **PASS** — found in `~/.memory/`, not in project memory |
| 2 | From webapp-project, search for project-specific "REST vs GraphQL" | **PASS** — found in project memory, not in shared |
| 3 | From mempalace, search for webapp-project's "REST vs GraphQL" | **PASS** — not found (correct isolation) |
| 4 | From mempalace, search for shared "design system" memory | **PASS** — found in `~/.memory/` |
| 5 | From webapp-project, search for "all decisions" | **PASS** — merged: 1 [project] + 1 [shared] |
| 6 | From mempalace, search for "all decisions" | **PASS** — merged: 10 [project] + 1 [shared], 0 from webapp |

**6/6 tests passed.** Shared memories are visible from every project. Project memories stay scoped to their repo. No leakage between projects.

---

## An Honest Assessment

I want to be upfront: **the individual components here are not novel.**

- File-based storage — standard
- Directory hierarchy — standard
- 30/60/90 tiering — decades-old data lifecycle management
- Auto-save hooks — MemPalace already ships these

Claude Code already has Auto Memory, CLAUDE.md, Hooks (26 event types), Skills, and the `/memory` command. The team hasn't built a full memory palace because Auto Memory solves 80% of the need, and MCP is their answer for the power-user 20%.

I was inspired by MemPalace's philosophy — especially the "store raw text, don't summarize" approach. What I added on top is lifecycle management, because that's the gap I kept hitting. Memories need to age, compress, and eventually get out of the way. MemPalace doesn't do this. Auto Memory doesn't do this. Nothing I've found does this.

What I've built is working for me. But I'm sure there are better approaches I haven't considered.

---

## A Few Things I Noticed Along the Way

**Skills are underrated.** They're 5-10x cheaper than MCP on token budget and more flexible. For tasks where Claude's built-in tools are sufficient (and they usually are), skills are the better primitive. I think more people should be reaching for skills before MCP.

**CLAUDE.md + Skills + Hooks is surprisingly powerful as a combo.** The protocol layer (CLAUDE.md) defines session behavior. The skill layer handles on-demand commands. The hook layer handles automation. This combination built a complete memory management system — including cross-project memory — with zero external dependencies. I didn't expect it to work this well.

**The test mode pattern saved me a lot of time.** `lifecycle.json` supports minute-based thresholds so you can validate the full pipeline in minutes instead of waiting 30 days. If you're building anything with time-based behavior, I'd recommend shipping a fast-validation mode alongside it.

**Cross-project memory was simpler than I expected.** I initially thought the hybrid (project + shared) setup would be complex. It's not. Skills search two paths instead of one. `/save-memory` asks one extra question (or infers scope from the topic). That's it. Everything else stays identical.

---

## Try It Yourself

The full implementation: [vikingh27/mempalace](https://github.com/vikingh27/mempalace)

```
~/.claude/skills/           — 4 skill definitions (global — work in any project)
~/.claude/CLAUDE.md         — Memory protocol (loads every session)
~/.claude/settings.json     — Hook config (fires every session)
~/.memory/                  — Shared cross-project memories
<repo>/.memory/             — Project-specific memories
```

Quick test:
```bash
git clone https://github.com/vikingh27/mempalace
cd mempalace
# Open with Claude Code, then:
/memory-status                        # see both project + shared dashboards
/recall <any question>                # test retrieval across both scopes
# edit .memory/lifecycle.json → set test_mode.enabled: true
/memory-cleanup                       # watch memories tier
/memory-status                        # see the after state
```

---

## What Do You Think?

This is working for me, but it's a v1 and I know there's room to improve. I'd genuinely love feedback, suggestions, or completely different approaches:

- **Is 30/60/90 day tiering the right model?** Maybe usage-based decay makes more sense — memories that get recalled frequently stay active longer, rarely-accessed ones age faster. Or importance scoring at save time. What would you do differently?
- **How are you managing memory growth?** If you're using Claude Code across long-running projects with many sessions, I'm curious what's working for you. Are you hitting the same scaling wall, or have you found a different solution?
- **Is there a better way to handle cross-project memory?** My shared `~/.memory/` approach is simple but basic. Maybe there's a more elegant way to share context across repos.
- **Has anyone else hit the "Auto Memory just accumulates" wall?** Would a built-in lifecycle/aging feature for Auto Memory be useful, or is this too niche?
- **What else have you built with Skills + Hooks + CLAUDE.md?** I feel like this combination is underexplored. I'd love to see what other systems people are building with it.

I started this because my manual document-tracking workflow wasn't scaling, and MemPalace inspired me to think about memory more seriously. This is where I've landed so far. If you're solving the same problem — or if you see obvious improvements I'm missing — I'd really love to hear about it.
