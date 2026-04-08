---
name: save-memory
description: Save a memory from the current session to the memory palace with metadata and classification
---

# Save Memory

Save important context from the current conversation to the memory palace.

## Memory Locations

This system supports two memory scopes:

- **Project memory** (`<repo>/.memory/`) — decisions specific to the current project. Travels with the repo.
- **Shared memory** (`~/.memory/`) — cross-project decisions (design systems, conventions, infrastructure). Accessible from any project.

## Instructions

When invoked, do the following:

1. **Identify what to save.** If the user specified what to save, use that. Otherwise, review the recent conversation and identify:
   - Decisions made (hall: decisions)
   - Discoveries or insights (hall: discoveries)
   - Preferences expressed (hall: preferences)
   - Events or milestones (hall: events)
   - Advice or recommendations (hall: advice)

2. **Classify the memory:**
   - **wing**: The project or topic this relates to (e.g., `mempalace`, `design-system`, `conventions`). Use lowercase, hyphenated slugs.
   - **room**: The specific topic (e.g., `memory-philosophy`, `auth-migration`). Use lowercase, hyphenated slugs.
   - **hall**: One of: `decisions`, `discoveries`, `preferences`, `events`, `advice`

3. **Determine scope** — project or shared:
   - **Shared** if the memory applies across multiple projects: design system decisions, coding conventions, infrastructure patterns, personal preferences, tool configurations.
   - **Project** if the memory is specific to the current codebase: API design, database schema, project-specific architecture, feature decisions.
   - If unclear, ask the user: "Should this be project-specific or shared across all your projects?"
   - **Heuristic**: If the wing name matches a cross-cutting concern (e.g., `design-system`, `conventions`, `infrastructure`, `tooling`, `personal`), default to shared. If it matches the current repo name or a project-specific topic, default to project.

4. **Create the directory** if it doesn't exist:
   - Project scope: `<repo>/.memory/active/<wing>/<room>/`
   - Shared scope: `~/.memory/active/<wing>/<room>/`

5. **Write the memory file** as markdown with YAML frontmatter:
   - Filename: `YYYY-MM-DD_HH-MM_<short-slug>.md`
   - Frontmatter fields: `date`, `wing`, `room`, `hall`, `tags` (list), `source` (session/manual), `scope` (project/shared)
   - Body: **Verbatim or near-verbatim** content. Do NOT over-summarize. Preserve the reasoning, context, and exact words where possible. The philosophy is: raw context > extracted facts.

6. **Update the wing index.** Read or create `_index.md` in the appropriate active wing directory and add/update an entry for this room.

## Example Memory File

```markdown
---
date: "2026-04-08T14:30:00"
wing: design-system
room: color-tokens
hall: decisions
tags: [design-system, tokens, colors]
source: session
scope: shared
---

## Decision: Use semantic color tokens, not raw values

We decided all components must reference semantic tokens (e.g., --color-primary)
instead of raw hex values. This applies to all projects using our design system.

Rationale: Enables theme switching and ensures consistency across repos.
```

## Rules

- ALWAYS use verbatim quotes and specific details. Never reduce a 3-paragraph rationale to "decided to use tiering."
- One memory per file. If saving multiple things, create multiple files.
- If unsure about wing/room, ask the user.
- If unsure about scope, ask the user.
- After saving, report what was saved, where, and the scope (project/shared).
- Ensure `~/.memory/` directories exist before writing shared memories (create if needed).
