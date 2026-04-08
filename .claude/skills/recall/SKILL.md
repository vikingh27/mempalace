---
name: recall
description: Search memory files to recall past decisions, discussions, and context
---

# Recall

Search the memory palace to find past decisions, discussions, insights, and context.

## Memory Locations

This skill searches **both** memory locations:
- **Project memory** (`<repo>/.memory/`) — current project's memories
- **Shared memory** (`~/.memory/`) — cross-project memories accessible from any repo

Results from both locations are merged and clearly labeled with their source.

## Instructions

When invoked (e.g., `/recall why did we choose tiering?`), do the following:

1. **Parse the query.** Understand what the user is looking for — a decision, a person, a project detail, a past discussion.

2. **Search strategy — narrow before deep, search BOTH locations:**

   a. **Check indexes first.** Read `_index.md` files from both:
      - `<repo>/.memory/active/*/_index.md` (project memories)
      - `~/.memory/active/*/_index.md` (shared memories)
      This identifies which wings and rooms are likely relevant.

   b. **Grep for keywords.** Use the Grep tool to search across all four locations:
      - `<repo>/.memory/active/` and `<repo>/.memory/warm/` (project)
      - `~/.memory/active/` and `~/.memory/warm/` (shared)
      If a directory doesn't exist, skip it gracefully.

   c. **Read matching files.** Read the top hits (up to 10 files). Understand the full context of each.

   d. **Check cold manifests.** If nothing found in active/warm, check both:
      - `<repo>/.memory/cold/manifest.md`
      - `~/.memory/cold/manifest.md`

3. **Synthesize an answer.** Don't just dump file contents. Combine what you found into a coherent answer that:
   - Directly addresses the query
   - Cites which memory files the answer came from (path + date)
   - **Labels each source as [project] or [shared]** so the user knows the scope
   - Includes relevant verbatim quotes where they add value
   - Notes if there are conflicting memories or if context has evolved

4. **Report gaps.** If you can't find what was asked, say so clearly. Suggest what might be missing and whether it should be saved.

## Search Priority

1. Project `.memory/active/` — full verbatim, most recent, current project
2. Shared `~/.memory/active/` — full verbatim, most recent, cross-project
3. Project `.memory/warm/` — summarized project memories
4. Shared `~/.memory/warm/` — summarized shared memories
5. Cold manifests (both locations)
6. `.memory/diary/` — agent diary entries for session history

## Rules

- NEVER guess or fabricate if a memory isn't found. Say "I don't have a memory of that."
- Always cite the source file path and date.
- Always label whether each result is from [project] or [shared] memory.
- If the query is ambiguous, search broadly first, then ask the user to narrow down.
- If `~/.memory/` doesn't exist, skip shared search gracefully (don't error).
