# AI Memory Has a Garbage Collection Problem — Here's How I Fixed It With Claude Code Skills

## The Problem Nobody Talks About

Every AI memory system I've looked at — MCP servers, vector databases, Auto Memory — focuses on the same two things: **storage** and **retrieval**. Save everything. Search it later.

Nobody asks: *what happens after 6 months?*

I analyzed [MemPalace](https://github.com/igorls/mempalace), one of the most impressive open-source AI memory projects out there. It scores 96.6% on LongMemEval with zero API calls. Genuinely great engineering. But it has 22,000+ memories with **zero lifecycle policy**. No TTL. No archival. No tiering. It stores everything forever — which is exactly why it needs ChromaDB and vector search to cope with the growing haystack.

Claude Code's Auto Memory has the same pattern. It grows until it hits the line limit, and then... what? The behavior isn't well-defined.

**The gap isn't retrieval. It's decay.**

## What I Built: A Memory Lifecycle System Using Only Claude Code Skills

Instead of building another MCP server, I built a file-based memory system using **4 Claude Code skills, 1 hook, and a CLAUDE.md protocol**. The entire thing runs on structured markdown files — no vector database, no external dependencies.

### The Architecture

```
.memory/
  active/          # Full verbatim memories (0-30 days)
    <wing>/        # Project grouping
      <room>/      # Topic grouping
        _index.md
        2026-04-08_14-30_skills-over-mcp.md
  warm/            # Summarized (30-60 days)
  cold/            # Index entries only (60-90 days)
    manifest.md    # One-line per memory
  archive/         # Permanent deep storage (90+ days)
  diary/           # Session summaries
  lifecycle.json   # Tiering config
```

### The 30/60/90 Tiering Flow

```
Active (full verbatim) ──30 days──> Warm (3-5 bullet summary)
                                         │
                                    60 days
                                         │
                                         v
                                    Cold (one-line in manifest)
                                         │
                                    90 days
                                         │
                                         v
                                    Purge from index
                                    (archive copy stays forever)
```

**Why this matters:** It keeps the active search space at ~200-300 files max. At that scale, Claude Code's native `grep + read + reason` is not just "good enough" — it's arguably **better** than vector search because you get reasoning, not just retrieval. Claude handles ambiguity, cross-references across files, and synthesizes answers. ChromaDB gives you the 5 nearest text chunks.

### The 4 Skills

| Skill | What It Does |
|-------|-------------|
| `/save-memory` | Files a memory with wing/room/hall classification and YAML frontmatter |
| `/recall` | Searches indexes, greps for keywords, reads matches, synthesizes a cited answer |
| `/memory-cleanup` | Runs the lifecycle engine — ages, summarizes, archives, purges |
| `/memory-status` | Dashboard showing tier sizes, wings, rooms, aging alerts |

### The Hook

`memory_save_hook.sh` fires on Claude Code's Stop event. Every 12 human messages, it blocks the stop and tells Claude to save important context before continuing. Zero extra tokens — it's a bash script that counts messages in the JSONL transcript.

## Why Skills Over MCP?

This was a deliberate choice, not a shortcut.

| | MCP Server (19 tools) | Native Skills (4 skills) |
|---|---|---|
| **Startup cost** | ~3,054 tokens (all schemas loaded) | ~300-600 tokens (descriptions only, content on-demand) |
| **Token efficiency** | 1x | **5-10x cheaper** |
| **Tool access** | Limited to server implementation | All of Claude's tools (Read, Grep, Edit, Bash, Write) |
| **Flexibility** | Structured schemas | Natural language instructions with full reasoning |

Skills load only a one-line description into context. The full instructions are fetched on-demand when you invoke them. An MCP server dumps all 19 tool schemas into the context window at session start, every session, whether you use them or not.

## Live Demo: The Lifecycle In Action

I validated the full pipeline in a single session using test mode (minute-based thresholds instead of 30-day waits).

### Before cleanup — `/memory-status`
```
| Tier    | Files | Description              |
|---------|-------|--------------------------|
| Active  | 5     | Full verbatim, 0-30 days |
| Warm    | 0     | Summarized, 30-60 days   |
| Cold    | 0     | Index only, 60-90 days   |
| Archive | 0     | Permanent deep storage   |
```

### Run `/memory-cleanup`
5 memories aged past the test threshold. Each one:
- Summarized to 3-5 bullet points preserving exact quotes and key decisions
- Summary written to `warm/`
- Original moved to `archive/` (never deleted)

### After cleanup — `/memory-status`
```
| Tier    | Files | Description              |
|---------|-------|--------------------------|
| Active  | 0     | Full verbatim, 0-30 days |
| Warm    | 5     | Summarized, 30-60 days   |
| Cold    | 0     | Index only, 60-90 days   |
| Archive | 5     | Permanent deep storage   |
```

### Example: What summarization looks like

**Before (active — full verbatim, 38 lines):**
> We evaluated using MemPalace as an MCP server vs building native Claude Code skills. Decision: native skills.
>
> Token comparison: MemPalace MCP (19 tools): ~3,054 tokens at startup... Native skills (6 skills): ~300-600 tokens...
>
> Capability comparison: MCP gives structured tool schemas... Skills give instructions with more flexibility... Skills can use ALL of Claude's tools...
>
> The user's key observation: "Cloud Code can achieve that without ChromaDB in my understanding." This is correct for sub-10K memory files...
>
> Architecture chosen: /save-memory, /recall, /memory-cleanup, /memory-status, Stop hook, CLAUDE.md protocol...

**After (warm — summarized, 4 bullets):**
> - **Token cost**: MCP (19 tools) costs ~3,054 tokens at startup; native skills (4 skills) cost ~300-600 tokens — skills are 5-10x cheaper on context window
> - **Capability**: Skills can use ALL of Claude's tools (Read, Grep, Bash, Write, Edit); MCP tools limited to what the server implements
> - User's key observation: "Cloud Code can achieve that without ChromaDB" — correct for sub-10K files
> - **Architecture chosen**: `/save-memory`, `/recall`, `/memory-cleanup`, `/memory-status`, stop hook, CLAUDE.md protocol

The summary preserves the exact quote, the specific numbers, and the decision rationale. Context is compressed, not lost.

### `/recall` — Retrieval with citations

Query: *"Why did we decide against using MCP?"*

Response (synthesized from memory files):
> The decision came down to three factors:
> 1. **Token cost** — MCP loads all 19 tool schemas at startup (~3,054 tokens). Skills load descriptions only (~300-600 tokens). 5-10x cheaper.
> 2. **Capability** — Skills can use all of Claude's built-in tools. MCP is limited to what the server implements.
> 3. **Claude Code IS semantic search** — For sub-10K files, Claude reading structured files and reasoning over them is effectively semantic search without needing a vector database.
>
> *Source: `.memory/active/mempalace/memory-philosophy/2026-04-08_14-30_skills-over-mcp.md`*

Not a keyword match. Not nearest-embedding retrieval. A reasoned answer with citations.

## An Honest Assessment

I want to be transparent: **the individual components here are not novel.**

- File-based storage — standard
- Directory hierarchy for organization — standard
- 30/60/90 tiering — decades-old data lifecycle management
- Auto-save hooks — MemPalace already ships these

Claude Code already has Auto Memory, CLAUDE.md, Hooks (26 event types), Skills, and the `/memory` command. The team hasn't built a full memory palace because Auto Memory solves 80% of the need for most users, and MCP is their answer for the power-user 20%.

**What's genuinely new here is applying lifecycle discipline to AI memory.** Nobody — not MemPalace, not Auto Memory, not Mem0 — has aging, summarization, and archival built in. They all accumulate forever. The fix isn't a product. It's a practice, codified into skills.

## What This Means for Claude Code

A few observations for the community and team:

1. **Skills are underrated.** They're dramatically cheaper than MCP on token budget and more flexible. For tasks where Claude's built-in tools are sufficient (and they usually are), skills are the better primitive.

2. **CLAUDE.md + Skills + Hooks is a full application framework.** The protocol layer (CLAUDE.md) defines behavior. The skill layer (on-demand instructions) handles commands. The hook layer (bash scripts on events) handles automation. This combination can build surprisingly complete systems with zero external dependencies.

3. **Memory lifecycle is a real gap.** As people use Claude Code for longer projects across more sessions, the "just accumulate everything" approach will hit scaling walls. Even at hundreds of files, some form of aging and summarization keeps the context window efficient.

4. **The test mode pattern is useful.** `lifecycle.json` supports minute-based thresholds so you can validate the full lifecycle pipeline in minutes instead of waiting 30 days. More configs should ship with a fast-validation mode.

## Try It

The full implementation is at [vikingh27/mempalace](https://github.com/vikingh27/mempalace). The relevant files:

- `.claude/skills/` — All 4 skill definitions
- `.memory/` — The tiered directory structure
- `hooks/memory_save_hook.sh` — Auto-save hook
- `CLAUDE.md` — The memory protocol
- `.memory/lifecycle.json` — Tiering config (set `test_mode.enabled: true` to try it fast)

To test the full lifecycle:
```
1. git clone the repo
2. Open with Claude Code
3. /memory-status          — see the dashboard
4. /recall <any question>  — test retrieval
5. Edit lifecycle.json     — enable test mode
6. /memory-cleanup         — watch memories tier
7. /memory-status          — see the after state
```

## Discussion

I'd love to hear from others working on memory management with Claude Code:

- How are you handling memory growth over time?
- Has anyone else hit the "Auto Memory just accumulates" wall?
- Would a built-in lifecycle/aging feature for Auto Memory be useful?
- Are there other creative uses of the Skills + Hooks + CLAUDE.md combo?

The gap is real. The fix doesn't require new infrastructure — just discipline applied to what already exists.
