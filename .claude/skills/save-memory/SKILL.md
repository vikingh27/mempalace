---
name: save-memory
description: Save a memory from the current session to the memory palace with metadata and classification
---

# Save Memory

Save important context from the current conversation to the memory palace.

## Instructions

When invoked, do the following:

1. **Identify what to save.** If the user specified what to save, use that. Otherwise, review the recent conversation and identify:
   - Decisions made (hall: decisions)
   - Discoveries or insights (hall: discoveries)
   - Preferences expressed (hall: preferences)
   - Events or milestones (hall: events)
   - Advice or recommendations (hall: advice)

2. **Classify the memory:**
   - **wing**: The project or person this relates to (e.g., `mempalace`, `personal`, a project name). Use lowercase, hyphenated slugs.
   - **room**: The specific topic (e.g., `memory-philosophy`, `auth-migration`). Use lowercase, hyphenated slugs.
   - **hall**: One of: `decisions`, `discoveries`, `preferences`, `events`, `advice`

3. **Create the directory** if it doesn't exist:
   ```
   .memory/active/<wing>/<room>/
   ```

4. **Write the memory file** as markdown with YAML frontmatter:
   - Filename: `YYYY-MM-DD_HH-MM_<short-slug>.md`
   - Frontmatter fields: `date`, `wing`, `room`, `hall`, `tags` (list), `source` (session/manual)
   - Body: **Verbatim or near-verbatim** content. Do NOT over-summarize. Preserve the reasoning, context, and exact words where possible. The philosophy is: raw context > extracted facts.

5. **Update the wing index.** Read or create `.memory/active/<wing>/_index.md` and add/update an entry for this room.

## Example Memory File

```markdown
---
date: "2026-04-08T14:30:00"
wing: mempalace
room: memory-philosophy
hall: decisions
tags: [lifecycle, tiering, claude-code]
source: session
---

## Decision: Memory lifecycle tiering strategy

We decided that the real gap in AI memory systems is not retrieval but lifecycle
management. Neither MemPalace nor Claude Code's Auto Memory handles aging,
archival, or cleanup of memories.

Agreed approach: 30/60/90 day tiering:
- Active (0-30 days): Full verbatim files, directly searchable
- Warm (30-60 days): Summarized, key decisions preserved
- Cold (60-90 days): Index entries only
- Archive (90+ days): Deep storage, not searched

Rationale: This keeps the active search space at ~200-300 files max, which
Claude Code can handle efficiently with grep + read. No vector DB needed
at this scale.
```

## Rules

- ALWAYS use verbatim quotes and specific details. Never reduce a 3-paragraph rationale to "decided to use tiering."
- One memory per file. If saving multiple things, create multiple files.
- If unsure about wing/room, ask the user.
- After saving, report what was saved and where.
